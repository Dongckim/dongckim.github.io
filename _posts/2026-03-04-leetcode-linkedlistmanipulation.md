---
title: "Blind75 - Linked List Manipulation (포인터의 흐름과 간격의 미학)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, linked-list, two-pointer, divide-and-conquer]
use_math: true
---

## Linked List (연결 리스트) 핵심 개념

연결 리스트 문제는 단순히 데이터를 담는 것이 아니라, 각 노드가 다음 노드를 가리키는 '이정표(next)'를 어떻게 재구성하느냐가 핵심이다.

`head`에서 시작해 `while head:`로 순회하는 법은 알지만, 그 과정에서 노드를 끊고 다시 잇는 **순서를 결정하는 것**이 가장 어렵다.

이를 해결하기 위해 **Dummy Node**와 **Two-Pointer** 기술이 필수적으로 사용된다.

---

## 초기 접근 및 시행착오 (Logic Check)

**Remove Nth Node From End** 문제를 풀 때, 처음에는 리스트의 전체 길이를 먼저 구하려고 시도했다.

### 실패했던 초기 코드 (미완성)

```python
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        count = 0
        curr = head
        while curr:
            curr = curr.next
            count += 1
        
        # 여기서 다시 처음으로 돌아가 (count - n)번 이동해야 함
        # 리스트를 두 번 훑어야 하므로 비효율적이고 코드가 복잡해짐
```

### 논리 오류 및 어려움 분석

- **단방향의 한계**: 연결 리스트는 뒤로 돌아갈 수 없다. 끝에 도달하면 다시 `head`로 돌아가야 하는 번거로움이 생긴다.
- **삭제 타겟 설정**: "뒤에서 n번째"를 지우려면 "뒤에서 n+1번째" 노드를 찾아야 하는데, 이동 횟수 계산이 헷갈리기 쉽다.
- **엣지 케이스**: 노드가 하나뿐인데 그걸 지워야 하는 경우(빈 리스트 반환) 처리가 까다롭다.

---

## 해결 전략 1: Two-Pointer (간격 유지법)

리스트를 두 번 순회하지 않고, 두 포인터 사이의 거리를 `n`만큼 벌린 채 이동하는 직관적인 방식이다.

### 핵심 로직

- `fast` 포인터를 먼저 `n`보 전진시킨다.
- 그 후 `slow`와 `fast`를 동시에 한 칸씩 이동시킨다.
- `fast`가 끝에 도달하면, `slow`는 정확히 **삭제할 노드 바로 직전**에 멈춘다.

```
초기 상태: dummy → 1 → 2 → 3 → 4 → 5, n=2
                   ↑                  ↑
                  slow               fast (n칸 앞)

이동 후:   dummy → 1 → 2 → 3 → 4 → 5 → None
                               ↑         ↑
                              slow      fast
→ slow.next(4)를 건너뛰어 slow.next = slow.next.next
```

### 최적화 코드

```python
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        dummy = ListNode(0)
        dummy.next = head
        slow, fast = dummy, dummy

        # fast를 n+1칸 먼저 이동 (slow가 삭제 노드의 이전에 멈추게 하기 위해)
        for _ in range(n + 1):
            fast = fast.next

        # fast가 None이 될 때까지 동시 이동
        while fast:
            slow = slow.next
            fast = fast.next

        # slow는 삭제 대상의 직전 노드
        slow.next = slow.next.next
        return dummy.next
```

---

## 해결 전략 2: Divide and Conquer (분할 정복)

**Merge K Sorted Lists** 같은 Hard 난이도 문제는 한꺼번에 합치려 하면 복잡도가 폭발한다. 이를 **2개씩 짝지어 합치는 토너먼트 방식**으로 해결한다.

### 핵심 공식

- **Base Case**: 리스트가 하나 남을 때까지 반복한다.
- **Merge**: 두 개의 정렬된 리스트를 합치는 로직(`mergeTwoLists`)을 재사용하여 효율성을 극대화한다.

```
Round 1: [L1, L2, L3, L4] → [merge(L1,L2), merge(L3,L4)]
Round 2: [M1, M2]         → [merge(M1,M2)]
Result:  최종 정렬 리스트
```

### 최종 최적화 코드 (Merge K Sorted Lists)

```python
class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        if not lists: return None
        if len(lists) == 1: return lists[0]
        
        # 리스트가 하나가 될 때까지 2개씩 병합 (Divide and Conquer)
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

## 요약 및 회고

### 왜 이 방식들이 효율적인가?

- **Two-Pointer**: 한 번의 순회($O(N)$)로 공간 효율성($O(1)$)까지 챙기며 타겟을 찾아낸다.
- **Divide and Conquer**: $K$개의 리스트를 무작정 합치면 $O(NK)$지만, 분할 정복은 $O(N \log K)$의 성능을 보장한다.

### 핵심 교훈

**"노드의 연결을 바꾸기 전, 미리 가야 할 곳을 찜해두자."**

연결 리스트에서 `next`를 바꾸는 순간 이전의 길은 끊긴다. 따라서 **Dummy Node**로 시작점을 고정하고, **Multiple Pointers**로 필요한 지점들을 미리 확보해두는 습관이 정답으로 가는 지름길임을 깨달았다.
