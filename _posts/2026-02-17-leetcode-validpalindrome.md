---
title: "Blind75 - Valid Palindrome (로직 교정과 투 포인터)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, twopointer, logic-check]
use_math: true
---

## Valid Palindrome (팰린드롬 검증)

주어진 문자열이 **팰린드롬(Palindrome, 앞뒤가 똑같은 문장)**인지 확인하는 문제이다.

- 대소문자를 구분하지 않는다.
- 영문자와 숫자(Alphanumeric)를 제외한 모든 특수문자는 무시한다.
- 빈 문자열은 팰린드롬으로 간주한다.

---

## 초기 접근 및 시행착오 (Logic Check)

처음에는 단순히 공백만 제거하고 인덱스로 비교하는 방식을 시도했으나, 몇 가지 치명적인 결함이 발견되었다.

### 실패했던 초기 코드

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        s = "".join(s.split())  # 오류 1: 특수문자 처리가 안 됨
        n = len(s)
        for i in range(n):
            if s[i] == s[n-1-i]:
                continue
            else:
                return False
            if i == n/2:  # 오류 2: float 비교 문제 및 루프 종료 위치 부적절
                return True
```

### 논리 오류 분석

- **특수문자 처리 부재**: `s.split()`은 공백만 제거한다. 문제 조건인 쉼표(`,`), 마침표(`.`) 등은 여전히 남아서 비교를 방해한다.
- **잘못된 종료 조건**: `i == n/2`는 정수와 실수를 비교하는 오류가 있으며, 사실 루프가 끝난 뒤에 `True`를 반환하는 것이 가장 안전하다.
- **불필요한 연산**: 팰린드롬은 문자열의 절반만 확인하면 검증이 끝난다.

---

## 해결 전략: 데이터 정제와 비교 최적화

### 필터링과 슬라이싱 (Pythonic)

파이썬의 `isalnum()`과 슬라이싱(`[::-1]`)을 활용하면 매우 짧고 강력하게 풀 수 있다.

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        # 알파벳과 숫자만 남기고 소문자로 통일
        filtered_s = "".join(char.lower() for char in s if char.isalnum())
        # 뒤집은 문자열과 비교
        return filtered_s == filtered_s[::-1]
```

---

## 심화: 메모리 최적화를 위한 투 포인터(Two Pointer)

위의 슬라이싱 방식은 이해하기 쉽지만, `filtered_s`라는 새로운 문자열 공간을 생성해야 하므로 데이터가 매우 클 경우 메모리 부담이 생길 수 있다. 이를 방지하기 위해 **공간 복잡도 $O(1)$**의 투 포인터 방식을 사용한다.

### 최종 최적화 코드

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        left, right = 0, len(s) - 1
        
        while left < right:
            # 왼쪽이 특수문자면 한 칸 오른쪽으로
            if not s[left].isalnum():
                left += 1
            # 오른쪽이 특수문자면 한 칸 왼쪽으로
            elif not s[right].isalnum():
                right -= 1
            # 둘 다 글자일 때 비교 시작
            else:
                if s[left].lower() != s[right].lower():
                    return False
                left += 1
                right -= 1
                
        return True
```

---

## 요약 및 회고

### 왜 투 포인터가 좋은가?

- **공간 효율성**: 새로운 문자열을 만들지 않고 인덱스만 조작하므로 메모리 사용량이 극히 적다.
- **현업에서의 가치**: 대용량 데이터나 임베디드 환경에서는 데이터를 '가공(Copy)'하는 것보다 원본을 '탐색(Search)'하는 방식이 훨씬 선호된다.

### 핵심 교훈

**데이터를 가공할 것인가, 탐색할 것인가?** 이 질문은 메모리 효율성을 결정짓는 중요한 기준이다. 코딩 테스트에서는 로직의 정교함뿐만 아니라, 공간 복잡도에 대한 고민이 실력을 가른다.
