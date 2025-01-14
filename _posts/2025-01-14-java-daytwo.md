---
title: "99클럽 JAVA 코딩테스트 예시답안 2일차 [그대로 출력하기2]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
# use_math: true
---
그대로 출력하기 2

쉬운 문제라도 명확하게 따져가면서 푸는 방법을 익혀야 한다.

## 문제 안에서 특이사항 확인하기

특이사항 없음.

이 문제를 통해서는 백준 문제를 어떤 식으로 입출력을 받을 수 있는지에 대해서 조금 많이 생각해보면 좋을 것 같다.

## 제한사항 꼭 확인하기

- 입력은 최대 100줄
- 알파벳 소문자, 대문자, 공백, 숫자
- 빈 줄이 주어질 수도 있고, 각 줄의 앞 뒤에 공백이 있을 수도 있다.

제한사항을 보면서 느낄 수 있는 건 느껴야겠다. 

## 입출력 비교하기

![]({{site.url}}/images/2025-01-14-java-daytwo/io.png)

그대로 나와야하는건 알겠지만, 조금 더 포인트를 짚어볼까?

- 띄어쓰기? 
- 탭?
- 한 줄 띄기?

어떻게 구현될까?

## Psudo Code
```java
import java.util.Scanner;

public class Main {
	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		while (sc.hasNextLine()) {
			String input = sc.nextLine();
			System.out.println(input);
		}
		sc.close();
	}
}
```

Scanner는 느립니다! 

### What if Python?
```python
while True:
    try:
        print(input())
    except EOFError:
        break
```
*SIBAL 이래서 JAVA가 불리하긴 하다.*

## 조금더 심화?
```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String line;

        while ((line = br.readLine()) != null) { // 입력이 null일 때 종료
            System.out.println(line); // 입력받은 줄을 그대로 출력
        }
    }
}
```

![]({{site.url}}/images/2025-01-14-java-daytwo/compare.png)

### 백준 뿐만 아니라 JAVA개발 안에서 입출력에 대한 고찰
일반적으로 Scanner sc = new Scanner(System.in); 을 통해 입력을 받곤 합니다. 
- 그런데 Scanner는 내부적으로 nextFloat() 이런걸 호출시에는 다음 입력을 찾기 위해 정규식을 사용하므로 느릴 수 밖에 없는 원리입니다. 

>*이걸 쓰면 로직은 맞았는데도 시간초과나서 맞왜틀을 외치게될 수 있다.*

- java, java.util.Scanner -> 6.068초
- java, java.io.BufferedReader -> 0.934초

### 입력은 하나만!
입력을 위한 클래스는 하나만 쓸 것!
- Scanner나 BufferedReader나 둘 다 미리 일정량을 읽어들인 후 사용자의 요청(readLine() 등)에 따라 해당 버퍼에서 꺼내옵니다. 
- 따라서 이걸 여러개 선언해버리면 이미 다른 클래스에서 버퍼에 쌓인 부분때문에 제대로 읽을 수 없게 됩니다.

### 근데 이게 뭘까요?
```java
public static void main(String[] args) throws IOException
```
하나하나 개념을 설명하면 저랑 밤새야됩니다.(술 사주면 ㄱㄴ)
- public과 private의 개념
- static과 dynamic 메서드의 개념
- void 개념
- argument에 대한 개념
- throw와 throws의 대한 개념
- IOException을 포함한 다양한 예외처리에 대한 개념

이를 한번 공부해보시면, 코테뿐만 아니라 앞으로 개발자로서 더욱 더 성장할 수 있지 않을까 생각이 있습니다.
> 궁금한거나 헷갈리는 것이 있다면 질문 주세요!

### 실행속도에 대한 고찰
[https://www.acmicpc.net/blog/view/57](https://www.acmicpc.net/blog/view/57)
```java
System.out.println()
```
얘는 느립니다. 쓰지마세요.(약 30배) BufferedWriter로 출력해보자. (참고로 BufferedReader나 BufferedWriter 둘 다 깔끔하게 맨 끝에 close를 해주는게 좋긴하다.)
![]({{site.url}}/images/2025-01-14-java-daytwo/table.png)