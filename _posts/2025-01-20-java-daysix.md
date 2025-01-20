---
title: "99클럽 JAVA 코딩테스트 예시답안 6일차 [할리갈리]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
# use_math: true
---

할리갈리

## 문제 안에서 특이사항 확인하기

그냥 할리갈리 잘할 수 있게 해주세요?

## 제한사항 꼭 확인하기

`STRAWBERRY, BANANA, LIME, PLUM` 제발 스펠링 틀리지 말자.

- 1 < N < 1,000,000
- 1 < X < 5

## 입출력 비교하기

![]({{site.url}}/images/2025-01-20-java-daysix/io.png)

각 과일의 갯수를 세다가 5개가 되면 `YES`출력 아니면 `NO` 출력이겠다.

## Psudo Code

코드를 보기 전에 생각을 해보자.

다들 해시로 문제를 풀었다는 건 알겠다. 왜 해시지? 해시가 왜 더 빠르지? 이 생각을 하면서 풀었다면 조금 더 건강한 방식이지 않을까?

![]({{site.url}}/images/2025-01-20-java-daysix/compare.png)
- 위에서부터, 5개의 Arraylist방식, 과일 이름에 맞는 배열 인덱스, 해시 방식이다. 당연히 해시가 더 효율적인 방법인지는 알겠으나, 얼마나 더 빠른지 알고 있다면 좋을 것 같아서 가져왔다.

### 5개의 Arraylist
```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // 카드의 개수 N
        int N = sc.nextInt();
        sc.nextLine();  // 개수 입력 후 개행문자 처리

        // 각 과일에 해당하는 배열을 선언
        List<Integer> strawberry = new ArrayList<>();
        List<Integer> banana = new ArrayList<>();
        List<Integer> lime = new ArrayList<>();
        List<Integer> plum = new ArrayList<>();

        // 카드 정보 입력받고 과일 개수 합산
        for (int i = 0; i < N; i++) {
            String[] card = sc.nextLine().split(" ");
            String fruit = card[0];
            int count = Integer.parseInt(card[1]);

            // 각 과일 이름에 해당하는 배열에 개수만큼 추가
            switch (fruit) {
                case "STRAWBERRY":
                    for (int j = 0; j < count; j++) {
                        strawberry.add(1);
                    }
                    break;
                case "BANANA":
                    for (int j = 0; j < count; j++) {
                        banana.add(1);
                    }
                    break;
                case "LIME":
                    for (int j = 0; j < count; j++) {
                        lime.add(1);
                    }
                    break;
                case "PLUM":
                    for (int j = 0; j < count; j++) {
                        plum.add(1);
                    }
                    break;
            }
        }

        // 각 과일 배열의 길이를 확인하고, 하나라도 5개인 과일이 있으면 YES
        if (strawberry.size() == 5 || banana.size() == 5 || lime.size() == 5 || plum.size() == 5) {
            System.out.println("YES");
        } else {
            System.out.println("NO");
        }
    }
}

```

### 과일 이름별로 인덱스 만들어 계산
```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // 카드의 개수 N
        int N = sc.nextInt();
        sc.nextLine();  // 개수 입력 후 개행문자 처리

        // 과일 이름에 맞는 배열 인덱스
        String[] fruits = {"STRAWBERRY", "BANANA", "LIME", "PLUM"};
        int[] fruitCount = new int[4];  // 각 과일의 개수를 셀 배열

        // 카드 정보 입력받고 과일 개수 합산
        for (int i = 0; i < N; i++) {
            String[] card = sc.nextLine().split(" ");
            String fruit = card[0];
            int count = Integer.parseInt(card[1]);

            // 과일 이름에 맞는 인덱스를 찾아서 개수 합산
            for (int j = 0; j < 4; j++) {
                if (fruits[j].equals(fruit)) {
                    fruitCount[j] += count;
                    break;
                }
            }
        }

        // 과일 개수 중 하나라도 정확히 5개인 과일이 있는지 확인
        for (int count : fruitCount) {
            if (count == 5) {
                System.out.println("YES");
                return;
            }
        }

        // 5개인 과일이 없으면 NO 출력
        System.out.println("NO");
    }
}
```
### 해시
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;

// 과일이 5개 있는 경우에만 YES 아니면 모두 NO
// MAP을 이용해 전체 갯수를 담은 후 5개 있으면 YES / 아니면 NO
public class Main {

  public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    int count = Integer.parseInt(br.readLine());
    HashMap<String, Integer> fruit = new HashMap<>();

    for (int i = 0; i < count; i++) {
      String[] split = br.readLine().split(" ");
      int existValue = fruit.getOrDefault(split[0], 0);
      fruit.put(split[0], existValue + Integer.parseInt(split[1]));
    }

    if (fruit.containsValue(5)) {
      System.out.println("YES");
    } else {
      System.out.println("NO");
    }
  }
}
```

## 조금더 심화?

1. 시간복잡도의 차이
- Map을 사용하면 각 과일에 대해 O(1)의 시간 복잡도로 값을 업데이트하고 확인할 수 있음. 즉, Map은 해시 테이블을 사용하여 빠르게 값을 조회하고 업데이트.
- 배열을 사용하는 경우, count만큼 요소를 추가하는 데 시간이 걸리므로 최악의 경우 O(count) 시간이 걸림. 예를 들어, count가 5일 때는 O(5)가 걸리며, count가 1일 때는 O(1)이지만, 평균적으로 배열에 많은 요소를 추가하는 경우 성능이 저하발생

2. `size()`호출의 비효율성
- Map을 사용할 경우 과일 종류별로 개수를 한 번에 저장하고, 그 값을 바로 확인할 수 있기 때문에 size()를 호출할 필요.
- 각 배열의 size()를 확인하여 그 길이가 5인지 비교하는 방식은 배열의 크기가 커질수록 시간이 더 많이 걸리기 마련이다. 특히, 배열에 많은 요소가 추가될 경우 size()를 계산하는 비용이 매우 방대해짐.