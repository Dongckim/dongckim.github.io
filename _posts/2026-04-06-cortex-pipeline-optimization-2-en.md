---
title: "Cortex Inference Pipeline Optimization (2) — ML Compiler Design and JIT Optimization"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, numba, llvm, jit, compiler, tvm, byoc, ml-compiler, profiling]
use_math: true
---

## I Picked the Wrong Target First

After building the Graph IR in [Part 1]({% post_url 2026-04-06-cortex-pipeline-optimization-1-en %}), the next step felt obvious — "JIT-compile the compilable nodes." Solid plan.

The first thing that caught my eye was `score_fusion`.

$$S = w_c \cdot S_c + w_t \cdot S_t + w_s \cdot S_s + w_m \cdot S_m$$

When Python evaluates this, it allocates seven temporary arrays. It looked exactly like the operator fusion examples from TVM papers. "Compile this and it'll be faster" — that was the thought.

So I profiled it.

```
score_fusion: 0.01ms
```

0.01ms. Smaller than the numba kernel launch overhead. **Compiling this would actually make it slower.** score_fusion operates on 48 cells (6×8 grid) — scalar operations that NumPy already handles in microseconds. This node had already hit the "numpy floor." I'd chosen a compilation target that had no room to improve.

This is what happens when you pick a target without measuring first. Looking at code and thinking "this could be optimized" is a completely different question from whether it's actually a bottleneck.

---

## Started Over with Measurement

I measured per-frame latency for each node independently — 640×480 frames, N=100 iterations. Logged everything to `examples/profiling_report.txt`. The results were more revealing than I expected.

```
=== Profiling Report (640×480, N=100) ===

Node              Type            ms/frame
─────────────────────────────────────────
text_roi_mser     external_call   6.79 ms   ← bottleneck
motion_map        compilable      0.79 ms
saliency_dft      compilable      0.35 ms   ← primary target
center_crop       compilable      0.03 ms
score_fusion      compilable      0.01 ms   ← numpy floor
ema_smooth        compilable      0.00 ms
─────────────────────────────────────────
Total                             7.97 ms
```

`text_roi_mser` was consuming 6.79ms — 85.2% of total runtime. And it's an OpenCV C++ implementation. There's no way in. The `cv2.dft` path is off-limits for the same reason.

This is where Amdahl's Law becomes real. If 85% of the runtime is unoptimizable, no matter how fast you make the remaining 15%, the overall speedup ceiling is already set. Accepting that made the next step clear.

**Find a node that's compilable and large enough that the JIT overhead doesn't eat the gain.** That was `saliency_dft` at 0.35ms.

---

## compiler.partition(): Splitting the Phase 1 Graph

`CortexCompiler` takes the Graph built in Phase 1 directly as input. That's how the two phases connect.

```python
# Python ops → Graph IR → Optimization Passes → Partitioning → Codegen → Runtime
#                                                ↑ here

class CortexCompiler:
    def __init__(self, graph: Graph):
        self.graph = graph

    def partition(self, graph: Graph) -> tuple[list[Node], list[Node]]:
        """
        Split the Phase 1 Graph by is_compilable flag.
        Same concept as TVM's partition_for_<target>() / BYOC pattern.

        is_compilable=True  → compilable subgraph (numba JIT target)
        is_compilable=False → external_call boundary (run as-is)
        """
        compilable, external = [], []
        for name in graph._topo_order:
            node = graph.nodes[name]
            (compilable if node.is_compilable else external).append(node)
        return compilable, external
```

Result:

```
Compilable subgraph:
  center_crop    (numpy: Gaussian window)
  saliency_dft   (numpy: FFT pipeline)     ← primary target
  motion_map     (numpy: absdiff + pool)
  score_fusion   (numpy: weighted sum)
  ema_smooth     (numpy: EMA)

External call boundary:
  text_roi_mser  (cv2.MSER — OpenCV black box)
```

I documented why `text_roi_mser` and `scene_change` (SSIM) are external_call nodes in `cortex/compiler/boundaries.md`. The short answer: anything that can't run in numba's `nopython=True` mode goes behind the external_call boundary. This maps directly to MLIR's `external func` and TVM BYOC's bring_your_own_codegen.

---

## The saliency_dft Kernel: Three Versions

`cortex/compiler/saliency_kernel.py` has three implementations. All three must produce numerically identical output — no compromise on that.

### Baseline

```python
# Python ops → [Graph IR] → ...
# baseline: interpreter re-evaluates bytecode on every iteration

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

The goal was to eliminate Python loops. `np.add.reduceat` handles grid-level aggregation without any explicit iteration.

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

My first attempt used `@njit(parallel=True)`. Parallelism should be faster, right? It was actually slower. The grid is 6×8=48 cells — thread creation overhead exceeded the actual computation. Rolled back to `@jit(nopython=True)`.

```python
# ... → Partitioning → [Codegen] → Runtime
# numba @jit → LLVM IR → native machine code

from numba import jit

@jit(nopython=True, cache=True)
def saliency_jit(gray: np.ndarray, grid_h: int = 6, grid_w: int = 8) -> np.ndarray:
    """
    nopython=True: pure native execution — no Python objects (blocks object mode fallback)
    cache=True: first compilation result saved to disk — no warmup cost on restart

    Note: cv2.dft can't enter numba nopython mode.
    → stays behind external_call boundary, same reason as MSER.
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

## Actual Benchmark Results

Measured in `examples/compiler_benchmark.py` (640×480, N=100):

```
=== Compilation Results ===

Kernel        ms/frame   Speedup   Pass
──────────────────────────────────────────
baseline      0.31 ms    1.0×      Python loop
vectorized    0.17 ms    1.9×      np.add.reduceat
jit           0.15 ms    2.1×      numba @jit → LLVM
```

Honestly, 2.1× was smaller than I expected. I had assumed something closer to 10×. But the baseline was already 0.31ms — NumPy is doing a lot of heavy lifting internally. The JIT headroom was narrow from the start.

### What That Means for the Full Pipeline

Plugging the actual measured values into Amdahl's Law:

$$\text{Pipeline Speedup} = \frac{1}{0.852 + \dfrac{0.044}{2.1} + 0.104} \approx 1.024$$

- $0.852$: text\_roi\_mser fraction (not compilable)
- $0.044$: saliency\_dft fraction (2.1× speedup)
- $0.104$: remaining compilable nodes

A 2.1× kernel speedup dilutes to **2.4%** at the pipeline level. That's the "2%" from the summary.

| Node | ms/frame | % of total |
|---|---|---|
| `text_roi_mser` | 6.79ms | 85.2% — bottleneck, untouchable |
| `motion_map` | 0.79ms | 9.9% — compilable |
| `saliency_dft` | 0.35ms | 4.4% — **primary target (2.1×)** |
| `center_crop` | 0.03ms | 0.4% |
| `score_fusion` | 0.01ms | <0.1% — numpy floor |
| `ema_smooth` | 0.00ms | — |
| **Total** | **7.97ms** | |

---

## The Full Pipeline, One More Time

```
Python ops
    ↓
Graph IR construction (Phase 1)
    ↓
Optimization Passes
  └── dead_node_elimination (POWER_SAVE: removes saliency_dft)
    ↓
Partitioning
  ├── compilable: saliency_dft, motion_map, center_crop, score_fusion, ema_smooth
  └── external_call: text_roi_mser
    ↓
Codegen
  └── saliency_dft → @jit → LLVM IR → native machine code
    ↓
Runtime
  ├── compiled kernel execution
  └── external_call nodes run as original functions
```

In code:

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

## Things I Discovered While Writing Tests

30 tests total. Three core ones:

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

The third test turned out to matter more than I expected. It verifies that the benchmark function is actually measuring something — not just calling the function without recording results. Without it, I would have been trusting wrong numbers for a while.

---

## Looking Back

Two things stuck with me from this project.

One: **never pick an optimization target without measuring first**. Looking at `score_fusion` and thinking "that can be optimized" was me guessing at the bottleneck from code structure alone. It was less than 0.1% of total runtime.

Two: **deciding what to do when the bottleneck is uncompilable is the harder engineering problem.** There's nothing I can do about MSER taking 85.2%. Accepting that, and then precisely finding and optimizing the meaningful fraction within the remaining 14.8% — that's what this project actually did.

2.4% looks small. But it came from measurement, not estimation. And I can explain exactly why it's 2.4% with a formula. I think that matters more.
