---
title: "Blind75 - 3Sum (The Pitfall of Duplicates & Two Pointer Efficiency)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, twopointer, logic-check]
use_math: true
---

## 3Sum

Given an array `nums`, find all unique triplets that sum to zero.

- Duplicate combinations must not be included in the result.
- The order of the answer does not matter.
- An efficient solution around $O(N^2)$ is expected.

---

## Initial Approach and Trial-and-Error (Logic Check)

The first attempts involved using `set()` to remove duplicates or tracking used indices, but both led to logical flaws and performance issues.

### Analysis of the Failing Initial Code

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums = list(set(nums))  # Bug 1: Cannot form triplets like [0,0,0]
        answer = []
        n = len(nums)
        used = [False] * n

        for i in range(n):
            if used[i]: continue
            for j in range(i+1, n):
                if used[j]: continue
                twoSum = nums[i] + nums[j]
                if (-twoSum) in nums:
                    used[i], used[j] = True, True  # Bug 2: A number can be part of multiple valid combinations
                    answer.append([nums[i], nums[j], -twoSum])
        return answer
```

### Logic Errors and Performance Analysis

- **Data loss**: Applying `set(nums)` makes it impossible to solve cases like `[-1, -1, 2]` where the same number must be used twice.
- **Pointer isolation**: Locking a number with `used[i] = True` prevents it from being found in other valid combinations.
- **Time complexity**: The `in` operation inside nested `for` loops results in near-$O(N^3)$ performance, causing Time Limit Exceeded on large datasets.

---

## Solution Strategy: Sorting and Two Pointers

The most efficient approach is to sort the array, fix one anchor point, and narrow the remaining two points inward from both ends using a two-pointer strategy.

### The Core Deduplication Logic: `i > 0 and nums[i] == nums[i-1]`

This condition is the most elegant way to avoid duplicate results.

- **Granting the first chance**: When `i=0`, `i > 0` evaluates to `False`, so the `if` statement is bypassed and the computation proceeds. (Short-circuit evaluation)
- **Blocking subsequent duplicates**: From `i=1` onward, if the current value equals the previous one (`nums[i-1]`), the combination was already processed in a prior iteration, so it is skipped with `continue`.
- **Solving the [0, 0, 0] case**: With this logic, the first `0` serves as the anchor, while the remaining two `0`s are found by the pointers (`left`, `right`), correctly returning `[[0, 0, 0]]`.

---

## Final Optimized Code

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums.sort()
        answer = []
        n = len(nums)

        for i in range(n - 2):
            # Skip duplicate anchor points (key lesson)
            if i > 0 and nums[i] == nums[i - 1]:
                continue
            
            left, right = i + 1, n - 1
            while left < right:
                total = nums[i] + nums[left] + nums[right]
                
                if total < 0:
                    left += 1
                elif total > 0:
                    right -= 1
                else:
                    answer.append([nums[i], nums[left], nums[right]])
                    # Skip duplicate elements for left and right
                    while left < right and nums[left] == nums[left + 1]:
                        left += 1
                    while left < right and nums[right] == nums[right - 1]:
                        right -= 1
                    left += 1
                    right -= 1
        return answer
```

---

## Summary & Reflection

### Why Is Brute Force ($O(N^3)$) Bad?

- **Space-time trade-off**: Simply dumping results into a `set()` is easy to implement but wastes unnecessary computation and memory (Set storage).
- **Algorithmic elegance**: The two-pointer approach maintains $O(1)$ space complexity (excluding the result array) while optimizing time complexity to $O(N^2)$.

### Key Lesson

**"Grant the first opportunity, but screen from the second onward."**

Rather than blindly removing data with `set` to handle duplicates, controlling the flow by comparing with the previous index in a sorted state is far safer and more efficient. Additionally, leveraging Python's short-circuit evaluation of `and` prevents index errors while keeping the logic concise.
