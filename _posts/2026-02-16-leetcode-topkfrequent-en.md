---
title: "Blind75 - Top K Frequent Elements (Heap & Bucket Sort)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, hashmap, heap, bucketsort]
use_math: true
---

## Top K Frequent Elements

Given an integer array `nums`, extract the top `k` most frequent elements.  
This post covers the journey from a naive loop approach to achieving $O(N)$ time complexity.

---

## First Approach: The Limits of Loops and Conditionals

My initial idea was to compute frequencies with `Counter`, then iterate and find values satisfying a specific condition (`n > k`).

### Trial-and-Error Code (Logic Error)

```python
def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    answer = []
    count = Counter(nums)
    for i, n in count:  # Tuple unpacking error
        if n > k:  # Logic error: checking 'frequency > k' instead of 'top k'
            answer.append(count[i])
    return count
```

### Key Takeaway

- When iterating over a `Counter` object, use `.items()` to get (number, frequency) pairs.
- To find the "top $k$", simple comparison is not enough — **Sorting** or a **Priority Queue (Heap)** is required.

---

## Second Approach: Sorting (O(N log N))

Sort all data in descending order by frequency, then slice the top $k$ elements.

### Core Principle

- Extract frequency data via `Counter(nums).items()`.
- Use the `key` parameter of `sorted()` to sort by frequency.

```python
from collections import Counter

def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    # Sort in descending order by frequency (x[1])
    sorted_counts = sorted(count.items(), key=lambda x: x[1], reverse=True)
    
    # Extract only the 'numbers' of the top k using list comprehension
    return [x[0] for x in sorted_counts[:k]]
```

---

## Third Approach: Heap Optimization (O(N log k))

Instead of sorting all elements, maintain a heap of size $k$ and pick only the top elements.  
This is efficient when the dataset is large and $k$ is relatively small.

### Pythonic Way (heapq)

Using Python's `heapq.nlargest`, the heap algorithm is executed internally,  
allowing a clean one-liner solution.

```python
import heapq
from collections import Counter

def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    # Extract top k elements by frequency
    return heapq.nlargest(k, count.keys(), key=count.get)
```

---

## Final Optimization: Bucket Sort (O(N))

Going beyond the $O(N \log N)$ limit of comparison-based sorting,  
this approach uses frequency itself as an index to solve the problem in $O(N)$.

### The Art of Classification

- The index becomes the 'frequency', and the value at that index is 'a list of numbers with that frequency'.
- Scan in reverse from the largest index (maximum frequency) and collect $k$ elements.

### Final Optimized Code

```python
from collections import Counter

def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    # Create buckets using frequency as index (max frequency is len(nums))
    bucket = [[] for _ in range(len(nums) + 1)]
    
    for num, freq in count.items():
        bucket[freq].append(num)
        
    answer = []
    # Scan from highest frequency (back of the bucket) and fill answer
    for i in range(len(bucket) - 1, 0, -1):
        for n in bucket[i]:
            answer.append(n)
            if len(answer) == k:
                return answer
```

---

## Summary & Reflection

### Choosing the Right Data Structure
- **Sorting** ($O(N \log N)$): The most intuitive implementation, but average performance.
- **Heap** ($O(N \log k)$): Memory-efficient and optimized for maintaining the top $k$ elements.
- **Bucket Sort** ($O(N)$): Theoretically the fastest when the frequency range is bounded (within array length).

### Conclusion

What started as a simple loop evolved into the powerful Bucket Sort algorithm.  
This was an insightful process that demonstrated how  
"placement by index" instead of "comparison-based sorting"  
can dramatically boost performance.
