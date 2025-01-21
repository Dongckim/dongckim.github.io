---
title: "99클럽 JAVA 코딩테스트 예시답안 7일차 [숨바꼭질]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
use_math: true
---

숨바꼭질

## 문제 안에서 특이사항 확인하기

100,000이 범위에 좀 확실하게 눈을 떠봐야겠다. 완전탐색은 힘들어보이네요?

움직임의 방식이 모두 세 가지이다.
- $x + 1$
- $x - 1$
- $x * 2$

이 세 가지 경우의 수를 각각 따져보면서, 생각해봐야겠다. 이걸 생각해보자. 이전 스텝이 중요한가?
- $3$ 이라는 좌표에서 저 세 가지 방식으로 움직여질 텐데, $3$이 어떤 방식을 거쳐서 왔는지가 중요한가?
- 그냥 몇초에 거쳐서 $3$에 왔는지가 중요하지 않나?


## 제한사항 꼭 확인하기

제한 사항은 없다.

## 입출력 비교하기

| 입력 예제 | 출력 예제 |
|-----------|-----------|
| 5 17      | 4         |

BFS는 그래프 탐색에서 최단 경로를 보장한다. 만약 수빈이가 동생을 2 초에 만났고, BFS 탐색이 3 초를 넘어가면 이미 최단 시간이 아니기 때문에 그 이후는 더 볼 필요가 없다는 것이다.

그래프 탐색이 아닌, 단순히 정렬된 데이터에서 값을 찾는 문제였다면 이진 탐색이 적합할 것 같다.

타 방법들을 비교해보면 DFS는 재귀를 바탕으로 찾아나가는데 말이 안된다.

## Psudo Code

BFS 기본 뼈대 다들 알고 있지 않나?

```python
from collections import deque

def find_fastest_time(N, K):
    # 최대 범위 설정
    max_limit = 100000
    visited = [False] * (max_limit + 1)  # 방문 여부 확인
    queue = deque([(N, 0)])  # (현재 위치, 시간)
    
    while queue:
        current, time = queue.popleft()  # 큐에서 현재 위치와 시간을 꺼냄
        
        # 동생을 찾으면 시간 반환
        if current == K:
            return time
        
        # 방문하지 않은 위치를 큐에 추가
        for next_pos in (current - 1, current + 1, current * 2):
            if 0 <= next_pos <= max_limit and not visited[next_pos]:
                visited[next_pos] = True  # 방문 표시
                queue.append((next_pos, time + 1))  # 큐에 다음 위치와 시간을 추가
```

### BFS 조금 살펴볼까?

무조건 이거만 기억하자. `초기화(1) → 반복 탐색(2) → 종료 조건(3) → 다음 위치 탐색(4)`

1. BFS의 기본 베이스는 `queue`를 사용하는 것이다. FIFO(First-In-First-Out) 특성 덕분에 항상 최단 경로를 먼저 탐색하는 것이라는 점은 충분히 알고 있다고 생각하고 넘어가면 되겠다.

2. `visited` 매우 중요하다. 방문했던 노드(인덱스)를 다시 한번 방문하는 것은 말이 안될 꺼 같다. visited를 꼭 만들어서 중복 검사를 진행하면서 순회하면 되겠다.

3. 큐가 빌 때까지 반복한다. queue.popleft()를 통해 큐의 맨 앞에서 현재 위치와 소요 시간을 꺼내서 비교한다.

4. 큐가 빌 때까지 반복은 하지만, 종료조건이 필요하겠다. 문제로 보면 `동생이 있는 위치와 만나게 될 때` 라는 종료 조건이 필수로 있어야겠다.

5. 다음 위치 탐색을 하는데에는 위에서 언급한 3가지 방식이 필요해 보인다. X−1 (한 칸 뒤로 걷기), X+1 (한 칸 앞으로 걷기), 2×X (순간이동)
    - 각 위치가 유효한 범위(0 ≤ 위치 ≤ 100,000) 내에 있고, 아직 방문하지 않았다면 큐에 추가

### 정리

BFS는 큐를 사용하여 현재 시간에서 갈 수 있는 모든 위치를 탐색한다.

이 좌표가 어떻게 왔는지는 중요하지 않다. 어떤 방식이든, 동생 위치에 처음 도달하는 순간의 시간이 최단 시간이 되는 것이다.

## Java Code

```java
import java.util.*;

public class Main {
    public static int findFastestTime(int N, int K) {
        int maxLimit = 100000;
        boolean[] visited = new boolean[maxLimit + 1];
        Queue<int[]> queue = new LinkedList<>(); 
        queue.add(new int[]{N, 0});
        
        while (!queue.isEmpty()) {
            int[] current = queue.poll(); 
            int position = current[0];
            int time = current[1];
            
            if (position == K) {
                return time;
            }
            
            for (int nextPos : new int[]{position - 1, position + 1, position * 2}) {
                if (nextPos >= 0 && nextPos <= maxLimit && !visited[nextPos]) {
                    visited[nextPos] = true;
                    queue.add(new int[]{nextPos, time + 1});
                }
            }
        }
        return -1;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int N = scanner.nextInt(); 
        int K = scanner.nextInt();

        System.out.println(findFastestTime(N, K));
    }
}
```
