---
title: "Cortex 추론 파이프라인 최적화 (1) — Graph IR과 Dead Node Elimination"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, compiler, graph-ir, dead-node-elimination, mlir, tvm, edge-computing]
use_math: true
---

## 시작하기 전에

솔직히 말하면, 이 작업을 처음 시작할 때 목표가 거창했다. "ML 컴파일러를 만들겠다"는 생각이었다. 그런데 막상 코드를 뜯어보고 나서야 알게 됐다 — 컴파일러를 만들기 전에 파이프라인을 **볼 수 있어야** 한다는 걸. 어디가 느린지 모르는데 뭘 최적화한다는 건지.

그게 이 글의 출발점이다. `cortex/graph/`는 화려한 최적화 전에, 파이프라인을 처음으로 **눈으로 볼 수 있게** 만든 작업이었다.

---

## 문제: 블랙박스 파이프라인

Cortex의 L2 파이프라인은 이런 흐름이었다.

```
blur_detector → scene_change → hybrid_roi_scoring → encoder
```

코드를 보면 함수들이 순서대로 호출된다. 딱 거기서 끝이다. 어떤 노드가 실행됐는지, 어떤 노드가 느린지, 어떤 노드는 사실 실행 안 해도 되는지 — 아무것도 알 수 없었다.

특히 `hybrid_roi_scoring` 안에는 이런 sub-op들이 있었다.

| 노드 | 연산 | 비고 |
|---|---|---|
| `center_crop` | Gaussian window → $S_c$ | 순수 NumPy |
| `text_roi` | MSER → $S_t$ | OpenCV C++ |
| `saliency_dft` | FFT pipeline → $S_s$ | 순수 NumPy |
| `motion_map` | absdiff + grid pool → $S_m$ | 순수 NumPy |
| `score_fusion` | weighted sum → $S$ | 순수 NumPy |

POWER_SAVE 배터리 모드에서 `ws=0.0`이 되면 `saliency_dft`의 결과는 최종 점수에 아무 영향도 없다.

$$S = w_c \cdot S_c + w_t \cdot S_t + (0.0 \cdot S_s) + \cdots \quad \because w_s = 0.0$$

그런데도 `saliency_dft`는 매 프레임 실행된다. 왜냐면 파이프라인이 그냥 순서대로 실행되는 구조라서, 결과가 쓰이든 안 쓰이든 신경 쓰지 않았으니까.

**이걸 고치려면 파이프라인을 데이터 구조로 표현해야 했다.** 코드 흐름이 아니라, 노드와 의존 관계가 명시된 그래프로.

---

## Graph IR 설계: 무엇을 담을 것인가

MLIR이나 TVM 문서를 보면서 설계했다. 핵심은 두 가지였다.

**첫째, 노드마다 컴파일 가능 여부를 명시한다.**

이게 나중에 Phase 3 컴파일러와 연결되는 열쇠다. `is_compilable=True`인 노드만 JIT 컴파일 대상이 된다. OpenCV처럼 C++ 내부로 들어갈 수 없는 노드는 `op_type="external_call"`, `is_compilable=False`로 표현한다. TVM에서 컴파일러가 lower하지 못하는 연산을 외부 함수 호출로 유지하는 것과 같은 개념이다.

```python
@dataclass
class Node:
    op_type: str          # "mul", "add", "fft2", "external_call"
    inputs: List[Any]     # Node 참조 또는 상수값
    outputs: List[str]
    metadata: Dict[str, Any] = field(default_factory=dict)
    is_compilable: bool = False
```

**둘째, temporal state를 named node로 만든다.**

처음엔 그냥 인스턴스 변수로 관리했다(`self._prev_gray`). 그런데 이렇게 하면 패스(pass)가 state dependency를 그래프 레벨에서 추론할 수 없다. dead_node_elimination이 state를 가진 노드를 처리할 때 잘못된 판단을 내릴 수 있다는 걸 테스트 짜다가 발견했다.

```python
graph.add_node("_prev_gray",  Node(op_type="input", inputs=[], outputs=["prev_gray"]))
graph.add_node("_prev_score", Node(op_type="input", inputs=[], outputs=["prev_score"]))
```

이걸 명시하고 나서야 `scene_change`(SSIM)와 `motion_map`의 의존 관계가 그래프에 정확히 표현됐다.

---

## dead_node_elimination 패스

처음엔 `constant_folding` 패스도 같이 만들려고 했다. 근데 생각해보니 가중치(`wc`, `wt`, `ws`, `wm`)가 `RequestType`마다 런타임에 바뀐다. 실제로 fold 가능한 케이스가 거의 없다. 복잡도만 올라가고 실익이 없어서 뺐다. 이런 결정이 처음엔 "이게 맞나?" 싶었는데, 돌아보면 맞는 선택이었다.

`dead_node_elimination`은 단순하다. 출력이 downstream에서 소비되지 않는 노드를 제거한다.

```python
def dead_node_elimination(graph: Graph, mode_weights: Dict[str, float]) -> Graph:
    dead = set()

    # 1단계: 가중치 0인 노드를 dead로 마킹
    for name, node in graph.nodes.items():
        weight_key = node.metadata.get("weight_key")
        if weight_key and mode_weights.get(weight_key, 1.0) == 0.0:
            dead.add(name)

    # 2단계: dead 노드에 의존하는 노드도 연쇄 전파
    changed = True
    while changed:
        changed = False
        for name, node in graph.nodes.items():
            if name not in dead:
                if any(inp in dead for inp in node.inputs
                       if isinstance(inp, str)):
                    dead.add(name)
                    changed = True

    # 3단계: 제거된 노드와 절약된 시간 출력
    eliminated = [n for n in dead if n in graph.nodes]
    if eliminated:
        saved_ms = sum(
            graph.nodes[n].metadata.get("profile_ms", 0.0)
            for n in eliminated
        )
        print(f"[dead_node_elimination] 제거: {eliminated}")
        print(f"[dead_node_elimination] 절약: ~{saved_ms:.1f}ms")

    return graph.subgraph(set(graph.nodes.keys()) - dead)
```

### POWER_SAVE 모드에서 실제로 무슨 일이 일어나냐면

POWER_SAVE 모드는 `ws=0.0`, `wm=0.0`으로 설정된다. saliency 계산을 아예 스킵하고 가중치를 `wc`에 재분배하는 방식이다.

```python
POWER_SAVE_WEIGHTS = {"wc": 0.7, "wt": 0.3, "ws": 0.0, "wm": 0.0}
```

이걸 패스에 넣으면 `saliency_dft`와 `motion_map`이 그래프에서 사라진다.

```
[dead_node_elimination] 제거: ['saliency_dft', 'motion_map']
[dead_node_elimination] 절약: ~4.5ms

Before:
  blur_detector    1.2ms
  scene_change    12.3ms
  center_crop      0.8ms
  text_roi        35.2ms
  saliency_dft     3.1ms  ← ws=0.0 → 출력 미사용
  motion_map       1.4ms  ← wm=0.0 → 출력 미사용
  score_fusion     0.01ms
  Total:          54.0ms

After:
  blur_detector    1.2ms
  scene_change    12.3ms
  center_crop      0.8ms
  text_roi        35.2ms
  score_fusion     0.01ms
  Total:          49.5ms  (약 4.5ms 절약)
```

숫자 자체보다 중요한 건, Graph IR을 도입하기 전엔 **이런 최적화가 구조적으로 불가능했다**는 점이다. 파이프라인이 코드 흐름이었을 땐 노드를 제거하려면 코드를 직접 수정해야 했다.

---

## GraphVisualizer: 그래프를 눈으로 보기

이걸 만들면서 처음으로 파이프라인이 어떻게 생겼는지 눈으로 볼 수 있었다. 단순한 기능인데 생각보다 유용했다. 특히 `[compilable]`과 `[external_call]`을 나란히 보면 어디가 최적화 가능한 영역인지 바로 보인다.

```python
class GraphVisualizer:
    def print(self, graph: Graph) -> None:
        print("=" * 55)
        print("Cortex Graph IR")
        print("=" * 55)
        for name in graph._topo_order:
            node = graph.nodes[name]
            tag = "[compilable]   " if node.is_compilable else "[external_call]"
            ms = node.metadata.get("profile_ms", 0.0)
            print(f"  {tag}  {name:<20} ({ms:.2f}ms)")
        print("=" * 55)

    def print_diff(self, before: Graph, after: Graph) -> None:
        removed = set(before.nodes) - set(after.nodes)
        print("\n[dead_node_elimination diff]")
        for name in before._topo_order:
            prefix = "  - REMOVED" if name in removed else "  ✓        "
            ms = before.nodes[name].metadata.get("profile_ms", 0.0)
            print(f"{prefix}  {name:<20} (~{ms:.1f}ms)")
```

실제 출력 (POWER_SAVE 모드 전/후):

```
==================================================
Cortex Graph IR
==================================================
  [external_call]  blur_detector        (1.20ms)
  [external_call]  scene_change         (12.30ms)
  [compilable]     center_crop          (0.80ms)
  [external_call]  text_roi             (35.20ms)
  [compilable]     saliency_dft         (3.10ms)
  [compilable]     motion_map           (1.40ms)
  [compilable]     score_fusion         (0.01ms)
  [external_call]  encoder              (0.50ms)
==================================================

[dead_node_elimination diff]
  ✓          blur_detector        (~1.2ms)
  ✓          scene_change         (~12.3ms)
  ✓          center_crop          (~0.8ms)
  ✓          text_roi             (~35.2ms)
  - REMOVED  saliency_dft         (~3.1ms)
  - REMOVED  motion_map           (~1.4ms)
  ✓          score_fusion         (~0.01ms)
  ✓          encoder              (~0.5ms)
```

---

## 테스트를 먼저 짰어야 했다

33개 테스트를 짜면서 설계 실수를 두 군데 발견했다. temporal state 노드 문제도 그 중 하나였다. 처음부터 테스트를 먼저 짰더라면 설계를 더 빨리 잡았을 텐데 — 이건 반성이다.

핵심 케이스 세 가지:

```python
def test_saliency_dft_eliminated_in_power_save():
    graph = build_l2_graph()
    optimized = dead_node_elimination(graph, POWER_SAVE_WEIGHTS)
    assert "saliency_dft" not in optimized.nodes
    assert "motion_map" not in optimized.nodes

def test_execute_output_identical_before_after_elimination():
    graph = build_l2_graph()
    frame = np.zeros((480, 640, 3), dtype=np.uint8)
    out_before = graph.execute({"frame": frame, **POWER_SAVE_INPUTS})
    optimized = dead_node_elimination(graph, POWER_SAVE_WEIGHTS)
    out_after = optimized.execute({"frame": frame, **POWER_SAVE_INPUTS})
    np.testing.assert_allclose(
        out_before["score_fusion"], out_after["score_fusion"]
    )

def test_is_compilable_correct_for_all_nodes():
    graph = build_l2_graph()
    assert graph.nodes["text_roi"].is_compilable is False    # MSER = external_call
    assert graph.nodes["saliency_dft"].is_compilable is True # 순수 NumPy
    assert graph.nodes["motion_map"].is_compilable is True   # 순수 NumPy
    assert graph.nodes["score_fusion"].is_compilable is True # 순수 NumPy
```

---

## 돌아보며

이 작업에서 배운 게 있다면, "최적화하기 전에 먼저 볼 수 있어야 한다"는 것이다. Graph IR은 그 자체로 엄청난 성능 향상을 가져다주진 않는다. 하지만 **이게 없었다면 Phase 3 컴파일러는 존재할 수 없었다.**

`is_compilable` 플래그는 단순한 메타데이터처럼 보이지만, Phase 3에서 `CortexCompiler`가 Graph를 받아 `partition()`을 호출할 때 이 필드 하나로 compilable subgraph와 external_call boundary를 나눈다. 두 Phase가 그 한 줄로 연결된다.

| 이 프로젝트 | ML 컴파일러 용어 |
|---|---|
| `Node.op_type` | operation / dialect |
| `dead_node_elimination()` | DCE (Dead Code Elimination) pass |
| `is_compilable=False` 노드 | external_call boundary |
| `Graph.execute()` | eager mode interpretation |
| Phase 3 파티셔닝 | BYOC (Bring Your Own Codegen) |

2편에서는 이 Graph를 받아 JIT 컴파일하는 `CortexCompiler`를 다룬다. 그쪽에서도 처음 생각했던 것과 실제가 꽤 달랐다.
