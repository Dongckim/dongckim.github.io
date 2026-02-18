---
title: "Blind75 - Container with Most Water (Two Pointer Approach)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, two-pointer, optimization]
use_math: true
---

## Container with Most Water

Given an array of vertical lines (pillars), choose two pillars and assume the space between them is filled with water. Find the maximum area of water that can be contained between any two pillars.

This post examines the limitations of a common **brute-force (nested loop)** approach and shows how to improve it to an efficient $O(N)$ solution using **Two Pointers**.

---

## Efficiency Analysis of the Initial Approach

The first idea might be to compare every pair of pillars and calculate the area for each.

### The Brute-force Code

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        n = len(heights)
        areas = []
        for i in range(n):
            for j in range(i+1, n):
                # The shorter pillar determines the water height
                if heights[i] < heights[j]:
                    area = (j-i) * heights[i]
                else: 
                    area = (j-i) * heights[j]
                areas.append(area)
        return sorted(areas)[-1]
```

### Why Does It Need Improvement?

This code has two shortcomings.

- **Time complexity**: Using nested `for` loops results in $O(N^2)$ time. With 100,000 pillars, the number of operations reaches 10 billion, causing Time Limit Exceeded.
- **Memory waste**: All area values are stored in the `areas` list and then sorted. Instead of keeping every value, we can simply track the maximum in real time to save memory.

---

## Solution Strategy: Two Pointers Closing In from Both Ends

Area is determined by width * height. Starting from both ends, we reduce the width from maximum to minimum while searching for the optimal height.

### Core Logic

- **Width always decreases**: Each time a pointer moves inward, the horizontal distance shrinks by 1.
- **Preserve the height**: The total area is determined by the shorter of the two pillars. Therefore, we must search for taller pillars to have any chance of increasing the area.
- **Move the shorter side**: Move the pointer pointing to the shorter of `heights[left]` and `heights[right]` inward.

---

## Final Implementation (Optimized Code)

Instead of creating a list, a single variable (`maxA`) is updated in real time with the maximum value, maximizing efficiency.

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        n = len(heights)
        left, right = 0, n-1
        maxA = 0
        
        while left < right:
            # Calculate height and width between the two pointers
            h = min(heights[left], heights[right])
            w = right - left
            
            # Update maximum in real time (using max instead of sorting a list)
            maxA = max(h * w, maxA)

            # Key: move the pointer on the shorter side to search for taller pillars
            if heights[left] < heights[right]:
                left += 1
            else:
                right -= 1
                
        return maxA
```

---

## Summary & Reflection

### Why Does Two Pointers Guarantee the Optimal Solution?

- **Narrowing the search space**: If we keep the shorter pillar fixed and move the taller one inward, the width decreases while the water height remains capped by the shorter pillar — so the area can only decrease. Therefore, the only scenario where the area could increase is by moving the shorter side.
- **Maximum tracking**: By managing the maximum with `max()` separately, we eliminate the need for unnecessary list storage and sorting.

### Performance Consideration

This approach scans the array only once, resulting in **$O(N)$** time complexity. The jump from $O(N^2)$ to $O(N)$ makes an enormous performance difference when processing large-scale data.
