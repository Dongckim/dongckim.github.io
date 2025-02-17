---
title: "99클럽 타임어택 보너스 문제풀이 [센서]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
use_math: true
---

센서, 저번기수 타임어택 문제이다.

## 문제 안에서 특이사항 확인하기

- 집중국의 수신 가능 영역은 고속도로 상에서 연결된 구간으로 나타나게 된다.(평면상의 직선)
- N개의 센서가 **적어도 하나의 집중국과는 통신이 가능**해야 하며, 집중국의 유지비 문제로 인해 각 집중국의 수신 가능 영역의 길이의 합을 최소화
- 길이의 합을 최소화? → 작은애들끼리만 모여있으면 되겠다!
-  각 집중국의 수신 가능영역의 거리의 합의 최솟값을 구하는 프로그램 → 수신 가능영역의 거리를 어떻게 구하면 되지?


## 제한사항 꼭 확인하기

- (1 ≤ N ≤ 10,000)
- (1 ≤ K ≤ 1000)
- 뭐든 가능하겠다.

## 입출력 비교하기

1. 센서를 정렬
센서의 위치가 랜덤하게 주어지기 때문에, 먼저 정렬을 해주는게 중요
```
정렬된 센서 위치: 1, 3, 6, 6, 7, 9
```

2. 센서 간의 거리 구하기
센서들이 떨어져 있는 거리를 계산
```
(3 - 1) = 2
(6 - 3) = 3
(6 - 6) = 0
(7 - 6) = 1
(9 - 7) = 2
```
각 센서 간의 거리를 나열하면 `[2, 3, 0, 1, 2]`

3. 큰 간격을 끊어서 K개의 그룹 만들기
큰 간격을 우선적으로 끊어서 K개의 구역을 만든다.
- 거리 리스트: [2, 3, 0, 1, 2] 
- 가장 큰 간격(3)을 끊으면, 구역이 나뉘어짐

4. 최소 거리 합 = 2 + 3 = 5 
```
1, 3 → 하나의 구역 (길이 2)
6, 6, 7, 9 → 하나의 구역 (길이 3)
최소 거리 합 = 2 + 3 = 5
```

## 핵심 방법 정리
- 센서 간 거리를 계산하고, 큰 간격부터 끊어서 그룹을 만든다.
- K개의 집중국을 배치해야 하므로, (K-1)개의 큰 간격을 제거하면 된다.
- 남은 거리들의 합이 최소의 수신 가능 영역 길이가 된다.

## psudo code

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.StringTokenizer;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        int N = Integer.parseInt(br.readLine());
        int K = Integer.parseInt(br.readLine());
        int[] sensors = new int[N];

        StringTokenizer st = new StringTokenizer(br.readLine());
        for (int i = 0; i < N; i++) {
            sensors[i] = Integer.parseInt(st.nextToken());
        }

        Arrays.sort(sensors);

        int[] distances = new int[N - 1];
        for (int i = 0; i < N - 1; i++) {
            distances[i] = sensors[i + 1] - sensors[i];
        }

        Arrays.sort(distances);

        int minLength = 0;
        for (int i = 0; i < N - K; i++) {
            minLength += distances[i];
        }

        System.out.println(minLength);
    }
}
```