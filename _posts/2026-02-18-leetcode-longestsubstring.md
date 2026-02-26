---
title: "Blind75 - Longest Substring Without Repeating Characters (문자열 슬라이싱과 슬라이딩 윈도우)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, sliding-window, logic-check]
use_math: true
---

## Longest Substring Without Repeating Characters (중복 없는 가장 긴 부분 문자열)

주어진 문자열에서 중복된 문자가 없는 가장 긴 부분 문자열(Substring)의 길이를 찾는 문제이다.

- "Substring"이므로 반드시 연속된 문자열이어야 한다.
- 대소문자 및 공백, 특수문자를 모두 포함할 수 있다.

---

## 초기 접근 및 시행착오 (Logic Check)

처음에는 한 글자씩 더해가며 중복을 체크하고, 중복 시 건너뛰는 방식을 시도했으나 논리적 결함이 발견되었다.

### 실패했던 초기 코드

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        answer = ''
        for i, letter in enumerate(s):
            if not answer: continue  # 첫 글자부터 막힘
            if letter == s[len(s)-1]:  # 마지막 글자와 비교하는 엉뚱한 로직
                answer = ''
            if letter in answer:
                continue  # 중복 시 그냥 건너뜀 (연속성 파괴)
            answer += letter
        return len(answer)
```

### 논리 오류 분석

- **첫 글자 무시**: `if not answer: continue` 조건 때문에 첫 번째 루프에서 아무 일도 일어나지 않는다.
- **중복 처리의 오해**: 중복이 발생했을 때 해당 글자를 그냥 `continue`로 넘기면, 문제에서 요구하는 '연속된 부분 문자열'의 흐름이 끊기게 된다.
- **최대 길이 갱신 누락**: 루프가 끝난 시점의 `answer` 길이만 반환하면, 중간에 가장 길었던 구간을 놓치게 된다.

---

## 해결 전략: 문자열 슬라이싱 (Intuitive Approach)

기존의 `answer`를 유지하면서, 중복이 발생했을 때 중복된 글자 이전의 과거를 과감히 버리는 방식이다.

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        answer = ""
        max_len = 0
        
        for letter in s:
            if letter in answer:
                # 중복 글자의 위치를 찾아 그 다음부터만 살림
                index = answer.find(letter)
                answer = answer[index + 1:]
            
            answer += letter
            # 매 루프마다 최대 길이 갱신
            max_len = max(max_len, len(answer))
                
        return max_len
```

---

## 심화: 최적화를 위한 슬라이딩 윈도우 (Sliding Window)

문자열 슬라이싱은 매번 새로운 문자열 객체를 생성하므로, 대규모 데이터에서는 두 개의 포인터와 **Set(집합)**을 사용하는 것이 더 효율적이다.

### 최종 최적화 코드 (Two Pointer)

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        char_set = set()
        left = 0
        max_len = 0
        
        for right in range(len(s)):
            # 새로 들어올 글자가 중복이라면, 빠져나갈 때까지 왼쪽 포인터 이동
            while s[right] in char_set:
                char_set.remove(s[left])
                left += 1
            
            char_set.add(s[right])
            # 윈도우의 크기 (right - left + 1) 측정
            max_len = max(max_len, right - left + 1)
            
        return max_len
```

---

## 요약 및 회고

### 왜 슬라이딩 윈도우인가?

- **효율성**: 전체 문자열을 딱 한 번 훑는 $O(n)$ 시간에 해결 가능하다.
- **유연성**: 중복이 발견될 때 윈도우의 크기를 유동적으로 조절하며 최적의 범위를 유지할 수 있다.

### 핵심 교훈

**"중복이 발생했을 때 어디서부터 다시 시작할 것인가?"** 이 질문이 슬라이딩 윈도우의 시작이다. 단순히 버리는 것이 아니라, 유효한 구간을 살리면서 범위를 좁혀가는 감각을 익히는 것이 중요하다.
