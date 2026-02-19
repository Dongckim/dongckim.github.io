---
title: "Blind75 - Best Time to Buy and Sell Stock (로직 교정과 슬라이딩 윈도우)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, sliding-window, twopointer, logic-check]
use_math: true
---

## Best Time to Buy and Sell Stock (주식 최대 수익 찾기)

한 번의 거래(한 번 사고 한 번 팔기)로 낼 수 있는 최대 이익을 산출하는 문제이다.

- 주식을 산 날 이후에만 팔 수 있다 (시간의 순방향성).
- 수익을 낼 수 없는 경우에는 0을 반환한다.

---

## 초기 접근 및 시행착오 (Logic Check)

처음에는 배열 전체에서 단순히 가장 큰 값과 가장 작은 값을 찾아 그 차이를 구하려 했으나, 이는 주식 시장의 대원칙인 **"시간 순서"**를 무시하는 결과를 초래했다.

### 실패했던 초기 코드

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        maxP = 0
        minP = prices[0]
        n = len(prices)

        for i in range(n-1):
            if prices[i] > prices[i+1]:
                maxP = max(maxP, prices[i])
            # 논리 오류: maxP와 minP가 독립적으로 업데이트됨
            # 이로 인해 '산 시점'이 '판 시점'보다 뒤에 오는 상황 발생 가능
            
        return maxP - minP
```

### 논리 오류 분석

- **시간 순서 무시**: 주식은 반드시 저점(Buy)이 고점(Sell)보다 먼저 나타나야 한다. 위 코드는 `maxP`와 `minP`가 인덱스 순서와 상관없이 결정될 위험이 있다.
- **최대 수익의 정의**: 우리가 찾는 것은 단순한 '최댓값'이 아니라, $Price(Sell) - Price(Buy)$의 최댓값이다.
- **반례**: `prices = [10, 1]`인 경우, 수익은 0이어야 하지만 위 로직은 잘못된 계산을 수행할 수 있다.

---

## 해결 전략: 최저점 유지와 차이 계산

### 그리디(Greedy) 방식의 접근

배열을 한 번 순회($O(n)$)하면서, **"지금까지 본 최저가"**를 계속 갱신하고, **"오늘 팔았을 때의 수익"**을 매일 계산한다.

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = float('inf')  # 매우 큰 값으로 초기화
        max_profit = 0
        
        for price in prices:
            # 현재까지의 최저가 갱신
            if price < min_price:
                min_price = price
            # 오늘 팔았을 때의 수익이 역대급인지 확인
            elif price - min_price > max_profit:
                max_profit = price - min_price
                
        return max_profit
```

---

## 심화: 가변 슬라이딩 윈도우(Sliding Window)

이 문제는 투 포인터를 활용한 슬라이딩 윈도우 관점에서도 해석될 수 있다. 윈도우의 시작점은 '구매일', 끝점은 '판매일'이 된다.

### 투 포인터 최적화 코드

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        l, r = 0, 1  # l: 구매일, r: 판매일
        max_profit = 0
        
        while r < len(prices):
            # 수익이 나는 구조 (저점 발견 상태)
            if prices[l] < prices[r]:
                profit = prices[r] - prices[l]
                max_profit = max(max_profit, profit)
            # 오늘 가격이 더 싸다면? 구매 시점을 오늘로 '슬라이딩'!
            else:
                l = r
            r += 1  # 판매일은 매일 한 칸씩 전진
            
        return max_profit
```

---

## 요약 및 회고

### 왜 슬라이딩 윈도우인가?

- **포인터 이동의 의미**: 오른쪽 포인터(`r`)는 매일 전진하며 시장을 탐색하고, 더 낮은 가격(`r`)을 만나는 순간 왼쪽 포인터(`l`)를 그 위치로 점프시킨다.
- **효율성**: 이 방식은 불필요한 이중 반복문($O(n^2)$)을 제거하고 단 한 번의 탐색($O(n)$)으로 문제를 해결하게 해준다.

### 핵심 교훈

**"과거의 데이터가 현재의 결정에 어떻게 영향을 주는가?"**를 파악하는 것이 중요하다. 주식 문제에서 최저가는 윈도우의 왼쪽 경계 역할을 하며, 더 나은 조건을 만났을 때 윈도우를 과감히 옮기는(Sliding) 판단이 알고리즘의 핵심이다.
