---
title: "Blind75 - Container with Most Water (투 포인터 접근법)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, two-pointer, optimization]
use_math: true
---

## Container with Most Water (가장 많은 물을 담을 수 있는 용기)

여러 개의 수직선(기둥)이 주어졌을 때, 두 기둥을 선택하여 그 사이의 공간에 물을 채운다고 가정한다. 이때 물을 가장 많이 담을 수 있는 두 기둥 사이의 최대 넓이를 구하는 문제이다.

이 글에서는 흔히 사용하는 **이중 반복문(Brute-force)**의 한계를 짚어보고, 이를 **투 포인터(Two Pointers)**를 통해 어떻게 $O(N)$의 효율적인 로직으로 개선할 수 있는지 정리해본다.

---

## 초기 접근의 효율성 분석

처음에는 모든 기둥의 쌍을 일일이 비교하여 넓이를 계산하는 방식을 떠올릴 수 있다.

### 문제의 코드 (Brute-force)

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        n = len(heights)
        areas = []
        for i in range(n):
            for j in range(i+1, n):
                # 낮은 기둥의 높이가 물의 높이가 됨
                if heights[i] < heights[j]:
                    area = (j-i) * heights[i]
                else: 
                    area = (j-i) * heights[j]
                areas.append(area)
        return sorted(areas)[-1]
```

### 왜 개선이 필요한가?

이 코드에는 두 가지 아쉬운 점이 있다.

- **시간 복잡도**: 이중 `for`문을 사용하므로 $O(N^2)$의 시간이 걸린다. 기둥이 10만 개라면 연산 횟수가 100억 번에 달해 시간 초과(Time Limit Exceeded)가 발생한다.
- **메모리 낭비**: 모든 넓이 값을 `areas` 리스트에 저장한 뒤 정렬한다. 굳이 모든 값을 가질 필요 없이 실시간으로 최댓값만 갱신하면 메모리를 아낄 수 있다.

---

## 해결 전략: 양 끝에서 좁혀오는 투 포인터

넓이는 가로(너비) * 세로(높이)로 결정된다. 양 끝에서 시작하여 너비를 최대에서 최소로 줄여나가며 최적의 높이를 찾는 전략을 사용한다.

### 핵심 로직

- **너비는 무조건 줄어든다**: 포인터를 안쪽으로 옮길 때마다 가로 길이는 1씩 감소한다.
- **높이를 보존하라**: 전체 넓이는 둘 중 더 낮은 기둥에 의해 결정된다. 따라서 더 높은 기둥을 찾아야 넓이가 커질 가능성이 생긴다.
- **낮은 쪽을 이동**: `heights[left]`와 `heights[right]` 중 더 낮은 값을 가진 포인터를 안쪽으로 이동시킨다.

---

## 최종 구현 (수정된 코드)

리스트를 생성하지 않고 변수 하나(`maxA`)에 최댓값을 실시간으로 업데이트하여 효율성을 극대화했다.

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        n = len(heights)
        left, right = 0, n-1
        maxA = 0
        
        while left < right:
            # 현재 두 포인터 사이의 높이와 너비 계산
            h = min(heights[left], heights[right])
            w = right - left
            
            # 실시간 최댓값 갱신 (리스트 정렬 대신 max 함수 활용)
            maxA = max(h * w, maxA)

            # 핵심: 더 낮은 기둥 쪽의 포인터를 이동시켜 더 높은 기둥을 탐색
            if heights[left] < heights[right]:
                left += 1
            else:
                right -= 1
                
        return maxA
```

---

## 요약 및 회고

### 왜 투 포인터가 최적의 해를 보장하는가?

- **탐색 범위의 축소**: 낮은 기둥을 고정하고 높은 기둥을 안으로 옮겨봤자, 너비는 줄어들고 물의 높이는 여전히 낮은 기둥에 갇히기 때문에 넓이는 무조건 감소한다. 따라서 낮은 쪽을 옮기는 것 외에는 넓이가 증가할 시나리오가 없다.
- **최대값 추적**: `max()` 함수를 따로 빼서 관리함으로써 불필요한 리스트 저장과 정렬 과정을 생략할 수 있었다.

### 성능 고려

이 방식은 배열을 단 한 번만 훑기 때문에 **$O(N)$**의 시간 복잡도를 가진다. $O(N^2)$에서 $O(N)$으로의 전환은 대규모 데이터 처리에서 엄청난 성능 차이를 만든다.
