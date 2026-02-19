---
title: "Blind75 - Valid Parentheses (스택과 문자열 소거법)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, stack, logic-check]
use_math: true
---

## Valid Parentheses (유효한 괄호)

주어진 문자열 `s`가 `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`로 이루어져 있을 때, 괄호가 올바른 순서로 닫히는지 확인하는 문제이다.

- 열린 괄호는 반드시 같은 타입의 닫힌 괄호로 닫혀야 한다.
- 열린 괄호는 반드시 나중에 열린 순서대로 먼저 닫혀야 한다 (LIFO 구조).

---

## 초기 접근 및 시행착오 (Logic Check)

처음에는 괄호의 쌍이 대칭(Mirror) 구조를 이룰 것이라 생각하여 인덱스를 계산해 비교하는 방식을 시도했다.

### 실패했던 초기 코드

```python
class Solution:
    def isValid(self, s: str) -> bool:
        char_valid = ['(', ')', '{', '}', '[', ']']
        for i, char in enumerate(s):
            if char == char_valid[0]:  # '(' 인 경우
                # 닫는 괄호가 문자열의 '대칭' 위치에 있을 것이라 가정함
                if s.find(')') != (len(s) - i): 
                    return False
            # ... 다른 괄호들도 동일한 논리 적용
        return True
```

### 논리 오류 분석

- **대칭 구조의 오해**: `()[]{}` 같은 형태는 유효하지만 대칭이 아니다. 위 코드는 `(( ))` 처럼 완벽히 거울처럼 배치된 경우만 고려했다.
- **`.find()`의 한계**: `find()`는 항상 문자열의 첫 번째 인덱스만 반환한다. 중복된 괄호가 나올 경우 정확한 짝을 찾지 못한다.
- **순서 무시**: `)(` 처럼 닫는 괄호가 먼저 나오는 경우를 잡아낼 수 없다.

---

## 해결 전략 1: 문자열 소거법 (Intuitive Approach)

"가장 안쪽의 짝부터 지워나간다"는 직관적인 아이디어다. 인덱스 계산 대신, 붙어 있는 쌍(`()`, `[]`, `{}`)을 빈 문자열로 치환하며 크기를 줄인다.

### 핵심 로직

- 문자열 안에 `()`, `[]`, `{}` 중 하나라도 있다면 계속해서 `replace`를 수행한다.
- 모든 처리가 끝난 후 빈 문자열(`""`)이 남으면 성공, 찌꺼기가 남으면 실패다.

---

## 해결 전략 2: 스택 (Stack - 최적화)

괄호의 "열린 순서"와 "닫히는 순서"가 반대라는 점(Last-In, First-Out)을 이용하여 스택을 활용한다.

### 핵심 공식

- **열린 괄호**: 스택에 담아둔다 (Push).
- **닫힌 괄호**: 스택의 가장 위(Top)에 있는 괄호와 짝이 맞는지 확인한다 (Pop).

---

## 최종 최적화 코드

```python
class Solution:
    def isValid(self, s: str) -> bool:
        # 닫는 괄호를 키로, 여는 괄호를 값으로 하는 딕셔너리
        lookup = {')': '(', '}': '{', ']': '['}
        stack = []
        
        for char in s:
            if char in lookup:  # 닫는 괄호를 만났을 때
                # 스택이 비어있지 않다면 pop, 비어있다면 가짜값 '#' 대입
                top_element = stack.pop() if stack else '#'
                if lookup[char] != top_element:
                    return False
            else:  # 여는 괄호인 경우
                stack.append(char)
        
        # 모든 순회가 끝난 후 스택이 비어있어야 True
        return not stack
```

---

## 요약 및 회고

### 왜 스택 방식이 효율적인가?

- **시간 복잡도**: $O(n)$ — 문자열을 한 번만 훑으면 된다. (문자열 소거법은 $O(n^2)$까지 늘어날 수 있음)
- **정확성**: `{[(}])`와 같이 괄호가 꼬여 있는 (Interlocking) 케이스를 즉시 잡아낼 수 있다.

### 핵심 교훈

**"가장 최근에 발생한 사건을 가장 먼저 처리해야 할 때"**는 **스택(Stack)**이 정답이다. 괄호 문제는 겉보기엔 인덱스 게임 같지만, 본질은 데이터의 계층(Nesting) 구조를 파악하는 문제임을 배웠다.
