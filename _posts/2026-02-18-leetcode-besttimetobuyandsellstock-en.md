---
title: "Blind75 - Best Time to Buy and Sell Stock (Logic Correction & Sliding Window)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, array, sliding-window, twopointer, logic-check]
use_math: true
---

## Best Time to Buy and Sell Stock

Find the maximum profit achievable from a single transaction (buy once and sell once).

- You can only sell on a day after you buy (time flows forward).
- If no profit is possible, return 0.

---

## Initial Approach and Trial-and-Error (Logic Check)

The first idea was to simply find the largest and smallest values in the entire array and compute their difference. However, this ignores the fundamental rule of the stock market: **"time order."**

### The Failing Initial Code

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        maxP = 0
        minP = prices[0]
        n = len(prices)

        for i in range(n-1):
            if prices[i] > prices[i+1]:
                maxP = max(maxP, prices[i])
            # Logic error: maxP and minP are updated independently
            # This allows 'buy point' to come after 'sell point'
            
        return maxP - minP
```

### Logic Error Analysis

- **Ignoring time order**: The buy point (low) must always appear before the sell point (high). The code above risks determining `maxP` and `minP` regardless of index order.
- **Definition of max profit**: What we're looking for is not simply the 'maximum value', but the maximum of $Price(Sell) - Price(Buy)$.
- **Counterexample**: For `prices = [10, 1]`, the profit should be 0, but the logic above can produce an incorrect calculation.

---

## Solution Strategy: Maintaining the Minimum and Computing Differences

### Greedy Approach

Traverse the array once ($O(n)$), continuously updating **"the lowest price seen so far"** and calculating **"today's profit if sold"** every day.

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = float('inf')  # Initialize to a very large value
        max_profit = 0
        
        for price in prices:
            # Update the lowest price so far
            if price < min_price:
                min_price = price
            # Check if selling today yields a record profit
            elif price - min_price > max_profit:
                max_profit = price - min_price
                
        return max_profit
```

---

## Advanced: Variable Sliding Window

This problem can also be interpreted through a sliding window lens using two pointers. The window's start is the 'buy day' and the end is the 'sell day'.

### Two Pointer Optimized Code

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        l, r = 0, 1  # l: buy day, r: sell day
        max_profit = 0
        
        while r < len(prices):
            # Profitable structure (low point discovered)
            if prices[l] < prices[r]:
                profit = prices[r] - prices[l]
                max_profit = max(max_profit, profit)
            # Today's price is cheaper? Slide the buy point to today!
            else:
                l = r
            r += 1  # Sell day advances one step every day
            
        return max_profit
```

---

## Summary & Reflection

### Why Sliding Window?

- **Meaning of pointer movement**: The right pointer (`r`) advances daily to explore the market, and the moment it encounters a lower price, the left pointer (`l`) jumps to that position.
- **Efficiency**: This approach eliminates unnecessary nested loops ($O(n^2)$) and solves the problem with a single pass ($O(n)$).

### Key Lesson

The key is understanding **"how does past data influence present decisions?"** In the stock problem, the minimum price serves as the left boundary of the window, and the core of the algorithm lies in the boldness to slide the window when a better condition is found.
