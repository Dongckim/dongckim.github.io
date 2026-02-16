---
title: "Python Coding Test - Top K Frequent Elements (Heap & Bucket Sort)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, hashmap, heap, bucketsort]
use_math: true
---

## Top K Frequent Elements (상위 K 빈도 요소)

주어진 정수 배열 `nums`에서 빈도수가 가장 높은 상위 `k`개의 요소를 추출하는 문제이다.  
단순한 반복문 접근부터 $O(N)$ 시간 복잡도를 달성하는 최적화 기법까지 정리해본다.

---

## 첫 번째 접근: 반복문과 조건문의 한계

처음에는 `Counter`로 빈도를 계산한 뒤, 반복문을 돌며 특정 조건(`n > k`)을 만족하는 값을 찾으려 했다.

### 시행착오 코드 (Logic Error)

```python
def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    answer = []
    count = Counter(nums)
    for i, n in count:  # 튜플 언패킹 오류 발생
        if n > k:  # '상위 k등'이 아닌 '빈도수 k 초과'를 체크하는 논리 오류
            answer.append(count[i])
    return count
```

### 깨달은 점

- `Counter` 객체를 순회할 때는 `.items()`를 사용하여 (숫자, 빈도) 쌍을 가져와야 한다.
- "상위 $k$개"를 구하기 위해서는 단순 비교가 아니라 **정렬(Sorting)**이나 **우선순위 큐(Heap)**가 필요하다.

---

## 두 번째 접근: 정렬을 이용한 풀이 (O(N log N))

빈도수를 기준으로 전체 데이터를 내림차순 정렬한 뒤 상위 $k$개를 슬라이싱하는 방식이다.

### 핵심 원리

- `Counter(nums).items()`를 통해 빈도 데이터를 추출한다.
- `sorted()` 함수의 `key` 인자에 빈도수를 지정하여 정렬한다.

```python
from collections import Counter

def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    # 빈도수(x[1])를 기준으로 내림차순 정렬
    sorted_counts = sorted(count.items(), key=lambda x: x[1], reverse=True)
    
    # 상위 k개의 '숫자'만 리스트 컴프리헨션으로 추출
    return [x[0] for x in sorted_counts[:k]]
```

---

## 세 번째 접근: Heap을 이용한 최적화 (O(N log k))

모든 요소를 정렬하는 대신, 크기가 $k$인 힙을 유지하며 최상위 요소만 골라내는 방식이다.  
데이터가 방대하고 $k$가 상대적으로 작을 때 효율적이다.

### Pythonic Way (heapq)

파이썬의 `heapq.nlargest`를 사용하면 내부적으로 힙 알고리즘을 수행하여  
깔끔하게 한 줄로 처리할 수 있다.

```python
import heapq
from collections import Counter

def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    # 빈도수가 높은 순서대로 k개 추출
    return heapq.nlargest(k, count.keys(), key=count.get)
```

---

## 최종 최적화: Bucket Sort (O(N))

정렬 알고리즘의 한계인 $O(N \log N)$을 넘어,  
빈도수 자체를 인덱스로 활용하여 $O(N)$ 만에 해결하는 방식이다.

### 분류의 미학

- 인덱스가 '빈도수'가 되고, 해당 인덱스의 값은 '그 빈도를 가진 숫자들의 리스트'가 된다.
- 가장 큰 인덱스(최대 빈도)부터 역순으로 스캔하며 $k$개를 채운다.

### 최종 최적화 코드

```python
from collections import Counter

def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    # 빈도수를 인덱스로 사용하는 버킷 생성 (최대 빈도는 len(nums))
    bucket = [[] for _ in range(len(nums) + 1)]
    
    for num, freq in count.items():
        bucket[freq].append(num)
        
    answer = []
    # 높은 빈도(뒤쪽 인덱스)부터 확인하며 answer 채우기
    for i in range(len(bucket) - 1, 0, -1):
        for n in bucket[i]:
            answer.append(n)
            if len(answer) == k:
                return answer
```

---

## 요약 및 회고

### 데이터 구조의 선택
- **정렬**($O(N \log N)$): 구현이 가장 직관적이지만 성능이 평이함.
- **힙**($O(N \log k)$): 메모리 효율이 좋으며 상위 $k$개를 유지하는 데 최적화됨.
- **버킷 정렬**($O(N)$): 빈도수의 범위가 정해져 있을 때(배열 길이 내) 이론상 최속의 성능을 냄.

### 결론

단순 반복문으로 시작했던 고민이 Bucket Sort라는 강력한 알고리즘으로 이어졌다.  
"비교를 통한 정렬"이 아닌 "인덱스를 이용한 배치"가  
성능을 얼마나 비약적으로 상승시키는지 이해하게 된 유익한 과정이었다.
