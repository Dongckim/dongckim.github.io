---
title: "99클럽 타임어택 보너스 문제풀이 [효율적인 해킹]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
---

효율적인 해킹

>김지민은 귀찮기 때문에, 한 번의 해킹으로 여러 개의 컴퓨터를 해킹 할 수 있는 컴퓨터를 해킹
> 어이가 없을 뿐이다.

## 예시로 다시 이해해볼까

```
5 4
3 1
3 2
4 3
5 3
```

첫째 줄 N은 컴퓨터 갯수(1~N까지 가능), M은 신뢰관계의 갯수

그럼, N=5 M=4일 때, 5대의 컴퓨터가 4개의 신뢰관계를 가지고 있다고 이해하고 넘어가자.

- `3 1`  -> 3이 1을 신뢰한다.
- `3 2`  -> 3이 2를 신뢰한다.
- `4 3`  -> 4가 3을 신뢰한다.
- `5 3`  -> 5가 3을 신뢰한다.

이렇게 해석할 수 있고, 이를 역방향 그래프로 해킹의 원리가 될 것이다.

정리하면,

- 3 → [1, 2]
- 4 → 3
- 5 → 3

시뮬레이션을 돌려볼까?

### 시뮬레이션

1번 컴퓨터를 해킹하면, 1번을 신뢰하는 3을 해킹가능해지고, → 3번을 해킹 가능해지면, 4 or 5 까지 해킹 가능해진다.(총 3대)

2번도 동일하다. 2번 컴퓨터를 해킹하면 2번을 신뢰하는 3번을 해킹가능해지고, → 3번을 해킹 가능해지면, 4 or 5까지 해킹 가능해진다.(총 3대)

3번은 어떨까? 3번 컴퓨터를 해킹하면, 4 or 5까지 해킹 가능하면서 2개의 컴퓨터만 해킹할 수 있게 된다.(총 2대)

4,5번은 동일하게 신뢰하는 컴퓨터가 없어서 1대씩만 가능할 것이다.(총 1대)

## 생각과정

"어떤 컴퓨터에서 시작해서 몇 개의 컴퓨터를 해킹할 수 있는지", 즉 "시작 노드에서 연결된 노드를 얼마나 많이 탐색할 수 있는지"를 묻고 있음.

그래서 그래프 탐색 알고리즘을 써야 한다는 결론

- 컴퓨터에서 시작해서 연결된 다른 컴퓨터들을 해킹하는 과정이므로, DFS가 적합
- DFS를 사용하면 한 컴퓨터에서 출발해 연결된 컴퓨터를 끝까지 탐색한 후, 그 컴퓨터가 신뢰하는 다른 컴퓨터를 다시 탐색하는 방식이 잘 맞는 것 같다.

아마 생각하는 과정 중에, N은 10,000보다 작거나 같은 자연수, M은 100,000보다 작거나 같은 자연수 라는 조건이 마음에 걸렸을 지도 모른다.
-  10,000개의 노드와 100,000개의 간선

>DFS + 메모이제이션을 활용하여 중복 연산을 방지 → O(N + M)으로 개선을 할 수 있어보이나, 현재 백준에서의 메모리가 한계가 있어서 이 시도는 백준 내에서는 실패로 뜬다.

결국 우리는 python의 한계를 여기서 느낄 수 있을 것이다. 당신의 탓이 아님.

## Pypy3와 cPython의 차이점

### CPython
한 줄씩 인터프리트(Interpretation) 방식으로 실행
→ 실행할 때마다 바이트코드 해석이 필요하여 속도가 상대적으로 느림

#### Python의 재귀 한계
- Python3의 기본 재귀 한도는 1000 (너무 깊어지면 RecursionError)
- sys.setrecursionlimit(10**6)을 사용해도 Python3는 함수 호출 오버헤드가 크기 때문에 속도가 느려짐.

### PyPy
JIT(Just-In-Time) 컴파일 적용 (= Java)
→ 실행 중에 자주 사용하는 코드(반복되는 루프, 재귀 등)를 미리 기계어로 변환하여 속도를 높임

#### PyPy는 리스트와 딕셔너리 연산이 빠름.
- PyPy는 리스트, 딕셔너리 연산 최적화가 되어 있어서 그래프 탐색에서 성능이 향상됨
- 이 문제에서는 인접 리스트로 그래프를 저장하고 탐색하는 과정에서 많은 리스트 연산이 수행됨
→ PyPy가 이를 최적화하여 실행 속도를 높여줌

## 예시코드

### 자바
```java
import java.io.*;
import java.util.*;

public class Main {
	static int n, m;					// n개 컴퓨터, m개 컴퓨타 신뢰 관계
	static List<Integer>[] lists;		// 컴퓨터 신뢰 관계 (인접 리스트)
	static int[] hackCounts;			// [i]번 컴퓨터를 해킹 했을때, 해킹 가능한 컴퓨터 수
	static boolean[] visited;

	static void dfs(int currentIdx) {
		for (int nextIdx : lists[currentIdx]) {
			if (!visited[nextIdx]) {
				visited[nextIdx] = true;
				hackCounts[nextIdx]++;
				dfs(nextIdx);
			}
		}
	}

	public static void main(String[] args) throws IOException {
		BufferedReader br = new BufferedReader(
				new InputStreamReader(System.in)
		);
		StringTokenizer st = new StringTokenizer(br.readLine());

		n = Integer.parseInt(st.nextToken());
		m = Integer.parseInt(st.nextToken());

		lists = new List[n + 1];			// 컴퓨터(정점) 번호 [1] ~ [n] 사용
		hackCounts = new int[n + 1];
		for (int i = 1; i <= n; i++) {
			lists[i] = new ArrayList<>();
		}

		for (int i = 0; i < m; i++) {
			st = new StringTokenizer(br.readLine());
			int start = Integer.parseInt(st.nextToken());
			int end = Integer.parseInt(st.nextToken());

			lists[start].add(end);
		}

		// 각 [i]번 컴퓨터를 시작 노드로 하여 DFS 탐색 수행
		for (int i = 1; i <= n; i++) {
			visited = new boolean[n + 1];	// 방문 처리 배열 초기화

			visited[i] = true;
			hackCounts[i]++;
			dfs(i);
		}

		int maxHackCount = 0;
		for (int i = 1; i <= n; i++) {
			maxHackCount = Math.max(maxHackCount, hackCounts[i]);
		}

		StringBuilder sb = new StringBuilder();
		for (int i = 1; i <= n; i++) {
			if (hackCounts[i] == maxHackCount) {
				sb.append(i).append(" ");
			}
		}
		System.out.println(sb);
	}
}
```

### 파이썬 (BFS) → 되는 코드

```python
import sys
from collections import deque

N, M = map(int, sys.stdin.readline().split())

graph = [[] for _ in range(N + 1)]
for _ in range(M):
    A, B = map(int, sys.stdin.readline().split())
    graph[B].append(A)

def BFS(v):
    visited[v] = False
    queue = deque()
    queue.append(v)

    while (queue):
        now = queue.popleft()
        for i in graph[now]:
            if (visited[i]):
                queue.append(i)
                visited[i] = False

    return visited.count(False)

result = [0] * (N + 1)
for i in range(1, N + 1):
    visited = [True] * (N + 1)
    result[i] = BFS(i)

MAX = max(result)
for i in range(1, N + 1):
    if (result[i] == MAX):
        print(i, end=' ')
```

### 파이썬 (DFS+메모이제이션) → 안되는 코드

근데 메모리초과 나옴. JAVA랑 다른 가장 큰 원인.
- PyPy는 JIT 최적화 때문에 함수 호출 스택을 더 많이 사용
- PyPy의 GC방식이 Java보다 메모리를 더 많이 잡아두는 경향

```python
import sys
sys.setrecursionlimit(10**6)

def dfs(node):
    if visited[node] != -1:
        return visited[node]  # 이미 계산된 값이면 리턴
    visited[node] = 1  # 자기 자신 포함
    for neighbor in graph[node]:
        visited[node] += dfs(neighbor)
    return visited[node]

# 입력 처리
n, m = map(int, sys.stdin.readline().split())
graph = [[] for _ in range(n + 1)]
for _ in range(m):
    a, b = map(int, sys.stdin.readline().split())
    graph[b].append(a)  # 역방향 그래프 저장

# 방문 배열 (-1: 아직 계산되지 않음)
visited = [-1] * (n + 1)
max_hacked = 0
result = []

# 모든 노드에서 DFS 실행
for i in range(1, n + 1):
    hacked_count = dfs(i)
    
    if hacked_count > max_hacked:
        max_hacked = hacked_count
        result = [i]
    elif hacked_count == max_hacked:
        result.append(i)

# 결과 출력
print(*sorted(result))
```


## pypy3 되는 코드 (DFS + DP)

```python
from collections import deque
import sys

input = sys.stdin.read
data = input().split("\n")

n, m = map(int, data[0].split())
dic = {i: [] for i in range(1, n + 1)}

# 그래프 입력 받기
for i in range(1, m + 1):
    if data[i]:
        a, b = map(int, data[i].split())
        dic[b].append(a)  

dp = [-1] * (n + 1) # <------초기값을 -1로 설정.

def bfs(start):
    if dp[start] != -1:
        return dp[start]  # <------------if 초기값인 -1이 아니라면, 거쳐간 것으로 판단 후 그 값을 그냥 가져와 반영.

    visited = [False] * (n + 1)
    queue = deque([start])
    visited[start] = True
    count = 0

    while queue:
        node = queue.popleft()
        for neighbor in dic[node]:
            if not visited[neighbor]:
                visited[neighbor] = True
                queue.append(neighbor)
                count += 1 

    dp[start] = count  # <-------아니라면 queue를 바탕으로 계산된 값을 dp테이블에 저장해서 계속 사용할 수 있도록 합니다.
    return count # <--------dp에서 따로 가져올 필요없이, 처음 계산된 값이기 때문에 queue 과정을 모두 끝낸 count 값 리턴

max_hack = 0
result = []

for i in range(1, n + 1):
    hack_count = bfs(i)
    if hack_count > max_hack:
        max_hack = hack_count
        result = [i]
    elif hack_count == max_hack:
        result.append(i)

print(" ".join(map(str, result)))


```