---
title: "Blind75 - Valid Parentheses (Stack & String Elimination)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, stack, logic-check]
use_math: true
---

## Valid Parentheses

Given a string `s` consisting of `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, determine if the brackets are closed in the correct order.

- Every open bracket must be closed by a bracket of the same type.
- Open brackets must be closed in reverse order (LIFO structure).

---

## Initial Approach and Trial-and-Error (Logic Check)

The first idea was to assume that bracket pairs form a mirror (symmetric) structure, and compare them by calculating index positions.

### The Failing Initial Code

```python
class Solution:
    def isValid(self, s: str) -> bool:
        char_valid = ['(', ')', '{', '}', '[', ']']
        for i, char in enumerate(s):
            if char == char_valid[0]:  # If '('
                # Assumes the closing bracket is at the 'symmetric' position
                if s.find(')') != (len(s) - i): 
                    return False
            # ... same logic applied for other brackets
        return True
```

### Logic Error Analysis

- **Misunderstanding symmetry**: A pattern like `()[]{}` is valid but not symmetric. The code only accounted for perfectly mirrored arrangements like `(( ))`.
- **Limitations of `.find()`**: `find()` always returns the first index in the string. It cannot locate the correct match when duplicate brackets appear.
- **Ignoring order**: Cases like `)(` where a closing bracket comes first cannot be caught.

---

## Solution Strategy 1: String Elimination (Intuitive Approach)

An intuitive idea: "Remove the innermost pairs first." Instead of calculating indices, repeatedly replace adjacent pairs (`()`, `[]`, `{}`) with empty strings to reduce the size.

### Core Logic

- As long as any of `()`, `[]`, `{}` exists in the string, keep performing `replace`.
- If an empty string (`""`) remains after all processing, it's valid. If anything is left over, it's invalid.

---

## Solution Strategy 2: Stack (Optimized)

Leverage the fact that the "opening order" and "closing order" of brackets are reversed (Last-In, First-Out) by using a stack.

### Core Formula

- **Open bracket**: Push onto the stack.
- **Close bracket**: Check if it matches the bracket on top of the stack (Pop).

---

## Final Optimized Code

```python
class Solution:
    def isValid(self, s: str) -> bool:
        # Dictionary mapping closing brackets to their opening counterparts
        lookup = {')': '(', '}': '{', ']': '['}
        stack = []
        
        for char in s:
            if char in lookup:  # Encountered a closing bracket
                # Pop from stack if not empty, otherwise use dummy value '#'
                top_element = stack.pop() if stack else '#'
                if lookup[char] != top_element:
                    return False
            else:  # Opening bracket
                stack.append(char)
        
        # Stack must be empty after full traversal for the string to be valid
        return not stack
```

---

## Summary & Reflection

### Why Is the Stack Approach Efficient?

- **Time complexity**: $O(n)$ — only a single pass through the string is needed. (String elimination can grow up to $O(n^2)$)
- **Accuracy**: Immediately catches interlocking cases like `{[(}])`.

### Key Lesson

**"When the most recent event must be processed first"** — that's when a **Stack** is the answer. The brackets problem may look like an index game on the surface, but at its core, it's about understanding the nesting structure of data.
