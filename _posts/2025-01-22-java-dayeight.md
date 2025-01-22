---
title: "99클럽 JAVA 코딩테스트 예시답안 8일차 [단지번호붙이기]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
use_math: true
---

단지번호붙이기

## 문제 안에서 특이사항 확인하기

연결됨의 의미를 포인트를 잡고 넘어가야겠다. → 좌우 + 위아래 (what if 대각선도 포함된다면?)
- 현재 문제에서는 대각선은 고려하지 않기 때문에 대각선은 생각만 해보자.

그림부터가 그냥 BFS와 DFS가 떠올라야했다. 그럼 이 둘 중에 뭘 써야하지?에 대한 생각을 해보자.


## 제한사항 꼭 확인하기

애초에 5≤N≤25 이긴 하지만, 그림부터가 DFS과 BFS라는 걸 티 내고 있기 때문에, 범위가 커지고 작아지고에 따라 알고리즘이 달라지진 않겠다.

## 입출력 비교하기

![]({{site.url}}/images/2025-01-22-java-dayeight/io.png)

그래프 탐색의 기본 원리 :
- BFS와 DFS 모두 특정 노드에서 시작하여, 연결된 노드를 하나씩 방문하면서 탐색을 진행. 이 과정에서 모든 연결된 노드를 방문하면 "하나의 연결 요소"를 완전히 탐색한 것으로 따진다.
- 상하좌우로 연결된 경우 같은 단지로 간주됨, BFS와 DFS 모두 상하좌우를 탐색하며 연결 요소를 탐색하는 데 적합

**이 문제에서는 단지 내 집의 수를 구하는 것이 목표이므로, 두 알고리즘 모두 사용 가능**

### BFS 
- 단지 내 집의 개수를 한 번에 계산할 수 있음.
- 구현이 직관적이고, 탐색 순서가 일정.
- 모든 경로를 균등하게 탐색해야 할 때

**시간복잡도** : $O(V+E)$
- V는 정점의 개수, E는 간선의 개수

### DFS
- 재귀적으로 호출하면서 단지 내 집의 개수를 누적 계산.
- 메모리 사용량이 적으며, 작은 그래프에서는 구현이 간단.
- 하나의 경로를 끝까지 탐색 후 백트래킹
- 특정 경로를 빠르게 찾고 싶을 때

**시간복잡도** : $O(V+E)$
- V는 정점의 개수, E는 간선의 개수

## Psudo Code

BFS 기본 뼈대 다들 알고 있지 않나?

## Java Code


### DFS 풀이

```java
import java.util.*;

public class Main {
    static int[][] map;
    static boolean[][] visited;
    static int N;
    static int count; // 단지 내 집의 수를 세는 변수
    static List<Integer> result = new ArrayList<>(); // 단지별 집의 수 저장

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        N = sc.nextInt();
        map = new int[N][N];
        visited = new boolean[N][N];

        // 지도 입력
        for (int i = 0; i < N; i++) {
            String line = sc.next();
            for (int j = 0; j < N; j++) {
                map[i][j] = line.charAt(j) - '0';
            }
        }

        // 모든 위치를 탐색하며 단지 찾기
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                if (map[i][j] == 1 && !visited[i][j]) {
                    count = 0; // 새로운 단지를 찾았으므로 초기화
                    dfs(i, j); // DFS 탐색 시작
                    result.add(count); // 단지 내 집의 수 저장
                }
            }
        }

        // 결과 출력
        Collections.sort(result); // 집의 수를 오름차순으로 정렬
        System.out.println(result.size()); // 총 단지 수 출력
        for (int num : result) {
            System.out.println(num); // 각 단지 내 집의 수 출력
        }
    }

    // DFS 탐색
    static void dfs(int x, int y) {
        visited[x][y] = true;
        count++; // 집의 수 증가

        // 상하좌우 방향
        int[] dx = {-1, 1, 0, 0};
        int[] dy = {0, 0, -1, 1};

        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i];
            int ny = y + dy[i];

            // 유효한 위치인지 확인
            if (nx >= 0 && ny >= 0 && nx < N && ny < N) {
                if (map[nx][ny] == 1 && !visited[nx][ny]) {
                    dfs(nx, ny);
                }
            }
        }
    }
}
```

### BFS 풀이
```java
import java.util.*;

public class Main {
    static int[][] map;
    static boolean[][] visited;
    static int N;
    static List<Integer> result = new ArrayList<>();

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        N = sc.nextInt();
        map = new int[N][N];
        visited = new boolean[N][N];

        // 지도 입력
        for (int i = 0; i < N; i++) {
            String line = sc.next();
            for (int j = 0; j < N; j++) {
                map[i][j] = line.charAt(j) - '0';
            }
        }

        // 모든 위치를 탐색하며 단지 찾기
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                if (map[i][j] == 1 && !visited[i][j]) {
                    result.add(bfs(i, j)); // BFS로 단지 내 집의 수 계산
                }
            }
        }

        // 결과 출력
        Collections.sort(result); // 집의 수를 오름차순으로 정렬
        System.out.println(result.size()); // 총 단지 수 출력
        for (int num : result) {
            System.out.println(num); // 각 단지 내 집의 수 출력
        }
    }

    // BFS 탐색
    static int bfs(int x, int y) {
        Queue<int[]> queue = new LinkedList<>();
        queue.add(new int[]{x, y});
        visited[x][y] = true;
        int count = 1; // 시작 위치 포함

        // 상하좌우 방향
        int[] dx = {-1, 1, 0, 0};
        int[] dy = {0, 0, -1, 1};

        while (!queue.isEmpty()) {
            int[] current = queue.poll();
            int cx = current[0];
            int cy = current[1];

            for (int i = 0; i < 4; i++) {
                int nx = cx + dx[i];
                int ny = cy + dy[i];

                // 유효한 위치인지 확인
                if (nx >= 0 && ny >= 0 && nx < N && ny < N) {
                    if (map[nx][ny] == 1 && !visited[nx][ny]) {
                        queue.add(new int[]{nx, ny});
                        visited[nx][ny] = true;
                        count++; // 집의 수 증가
                    }
                }
            }
        }
        return count;
    }
}

```