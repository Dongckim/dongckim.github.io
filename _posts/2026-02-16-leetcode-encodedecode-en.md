---
title: "Python Coding Test - Encode and Decode Strings (Chunked Encoding)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, twopointer, design]
use_math: true
---

## Encode and Decode Strings

Design an algorithm to encode a list of strings to a single string. The encoded string is then sent over the network and should be decoded back to the original list of strings.

This post covers how to use **Chunked Encoding** with a **Two Pointer** approach to handle any character inclusion, including special delimiters.

---

## The Challenge: Handling Delimiters

A naive approach might use a simple delimiter like a comma (`,`) or a space. However, if the input strings themselves contain those characters, the decoding process will break. We need a way to distinguish between "data" and "metadata."

---

## Core Logic: Chunked Encoding

The most robust way to solve this is to prepend the **length of the string** and a **separator** before each actual string.

### Implementation

```python
class Solution:
    def encode(self, strs: List[str]) -> str:
        # Format: {length}#{word}
        res = ""
        for s in strs:
            res += str(len(s)) + "#" + s
        return res

    def decode(self, s: str) -> List[str]:
        res = []
        p1 = 0
        
        while p1 < len(s):
            p2 = p1
            # Seeker pointer: find the delimiter
            while s[p2] != "#":
                p2 += 1
            
            # Extract length and the word
            word_len = int(s[p1:p2])
            res.append(s[p2 + 1 : p2 + 1 + word_len])
            
            # Jump p1 to the start of the next chunk
            p1 = p2 + 1 + word_len
            
        return res
```

---

## Strategy: Two Pointer Entry Approach

Using a `while` loop with a two-pointer strategy is the most reliable way to navigate this encoded string.

- **p1 (Entry Pointer)**: Marks the start of the metadata (the length of the word).
- **p2 (Seeker Pointer)**: Moves from `p1` to find the `#` separator.
- **The "Jump"**: Once the length is parsed, we slice the exact amount of characters needed and "jump" `p1` to the next segment.

This "Dynamic Window" approach ensures that even if the string contains a `#` or numbers, the algorithm won't be confused because it only looks for delimiters at the specific indices where metadata is expected.

---

## Optimization & Considerations

### 1. String Concatenation Performance

In Python, strings are immutable. Repeatedly using `res += ...` creates a new string object each time, leading to $O(N^2)$ in the worst case for encoding.

**Improvement**: Use `"".join()` for $O(N)$ efficiency.

```python
def encode(self, strs: List[str]) -> str:
    return "".join(f"{len(s)}#{s}" for s in strs)
```

### 2. Time & Space Complexity

- **Time Complexity**: $O(N)$ for both encoding and decoding, where $N$ is the total number of characters across all strings.
- **Space Complexity**: $O(1)$ if we exclude the memory used for the output list/string.

---

## Summary & Reflection

### Why Two Pointers?

- **Robustness**: It handles edge cases like empty strings, strings with only numbers, or strings containing the delimiter itself.
- **Efficiency**: By skipping over the "body" of the string using the length metadata, we avoid unnecessary character checks.

### Conclusion

The Chunked Encoding method, paired with a Two Pointer decoding strategy, is a masterclass in handling serialized data. It teaches us how to design a protocol that is both simple and bulletproof against unexpected input data.
