---
title: "99클럽 JAVA 코딩테스트 예시답안 5일차 [세로읽기]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
# use_math: true
---

세로 읽기

## 문제 안에서 특이사항 확인하기

대문자 ‘A’부터 ‘Z’, 영어 소문자 ‘a’부터 ‘z’, 숫자 ‘0’부터 ‘9’이다. 
- 다만, 이것들을 uppercase나 이런 식의 처리는 필요없어 보인다.

한 줄의 단어는 글자들을 빈칸 없이 연속으로 나열해서 최대 15개의 글자

예시와 같이 두 번째 줄의 다섯 번째 자리의 글자는 없다. 이런 경우처럼 세로로 읽을 때 해당 자리의 글자가 없으면, 읽지 않고 그 다음 글자를 계속 읽는다.

## 제한사항 꼭 확인하기

특이사항 없음.

## 입출력 비교하기

![]({{site.url}}/images/2025-01-17-java-dayfive/compare.png)

그냥 열마다 출력하고, 빈칸이라면 해당 순서를 건너뛰면 되지 않을까 싶다.
- 2차원 배열의 조작법에 익숙해져보면 좋아보인다.

## Psudo Code

char형 배열 arr[5][15]을 선언하고 charAt을 이용해서 한 자씩 배열에 넣고 for문으로 세로로 출력. 
- 이때, 열의 크기를 15로 선언했으므로 값이 없는 열은 arr[i][j] == 0로 null인지 체크해주고 출력해주면 베스트이다.

```java
import java.io.*;

public class Main {
    public static void main(String[] rgs) throws IOException {
        BufferedReader bfr = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bfw = new BufferedWriter(new OutputStreamWriter(System.out));

        char arr[][] = new char[5][15];

        for (int i = 0; i < 5; i++) {
            String str = bfr.readLine();
            for (int j = 0; j < str.length(); j++) {
                arr[i][j] = str.charAt(j);
            }
        }

        for (int j = 0; j < arr[0].length; j++) {	// 뽀인트
            for (int i = 0; i < 5; i++) {
                if (arr[i][j] == 0) {
                    continue;
                }
                bfw.write(String.valueOf(arr[i][j]));
            }
        }

        bfr.close();
        bfw.flush();
        bfw.close();
    }
}
```

## 조금더 심화?

무난한 문제였다.