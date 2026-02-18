---
title: "Blind75 - Longest Consecutive Sequence (정렬 기반 접근법)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, sorting, logic-check]
use_math: true
---

## Longest Consecutive Sequence (가장 긴 연속된 요소)

정렬되지 않은 정수 배열 `nums`가 주어졌을 때, 숫자들이 연속적인 구간을 형성하는 가장 긴 길이를 찾는 문제이다.

이 글에서는 초기에 흔히 저지를 수 있는 논리 오류를 짚어보고, 이를 **정렬(Sorting)**과 **중복 제거(Set)**를 통해 어떻게 견고한 로직으로 수정할 수 있는지 정리해본다.

---

## 초기 접근의 논리 오류 분석

처음에는 다음과 같이 "현재 기준값(`lowest`)의 다음 값(`lowest + 1`)이 배열에 있는지" 확인하며 카운팅하는 방식을 떠올릴 수 있다.

### 문제의 코드

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
                lowest = nums[i]  # 핵심 오류 지점
                in_row = 1
        return max(lst)
```

### 왜 오류가 발생하는가?

이 코드에는 두 가지 치명적인 결함이 있다.

- **중복값의 간섭**: `[1, 2, 2, 3]`과 같은 입력에서, 중복된 `2`를 만나는 순간 `lowest`는 이미 `3`인 상태인데 `lowest + 1`인 `4`를 찾지 못해 `else`문으로 빠져 카운트가 리셋된다.
- **순서 보장 부재**: `lowest = nums[i]` 부분에서, 반복문이 도는 원본 `nums`의 인덱스 순서가 "다음으로 작은 숫자"라는 보장이 없다. 따라서 이미 확인한 구간으로 `lowest`가 되돌아가거나 흐름이 끊기게 된다.

---

## 해결 전략: 데이터 정제와 정렬

위의 논리 오류를 해결하기 위해서는 **데이터의 고유성(Uniqueness)**과 **순서(Order)**를 먼저 확보해야 한다.

### 핵심 로직

- `set(nums)`: 중복된 숫자를 제거하여 중복값에 의해 카운트가 리셋되는 현상을 방지한다.
- `sorted()`: 숫자를 오름차순으로 정렬하여, 리스트를 순회할 때 다음 인덱스의 값이 반드시 현재 값보다 크거나 같음을 보장한다.

---

## 최종 구현 (수정된 코드)

질문했던 코드의 원형을 최대한 살리면서 논리적 결함만 보완한 형태이다.

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        # 중복 제거 후 정렬하여 순차 탐색의 기반을 마련한다.
        nums = sorted(list(set(nums)))
        
        lst = []
        in_row = 1
        n = len(nums)
        lowest = nums[0]

        for i in range(n):
            # 정렬된 상태이므로 (lowest + 1) 확인이 유효해진다.
            if (lowest + 1) in nums:
                in_row += 1
                lowest += 1
            else:
                # 연속이 끊기면 현재까지의 길이를 저장하고 리셋
                lst.append(in_row)
                lowest = nums[i]
                in_row = 1
        
        # 마지막으로 계산된 구간의 길이를 추가
        lst.append(in_row)
        
        return max(lst)
```

---

## 요약 및 회고

### 왜 정렬과 중복 제거가 필수인가?

- **일관성**: 정렬을 해야만 `nums[i]`가 "연속이 끊긴 지점 이후의 가장 작은 후보"가 될 수 있다.
- **안정성**: 중복을 제거해야만 동일한 값 때문에 `lowest` 업데이트와 `in_row` 카운팅이 꼬이지 않는다.

### 성능 고려

위 방식은 정렬 때문에 $O(N \log N)$의 시간 복잡도를 가진다. 만약 문제의 제약 조건이 $O(N)$을 요구한다면, 정렬 대신 Hash Set만을 활용하여 각 숫자가 연속 구간의 시작점인지를 체크하는 최적화 기법을 적용할 수 있다.
