---
title: "99클럽 JAVA 코딩테스트 예시답안 1일차 [문자열 내 P와 Y의 개수]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
# use_math: true
---
문자열 내 p와 y의 개수

## 문제 안에서 특이사항 확인하기
1. 'p'와 'y' 모두 하나도 없는 경우는 항상 `True`를 리턴한다.
2. 단, 개수를 셀 때는 `대문자`와 `소문자` 따로 셀 필요는 없음. (p = P)

## 제한사항 꼭 확인하기
- 문자열 s의 길이: 50이하의 자연수.
- 문자열 s는 알파벳으로만 이루어져 있다.

제한사항을 보면서 느낄 수 있는 건 느껴야겠다. 
1. 문자열 내에서 내가 검열해야 할 것은 없겠구나.
2. 문자열 길이가 생각보단 길지는 않겠구나. (요정도면 완전 탐색 정도는 가능할 수 있겠구나까지 떠올렸다면 best)

## 입출력 비교하기

| s         | answer    |         |
|-----------|-----------|--------------|
| "pPoooyY" | true      | → p의 개수 = 2, y의 개수 2
| "Pyy"     | false     | → p의 개수 = 1, y의 개수 2


## Psudo Code
```java
//string으로 s을 입력받아 = s로 지정할꺼임
class Solution {
     boolean solution(String s) {
// 1. 이 s는 무조건 갯수 세기 편해야 하기 때문에, 
// 안에 내용물이 모두 대문자이거나, 모두 소문자로 변형시켜주면 갯수 새기 편하겠다.

    s = s.toUpperCase();

// 각 문자열을 순회하면서 P이거나 y이면 각각 count +=1
    int p = 0;
    int y = 0;

    for (int i = 0; i < s.length(); i++){
        if(s.charAt(i) == 'p'){
            p++
        }else if(s.charAt(i) == 'y'){
            y++
        }
    }

// 순회가 끝난 시점에서 갯수 비교하면 되겠다.

    if(p != y) {
        answer = false;
    }
     }
}
```

## 조금더 심화?
```java
p = (int) s.chars().filter(c -> c == 'p').count();
y = (int) s.chars().filter(c -> c == 'y').count();
```
filter가 뭔지 한번 공부해볼까요?

공부한다면 알겠지만, 매우 비효율적인 코드이다. 왜? 순회를 p따로 y따로 2번 도는 코드이다. 그렇다면 한번에 돌게 할 순 없을까?

```java
s.chars().filter( e -> 'P'== e).count() == s.chars().filter( e -> 'Y'== e).count();
```
오 이러면 filter 한번으로 끝난 걸까요?
아닙니다. 이것또한 마찬가지로 두번 순회를 하게 됩니다.

## GPT한테 물어볼까요?
```java
AtomicInteger pCount = new AtomicInteger(0);
AtomicInteger yCount = new AtomicInteger(0);

s.chars().filter(c -> {
    if (c == 'p' || c == 'P') {
        pCount.incrementAndGet();
    } else if (c == 'y' || c == 'Y') {
        yCount.incrementAndGet();
    }
    return false; // 실제 필터링은 하지 않고, 카운팅만 진행
}).count();

boolean result = pCount.get() == yCount.get();
```
이 코드를 이해할 필요는 없습니다. 그냥 filter안에 if 문으로 나눠서 진행하는 형태를 볼 수 있습니다.

즉, psudo code에서의 코드가 가장 직관적이며 효율적인 방법임을 깨닫게 된다면 제가 의도한 바가 전달된 것 같습니다. (튜닝의 끝은 순정입니다.)

가끔보면 이런 간지나는 코드들이 있는데, 부러워하지마세요 비효율적입니다.
```java
class Solution {
    boolean solution(String s) {
        s = s.toUpperCase();
        return s.chars().filter( e -> 'P'== e).count() == s.chars().filter( e -> 'Y'== e).count();
    }
}
```