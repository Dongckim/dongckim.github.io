---
title: "Blind75 - Longest Repeating Character Replacement (Sliding Window & Frequency Check)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, sliding-window, logic-check]
use_math: true
---

## Longest Repeating Character Replacement

Given a string `s` and an integer `k`, find the length of the longest substring containing the same letter after performing at most $k$ character replacements.

- Must be a contiguous "Substring."
- The result depends on how the $k$ opportunities are used.

---

## Initial Approach and Trial-and-Error (Logic Check)

The first attempt involved moving pointers, simply appending characters, and looping through $k$ opportunities. However, errors arose in the window concept and data storage.

### The Failing Initial Code

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        n = len(s)
        counter = {}
        max_freq = 0
        left = 0
        answer = 0
        for right in range(n):
            # 1. Only retrieves the value without updating the dictionary
            counter.get(s[right], 0) + 1 
            
            # 2. Confuses max_freq with 'window length' instead of 'frequency'
            max_freq = max(max_freq, right - left + 1)
            
            while (right - left + 1) - max_freq > k:
                counter[s[left]] -= 1
                left += 1
            
            # 3. Updates max_freq again instead of the answer variable
            max_freq = max(answer, right - left + 1)
        return answer
```

### Logic Error Analysis

- **Missing update**: `counter.get()` only returns a value; without assignment (`=`), nothing is stored in the dictionary.
- **Variable role confusion**: `max_freq` should represent "the count of the most frequent character in the window," but it was confused with "the current window length," neutralizing the `while` condition.
- **Result never updated**: The `answer` variable was never updated, so the initial value `0` was always returned.

---

## Solution Strategy: Sliding Window

Fix the most frequent character in the window as the base, and check whether the remaining characters can be replaced within $k$ times to expand the window.

### Core Formula

$$\text{Window Length} - \text{Max Frequency} \le k$$

**(Total window length) - (Count of the most frequent character)** equals **"the number of characters that need to be replaced."** This value must be less than or equal to $k$ for the window to be valid.

---

## Final Optimized Code (Two Pointer)

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        counter = {}
        max_f = 0  # Max frequency within the window
        left = 0
        answer = 0
        
        for right in range(len(s)):
            # Add the right character to the window and update frequency
            counter[s[right]] = counter.get(s[right], 0) + 1
            max_f = max(max_f, counter[s[right]])
            
            # If the window is invalid (replacements needed exceed k), shrink from the left
            while (right - left + 1) - max_f > k:
                counter[s[left]] -= 1
                left += 1
            
            # Update max length
            answer = max(answer, right - left + 1)
            
        return answer
```

---

## Summary & Reflection

### Why Is This Approach Efficient?

- **Time complexity**: $O(n)$ — the string is traversed only once to find the answer.
- **Space complexity**: Since only uppercase letters are used, the dictionary size is capped at 26, making it effectively $O(1)$.

### Key Lesson

**"Set a standard and adjust the rest."** In this problem, the standard is always 'the most frequent character.' This was a great exercise in learning the classic sliding window mechanism: identify the dominant force within the window, absorb the weaker forces (up to $k$), and expand the territory.
