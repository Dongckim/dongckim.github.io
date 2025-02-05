---
title: "JAVA Coding Test Brute-Force"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: brute-force
use_math: true
---

코딩테스트의 brute force 과정을 한번 이해해보자

## 완전탐색이란?

가능한 모든 경우의 수를 탐색하여 문제의 해를 찾는 방법이다. 모든 경우를 고려하기 때문에 반드시 최적의 해를 찾을 수 있다. 

### 특징과 한계

- 모든 경우의 수를 확인 → 가능한 모든 조합, 배열, 선택 등을 탐색.
- 모든 경우를 확인하기 때문에, 당연히 비효율적인 방법일 수 있다.
    - 입력의 크기가 커지면 커질 수록, 계산량이 폭발적으로 늘어나므로 비효율적이다.
- 하지만, 확실한 정답을 보장한다.
    - 내 컴퓨터나 메모리/시간등이 여유롭다면 충분히 해도해볼만한 방법이긴 하다. 가장 완벽하고 확실하게 정답을 찾아내는 방법이며, 또 정답이 없다면 정답이 없다는 사실조차 반증할 수 있는 방법이 된다.

### 완전 탐색의 종류
- 순열과 조합: 주어진 데이터에서 모든 가능한 순서(순열)나 조합을 생성하여 문제를 해결(ex. 암호해독) → 사실 암호해독도 완전탐색으로 사용하진 않긴 함.
- 재귀: 재귀적으로 모든 경우를 탐색(ex. N-queen) → 일종의 DFS/BFS느낌으로 탐색하는. 관련 문제 살펴볼 예정.
- 반복문을 이용한 구현: 여러 중첩된 반복문으로 경우의 수를 전부 나열(ex. 2D Matrix 탐색 등)
- 비트마스킹: 부분집합 문제에서 모든 경우를 탐색하기 위해 비트를 활용(ex. 모든 부분 집합 찾기)

**사실 완전탐색 문제는 개념적으로는 짚고 넘어가지만, 거의 나오지 않는다고 봐도 무방하다. 너무 단순하기 때문.**

## 완전탐색 다음?

그럼에도 이렇게 짚고 넘어가는 이유는, 다음 스텝으로 넘어가기 위함이라고 생각하면 좋을 꺼 같다.

### 완전탐색으로 한계 극복!

1. **가지치기(Pruning)** → 탐색 중간 유망하지 않은 경로를 제외해서 탐색하는 방식 (백트래킹: 조건에 맞지 않는 친구들이 나오면 그 전 단계로 넘어가서 다음 단계로 넘어가는 방식)

2. **DP** → 이미 계산된 부분을 저장하여 중복 탐색을 방지

3. **그리디 알고리즘** → 항상 최적해를 보장한다는 가정하에, 다음 스텝이 최적해가 아니라면 끝내 버리기 때문에 완전탐색에서 한계를 극복하기 위해 만든 방법으로 볼 수 있다.

4. **Divide and Conquer(분할 정복)** → 문제를 세분화해서 탐색. 세부적으로 쪼갠 뒤, 각각의 세부문제에서 최적값을 조합해서 최적값을 만든다.

5. **Branch and Bound(분지 한정법)** → 그리디 + 부분적 완전탐색, 일부 조건에서 그리디/부분 조건에서 완전 탐색

6. **Heuristic(휴리스틱)** → 거의 보기 힘든 유형이지만, 최적해가 아닌 근사해를 찾는 방법이다. 너무 불분명하고 정답을 내기 매우 힘들기 때문에, 데이터를 다룰 때는 만나보실 수도 있지만, 코테 문제로서는 만나기 힘들지 않을까... 싶다.

7. **이분탐색** → 조건에 따라 탐색 범위를 줄여가며 탐색

8. **병렬 처리 (결과론적으로 완전 탐색)** → 병렬로 탐색 범위를 분할하여 탐색

9. lots of other else...

## 완전탐색을 자신있게 시도하는 상황

- 변수의 범위가 작다. 1,000,0000 아래 쪽이면 시도해볼만하다. 
- 문제에서 주어진 것들을 써가다 보면 풀릴 수 있는게 완전 탐색이라고 본다. 그래서 코테에서 많이 안나옴.

## [N-Queen 문제](https://www.acmicpc.net/problem/9663)

대표적인 백트래킹 문제이다. 이를 완전탐색과 완전탐색보다 더 효율적인 방법인 백트래킹 방법을 활용하여 느껴보는게 주 목적이다.

### brute-force방식
permutation을 이용해서 퀸 배치순서에 대한 모든 가능성을 다 돌아본 후, 예를들면, **4개의 숫자를 순서를 바꿔 나열하는 모든 경우(4! = 24개)**의 경우를 다 for문으로 돌아본다.
- 이후 같은 대각선에 있는 경우를 따져주기만 하면 될꺼 같다. → `is_valid` 함수 참고.

```python
from itertools import permutations
import time

def n_queens_bruteforce(N):
  def is_valid(board):
    for i in range(N):
      for j in range(i + 1, N):
        # 같은 대각선에 있는 경우 체크
        if abs(board[i] - board[j]) == abs(i-j):
          return False
    return True

  count = 0
  for perm in permutations(range(N)):
    if is_valid(perm):
      count += 1
  return count
```

### 백트래킹
- 현재 row에서 퀸을 놓을 수 있는 모든 열을 확인한다.
- 퀸은 같은 열이나 대각선 방향에 있으면 공격할 수 있다.
- `col_used[col] or diag1_used[row-col+N-1] or diag2_used[row+col]`의 경우는 안됨.
- 퀸을 놓을 수 있다면, 배치를 기록하고 다음 행(row + 1)에서 탐색을 계속
- solve(row + 1) → 재귀 호출 (다음 행으로 이동) → row + 1을 호출하여 다음 행에 퀸을 배치할 수 있는지 확인 → 만약 row == N이면 (즉, 마지막 행까지 도달하면), 유효한 배치 1개 찾은 것!

```python
def n_queens(N):
  def solve(row):
    if row == N:
      return 1
    count = 0
    for col in range(N):
      if col_used[col] or diag1_used[row-col+N-1] or diag2_used[row+col]:
        continue

      col_used[col] = diag1_used[row-col+N-1] = diag2_used[row+col] = True
      count += solve(row + 1)
      col_used[col] = diag1_used[row-col+N-1] = diag2_used[row+col] = False

    return count

  col_used = [False] * N
  diag1_used = [False] * (2 * N -1)
  diag2_used = [False] * (2 * N -1)

  return solve(0)
```

### 더 줄일 수 있나? (대칭? 회전?)

```Python
def n_queens_symmetry(N):
    def solve(row):
        nonlocal count
        if row == N:
            count += 1
            return

        # 대칭을 활용하여 체스판의 절반만 탐색
        start_col = 0 if row > 0 else (N + 1) // 2

        for col in range(start_col, N):
            if col_used[col] or diag1_used[row - col + N - 1] or diag2_used[row + col]:
                continue

            col_used[col] = diag1_used[row - col + N - 1] = diag2_used[row + col] = True
            solve(row + 1)
            col_used[col] = diag1_used[row - col + N - 1] = diag2_used[row + col] = False

    count = 0
    col_used = [False] * N
    diag1_used = [False] * (2 * N - 1)
    diag2_used = [False] * (2 * N - 1)

    solve(0)

    # 전체 해 개수는 찾은 해의 2배 (좌우 대칭 반영)
    return count * 2 if N % 2 == 0 else count * 2 - solve_half(N)

def solve_half(N):
    """ N이 홀수일 때, 중앙 열을 포함한 경우를 따로 계산 """
    col_used = [False] * N
    diag1_used = [False] * (2 * N - 1)
    diag2_used = [False] * (2 * N - 1)
    count = 0

    def solve(row):
        nonlocal count
        if row == N:
            count += 1
            return
        for col in range(N):
            if col_used[col] or diag1_used[row - col + N - 1] or diag2_used[row + col]:
                continue
            col_used[col] = diag1_used[row - col + N - 1] = diag2_used[row + col] = True
            solve(row + 1)
            col_used[col] = diag1_used[row - col + N - 1] = diag2_used[row + col] = False

    col_used[N // 2] = True
    diag1_used[-(N // 2) + N - 1] = True
    diag2_used[N // 2] = True
    solve(1)
    return count

```