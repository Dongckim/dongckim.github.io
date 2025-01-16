---
title: "99클럽 JAVA 코딩테스트 예시답안 4일차 [뜨거운 붕어빵]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
# use_math: true
---
뜨거운 붕어빵

## 문제 안에서 특이사항 확인하기

*“안녕, 안녕, 안녕하십니까, 아저씨! 붕어빵 두 개 주세요.”*

*“안녕을 세 번 외쳤으니 붕어빵 세 개!”*

*붕어빵 두 개의 값을 내고 세 개를 받은 호돌이는 기분이 좋았어요.*

**이 내용은 전혀 쓸데없는 내용이라는 걸 문제를 읽으면서 바로 알고 넘어갔어야 했겠다.**

## 제한사항 꼭 확인하기

- N과 M(0≤N,M≤10)
- 둘째 줄부터 N개의 줄에 걸쳐 붕어빵의 모양이 주어짐.

## 입출력 비교하기

![]({{site.url}}/images/2025-01-16-java-dayfour/io.png)

## Psudo Code
```java
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int x = sc.nextInt();
		int y = sc.nextInt();

		for (int i = 0; i < x; i++) {
			while (sc.hasNext()) {
				StringBuilder sb = new StringBuilder(sc.next());
				System.out.println(sb.reverse());
			}
		}
		sc.close();
	}

}
```
with Stringbuilder

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class Main {

	public static void main(String[] args) throws IOException {
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

		String str[] = br.readLine().split(" ");
		int N = Integer.parseInt(str[0]);
		int M = Integer.parseInt(str[1]);

		for (int i = 0; i < N; i++) {
			String result = br.readLine();
			for (int j = 0; j < M; j++) {
				bw.write(result.charAt(M - j - 1));
			}
			bw.write("\n");
		}
		bw.flush();
		bw.close();
		br.close();
	}

}
```
하나하나 끊어서 재조합하는 방식이다.

## 조금더 심화?

StringBuilder.reverse()의 시간 복잡도는 **O(n)**입니다. 문자열의 각 문자에 대해 한 번씩 접근하여 위치를 교환하는 역할.
1. StringBuilder의 내부적으로 사용되는 char 배열 (value[])에 접근합니다.
2. 문자열의 양 끝에서 시작하여, 앞쪽 문자와 뒤쪽 문자를 교환합니다.
3. 중간 지점까지 반복하면서 모든 문자를 뒤집습니다.

그래서 `각 입력 문자열에 대해 StringBuilder 객체를 생성`하게 됨.
- 객체 생성 비용 + GC(가비지 컬렉션) 
- Scanner가 반복적으로 입력을 읽으면서 처리하므로, 병목현상이 발생

![]({{site.url}}/images/2025-01-16-java-dayfour/compare.png)