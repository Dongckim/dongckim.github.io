---
title: "99클럽 JAVA 코딩테스트 예시답안 9일차 [빙산]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
use_math: true
---

빙산, 저번기수 타임어택 문제였다.

## 문제 안에서 특이사항 확인하기

문제의 핵심을 이해해보자.

1. 빙산이 녹는다
빙산의 높이는 주변에 있는 바다(0의 개수)만큼 줄어들고, 높이는 0보다 작아질 수 없다.

2. 빙산이 분리된다
빙산이 한 덩어리였다가 두 덩어리 이상으로 나뉘는 순간을 찾는다.

3. 분리되는 데 걸린 시간을 구한다.
빙산이 다 녹으면 종료
만약 빙산이 전부 녹았는데도 분리되지 않았다면, 답은 0이다.


## 제한사항 꼭 확인하기

제한 사항 없음.

## 입출력 비교하기

![]({{site.url}}/images/2025-01-23-java-dayten/io.png)

### 빙산 녹이기
빙산은 매년 다음 규칙에 따라 녹는 걸 알 수 있다.
각 칸에서 주변(동, 서, 남, 북) 바다(0)의 개수를 세고, 그만큼 높이를 줄인다. + 높이는 0보다 작아질 수 없음.
- 2는 주변에 바다가 1칸 있으니 2 - 1 = 1로 줄어듭니다.
- 4는 주변에 바다가 2칸 있으니 4 - 2 = 2로 줄어듭니다.

### 빙산 덩어리 확인
빙산이 몇 덩어리로 나뉘었는지 확인
- 덩어리란 동서남북으로 연결된 빙산 부분.
- 예를 들어, 예시에서는 빙산이 3개의 덩어리로 나뉘었음.

### 시뮬레이션 반복
1년씩 시간이 흐르며 반복:
- 빙산을 녹이기.
- 빙산 덩어리 개수를 세기.
- 덩어리가 2개 이상이면 현재 연도를 출력하고 종료.
- 빙산이 전부 녹으면 0을 출력하고 종료합니다.


## Psudo Code

```java
import java.util.*;

public class Iceberg {
    static int n, m;
    static int[][] iceberg;
	{% raw %}
    static int[][] directions = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
	{% endraw %}

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // 입력받기
        n = sc.nextInt();
        m = sc.nextInt();
        iceberg = new int[n][m];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                iceberg[i][j] = sc.nextInt();
            }
        }

        int years = 0;
        while (true) {
            // 빙산 덩어리 개수 확인
            int parts = countIcebergParts();
            if (parts >= 2) { // 두 덩어리 이상으로 분리된 경우
                System.out.println(years);
                break;
            }
            if (isAllMelted()) { // 빙산이 모두 녹은 경우
                System.out.println(0);
                break;
            }

            // 빙산 녹이기
            meltIceberg();
            years++;
        }

        sc.close();
    }

    // 빙산 녹이기 함수
    static void meltIceberg() {
        int[][] melted = new int[n][m]; // 새로 녹은 높이를 저장할 배열

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (iceberg[i][j] > 0) { // 빙산인 경우
                    int waterCount = 0;
                    for (int[] dir : directions) {
                        int nx = i + dir[0];
                        int ny = j + dir[1];
                        if (nx >= 0 && nx < n && ny >= 0 && ny < m && iceberg[nx][ny] == 0) {
                            waterCount++;
                        }
                    }
                    melted[i][j] = Math.max(0, iceberg[i][j] - waterCount); // 높이 줄이기
                }
            }
        }

        // 원본 배열 갱신
        for (int i = 0; i < n; i++) {
            System.arraycopy(melted[i], 0, iceberg[i], 0, m);
        }
    }

    // 빙산 덩어리 개수 세기 함수
    static int countIcebergParts() {
        boolean[][] visited = new boolean[n][m];
        int parts = 0;

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (iceberg[i][j] > 0 && !visited[i][j]) { // 방문하지 않은 빙산이라면
                    bfs(i, j, visited);
                    parts++;
                }
            }
        }
        return parts;
    }

    // BFS로 연결된 빙산 탐색
    static void bfs(int x, int y, boolean[][] visited) {
        Queue<int[]> queue = new LinkedList<>();
        queue.add(new int[]{x, y});
        visited[x][y] = true;

        while (!queue.isEmpty()) {
            int[] current = queue.poll();
            int cx = current[0];
            int cy = current[1];

            for (int[] dir : directions) {
                int nx = cx + dir[0];
                int ny = cy + dir[1];
                if (nx >= 0 && nx < n && ny >= 0 && ny < m && !visited[nx][ny] && iceberg[nx][ny] > 0) {
                    visited[nx][ny] = true;
                    queue.add(new int[]{nx, ny});
                }
            }
        }
    }

    // 빙산이 모두 녹았는지 확인
    static boolean isAllMelted() {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (iceberg[i][j] > 0) {
                    return false;
                }
            }
        }
        return true;
    }
}

```