---
title: "Cortex 추론 파이프라인 최적화 (2) — ML 컴파일러 설계와 JIT 최적화"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, numba, llvm, jit, compiler, tvm, byoc, ml-compiler, profiling]
use_math: true
---

## 처음 타겟을 잘못 잡았다

[1편]({% post_url 2026-02-16-cortex-pipeline-optimization-1 %})에서 Graph IR을 만든 뒤, 자연스럽게 다음 단계로 넘어갔다. "컴파일 가능한 노드를 JIT으로 최적화한다." 그럴듯한 계획이었다.

처음에 눈이 간 건 `score_fusion`이었다.

$$S = w_c \cdot S_c + w_t \cdot S_t + w_s \cdot S_s + w_m \cdot S_m$$

Python이 이 연산을 실행할 때 임시 배열을 7번 할당한다. TVM 논문에서 본 operator fusion 예제랑 똑같이 생겼다. "이걸 컴파일하면 빨라지겠다"는 생각이 들었다.

그래서 프로파일링을 돌렸다.

```
score_fusion: 0.01ms
```

0.01ms. numba 커널 실행 준비 시간보다 작다. **컴파일할수록 오히려 느려지는 구조였다.** score_fusion은 6×8=48개 셀에 대한 스칼라 연산이다. NumPy가 이미 마이크로초 단위로 처리하고 있었다. "numpy floor"에 이미 도달해 있는 노드를 컴파일 타겟으로 잡은 거였다.

이게 측정 없이 타겟을 고르면 어떻게 되는지 보여주는 사례다. 코드를 보고 "이거 최적화할 수 있겠다"고 생각하는 것과, 실제로 병목인지는 전혀 다른 이야기다.

---

## 측정부터 다시 했다

640×480 프레임, N=100 반복으로 각 노드의 per-frame latency를 따로 측정했다. 결과를 `examples/profiling_report.txt`에 기록했다. 예상보다 훨씬 적나라했다.

```
=== Profiling Report (640×480, N=100) ===

Node              Type            ms/frame
─────────────────────────────────────────
text_roi_mser     external_call   6.79 ms   ← 병목
motion_map        compilable      0.79 ms
saliency_dft      compilable      0.35 ms   ← 주요 타겟
center_crop       compilable      0.03 ms
score_fusion      compilable      0.01 ms   ← numpy floor
ema_smooth        compilable      0.00 ms
─────────────────────────────────────────
Total                             7.97 ms
```

`text_roi_mser`가 6.79ms로 전체의 85.2%를 먹고 있었다. 그리고 이건 OpenCV C++ 구현이라 내부로 들어갈 수 없다. `cv2.dft` 경로도 마찬가지 이유로 손댈 수 없다.

여기서 암달의 법칙이 작동한다는 걸 다시 실감했다. 최적화할 수 없는 부분이 85%면, 나머지 15%를 아무리 빠르게 만들어도 전체 향상의 상한이 정해진다. 이 현실을 받아들이고 나서야 다음 단계가 명확해졌다.

**컴파일 가능하면서, JIT 오버헤드보다 충분히 큰 노드를 찾는다.** 그게 `saliency_dft`(0.35ms)였다.

---

## compiler.partition(): Phase 1 Graph를 둘로 나누다

`CortexCompiler`는 Phase 1에서 만든 Graph를 그대로 받는다. 이게 두 Phase가 연결되는 방식이다.

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

결과:

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

`text_roi_mser`가 external_call인 이유, `scene_change`(SSIM)가 external_call인 이유를 `cortex/compiler/boundaries.md`에 정리했다. 결론은 하나다 — numba `nopython=True` 모드에서 실행 불가능한 모든 것은 external_call boundary 뒤에 둔다. MLIR의 `external func`나 TVM BYOC의 bring_your_own_codegen과 같은 개념이다.

---

## saliency_dft 커널: 세 가지 버전

`cortex/compiler/saliency_kernel.py`에 baseline, vectorized, jit 세 버전을 만들었다. 모든 버전은 수치적으로 동일한 출력을 보장해야 한다 — 이건 타협 없이.

### Baseline

```python
# Python ops → [Graph IR] → ...
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

### Vectorized

Python 루프를 없애는 게 목표였다. `np.add.reduceat`를 사용하면 그리드 분할과 집계를 루프 없이 처리할 수 있다.

```python
# Python ops → Graph IR → [Optimization Passes] → ...
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

### JIT

처음엔 `@njit(parallel=True)`를 썼다. 병렬화가 무조건 빠를 거라고 생각했다. 그런데 측정하니 오히려 느렸다. 그리드가 6×8=48셀이라 스레드 생성 오버헤드가 실제 계산보다 컸다. 결국 `@jit(nopython=True)`로 되돌렸다.

```python
# ... → Partitioning → [Codegen] → Runtime
# numba @jit → LLVM IR → 네이티브 기계어

from numba import jit

@jit(nopython=True, cache=True)
def saliency_jit(gray: np.ndarray, grid_h: int = 6, grid_w: int = 8) -> np.ndarray:
    """
    nopython=True: Python 객체 없이 순수 네이티브 실행 (object mode 차단)
    cache=True: 첫 컴파일 결과를 디스크에 저장 — 재시작할 때마다 웜업 반복 안 해도 됨
    
    cv2.dft 경로는 numba nopython mode에 진입 불가.
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

`examples/compiler_benchmark.py`에서 측정했다 (640×480, N=100).

```
=== Compilation Results ===

Kernel        ms/frame   Speedup   Pass
──────────────────────────────────────────
baseline      0.31 ms    1.0×      Python loop
vectorized    0.17 ms    1.9×      np.add.reduceat
jit           0.15 ms    2.1×      numba @jit → LLVM
```

솔직히 말하면, 2.1배는 기대보다 작은 숫자였다. 처음엔 10배는 나올 거라고 생각했다. 그런데 baseline이 이미 0.31ms로 작았다. NumPy가 이미 상당히 최적화돼 있기 때문이다. JIT의 이득이 크게 날 여지가 처음부터 좁았던 거다.

### 파이프라인 전체로 보면

실제 측정값으로 암달의 법칙을 계산했다.

$$\text{Pipeline Speedup} = \frac{1}{0.852 + \dfrac{0.044}{2.1} + 0.104} \approx 1.024$$

- $0.852$: text\_roi\_mser 비율 (컴파일 불가)
- $0.044$: saliency\_dft 비율 (2.1× 향상)
- $0.104$: 나머지 compilable 노드

커널 2.1배 향상이 파이프라인 전체에선 **2.4%**로 희석된다. 이게 처음에 설명한 2%다.

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

## 전체 파이프라인을 한 번 더 보면

```
Python ops
    ↓
Graph IR 생성 (Phase 1)
    ↓
Optimization Passes
  └── dead_node_elimination (POWER_SAVE: saliency_dft 제거)
    ↓
Partitioning
  ├── compilable: saliency_dft, motion_map, center_crop, score_fusion, ema_smooth
  └── external_call: text_roi_mser
    ↓
Codegen
  └── saliency_dft → @jit → LLVM IR → 네이티브 기계어
    ↓
Runtime
  ├── JIT 커널 실행
  └── external_call 노드 원본 실행
```

코드로 쓰면:

```python
graph = build_l2_graph()
graph = dead_node_elimination(graph, mode_weights)

compiler = CortexCompiler(graph)
compilable, external = compiler.partition(graph)
compiler.compile(compilable)
compiler.benchmark(n=100)

for frame in camera_stream:
    if l1_gate.should_process(frame):
        results = compiler.run(frame)
        vlm_api.call(frame, results["score_fusion"])
```

---

## 테스트 짜면서 발견한 것들

30개 테스트를 짰다. 핵심 세 가지:

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

세 번째 테스트가 생각보다 중요했다. 벤치마크 함수가 실제로 측정하고 있는지, 아니면 그냥 함수를 호출만 하고 있는지 검증하는 코드다. 이걸 안 짰으면 잘못된 숫자를 한참 믿고 있었을 것이다.

---

## 돌아보며

이 프로젝트에서 두 가지를 배웠다.

하나는 **측정 없이 타겟을 고르면 안 된다**는 것이다. score_fusion을 먼저 보고 "이걸 최적화하면 되겠다"고 생각했던 건 코드만 보고 병목을 추측한 거였다. 실제로는 전체 시간의 0.1%도 안 됐다.

다른 하나는 **병목이 컴파일 불가능한 영역에 있을 때 어떻게 할 것인가**를 결정하는 게 더 중요한 엔지니어링이라는 것이다. MSER 85.2%는 어찌할 수 없다. 이걸 인정하고 나머지 14.8% 중 의미 있는 부분을 찾아 정확히 최적화하는 것 — 그게 이 프로젝트가 실제로 한 일이다.

2.4%라는 숫자가 작아 보일 수 있다. 하지만 이 숫자는 추측이 아니라 측정에서 나왔고, 왜 이 숫자인지 수식으로 설명할 수 있다. 그게 더 중요하다고 생각한다.
