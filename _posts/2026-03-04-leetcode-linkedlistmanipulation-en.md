---
title: "Blind75 - Linked List Manipulation (The Art of Pointer Flow and Gap Keeping)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, linked-list, two-pointer, divide-and-conquer]
use_math: true
---

## Core Concepts of Linked Lists

Linked list problems aren't just about storing data — the real challenge lies in **how you rewire the `next` pointers** that each node uses to point to the next one.

Knowing how to traverse from `head` with `while head:` is the easy part. The hard part is **deciding the exact order** in which you cut and reattach nodes.

Two essential tools for solving this: **Dummy Node** and **Two-Pointer**.

---

## Initial Approach & Trial-and-Error (Logic Check)

When tackling **Remove Nth Node From End**, my first instinct was to calculate the total length of the list upfront.

### Failed Initial Code (Incomplete)

```python
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        count = 0
        curr = head
        while curr:
            curr = curr.next
            count += 1
        
        # Now we need to go back to head and move (count - n) steps
        # This requires two passes — inefficient and unnecessarily complex
```

### Analysis of the Errors

- **One-directional constraint**: You can't go backwards in a singly linked list. Once you reach the end, you have to restart from `head`.
- **Target offset confusion**: To delete the "nth from the end", you need to find the "(n+1)th from the end" — the step count arithmetic is easy to mess up.
- **Edge case**: Deleting the only node in the list (returning an empty list) requires special handling.

---

## Strategy 1: Two-Pointer (Gap Maintenance)

Instead of making two passes, we maintain a fixed gap of `n` between two pointers and move them together — elegant and intuitive.

### Core Logic

- Advance the `fast` pointer `n` steps ahead first.
- Then move both `slow` and `fast` forward one step at a time.
- When `fast` reaches the end, `slow` lands exactly **one node before the target to delete**.

```
Initial: dummy → 1 → 2 → 3 → 4 → 5, n=2
                  ↑                  ↑
                slow               fast (n steps ahead)

After:   dummy → 1 → 2 → 3 → 4 → 5 → None
                              ↑         ↑
                             slow      fast
→ Skip slow.next(4): slow.next = slow.next.next
```

### Optimized Code

```python
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        dummy = ListNode(0)
        dummy.next = head
        slow, fast = dummy, dummy

        # Move fast n+1 steps ahead so slow stops just before the target
        for _ in range(n + 1):
            fast = fast.next

        # Advance both until fast reaches None
        while fast:
            slow = slow.next
            fast = fast.next

        # slow is now the node just before the one to delete
        slow.next = slow.next.next
        return dummy.next
```

---

## Strategy 2: Divide and Conquer

For Hard problems like **Merge K Sorted Lists**, trying to merge everything at once causes complexity to explode. The solution is a **tournament-style approach** — merge pairs of lists round by round.

### Core Formula

- **Base Case**: Keep iterating until only one list remains.
- **Merge**: Reuse `mergeTwoLists` to merge sorted pairs, maximizing code reuse and efficiency.

```
Round 1: [L1, L2, L3, L4] → [merge(L1,L2), merge(L3,L4)]
Round 2: [M1, M2]         → [merge(M1,M2)]
Result:  Final sorted list
```

### Final Optimized Code (Merge K Sorted Lists)

```python
class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        if not lists: return None
        if len(lists) == 1: return lists[0]
        
        # Keep merging pairs until one list remains (Divide and Conquer)
        while len(lists) > 1:
            merged_lists = []
            for i in range(0, len(lists), 2):
                l1 = lists[i]
                l2 = lists[i + 1] if (i + 1) < len(lists) else None
                merged_lists.append(self.mergeTwoLists(l1, l2))
            lists = merged_lists
        return lists[0]

    def mergeTwoLists(self, l1, l2):
        dummy = ListNode(0)
        curr = dummy
        while l1 and l2:
            if l1.val < l2.val:
                curr.next, l1 = l1, l1.next
            else:
                curr.next, l2 = l2, l2.next
            curr = curr.next
        curr.next = l1 or l2
        return dummy.next
```

---

## Summary & Retrospective

### Why Are These Approaches Efficient?

- **Two-Pointer**: Finds the target in a single pass ($O(N)$) with constant space ($O(1)$) — no second traversal needed.
- **Divide and Conquer**: Naively merging all $K$ lists costs $O(NK)$, but the tournament approach guarantees $O(N \log K)$ performance.

### Key Takeaway

**"Before rewiring a node's connection, claim your destination first."**

The moment you reassign `next`, the old path is gone. That's why the habit of anchoring with a **Dummy Node** and pre-securing key positions with **Multiple Pointers** is the fastest route to a correct solution in linked list problems.
