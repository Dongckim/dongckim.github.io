---
title: "99클럽 타임어택 보너스 문제풀이 [입국심사]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
# use_math: true
---

입국심사

## 문제 이해하기

여러 개의 심사대가 있습니다. 각 심사대에는 심사관이 있고, 이 심사관들은 한 사람을 심사하는 데 시간이 다르네?

- 처음에 모든 심사대는 비어 있고.
- 사람들이 줄을 서 있다가, 비어 있는 심사대로 가서 심사를 받는다
- 만약 모든 심사대가 바쁘면, 심사가 가장 빨리 끝나는 심사대를 기다렸다가 그곳으로 간다.

## 예시로 다시 이해해볼까

사람수? `n = 6`
심사관별 심사 시간 `times = [7, 10]`

### 진행과정
- 처음 상태

심사대 A: 7분 <br>
심사대 B: 10분<br>
→ 두 심사대는 비어 있으니, 첫 두 사람은 바로 각각 A와 B로 간다.

- 7분 후

A 심사대가 비어 있고, 3번째 사람이 A로 간다.
B 심사대는 여전히 10분이 걸리므로 아직 끝나지 않았겠죠.

- 10분 후

B 심사대가 비어 있고, 4번째 사람이 B로 간다.
A 심사대는 14분까지 작업 중.

- 14분 후

A 심사대가 비어 있고, 5번째 사람이 A로 가겠지.

- 20분 후

B 심사대가 비어 있지만 더 오래걸리기 때문에, 6번째 사람은 1분 더 기다렸다가 28분에 A로 가는게 이득.

- 결과

모든 사람이 심사를 끝내는 데 28분이 걸린다.

## 생각과정
보통 생각의 과정은 A에서 얼마~~~  그동안 B에서 얼마~~~ 이런식의 생각을 하기 마련인데, 이런 방식은 최적화 방식은 아니다.

> 잘 생각해보자 A에서 몇명을 처리하는 지는 중요하지 않다. B에서도 몇명을 처리하는지도 중요한 게 아니다.

그냥 `모든 사람이 심사를 받는데 걸리는 시간의 최솟값` 이 제일 중요한 것이다. 이것에만 초점을 잡고 진행하면 될 것이다.

→ 즉, 어떤 시간동안 최대 인원을 계속 비교해보면 될 것이다.

### 제한사항?

1,000,000,000이 등장한다. 완전탐색 쓰지 말라고 노골적으로 표현한 것으로 이해하고 넘어가면 되겠다.

> 그럼 뭐쓰지?

이분탐색, DP, 그리디, 백트래킹, DFS, BFS 등등 여러 개가 있을 것 같다.이 중에서 뭐가 알맞을까?

- 이진 탐색? 정렬된 범위 안에서 값을 찾을 때
- DP? 중복계산이 많을 때
- 그리디 알고리즘? 각 단계에서 최적의 선택이 가능할 때
- DFS/BFS? 그래프 탐색 or 경로
- Hashing, Sliding Window, Two Pointer? 배열이나, 문자열과 관련될 때 보통 떠올린다.

이 문제에 어울리는건, 이진 탐색 혹은 그리디가 될 꺼 같다는 느낌만 가져가면 될 것 같다.

### 왜 그리디가 안될까?
- 20분 후

B 심사대가 비어 있지만 더 오래걸리기 때문에, 6번째 사람은 1분 더 기다렸다가 28분에 A로 가는게 이득.

**이 예시를 바탕으로 바로 판단이 되어야겠다.**

>미래의 상황을 고려하지 않고, 현재 단계에서 가장 최적화 된 선택을 따르는 것(국소 최적화)이 Greedy이다. 그래서 전체 시간의 최소화를 보장하지 못하지 않을까?

를 떠올린 후 바로 이분탐색으로 넘어왔다면 BEST!

## 이분탐색으로 선택했어 어캐할래? 범위는?

최소 시간은 1분, 최대 시간은 `(가장 느린 심사관의 시간) × n`이겠죠?

이 사이의 중간 값 `mid`를 기준으로 모든 사람이 심사를 받을 수 있는 확인해봐야겠다. 모든 심사관의 심사 가능 인원을 더해서 `n`명 이상을 처리할 수 있다면, 시간을 줄이고, 아니라면 시간을 늘리는 방식.

이 과정을 반복하면 당연히 최소시간을 찾을 수 있을 것 같다.

## 예시코드

### 자바
```java
class Solution {

    private long result;

    public long solution(int n, int[] times) {
        long maxTime = (long)1000000000 * (long)1000000000;
        long minTime = 1;
        result = maxTime;
        search(times, n, minTime, maxTime);
        return result;
    }

    private void search(int[] times, int goal, long start, long end) {
        while(start <= end) {
            long mid = (start + end) / 2;

            long timeCnt = 0;
            for (int time : times) {
                timeCnt += (mid / time);
            }

            if (goal <= timeCnt) {
                result = Math.min(result, mid);
                end = mid - 1;
            } else {
                start = mid + 1;
            }
        }
    }
}
```

### 파이썬

```python
def solution(n, times):
    left, right = 1, max(times) * n 
    answer = right

    while left <= right:
        mid = (left + right) // 2
        total = sum(mid // time for time in times)

        if total >= n:
            answer = mid
            right = mid - 1
        else:
            left = mid + 1

    return answer
```