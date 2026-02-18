---
title: "Blind75 - Valid Palindrome (Logic Correction & Two Pointer)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, twopointer, logic-check]
use_math: true
---

## Valid Palindrome

Determine whether a given string is a **palindrome** (reads the same forwards and backwards).

- Case-insensitive comparison.
- Ignore all non-alphanumeric characters.
- An empty string is considered a valid palindrome.

---

## Initial Approach and Trial-and-Error (Logic Check)

The first attempt was to simply remove spaces and compare characters by index, but several critical flaws were discovered.

### The Failing Initial Code

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        s = "".join(s.split())  # Bug 1: Special characters are not handled
        n = len(s)
        for i in range(n):
            if s[i] == s[n-1-i]:
                continue
            else:
                return False
            if i == n/2:  # Bug 2: Float comparison issue & improper loop exit
                return True
```

### Logic Error Analysis

- **Missing special character handling**: `s.split()` only removes whitespace. Characters like commas (`,`) and periods (`.`) remain and interfere with comparison.
- **Incorrect termination condition**: `i == n/2` compares an integer with a float, which is error-prone. It is safest to return `True` after the loop completes naturally.
- **Unnecessary computation**: A palindrome only needs to be checked up to the halfway point of the string.

---

## Solution Strategy: Data Cleansing and Comparison Optimization

### Filtering and Slicing (Pythonic)

Using Python's `isalnum()` and slicing (`[::-1]`), this can be solved concisely and powerfully.

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        # Keep only alphanumeric characters and convert to lowercase
        filtered_s = "".join(char.lower() for char in s if char.isalnum())
        # Compare with its reverse
        return filtered_s == filtered_s[::-1]
```

---

## Advanced: Memory Optimization with Two Pointers

The slicing approach above is easy to understand, but it requires creating a new string (`filtered_s`), which can cause memory overhead for very large inputs. To avoid this, we use a two-pointer approach with **$O(1)$ space complexity**.

### Final Optimized Code

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        left, right = 0, len(s) - 1
        
        while left < right:
            # Skip non-alphanumeric on the left
            if not s[left].isalnum():
                left += 1
            # Skip non-alphanumeric on the right
            elif not s[right].isalnum():
                right -= 1
            # Both are valid characters, compare them
            else:
                if s[left].lower() != s[right].lower():
                    return False
                left += 1
                right -= 1
                
        return True
```

---

## Summary & Reflection

### Why Two Pointers?

- **Space efficiency**: No new string is created; only indices are manipulated, resulting in minimal memory usage.
- **Real-world value**: In large-scale data processing or embedded environments, 'searching' the original data is far preferred over 'copying' and transforming it.

### Key Lesson

**Should you transform the data, or traverse it?** This question is a crucial criterion that determines memory efficiency. In coding tests, it is not just the precision of logic but also the consideration of space complexity that separates skill levels.
