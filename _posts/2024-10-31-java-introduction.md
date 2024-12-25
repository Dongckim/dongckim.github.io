---
title: "JAVA Coding Test Introduction"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: codingtest
use_math: true
---

## 백준


### 입출력
일반적으로 Scanner sc = new Scanner(System.in); 을 통해 입력을 받곤한다. 그런데 Scanner는 내부적으로 nextFloat() 이런걸 호출 시 다음 입력을 찾기 위해 정규식을 사용하므로 느리다. 이걸 쓰면 로직은 맞았는데도 시간초과나서 맞왜틀을 외치게될 수 있다. 

java,	java.util.Scanner	-> 6.068초

java,	java.io.BufferedReader	-> 0.934초

#### 입력은 하나만!
입력을 위한 클래스는 하나만 쓸 것! Scanner나 BufferedReader나 둘 다 미리 일정량을 읽어들인 후 사용자의 요청(readLine() 등)에 따라 해당 버퍼에서 꺼내온다. 따라서 이걸 여러개 선언해버리면 이미 다른 클래스에서 버퍼에 쌓인 부분때문에 제대로 읽을 수 없게된다.


#### BufferedReader (입력 TIP)

사용하려면 main클래스에 throws IOException을 추가해 주어야 한다.

```java
public static void main(String[] args) throws IOException
```

#### System.out.println() (출력 TIP)

![]({{site.url}}/images/2024-10-31-java-introduction/buffer.png)

```java
bw.writer();
bw.flush();
bw.close();
```

https://www.acmicpc.net/blog/view/57


느림. 쓰지마세요.(약 30배)
BufferedWriter로 출력해보자. (참고로 BufferedReader나 BufferedWriter 둘 다 깔끔하게 맨 끝에 close를 해주는게 좋긴하다.)

![]({{site.url}}/images/2024-10-31-java-introduction/1.png)
*java*

![]({{site.url}}/images/2024-10-31-java-introduction/.png)
*java*


### Main
클래스명은 'Main', 패키지는 없어야 한다. package없이 클래스명을 Main으로 두고 작성해야 한다.
![]({{site.url}}/images/2024-10-31-java-introduction/main.png)

### Other Class?
Main 이외의 클래스를 추가로 쓰고싶다면 public이 아닌 클래스 혹은 Inner 클래스를 쓰면 된다.
![](other.png)

### 바로 작성? or main args?

![]({{site.url}}/images/2024-10-31-java-introduction/dd.png)

 main문 자체가 static 함수이므로 거기서 사용하는 전역변수 및 모든 함수또한 static이어야 한다. 뭐 문제 푸는데 지장이 있는건 아니지만 그냥 static 안쓰면 동작이 안되니까 써둔걸로밖에 안보인다

vs

![]({{site.url}}/images/2024-10-31-java-introduction/args.png)

![]({{site.url}}/images/2024-10-31-java-introduction/example.png)

### 입출력 예시

```java
import java.io.*;                   	// JAVA IO import
import java.util.StringTokenizer;	// StringTokenizer import

public class Main {

    // Exception 처리
    public static void main(String[] args) throws Exception {
    
    	// BufferedReader 선언
    	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    	// BufferedWriter 선언
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        // 첫번째 라인 값 3 입력
        int len = Integer.parseInt(br.readLine());
        
        // 두번째, 세번째, 네번째 라인 값 split 사용, 공백 기준으로 입력 
        for(int i=0; i<len; i++) {
            // StringTokenizer를 이용한 공백 기준으로 자르기
            StringTokenizer st = new StringTokenizer(br.readLine(), " ");
            int a = Integer.parseInt(st.nextToken());
            int b = Integer.parseInt(st.nextToken());

            int sum = a + b;
            bw.write(sum+"\n");
        }
        br.close(); 
        
        // 최종 출력
        bw.flush();
        bw.close();
    }
}

```
백준의 문제들 중에서는 다음과 같이 한 줄에 여러 데이터가 입력받는 경우가 종종 있는데, 이때 split()보다 StringTokenizer 클래스를 이용해서 각 데이터를 분할하고 취득하는 것이 빠름.



### solved.ac
알고보니 원래 백준 온라인 저지에는 난이도가 없었지만, shiftpsh라는 분께서 난이도를 표시하는 서비스를 만들었다. ->  그게 이제 백준에로 통합되었다고.
우측 상단 설정에 들어가서 [solved.ac] 탭으로 이동하면 사용할 수 있다.

![]({{site.url}}/images/2024-10-31-java-introduction/baek.png)


그리고 [보기 설정]에서 solved.ac 티어 보기를 설정하면 끝

![]({{site.url}}/images/2024-10-31-java-introduction/tier.png)

티어가 아주 잘 보인다.

![]({{site.url}}/images/2024-10-31-java-introduction/result.png)


## Programmers
프로그래머스의 채점 기준은 정확성 테스트, 효율성 테스트 두 가지가 있습니다. 효율성 테스트가 있으면 정확성 테스트를 통과했더라도 시간 초과로 통과하지 못할 수 있습니다.

### 정확성 테스트란?

정확성 테스트는 제출한 코드 정답을 제대로 출력하는지 확인합니다. 각 테스트 케이스의 제한 시간을 10초로 넉넉하게 두고 정확성 여부만 테스트합니다.
#### 01단계 
제출한 코드를 기준으로 모든 테스트 케이스를 전부 수행합니다.
![]({{site.url}}/images/2024-10-31-java-introduction/test.png)

#### 02단계
각 테스트 케이스를 수행한 결과와 해당 문제에 대한 실제 정답을 비교하여 하나라도 다르면 오답으로 처리합니다.
![]({{site.url}}/images/2024-10-31-java-introduction/comp.png)

#### 03단계 
정확성 테스트는 정답이 맞으면서도 각 테스트 케이스 수행 시간이 10초 이내여야 합니다.
![]({{site.url}}/images/2024-10-31-java-introduction/loading.png)

### 효율성 테스트란?
효율성 테스트는 알고리즘의 성능을 확인합니다. 
10초라는 시간은 프로그램이 무한 루프에 빠질 가능성이 있는지 확인할 수 있는 수준의 시간이지 효율성을 논하기는 어려운 시간입니다. 예를 들어 효율성 테스트는 출제자가 의도한 문제의 시간 복잡도는 O(N)인데 여러분이 작성한 코드의 시간 복잡도가 O(N2)이면 오답 처리합니다. 
- 구체적으로 말하자면 효율성 테스트는 정답 코드를 기준으로 어느 정도 배수를 두고 시간 내에 코드가 수행되는지 체크합니다.


## 백준 & 프로그래머스 난이도 비교
![]({{site.url}}/images/2024-10-31-java-introduction/progbaek.png)
*참고만 하자*

<br/>

## 진짜 안알려주는

`String.valueOf()`와 `+""`는 둘 다 String 타입의 변환을 수행하는디.
- 전자는 가독성이 좋지만 여러 변환 과정이 있어서 속도가 느리고, `+""`는 가독성은 떨어지지만 속도가 좀 더 빠릅니다. 
- 따라서 코딩 테스트에서는 `+""`를 사용하는 것을 권장합니다.

- IDE의 디버그 기능 활용하기?


<br/>


```toc
```