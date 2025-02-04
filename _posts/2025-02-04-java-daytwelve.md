---
title: "99클럽 JAVA 코딩테스트 예시답안 12일차 [숫자 정사각형]"
layout: single
Typora-root-url: ../
categories: Three.js
tag: light
use_math: true
---

숫자 정사각형

## 문제 안에서 특이사항 확인하기

이 직사각형에서 꼭짓점에 쓰여 있는 수가 모두 같은 가장 큰 정사각형
-  정사각형은 행 또는 열에 평행
- 즉, 다이아몬드 형태의 정사각형은 인정하지 않음.

## 제한사항 꼭 확인하기

N과 M은 50보다 작거나 같은 자연수

## 입출력 비교하기

- 출력: 첫째 줄에 정답 정사각형의 크기를 출력한다.

### 생각과정

어떤 Length의 값을 바탕으로 범위 안에서 조사한다고 했을때, 범위 밖의 가능성에 대해 우리가 판단할 수 있는 근거는 1도 없다.
- 따라서 전부 다 조사를 해봐야 답을 알 수 있기 때문에 brute force의 방식을 사용해야만 하겠다.

## Psudo Code

### 구현 구현 구현

- **정사각형의 최대 길이**: \( \min(N, M) \) → 이를 `len`이라 하자.  
- **탐색 범위 최적화**:  
  - 세로: $ 0 \$ ~ $\( N - \text{len} \) $ 
  - 가로: $ 0 \$ ~ $\( M - \text{len} \) $ 
  - 배열 범위를 벗어나지 않도록 조정  
- **탐색 방식**:  
  1. 가장 긴 정사각형부터 (`len`부터 1씩 감소)  
  2. `(i, j)`를 기준점으로 잡고 **세 꼭짓점 비교**  
  3. 모두 같다면 `len × len` 출력 후 종료  
- **최적화된 종료 조건**:  
  - 가장 긴 길이부터 탐색했으므로 처음 발견한 정사각형이 최댓값  
  - 즉시 종료해도 됨  
- **완전 탐색 후에도 정사각형이 없으면** → `len`을 줄여서 반복  
- **시간 복잡도 최적화**:  
  - $ O(NM) \$보다 개선됨 (긴 길이부터 탐색하므로 조기 종료 가능)  


## JAVA CODE

```java
import java.io.*;
import java.util.*;

public class Main {
    static int N, M;
    static int[][] map;

    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());

        N = Integer.parseInt(st.nextToken());
        M = Integer.parseInt(st.nextToken());
        map = new int[N][M];

        for (int i = 0; i < N; i++) {
            String str = br.readLine();
            for (int j = 0; j < M; j++) {
                map[i][j] = str.charAt(j) - '0';
            }
        }

        int len = Math.min(N, M); // 정사각형의 최대 길이

        while (len > 1) {
            for (int i = 0; i <= N - len; i++) {
                for (int j = 0; j <= M - len; j++) {
                    int num = map[i][j];

                    // 네 꼭짓점 비교
                    if (num == map[i][j + len - 1] &&
                        num == map[i + len - 1][j] &&
                        num == map[i + len - 1][j + len - 1]) {
                        System.out.println(len * len);
                        return;
                    }
                }
            }
            len--; // 정사각형 크기 줄이기
        }

        System.out.println(1); // 정사각형이 없으면 크기 1 출력
    }
}

```
- `map[i][j] = str.charAt(j) - '0'`은 자주 사용하는 기법이니 알아두자.
- `str.charAt(j)` : j번째 문자열을 가져오고, 문자를 숫자로 변환하는 `- 0`을 하면
- `0`의 ASCII값인 48에서 뺄셈이 진행되면서, 정수가 가져와짐.
- `1` - `0` → `49 - 48` → `1`
- `2` - `0` → `50 - 48` → `2`
- `3` - `0` → `51 - 48` → `3`
