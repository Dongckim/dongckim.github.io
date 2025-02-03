---
title: "99클럽 JAVA 코딩테스트 예시답안 10일차 [체스판 다시 칠하기]"
layout: single
Typora-root-url: ../
categories: Three.js
tag: light
use_math: true
---

체스판 다시 칠하기

## 문제 안에서 특이사항 확인하기

M×N 크기의 보드 →  이 보드를 잘라서 8×8 크기의 체스판으로 만든다.

따라서 이 정의를 따르면 체스판을 색칠하는 경우는 두 가지뿐이다. 
- 하나는 맨 왼쪽 위 칸이 흰색인 경우, 
- 하나는 검은색인 경우이다.

## 제한사항 꼭 확인하기

N과 M은 8보다 크거나 같고, 50보다 작거나 같은 자연수

## 입출력 비교하기

첫째 줄에 지민이가 다시 칠해야 하는 정사각형 개수의 최솟값을 출력한다.

## Psudo Code

### 현재 상태 vs 체스판 변환 비용

| 현재 상태 | Black 체스판 (최소 비용) | Count | White 체스판 (최소 비용) | Count |
|-----------|----------------|-------|----------------|-------|
| B W <br> W B | B W <br> W B | 0 | `W` `B` <br> `B` `W` | 4 |
| W W <br> W B | `B` W <br> W B | 1 | W `B` <br> `B` `W` | 3 |
| W B <br> B B | `B` `W` <br> `W` B | 3 | W B <br> B `W` | 1 |

이때 우리는 패턴을 이해할 수 있다.
- 블랙체스판을 만들기 위한 최소 비용과 화이트 체스판을 만들기 위한 최소 비용을 합치면 항상 이 체스판을 구성하는 전체 칸 수가 된다.
- 블랙 체스판을 하기 위해서 1칸만 바꿔야 했다면, 화이트 체스판을 위해서는 나머지 3개를 바꾸면 된다는 것이다.
- 따라서 우리는 두가지 경우를 각각 계산하는 것이 아니라, 화이트 체스판의 경우만 구하고, 64에서 해당 값을 뺀 값을 비교만 해주면 될 것 같다.

>1. 체스판 자르기 → 8*8크기로 자른다
>2. 현 체스판의 최소 비용 구하기 → 현재 체스판을 완성하기 위한 최소 count 구하기
>3. 전체 최적의 값과 비교하여 갱신하기 → 현재 체스판의 최소 count를 전체 체스판의 최소 count와 비교해서 최적의 값을 구하기.

```java
import java.util.Scanner;

class Main {
    public static int getSolution(int startRow, int startCol, String[] board) {
        String[] orgBoard = { "BWBWBWBW", "WBWBWBWB" };
        int whiteSol = 0;

        for (int i = 0; i < 8; i++) {
            int row = startRow + i;
            for (int j = 0; j < 8; j++) {
                int col = startCol + j;
                if (board[row].charAt(col) != orgBoard[row % 2].charAt(j)) whiteSol++;
            }
        }

        return Math.min(whiteSol, 64 - whiteSol);
    }

    public static void main(String[] args) {
        // 입력 받기
        Scanner sc = new Scanner(System.in);
        int row = sc.nextInt();
        int col = sc.nextInt();
        sc.nextLine();

        String[] board = new String[row];
        for (int i = 0; i < row; i++) board[i] = sc.nextLine();

        // 체스판 자르기
        int sol = Integer.MAX_VALUE;
        for (int i = 0; i <= row - 8; i++) {
            for (int j = 0; j <= col - 8; j++) {
                // 현재 체스판의 최소 비용 구하기
                int curSol = getSolution(i, j, board);
                // 최적의 값 갱신
                if (sol > curSol) sol = curSol;
            }
        }

        System.out.println(sol);
        sc.close();
    }
}
```
- 체스판의 첫 줄이 "BWBWBWBW", 둘째 줄이 "WBWBWBWB"인 정해진 패턴과 비교.
- 바꿔야 하는 칸 수(whiteSol)를 계산하고, 64 - whiteSol과 비교하여 최소 비용을 반환.

### 시간복잡도
- 전체 탐색에서 (N-7) × (M-7) 개의 8×8 체스판을 검사 → O(NM)
- 각 8×8 체스판을 비교하는 getSolution 함수의 시간 복잡도 → O(64) = O(1)
- 따라서 전체 시간 복잡도는 O(NM).