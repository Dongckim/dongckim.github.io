---
title: "Dynamic Programming"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [Dynamic Programming]
use_math: true
---

## Dynamic Programming

- 중복 계산을 피하기 위해 중간 결과를 저장해두는 문제 해결 기법
- ‘모든 것을 다 하지 않는다’ → 효율적인 선택
- Memoization: 하위 문제의 결과를 저장
- Optimal Substructure: 전체 최적해에 포함된 부분 문제도 최적이라는 성질 (항상 성립하진 않음)

### 1. Divide and conquer
• Break down large problem into smaller subproblems
• Solve the subproblems to combine the results

### 2. Memoization (not memorization!)
• Storing intermediary results into memory – results from subproblems
• When forming the larger results, simply look-up the memory

## 예시
### 1. 피보나치 수열
재귀 방식은 비효율적 메모이제이션(또는 반복문)으로 중복 계산 방지 → 시간복잡도 감소
![]({{site.url}}/images/2025-06-06-dynamic-programming/fibo.png){: .align-center}

![]({{site.url}}/images/2025-06-06-dynamic-programming/fibo-tree.png){: .align-center}

![]({{site.url}}/images/2025-06-06-dynamic-programming/fibo-iter.png){: .align-center}

- Recall that fib(n) = fib(n – 1) + fib(n – 2)
    - fib(n-1) and fib(n-2) have already been computed numerous times! → Memoization
- Record the intermediary results and re-use later → No need to re-do everything

#### 1-1. 시간복잡도 (Time Complexity)
O(n)
0부터 n까지 한 번씩 계산하기 때문에 반복문은 총 n-1번 수행됨

각 반복은 O(1)이므로 전체 시간복잡도는 O(n)

#### 1-2. 공간복잡도 (Space Complexity)
기본 방식: O(n)
결과 저장을 위해 n+1 크기의 배열 dp[] 사용

### 2. 최단 경로 문제
- Dijkstra’s Algorithm (단일 출발점) 
- Floyd-Warshall Algorithm (모든 쌍 간 경로)

#### 기본 개념
- 기본 아이디어 1: Optimal Substructure     
    > **"최단 경로의 일부 경로도 역시 최단 경로다."**

    예: A → B → C가 최단 경로라면, A → B도 최단 경로여야 함.   
    이는 DP에서 필수적인 조건 중 하나 → Subpath(부분 경로) 가 최적이라는 것을 믿고, 이를 기반으로 더 큰 문제를 해결할 수 있음

- 기본 아이디어 2: Detour Might Be Better    
    > **"우회 경로가 더 짧으면 우회하라"**    

    (≒ 역삼각 부등식, reverse triangle inequality)
![]({{site.url}}/images/2025-06-06-dynamic-programming/detour.png){: .align-center}

## Matrix Chain Multiplication

행렬 곱샘 최적화! → 여러 개의 행렬을 곱할 때, 곱셈 연산 횟수를 최소화하는 곱셈 순서를 찾는 것.
- 행렬 곱셈은 결합법칙만 성립, 교환법칙은 X
- 행렬 A: (n × m), B: (m × k) 라면
    - 곱하는 데 드는 연산량은 n × m × k

### 테이블
- 행렬이 N개 있으면, N×N 크기의 2차원 테이블 m[i][j] 생성
    - m[i][j]: Ai ~ Aj를 곱하는 데 드는 최소 연산 횟수
    - i < j인 경우만 채운다 → 상삼각형만 사용

#### why 상삼각형?

- 행렬 곱셈 순서 문제의 핵심 조건
우리는 주어진 순서대로 행렬을 곱해야만 함. 즉,행렬 A₁, A₂, A₃, ..., Aₙ 이 있을 때 A₃ × A₂ × A₁ 같이 역순으로 곱하면 안 됨.
    - 행렬 곱셈은 결합법칙은 있지만, 교환법칙이 x.(AB)C = A(BC) 가능 AB ≠ BA ❌ 순서 바꾸면 안 됨.

### 점화식 이해 (슬라이드 4)
우리는 m(i, j) (Ai~Aj 곱하는 최소 연산량)를 구하고 싶음.

![]({{site.url}}/images/2025-06-06-dynamic-programming/matrix.png){: .align-center}

Ai × Ai+1 × … × Ak × Ak+1 × … × Aj

중간에 쪼개는 지점 k를 정한다면:
- m(i, k): Ai~Ak까지 곱하는 최소 비용
- m(k+1, j): Ak+1~Aj까지 곱하는 최소 비용
- row(Ai) * col(Ak) * col(Aj): 마지막으로 두 결과를 곱하는 비용

#### 최적 분할 위치 K를 어떻게 구할까?

모든 가능한 k를 다 시도해보고, 그 중에서 곱셈 연산량이 가장 작은 k를 고릅니다. → 이게 바로 "브루트포스 + 동적 계획법" 방식

![]({{site.url}}/images/2025-06-06-dynamic-programming/for-matrix.png){: .align-center}

#### 예시
목표: m[0][2]을 계산해보자 (즉, A₀ × A₁ × A₂) → 가능한 split은 k = 0 or k = 1

1. k = 0:

```txt
m[0][0] + m[1][2] + p[0] * p[1] * p[3]
        ↘   ↘           ↘    ↘    ↘
      A₀  A₁×A₂    A₀의 행  A₀의 열  A₂의 열
```
- A₀ 크기: 10×100 → p[0]×p[1]
- A₁×A₂ 결과 크기: 100×50 → (이건 계산 X, 크기만)
- 곱셈 비용: 10×100×50 = 50,000

2. k = 1:

```txt
m[0][1] + m[2][2] + p[0] * p[2] * p[3]

```
- A₀×A₁ 결과 크기: 10×5 → p[0]×p[2]
- A₂ 크기: 5×50 → p[2]×p[3]
- 곱셈 비용: 10×5×50 = 2,500

## Floyd-Warshall Algorithm 

(All Pairs Shortest Path) 모든 경유 노드 k를 기준으로, i에서 j까지 k를 거치면 더 짧아질 수 있는지를 반복적으로 갱신

### 전제 조건
음수 가중치(edge)는 가능 → 음수 사이클(negative cycle)은 불가능!
(경로 비용이 -∞가 되기 때문에 알고리즘이 무한히 반복될 수 있음)

### 구성요소
1. Initialize dist ← array of infinity
→ 모든 거리 값을 무한대로 초기화. 즉, 처음에는 모든 정점 간의 거리를 "아직 모른다"라고 가정

2. Initialize d[i, j] ← w(i, j)
→ 간선이 직접 연결된 경우는 가중치로 설정
    예: i에서 j로 가는 간선이 있고 가중치가 3이라면 d[i][j] = 3
    자기 자신은 d[i][i] = 0

3. 삼중 루프 (핵심!)
```python
for k in range(0, N):       # 경유 노드
    for i in range(0, N):   # 출발 노드
        for j in range(0, N): # 도착 노드
            d[i][j] = min(d[i][j], d[i][k] + d[k][j])
```
- i에서 j까지 가는 기존 거리(d[i][j]) 와 k를 거쳐 가는 거리(d[i][k] + d[k][j]) 를 비교해서 더 짧은 거리로 갱신

### 시간복잡도
총 N × N × N = O(N³)

### 공간복잡도: O(N²)
d[i][j] 배열이 정점 간 거리 저장용이기 때문


## Dijkstra’s Algorithm (단일 출발점) 
단일 출발점 최단 경로 (Single-Source Shortest Path, SSSP) 계산, **가중치가 있는 그래프**에서 **출발 노드로부터 모든 노드까지의 최단 거리**를 구함.
- Input: designated source node – start here
- Output: distance (& path) between source and each node

#### 저장구조
- dist[u]: 시작 노드로부터 u까지의 최단 거리
- pred[u]: u까지 오는 데 사용된 바로 이전 노드 (경로 추적용) u's predecessor in the shortest path.

![]({{site.url}}/images/2025-06-06-dynamic-programming/dijkstra.png){: .align-center}

#### 시간 복잡도

- Q를 Min-Heap / 우선순위 큐 (Priority Queue) 로 구현 (ex: Python heapq)
- min dist[u] 선택 → O(log V)

각 정점에서 나가는 간선들을 업데이트할 때 → O(log V)
- 총 간선 수 E만큼 큐 갱신 → O(E log V)

**최종 시간복잡도**: O((V + E) log V)
