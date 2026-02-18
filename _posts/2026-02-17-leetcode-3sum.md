---
title: "Blind75 - 3Sum (중복 처리의 늪과 투 포인터의 효율성)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, twopointer, logic-check]
use_math: true
---

## 3Sum (세 수의 합)

배열 `nums`가 주어졌을 때, 합이 0이 되는 세 개의 요소를 찾는 문제이다.

- 중복된 조합은 결과 리스트에 포함되지 않아야 한다.
- 정답의 순서는 상관없다.
- $O(N^2)$ 내외의 효율적인 처리가 요구된다.

---

## 초기 접근 및 시행착오 (Logic Check)

처음에는 중복을 제거하기 위해 `set()`을 사용하거나, 사용한 인덱스를 기록하는 방식을 시도했으나 논리적 결함과 성능 문제가 발생했다.

### 실패했던 초기 코드 분석

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums = list(set(nums))  # 오류 1: [0,0,0] 같은 중복 값 구성 불가
        answer = []
        n = len(nums)
        used = [False] * n

        for i in range(n):
            if used[i]: continue
            for j in range(i+1, n):
                if used[j]: continue
                twoSum = nums[i] + nums[j]
                if (-twoSum) in nums:
                    used[i], used[j] = True, True  # 오류 2: 한 숫자는 여러 조합에 쓰일 수 있음
                    answer.append([nums[i], nums[j], -twoSum])
        return answer
```

### 논리 오류 및 성능 분석

- **데이터 손실**: `set(nums)`를 수행하면 `[-1, -1, 2]`처럼 같은 숫자가 두 번 쓰여야 하는 케이스를 풀 수 없다.
- **포인터 고립**: `used[i] = True`로 숫자를 잠가버리면, 해당 숫자가 포함된 다른 유효한 조합을 찾지 못한다.
- **시간 복잡도**: `for`문 중첩 내에서 `in` 연산을 수행하여 $O(N^3)$에 가까운 성능을 보이며, 대규모 데이터에서 Time Limit Exceeded가 발생한다.

---

## 해결 전략: 정렬과 투 포인터(Two Pointer)

가장 효율적인 방법은 배열을 정렬한 후, 하나의 기준점을 잡고 나머지 두 지점을 양 끝에서 좁혀오는 투 포인터 전략을 사용하는 것이다.

### 중복 방지의 핵심 로직: `i > 0 and nums[i] == nums[i-1]`

이 조건은 중복된 결과값을 피하는 가장 우아한 방법이다.

- **첫 번째 기회 부여**: `i=0`일 때는 `i > 0`이 `False`이므로 `if`문을 통과하여 연산을 수행한다. (단락 평가: Short-circuit evaluation)
- **이후 중복 차단**: `i=1`부터는 이전 값(`nums[i-1]`)과 현재 값이 같다면 이미 앞선 루프에서 처리된 조합이므로 `continue`를 통해 건너뛴다.
- **[0, 0, 0] 케이스 해결**: 이 로직을 통해 첫 번째 `0`은 기준으로 삼고, 나머지 두 `0`은 포인터(`left`, `right`)가 찾아내어 정확히 `[[0, 0, 0]]`을 반환한다.

---

## 최종 최적화 코드

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums.sort()
        answer = []
        n = len(nums)

        for i in range(n - 2):
            # 중복된 기준점 건너뛰기 (핵심 교훈 포인트)
            if i > 0 and nums[i] == nums[i - 1]:
                continue
            
            left, right = i + 1, n - 1
            while left < right:
                total = nums[i] + nums[left] + nums[right]
                
                if total < 0:
                    left += 1
                elif total > 0:
                    right -= 1
                else:
                    answer.append([nums[i], nums[left], nums[right]])
                    # left와 right의 중복 요소 건너뛰기
                    while left < right and nums[left] == nums[left + 1]:
                        left += 1
                    while left < right and nums[right] == nums[right - 1]:
                        right -= 1
                    left += 1
                    right -= 1
        return answer
```

---

## 요약 및 회고

### 왜 브루트 포스($O(N^3)$)는 나쁜가?

- **공간과 시간의 트레이드오프**: 단순히 `set()`에 결과를 담는 방식은 구현은 쉬우나, 불필요한 연산과 메모리(Set 저장 공간)를 낭비한다.
- **알고리즘의 정교함**: 투 포인터 방식은 공간 복잡도를 $O(1)$(결과 배열 제외)로 유지하면서 시간 복잡도를 $O(N^2)$으로 최적화한다.

### 핵심 교훈

**"첫 번째 기회는 주되, 두 번째부터는 검문한다."**

중복을 제거할 때 무작정 데이터를 삭제(`set`)하기보다, 정렬된 상태에서 이전 인덱스와 비교하며 흐름을 제어하는 방식이 훨씬 안전하고 효율적이라는 것을 배웠다. 또한 파이썬의 `and` 연산이 가진 특성을 이용해 인덱스 에러를 방지하고 로직을 간결하게 만드는 법을 익혔다.
