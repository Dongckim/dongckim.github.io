---
title: "Blind75 - Longest Consecutive Sequence (Sorting-Based Approach)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, sorting, logic-check]
use_math: true
---

## Longest Consecutive Sequence

Given an unsorted integer array `nums`, find the length of the longest consecutive elements sequence.

This post examines a common logic error in the initial approach and shows how to fix it using **Sorting** and **Deduplication (Set)** to build a robust solution.

---

## Analyzing Logic Errors in the Initial Approach

A natural first idea is to check whether the next value (`lowest + 1`) exists in the array, counting consecutive hits as we go.

### The Flawed Code

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if not nums: return 0
        lst = []
        in_row = 1
        lowest = sorted(nums)[0]

        for i in range(len(nums)):
            if (lowest + 1) in nums:
                in_row += 1
                lowest += 1
            else:
                lst.append(in_row)
                lowest = nums[i]  # Critical bug
                in_row = 1
        return max(lst)
```

### Why Does It Fail?

This code has two critical flaws.

- **Duplicate interference**: For an input like `[1, 2, 2, 3]`, when the duplicate `2` is encountered, `lowest` is already at `3`. The check for `lowest + 1` (which is `4`) fails, causing the count to reset prematurely.
- **No order guarantee**: In the line `lowest = nums[i]`, the index order of the original `nums` does not guarantee that it points to "the next smallest number." As a result, `lowest` may jump back to an already-visited range, breaking the flow.

---

## Solution Strategy: Data Cleansing and Sorting

To fix the logic errors above, we must first ensure **Uniqueness** and **Order** in the data.

### Core Logic

- `set(nums)`: Removes duplicate numbers to prevent the count from resetting due to repeated values.
- `sorted()`: Sorts the numbers in ascending order, guaranteeing that the next index value is always greater than or equal to the current one.

---

## Final Implementation (Corrected Code)

This version preserves the structure of the original approach while fixing its logical flaws.

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        # Remove duplicates and sort to establish a sequential scan basis.
        nums = sorted(list(set(nums)))
        
        lst = []
        in_row = 1
        n = len(nums)
        lowest = nums[0]

        for i in range(n):
            # With sorted data, checking (lowest + 1) becomes valid.
            if (lowest + 1) in nums:
                in_row += 1
                lowest += 1
            else:
                # When the streak breaks, save the length and reset
                lst.append(in_row)
                lowest = nums[i]
                in_row = 1
        
        # Append the length of the last calculated streak
        lst.append(in_row)
        
        return max(lst)
```

---

## Summary & Reflection

### Why Are Sorting and Deduplication Essential?

- **Consistency**: Only with sorting can `nums[i]` reliably be "the smallest candidate after the streak breaks."
- **Stability**: Only with deduplication can we prevent the `lowest` update and `in_row` counting from getting confused by identical values.

### Performance Consideration

This approach has a time complexity of $O(N \log N)$ due to sorting. If the problem constraints require $O(N)$, an optimization using only a Hash Set can be applied — checking whether each number is the start of a consecutive sequence instead of sorting.
