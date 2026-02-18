---
title: "Blind75 - Group Anagrams (Hash Map & Visited)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, hashmap]
use_math: true
---

## Group Anagrams (아나그램 그룹화)

문자열 배열을 받아 애너그램(문자를 재배열했을 때 같아지는 단어들)끼리 그룹을 짓는 문제이다.  
파이썬의 기초적인 리스트 조작부터 효율적인 해시맵 활용까지 정리해본다.

---

## 첫 번째 접근: Bruteforce와 `visited`의 이해

처음에는 모든 단어를 하나씩 비교하는 방식을 생각했다.  
하지만 리스트를 순회하며 `pop()`을 하면 인덱스가 꼬이는 치명적인 문제가 발생한다.

이를 해결하기 위해 **`visited` 리스트**를 도입했다.

### 핵심 원리

- 원본 리스트를 건드리지 않고  
- "이 단어는 이미 그룹에 포함되었는가?"를 `True / False`로 기록한다.

### 깨달은 점

리스트의 요소를 직접 삭제하는 대신,  
상태를 기록하는 `visited` 배열을 활용하면  
인덱스 오류 없이 안전하게 전체 데이터를 제어할 수 있다.

---

### 수정된 코드 (O(N^2) 버전)

```python
from typing import List

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        answer = []
        n = len(strs)
        visited = [False] * n 
        
        for i in range(n):
            if visited[i]:  # 이미 처리된 단어라면 건너뜀
                continue
            
            trial = [strs[i]]
            visited[i] = True
            
            for j in range(i + 1, n):
                # 아직 방문 안 했고, 정렬 결과가 같다면 아나그램!
                if not visited[j] and sorted(strs[i]) == sorted(strs[j]):
                    trial.append(strs[j])
                    visited[j] = True
                    
            answer.append(trial)
            
        return answer
```

---

## 시간 복잡도의 아쉬움

위 코드는 논리적으로는 완벽하지만, 성능 면에서는 아쉬움이 있다.

### 시간 복잡도

O(N^2 · K log K)

- 외곽 루프: `N`
- 내부 루프: `N`
- 문자열 정렬: `K log K`

데이터가 10,000개만 되어도  
약 1억 번 이상의 연산이 필요하다.

실제 코딩 테스트에서는 **시간 초과(TLE)**가 발생할 가능성이 매우 높다.

---

## 최적화: Hash Map(Dictionary) 활용

비교 연산을 없애고,  
단어를 정렬한 값을 **Key(바구니 이름)** 로 사용하는 방식이다.

### 분류의 미학

Hash Map은 특정 "특성"에 따라 데이터를 즉시 분류하는 최적의 구조이다.

### Pythonic Way

`defaultdict(list)`를 사용하면  
새로운 Key를 만날 때마다 자동으로 빈 리스트를 생성해주어  
코드가 매우 간결해진다.

---

## 최종 최적화 코드 (O(N · K log K))

```python
from collections import defaultdict
from typing import List

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        # 정렬된 문자열을 Key로, 원본 문자열 리스트를 Value로 저장
        groups = defaultdict(list)
        
        for s in strs:
            # "eat", "tea", "ate" -> 모두 "aet"로 정렬됨 (유일한 Key)
            key = "".join(sorted(s))
            groups[key].append(s)
            
        return list(groups.values())
```

---

## 요약 및 회고

### visited 배열의 발견
- 리스트 수정 없이 상태를 추적할 때 매우 유용
- 인덱스 꼬임 방지의 핵심 전략

### Hash Map의 효율성
- "그룹화" 문제에서는 해시맵이 가장 강력한 도구
- 시간 복잡도를 O(N^2) 에서 O(N) 수준으로 개선 가능
- 코딩 테스트에서의 실전 최적화 핵심 패턴

### 결론

**Bruteforce → 상태 관리 → Hash Map 최적화**

이 과정을 통해 단순 구현을 넘어  
"자료구조 선택의 중요성"을 깊이 이해하게 되었다.
