---
title: "Blind75 - Longest Repeating Character Replacement (슬라이딩 윈도우와 빈도수 체크)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, sliding-window, logic-check]
use_math: true
---

## Longest Repeating Character Replacement (대치 후 가장 긴 반복 문자열)

주어진 문자열 `s`와 정수 `k`가 주어졌을 때, 최대 $k$번만큼 문자를 변경하여 만들 수 있는 동일한 문자로 이루어진 가장 긴 부분 문자열의 길이를 찾는 문제이다.

- "Substring"이므로 연속된 구간이어야 한다.
- $k$번의 기회를 어떻게 사용하느냐에 따라 결과가 달라진다.

---

## 초기 접근 및 시행착오 (Logic Check)

처음에는 포인터를 이동하며 단순히 문자를 더하고 $k$번의 기회를 루프로 돌리려 했으나, 윈도우의 개념과 데이터 저장 방식에서 오류가 발생했다.

### 실패했던 초기 코드

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        n = len(s)
        counter = {}
        max_freq = 0
        left = 0
        answer = 0
        for right in range(n):
            # 1. 값을 업데이트하지 않고 조회만 함
            counter.get(s[right], 0) + 1 
            
            # 2. max_freq에 빈도수가 아닌 '길이'를 저장하는 혼동
            max_freq = max(max_freq, right - left + 1)
            
            while (right - left + 1) - max_freq > k:
                counter[s[left]] -= 1
                left += 1
            
            # 3. 정답 변수가 아닌 max_freq를 다시 갱신함
            max_freq = max(answer, right - left + 1)
        return answer
```

### 논리 오류 분석

- **업데이트 누락**: `counter.get()`은 값을 반환만 할 뿐, 딕셔너리에 대입(`=`)하지 않으면 저장되지 않는다.
- **변수 역할 혼동**: `max_freq`는 윈도우 내 "최다 빈도 문자의 개수"여야 하는데, 이를 "현재 윈도우의 길이"와 혼동하여 `while` 조건문이 무력화되었다.
- **결과 값 미갱신**: 최종적으로 반환할 `answer` 변수를 갱신하지 않아 항상 초기값 `0`이 반환되는 구조였다.

---

## 해결 전략: 슬라이딩 윈도우 (Sliding Window)

구간 내에서 가장 많이 등장하는 문자를 기준으로 잡고, 나머지 문자들을 $k$번 이내로 바꿀 수 있는지 체크하며 윈도우를 확장한다.

### 핵심 공식

$$\text{Window Length} - \text{Max Frequency} \le k$$

**(현재 윈도우 전체 길이) - (가장 많이 나온 문자의 개수)**는 곧 **"바꿔야 할 문자의 개수"**를 의미한다. 이 값이 $k$보다 작거나 같아야 유효한 윈도우다.

---

## 최종 최적화 코드 (Two Pointer)

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        counter = {}
        max_f = 0  # 윈도우 내 최다 빈도수
        left = 0
        answer = 0
        
        for right in range(len(s)):
            # 오른쪽 문자를 윈도우에 추가 및 빈도수 갱신
            counter[s[right]] = counter.get(s[right], 0) + 1
            max_f = max(max_f, counter[s[right]])
            
            # 유효하지 않은 윈도우(바꿔야 할 문자가 k 초과)라면 왼쪽 축소
            while (right - left + 1) - max_f > k:
                counter[s[left]] -= 1
                left += 1
            
            # 최대 길이 갱신
            answer = max(answer, right - left + 1)
            
        return answer
```

---

## 요약 및 회고

### 왜 이 방식이 효율적인가?

- **시간 복잡도**: $O(n)$으로 문자열을 단 한 번 순회하며 답을 찾는다.
- **공간 복잡도**: 알파벳 대문자만 들어오므로 딕셔너리 크기는 최대 26으로 제한되어 $O(1)$에 가깝다.

### 핵심 교훈

**"기준을 정하고 나머지를 맞춘다."** 이 문제에서 기준은 언제나 '가장 많이 등장한 문자'이다. 윈도우 내에서 가장 강한 세력을 파악하고, 약한 세력($k$개 이하)을 흡수하며 영역을 넓혀가는 슬라이딩 윈도우의 전형적인 메커니즘을 배울 수 있었다.
