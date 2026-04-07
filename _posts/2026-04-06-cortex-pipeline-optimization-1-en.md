---
title: "Cortex Inference Pipeline Optimization (1) — Graph IR and Dead Node Elimination"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, compiler, graph-ir, dead-node-elimination, mlir, tvm, edge-computing]
use_math: true
---

## Before We Start

Honestly, when I first set out to do this, the goal was ambitious — "I'm going to build an ML compiler." But the moment I actually dug into the codebase, I realized something obvious I had overlooked: before you can build a compiler, you need to be able to **see** the pipeline. If you don't know what's slow, what are you even optimizing?

That's where this started. `cortex/graph/` wasn't the flashy optimization work. It was the prerequisite — making the pipeline **visible** for the first time.

---

## The Problem: A Black-Box Pipeline

Cortex's L2 pipeline looked like this:

```
blur_detector → scene_change → hybrid_roi_scoring → encoder
```

Functions called in sequence. That's it. There was no way to know which nodes actually ran, which ones were slow, or which ones didn't even need to run at all.

Inside `hybrid_roi_scoring`, specifically, there were these sub-ops:

| Node | Operation | Notes |
|---|---|---|
| `center_crop` | Gaussian window → $S_c$ | Pure NumPy |
| `text_roi` | MSER → $S_t$ | OpenCV C++ |
| `saliency_dft` | FFT pipeline → $S_s$ | Pure NumPy |
| `motion_map` | absdiff + grid pool → $S_m$ | Pure NumPy |
| `score_fusion` | weighted sum → $S$ | Pure NumPy |

In POWER_SAVE battery mode, `ws=0.0`, which means `saliency_dft`'s output contributes nothing to the final score:

$$S = w_c \cdot S_c + w_t \cdot S_t + (0.0 \cdot S_s) + \cdots \quad \because w_s = 0.0$$

And yet `saliency_dft` ran every single frame — because the pipeline executed sequentially and never checked whether a result was actually consumed.

**The fix required representing the pipeline as a data structure**, not a call stack. Nodes and their dependency relationships needed to be explicit.

---

## Designing the Graph IR

I went through the MLIR and TVM docs while designing this. Two things mattered most.

**First: mark every node with whether it's compilable.**

This is the key that connects to the Phase 3 compiler later. Only nodes with `is_compilable=True` become JIT compilation targets. Nodes backed by C++ internals we can't enter — like OpenCV — get `op_type="external_call"` and `is_compilable=False`. This is the same idea as TVM keeping ops it can't lower as external function calls.

```python
@dataclass
class Node:
    op_type: str          # "mul", "add", "fft2", "external_call"
    inputs: List[Any]     # Node references or constant values
    outputs: List[str]
    metadata: Dict[str, Any] = field(default_factory=dict)
    is_compilable: bool = False
```

**Second: make temporal state explicit as named nodes.**

My first instinct was to just manage state as instance variables (`self._prev_gray`). The problem is that passes can't reason about state dependency at the graph level that way. I discovered this while writing tests — `dead_node_elimination` was making wrong decisions about nodes that had state dependencies.

```python
graph.add_node("_prev_gray",  Node(op_type="input", inputs=[], outputs=["prev_gray"]))
graph.add_node("_prev_score", Node(op_type="input", inputs=[], outputs=["prev_score"]))
```

Once those were explicit, the dependency relationships for `scene_change` (SSIM) and `motion_map` were correctly represented in the graph.

---

## The dead_node_elimination Pass

I initially planned to implement `constant_folding` as well. But thinking about it more — the weights (`wc`, `wt`, `ws`, `wm`) change per `RequestType` at runtime. There are almost no cases that are actually foldable. Adding the complexity wasn't worth it. That decision felt uncertain at the time, but looking back, it was the right call.

`dead_node_elimination` itself is straightforward: remove nodes whose outputs are never consumed downstream.

```python
def dead_node_elimination(graph: Graph, mode_weights: Dict[str, float]) -> Graph:
    dead = set()

    # Step 1: mark nodes with zero weight as dead
    for name, node in graph.nodes.items():
        weight_key = node.metadata.get("weight_key")
        if weight_key and mode_weights.get(weight_key, 1.0) == 0.0:
            dead.add(name)

    # Step 2: propagate — any node depending on a dead node is also dead
    changed = True
    while changed:
        changed = False
        for name, node in graph.nodes.items():
            if name not in dead:
                if any(inp in dead for inp in node.inputs
                       if isinstance(inp, str)):
                    dead.add(name)
                    changed = True

    # Step 3: report what was removed and how much time was saved
    eliminated = [n for n in dead if n in graph.nodes]
    if eliminated:
        saved_ms = sum(
            graph.nodes[n].metadata.get("profile_ms", 0.0)
            for n in eliminated
        )
        print(f"[dead_node_elimination] removed: {eliminated}")
        print(f"[dead_node_elimination] saved: ~{saved_ms:.1f}ms")

    return graph.subgraph(set(graph.nodes.keys()) - dead)
```

### What Actually Happens in POWER_SAVE Mode

POWER_SAVE sets `ws=0.0` and `wm=0.0` — saliency computation is skipped entirely, with weights redistributed into `wc`.

```python
POWER_SAVE_WEIGHTS = {"wc": 0.7, "wt": 0.3, "ws": 0.0, "wm": 0.0}
```

Run the pass with those weights and `saliency_dft` and `motion_map` disappear from the graph:

```
[dead_node_elimination] removed: ['saliency_dft', 'motion_map']
[dead_node_elimination] saved: ~4.5ms

Before:
  blur_detector    1.2ms
  scene_change    12.3ms
  center_crop      0.8ms
  text_roi        35.2ms
  saliency_dft     3.1ms  ← ws=0.0 → output never consumed
  motion_map       1.4ms  ← wm=0.0 → output never consumed
  score_fusion     0.01ms
  Total:          54.0ms

After:
  blur_detector    1.2ms
  scene_change    12.3ms
  center_crop      0.8ms
  text_roi        35.2ms
  score_fusion     0.01ms
  Total:          49.5ms  (~4.5ms saved)
```

The numbers matter less than the structural point: before Graph IR, **this optimization was architecturally impossible**. When the pipeline was just a sequence of function calls, removing a node meant editing the code directly.

---

## GraphVisualizer: Actually Seeing the Graph

Building this was the first time I could actually look at the pipeline as a structure. It sounds simple, but it was surprisingly useful — especially seeing `[compilable]` and `[external_call]` side by side immediately makes it clear where optimization headroom exists.

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

Actual output (POWER_SAVE mode, before/after):

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

## I Should Have Written Tests First

Writing the 33 tests surfaced two design mistakes. The temporal state node issue was one of them. If I'd written tests first, I would have caught the design problems earlier — that's on me.

Three core cases:

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
    assert graph.nodes["saliency_dft"].is_compilable is True # pure NumPy
    assert graph.nodes["motion_map"].is_compilable is True   # pure NumPy
    assert graph.nodes["score_fusion"].is_compilable is True # pure NumPy
```

---

## Looking Back

If there's one thing I took away from this work, it's that you have to be able to **see** something before you can optimize it. Graph IR doesn't deliver dramatic performance gains on its own. But **without it, the Phase 3 compiler couldn't exist.**

The `is_compilable` flag looks like a simple metadata field, but it's the single line that connects the two phases — when `CortexCompiler` receives a Graph and calls `partition()`, that one field determines what goes into the compilable subgraph and what stays behind the external_call boundary.

| This Project | ML Compiler Terminology |
|---|---|
| `Node.op_type` | operation / dialect |
| `dead_node_elimination()` | DCE (Dead Code Elimination) pass |
| `is_compilable=False` node | external_call boundary |
| `Graph.execute()` | eager mode interpretation |
| Phase 3 partitioning | BYOC (Bring Your Own Codegen) |

Part 2 covers the `CortexCompiler` that takes this Graph and JIT-compiles it. That one had some surprises too.
