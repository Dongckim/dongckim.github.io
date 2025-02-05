---
title: "99클럽 JAVA 코딩테스트 예시답안 13일차 [부등호]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
use_math: true
---

부등호

## 문제 안에서 특이사항 확인하기

0~9 사이의 숫자들을 차례로 대입해가며 주어진 부등호에 만족한다면 진행하고 만족하지 않는다면 다시 돌아가는 방식으로 풀이가 가능하다. 전형적인 "백트래킹"

## 제한사항 꼭 확인하기

한 번 사용했던 숫자들은 사용할 수 없기때문에 used라는 boolean형 배열을 사용해 사용된 숫자들을 확인한다.

## 입출력 비교하기

- 출력: 최종적으로 정답 배열을 정렬시키면 배열의 첫 번째 원소는 최솟값, 마지막 원소는 최댓값이 된다. 

### 생각과정

조건에 만족한다면 num이라는 String형 변수에 붙여 나아간다. → 재귀를 이용
- 이때, 조건에 부합하지 못해 재귀의 호출이 종료되면 사용된 숫자의 사용 여부(used[i])를 false로 바꾸어 다시 사용할 수 있도록 설정한다. 
- 만약 num의 길이가 (주어진 부등호 개수(N) + 1)이 된다면, 조건을 모두 만족시킨 문자열이 만들어진 것이므로 이를 정답 배열에 저장한다.

## Psudo Code

### 구현 구현 구현

## JAVA CODE

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Collections;
 
public class Q2529 {
    static int N;
    static boolean[] used;
    static String[] operator;
    static ArrayList<String> number = new ArrayList<>();
 
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        N = Integer.parseInt(br.readLine());
 
        operator = br.readLine().split(" ");
 
        for (int i = 0; i < 10; i++) {
            used = new boolean[10]; // 범위 0~9
            used[i] = true;
            rec_func(i, 0, i+"");
            used[i] = false;
        }
 
        // 정렬한 뒤, 첫 번째 원소 = 최솟값, 마지막 원소 = 최댓값
        Collections.sort(number);
        System.out.println(number.get(number.size()-1) + "\n" + number.get(0));
    }
 
    static void rec_func(int start, int count, String num) {
        // num의 길이가 주어진 N보다 1이 클 경우 숫자가 완성됐으므로 종료
        if (num.length() == N + 1) {
            number.add(num);
        } else {
            for (int i = 0; i < 10; i++) {
                // 사용되지 않은 숫자일 경우만 사용 가능
                if (!used[i]) {
                    if (operator[count].equals("<")) {
                        if (start < i) {
                            // 조건을 만족하면 사용했다는 것을 체크
                            used[i] = true;
                            rec_func(i, count + 1, num + i);
                            // 재귀 함수를 빠져나오면 사용 해제
                            used[i] = false;
                        }
                    } else {
                        if (start > i) {
                            used[i] = true;
                            rec_func(i, count + 1, num + i);
                            used[i] = false;
                        }
                    }
                }
            }
        }
    }
}
```


```python
num = int(input())
op = input().split()
check = [False] * 10
mx , mn = "",""
def poss(i,j,k): # 부등호 조건이  만족할 때만 작동
    if k == ">":
        return i>j
    else:
        return i<j


def recu(cnt, s):
    global mx,mn
    if cnt > num: #맨처음 나타나는 값이 최소, 마지막 저장되는 것이 최대
        if len(mn) == 0:
            mn = s
        else:
            mx = s
        return
    for i in range(10): #재귀 함수
        if check[i] == False:
            if cnt == 0 or poss(s[-1],str(i),op[cnt-1]):
                check[i] = True
                recu(cnt+1,s+str(i))
                check[i] = False

recu(0,"")
print(mx)
print(mn)
```