---
title: "Blind75 - Longest Substring Without Repeating Characters (String Slicing & Sliding Window)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, sliding-window, logic-check]
use_math: true
---

## Longest Substring Without Repeating Characters

Find the length of the longest substring without repeating characters in a given string.

- Since it's a "Substring", the characters must be contiguous.
- May include uppercase, lowercase, spaces, and special characters.

---

## Initial Approach and Trial-and-Error (Logic Check)

The first attempt was to build a string character by character, checking for duplicates and skipping when found. However, several logical flaws were discovered.

### The Failing Initial Code

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        answer = ''
        for i, letter in enumerate(s):
            if not answer: continue  # Blocks the very first character
            if letter == s[len(s)-1]:  # Nonsensical comparison with the last character
                answer = ''
            if letter in answer:
                continue  # Skipping on duplicate breaks contiguity
            answer += letter
        return len(answer)
```

### Logic Error Analysis

- **First character ignored**: The `if not answer: continue` condition causes nothing to happen on the first iteration.
- **Misunderstanding duplicate handling**: Simply skipping a duplicate with `continue` breaks the flow of the 'contiguous substring' required by the problem.
- **Missing max length update**: Returning only the length of `answer` at the end of the loop misses the longest segment that may have occurred mid-iteration.

---

## Solution Strategy: String Slicing (Intuitive Approach)

Maintain the current `answer` and, when a duplicate is found, boldly discard everything before the duplicated character.

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        answer = ""
        max_len = 0
        
        for letter in s:
            if letter in answer:
                # Find the duplicate's position and keep only what comes after
                index = answer.find(letter)
                answer = answer[index + 1:]
            
            answer += letter
            # Update max length every iteration
            max_len = max(max_len, len(answer))
                
        return max_len
```

---

## Advanced: Sliding Window for Optimization

String slicing creates a new string object every time, so for large-scale data, using two pointers with a **Set** is more efficient.

### Final Optimized Code (Two Pointer)

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        char_set = set()
        left = 0
        max_len = 0
        
        for right in range(len(s)):
            # If the new character is a duplicate, move left pointer until it's gone
            while s[right] in char_set:
                char_set.remove(s[left])
                left += 1
            
            char_set.add(s[right])
            # Measure the window size (right - left + 1)
            max_len = max(max_len, right - left + 1)
            
        return max_len
```

---

## Summary & Reflection

### Why Sliding Window?

- **Efficiency**: Solves the problem in $O(n)$ time with a single pass through the string.
- **Flexibility**: Dynamically adjusts the window size when duplicates are found, maintaining the optimal range.

### Key Lesson

**"When a duplicate occurs, where do you restart from?"** This question is the starting point of the sliding window technique. The key is not simply discarding everything, but learning the instinct to narrow the range while preserving the valid segment.
