---
title: "Cortex 추론 파이프라인 최적화 (2) — 추측은 틀렸고, 암달의 법칙은 옳았다"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, numba, llvm, jit, compiler, tvm, byoc, ml-compiler, profiling]
use_math: true
---

## 청사진은 그럴듯했다

[1편]({% post_url 2026-04-06-cortex-pipeline-optimization-1 %})에서 Graph IR을 만들어 L2 파이프라인의 뼈대를 잡은 뒤, 자연스럽게 다음 단계로 넘어갔다. "이제 컴파일 가능한 노드를 JIT으로 최적화한다." 머릿속에 그린 청사진은 꽤 그럴듯했다.

가장 먼저 눈에 든 사냥감은 `score_fusion` 노드였다. 4개의 ROI 점수를 가중치와 곱해서 더하는 수식을 보면 Python이 임시 배열을 여러 번 할당할 게 뻔했다.

$$S = w_c S_c + w_t S_t + w_s S_s + w_m S_m$$

TVM 논문에서 보던 Operator Fusion 예제와 똑같이 생겼길래, "이걸 하나로 컴파일해서 퓨전해버리면 무조건 빨라지겠다"며 속으로 쾌재를 불렀다.

그리고 호기롭게 프로파일링을 돌려본 순간, 모니터에 찍힌 숫자를 보고 헛웃음이 났다.

```
score_fusion: 0.01ms
```

0.01ms. Numba가 커널 실행을 준비하는 데 걸리는 오버헤드보다도 작은 시간이었다. 6×8 사이즈의 작은 그리드에서 스칼라 연산을 하는 정도는 NumPy가 이미 마이크로초 단위로 바닥(numpy floor)까지 최적화해서 처리하고 있었던 것이다. 컴파일을 하면 할수록 오히려 느려지는 구조였다.

코드를 눈으로만 훑고 "이거 최적화하면 되겠다"고 덤벼드는 게 얼마나 위험한 실수인지, 측정 없이 타겟을 고르면 어떤 참사가 일어나는지 뼈저리게 체감한 순간이었다.

---

## 현실은 예상보다 훨씬 적나라했다

정신을 차리고, 프레임당 지연 시간을 각 노드별로 정확하게 다시 측정해 `examples/profiling_report.txt`를 뽑아봤다.

```
=== Profiling Report (640×480, N=100) ===

Node              Type            ms/frame
─────────────────────────────────────────
text_roi_mser     external_call   6.79 ms   ← 병목
motion_map        compilable      0.79 ms
saliency_dft      compilable      0.35 ms   ← JIT 타겟
center_crop       compilable      0.03 ms
score_fusion      compilable      0.01 ms   ← numpy floor
ema_smooth        compilable      0.00 ms
─────────────────────────────────────────
Total                             7.97 ms
```

전체 실행 시간 7.97ms 중 무려 85.2%인 6.79ms가 내가 짠 코드가 아니라, OpenCV의 MSER(텍스트 영역 추출) C++ 블랙박스 노드에서 소모되고 있었다. 컴퓨터 구조 시간에 교과서에서나 외우던 **암달의 법칙(Amdahl's Law)**이 멱살을 잡고 흔드는 기분이었다.

최적화할 수 없는 외부 블랙박스가 85%를 차지하고 있다면, 나머지 15%를 0초로 만들어버린다 한들 전체 성능 향상의 상한선은 이미 정해져 있는 것이다.

$$\text{Pipeline Speedup} = \frac{1}{0.852 + \dfrac{0.044}{2.1} + 0.104} \approx 1.024$$

- $0.852$: text\_roi\_mser 비율 (컴파일 불가)
- $0.044$: saliency\_dft 비율 (2.1× 향상)
- $0.104$: 나머지 compilable 노드

하지만 이 현실을 마주하고 나서야 비로소 엔지니어링의 다음 단계가 명확해졌다. "전체를 마법처럼 빠르게 만들 수는 없다. 그렇다면 컴파일러 프레임워크의 철학을 빌려 시스템의 경계를 우아하게 나누자."

---

## BYOC 패턴: 경계를 나누는 것이 먼저다

맹목적인 코드 튜닝을 멈추고, Apache TVM 같은 최신 ML 컴파일러들이 사용하는 **BYOC(Bring Your Own Codegen)** 패턴을 도입해 시스템을 재설계하기 시작했다.

`CortexCompiler`가 Phase 1에서 만든 Graph를 넘겨받으면, `is_compilable` 플래그를 확인해 그래프를 두 갈래로 파티셔닝한다.

```python
# Python ops → Graph IR → Optimization Passes → Partitioning → Codegen → Runtime
#                                                ↑ 여기

class CortexCompiler:
    def __init__(self, graph: Graph):
        self.graph = graph

    def partition(self, graph: Graph) -> tuple[list[Node], list[Node]]:
        """
        Phase 1 Graph를 is_compilable 플래그로 분리한다.
        TVM의 partition_for_<target>() / BYOC 패턴과 동일한 개념.

        is_compilable=True  → compilable subgraph (numba JIT 대상)
        is_compilable=False → external_call boundary (그대로 실행)
        """
        compilable, external = [], []
        for name in graph._topo_order:
            node = graph.nodes[name]
            (compilable if node.is_compilable else external).append(node)
        return compilable, external
```

OpenCV를 쓰는 MSER이나 SSIM처럼 건드릴 수 없는 노드들은 external_call 경계(Boundary) 밖으로 명확히 격리하고, 최적화가 가능한 서브그래프만 모아서 JIT 타겟팅했다.

```
Compilable subgraph:
  center_crop    (numpy: Gaussian window)
  saliency_dft   (numpy: FFT pipeline)     ← 주 타겟
  motion_map     (numpy: absdiff + pool)
  score_fusion   (numpy: weighted sum)
  ema_smooth     (numpy: EMA)

External call boundary:
  text_roi_mser  (cv2.MSER — OpenCV 블랙박스)
```

이 분리가 MLIR의 `external func`, TVM BYOC의 bring_your_own_codegen과 정확히 같은 개념이다. 컴파일러가 lower할 수 없는 연산은 외부 함수 호출로 남겨두는 것.

---

## saliency_dft 커널: 세 단계 최적화

가장 의미 있는 타겟이었던 `saliency_dft` 커널을 Python 루프 → 벡터화 → JIT 세 단계로 최적화했다. 모든 버전은 수치적으로 동일한 출력을 보장한다.

### Baseline — Python 루프

```python
# baseline: 인터프리터가 매 반복마다 바이트코드 해석

def saliency_baseline(gray: np.ndarray, grid_h: int = 6, grid_w: int = 8) -> np.ndarray:
    h, w = gray.shape
    cell_h, cell_w = h // grid_h, w // grid_w
    out = np.zeros((grid_h, grid_w), dtype=np.float32)
    for i in range(grid_h):
        for j in range(grid_w):
            cell = gray[i*cell_h:(i+1)*cell_h, j*cell_w:(j+1)*cell_w].astype(np.float32)
            out[i, j] = cell.std()
    return out
```

### Vectorized — Python 루프 제거

`np.add.reduceat`를 사용하면 그리드 분할과 집계를 루프 없이 처리할 수 있다.

```python
# vectorization pass: loop → np.add.reduceat

def saliency_vectorized(gray: np.ndarray, grid_h: int = 6, grid_w: int = 8) -> np.ndarray:
    h, w = gray.shape
    cell_h, cell_w = h // grid_h, w // grid_w
    f = gray[:grid_h * cell_h, :grid_w * cell_w].astype(np.float32)

    row_cuts = np.arange(0, grid_h * cell_h, cell_h)
    col_cuts = np.arange(0, grid_w * cell_w, cell_w)

    row_sum = np.add.reduceat(f, row_cuts, axis=0)
    grid_sum = np.add.reduceat(row_sum, col_cuts, axis=1)

    row_sq = np.add.reduceat(f ** 2, row_cuts, axis=0)
    grid_sq = np.add.reduceat(row_sq, col_cuts, axis=1)

    n = cell_h * cell_w
    mean = grid_sum / n
    variance = grid_sq / n - mean ** 2
    return np.sqrt(np.maximum(variance, 0)).astype(np.float32)
```

### JIT — LLVM 네이티브 코드

처음엔 `@njit(parallel=True)`를 썼는데 오히려 느렸다. 그리드가 6×8=48셀이라 스레드 생성 오버헤드가 실제 계산보다 컸기 때문이다. `@jit(nopython=True)`로 되돌렸다.

```python
# numba @jit → LLVM IR → 네이티브 기계어

from numba import jit

@jit(nopython=True, cache=True)
def saliency_jit(gray: np.ndarray, grid_h: int = 6, grid_w: int = 8) -> np.ndarray:
    """
    nopython=True: Python 객체 없이 순수 네이티브 실행
    cache=True: 첫 컴파일 결과를 디스크에 저장 — 재시작 시 웜업 불필요

    cv2.dft 경로는 numba nopython mode 진입 불가.
    → MSER과 동일한 이유로 external_call boundary에 머문다.
    """
    h, w = gray.shape
    cell_h = h // grid_h
    cell_w = w // grid_w
    out = np.zeros((grid_h, grid_w), dtype=np.float32)

    for i in range(grid_h):
        for j in range(grid_w):
            s = 0.0
            s2 = 0.0
            for r in range(cell_h):
                for c in range(cell_w):
                    v = float(gray[i * cell_h + r, j * cell_w + c])
                    s += v
                    s2 += v * v
            n = cell_h * cell_w
            mean = s / n
            out[i, j] = (s2 / n - mean * mean) ** 0.5

    return out
```

---

## 실제 측정 결과

```
=== Compilation Results (640×480, N=100) ===

Kernel        ms/frame   Speedup   Pass
──────────────────────────────────────────
baseline      0.31 ms    1.0×      Python loop
vectorized    0.17 ms    1.9×      np.add.reduceat
jit           0.15 ms    2.1×      numba @jit → LLVM
```

기존 0.31ms 걸리던 커널을 0.15ms로 줄이며 **2.1배의 속도 향상**을 이뤄냈다.

이 수치를 파이프라인 전체에 대입해보면 어떨까. 85%의 블랙박스는 그대로 두고, 4.4%의 비중을 차지하던 노드 하나를 2.1배 빠르게 만든 결과 — 파이프라인 전체의 엔드투엔드 속도 향상은 고작 **2.4%(1.024×)**에 불과하다.

| 노드 | ms/frame | 전체 비율 |
|---|---|---|
| `text_roi_mser` | 6.79ms | 85.2% — 병목, 건드릴 수 없음 |
| `motion_map` | 0.79ms | 9.9% — compilable |
| `saliency_dft` | 0.35ms | 4.4% — **주 컴파일 타겟 (2.1×)** |
| `center_crop` | 0.03ms | 0.4% |
| `score_fusion` | 0.01ms | <0.1% — numpy floor |
| `ema_smooth` | 0.00ms | — |
| **Total** | **7.97ms** | |

---

## 테스트: 이 시스템을 지탱하는 진짜 안전망

숫자가 진짜인지 검증하기 위해 30개의 테스트를 작성했다. 파티셔닝이 compilable/external 노드를 정확히 나누는지, JIT 커널이 Python 버전과 수치적으로 완전히 동일한 값을 내뱉는지, 벤치마크가 허상이 아닌 실제 레이턴시를 측정하고 있는지를 깐깐하게 물고 늘어지는 테스트들이다.

```python
def test_partition_splits_compilable_and_external():
    graph = build_l2_graph()
    compiler = CortexCompiler(graph)
    compilable, external = compiler.partition(graph)
    compilable_names = [n.name for n in compilable]
    external_names = [n.name for n in external]
    assert "saliency_dft" in compilable_names
    assert "text_roi_mser" in external_names
    assert "text_roi_mser" not in compilable_names

def test_all_kernel_versions_numerically_identical():
    gray = np.random.randint(0, 255, (480, 640), dtype=np.uint8)
    out_b = saliency_baseline(gray)
    out_v = saliency_vectorized(gray)
    out_j = saliency_jit(gray)
    np.testing.assert_allclose(out_b, out_v, rtol=1e-5)
    np.testing.assert_allclose(out_b, out_j, rtol=1e-5)

def test_benchmark_measures_actual_latency():
    graph = build_l2_graph()
    compiler = CortexCompiler(graph)
    compilable, _ = compiler.partition(graph)
    compiler.compile(compilable)
    report = compiler.benchmark(n=100)
    assert "saliency_dft" in report
    assert 0 < report["saliency_dft"]["mean_ms"] < 5.0
```

세 번째 테스트가 생각보다 중요했다. 벤치마크 함수가 실제로 측정하고 있는지, 아니면 그냥 함수를 호출만 하고 있는지 검증하는 코드다. 이걸 안 짰으면 잘못된 숫자를 한참 믿고 있었을 것이다. 테스트들이야말로 이 시스템을 지탱하는 진짜 안전망이다.

---

## 돌아보며: 추측은 틀렸고, 암달의 법칙은 옳았다

숫자만 보면 초라해 보일 수 있다. 하지만 역설적으로, 이 '2.4%의 개선'이 담긴 커밋이 이번 프로젝트에서 가장 자랑스럽다. 이 숫자는 단순한 감이나 추측이 아니라 엄밀한 측정의 결과물이며, 수식으로 설명할 수 있는 명확한 근거를 가지고 있기 때문이다.

모든 것을 무작정 최적화하려 들지 않고, 병목이 컴파일 불가능한 영역에 있을 때 이를 인정하며 시스템의 아키텍처 경계를 어떻게 분리할 것인가. 어쩌면 엔지니어의 진짜 실력은 코드를 얼마나 빠르게 깎아내느냐보다, 내 시스템의 한계를 어디까지 정직하게 인정하고 타협할 수 있는지에서 나오는 것일지도 모른다.

추측은 틀렸고, 암달의 법칙은 옳았다. 그리고 Cortex의 아키텍처는 이제 그 한계를 인지하는 만큼 훨씬 더 단단해졌다.
