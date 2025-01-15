---
title: "99클럽 JAVA 코딩테스트 예시답안 3일차 [문자열 반복]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
# use_math: true
---
문자열 반복

## 문제 안에서 특이사항 확인하기

`S에는 QR Code "alphanumeric" 문자만` 이라는 조건을 잘 생각해보자 무슨 뜻일까?

- 숫자: 0-9
- 대문자: A-Z
- 9개의 특수 문자: $ % * + - . / :

즉, 이 문자외에 소문자 + @, #, & 이런 문자들은 제외되는 것으로 이해하면 되겠다
(toUpper, toLower이런거 안써도 되겠다.)

## 제한사항 꼭 확인하기

- 문자열 S가 공백으로 구분
- 개수 T(1 ≤ T ≤ 1,000)가 주어진다.
- S의 길이는 적어도 1이며, 20글자를 넘지 않는다.

그리 복잡하지도 않으며, 시간초과 걱정은 안해도 되겠다.
- **(참고: 20000정도의 문자열 길이를 이중for문으로 돌렸을 때 5초정도의 시간이 걸립니다.)**

## 입출력 비교하기

![]({{site.url}}/images/2025-01-15-java-daythree/io.png)

- 첫번째 입력은 이후 입력을 받을 "line" 의 갯수를 의미한다. ← for문 반복 횟수가 되면 되겠다! 

- 3'\t'ABC 이런식으로 구성되어있는데, 띄어쓰기 기준으로 앞에는 반복횟수, 뒤에는 반복 당할(?) 문자열들이 있다.← 띄어쓰기 기준으로 어떻게 나눌까? 에 대한 고민을 시작해보면 좋은 모먼트.

- ABC 각각을 또 순회하면서 각각을 반복횟수만큼 늘린뒤 각각을 다시 이어서 출력하면 된다.

대충 이정도를 떠올렸다면 떠올려야할 모먼트에서 모든걸 다 떠올린 것 같다. 이제 구현해볼까?

## Psudo Code
```java
import java.util.Scanner;

public class b_2675 {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int n = sc.nextInt(); // 입력 받을 줄의 갯수

        for(int i=0; i<n; i++){
            int num = sc.nextInt(); // 반복할 숫자
            String str = sc.next();

            for(int j=0; j<str.length(); j++){
                for(int k=0; k<num; k++){
                    System.out.print(str.charAt(j));
                }
            }
            System.out.println();
        }
    }
}
```
Scanner는 느리다고 했죠?

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;
import java.util.StringTokenizer;
 
public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb = new StringBuilder();
 
        int T = Integer.parseInt(br.readLine()); // 테스트 케이스의 개수
 
        for (int i = 0; i < T; i++) {
 
            StringTokenizer st = new StringTokenizer(br.readLine());  // 띄어쓰기를 기준으로 문자열을 분리함
 
            int R = Integer.parseInt(st.nextToken()); // 반복 횟수
            String S = st.nextToken(); // 반복할 문자
 
            for (int j = 0; j < S.length(); j++) {
                for (int k = 0; k < R; k++) {
                    sb.append(S.charAt(j));
                }
            }
            sb.append("\n");
        }
        System.out.println(sb);
        br.close();
    }
}
```
⃖![]({{site.url}}/images/2025-01-15-java-daythree/compare.png)

Stringbuilder를 활용한 println()이 제일 빠릅니다!

## 조금더 심화?

딱히 없는 것 같습니다.