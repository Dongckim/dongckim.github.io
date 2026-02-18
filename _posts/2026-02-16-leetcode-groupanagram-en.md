---
title: "Blind75 - Group Anagrams (Hash Map & Visited)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, hashmap]
use_math: true
---

## Group Anagrams

Given an array of strings, group the anagrams together — words that become identical when their characters are rearranged.  
This post covers the journey from basic list manipulation to efficient hash map usage in Python.

---

## First Approach: Bruteforce with `visited`

My initial idea was to compare every word one by one.  
However, using `pop()` while iterating over a list causes a critical index corruption problem.

To solve this, I introduced a **`visited` list**.

### Core Principle

- Leave the original list untouched  
- Track "has this word already been grouped?" using `True / False`.

### Key Takeaway

Instead of directly deleting elements from a list,  
using a `visited` array to record state  
allows safe control over the entire dataset without index errors.

---

### Revised Code (O(N^2) Version)

```python
from typing import List

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        answer = []
        n = len(strs)
        visited = [False] * n 
        
        for i in range(n):
            if visited[i]:  # Skip already processed words
                continue
            
            trial = [strs[i]]
            visited[i] = True
            
            for j in range(i + 1, n):
                # Not yet visited and sorted result matches -> anagram!
                if not visited[j] and sorted(strs[i]) == sorted(strs[j]):
                    trial.append(strs[j])
                    visited[j] = True
                    
            answer.append(trial)
            
        return answer
```

---

## The Downside: Time Complexity

The code above is logically correct, but leaves much to be desired in terms of performance.

### Time Complexity

O(N^2 · K log K)

- Outer loop: `N`
- Inner loop: `N`
- String sorting: `K log K`

With just 10,000 entries,  
this requires over 100 million operations.

In real coding tests, this is very likely to result in **Time Limit Exceeded (TLE)**.

---

## Optimization: Using a Hash Map (Dictionary)

Eliminate comparison operations entirely  
by using the sorted version of each word as a **Key (bucket name)**.

### The Art of Classification

A Hash Map is the optimal structure for instantly classifying data based on a specific "characteristic".

### Pythonic Way

Using `defaultdict(list)`,  
a new empty list is automatically created whenever a new Key is encountered,  
making the code extremely concise.

---

## Final Optimized Code (O(N · K log K))

```python
from collections import defaultdict
from typing import List

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        # Use sorted string as Key, original string list as Value
        groups = defaultdict(list)
        
        for s in strs:
            # "eat", "tea", "ate" -> all sort to "aet" (unique Key)
            key = "".join(sorted(s))
            groups[key].append(s)
            
        return list(groups.values())
```

---

## Summary & Reflection

### The Discovery of the `visited` Array
- Extremely useful for tracking state without modifying the list
- A key strategy for preventing index corruption

### The Efficiency of Hash Maps
- Hash maps are the most powerful tool for "grouping" problems
- Can improve time complexity from O(N^2) to O(N) level
- A core optimization pattern for real coding tests

### Conclusion

**Bruteforce → State Management → Hash Map Optimization**

Through this process, I went beyond simple implementation  
and gained a deep understanding of "the importance of choosing the right data structure".
