---
title: "99클럽 타임어택 보너스 문제풀이 [소수 찾기]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
use_math: true
---

저번 기수 문제 중 하나였다. 소수 찾기

## 문제 안에서 특이사항 확인하기

한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.

- 흩어진 종이조각을 붙인다는건, 순서 상관없이 조합할 수 있다는 뜻이겠다.
- 조합? 바로 순열이 떠오르면 완전탐색에 대해 익숙해지는 단계인 것 같다.


## 제한사항 꼭 확인하기

- numbers는 길이 1 이상 7 이하인 문자열입니다.
- numbers는 0~9까지 숫자만으로 이루어져 있습니다.

n개의 서로 다른 숫자로 만들 수 있는 순열의 개수는:

$ n! = n × (n−1) × (n−2) × ... × 1 $

7 이하의 문자열이 제한조건이기 때문에, `7!`으로 최대 5040개의 숫자를 만들어서 소수인지 판별하면 되겠다! → 완전탐색 되겠는데?

## 입출력 비교하기

**예제 #1**
- [1, 7]으로는 소수 [7, 17, 71]를 만들 수 있습니다.

7 17 71이 가능하겠다. (1인 자기 스스로이기 때문에 안됨.)

**예제 #2**
- [0, 1, 1]으로는 소수 [11, 101]를 만들 수 있습니다.

11과 011은 같은 숫자로 취급함.


1. 나올 수 있는 모든 조합을 만들어서
2. 순회하면서 소수인지 아닌지 확인해보면 되겠다!

끝

## Psudo Code

```python
from itertools import permutations

def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

def solution(numbers):
    number_set = set()
    for length in range(1, len(numbers) + 1):
        for perm in permutations(numbers, length):
            num = int("".join(perm))
            number_set.add(num) 

    prime_count = sum(1 for num in number_set if is_prime(num))
    
    return prime_count
```

## 요런 생각은 어때?

### 에라토스테네스의 체?

[https://namu.wiki/](https://namu.wiki/w/%EC%97%90%EB%9D%BC%ED%86%A0%EC%8A%A4%ED%85%8C%EB%84%A4%EC%8A%A4%EC%9D%98%20%EC%B2%B4)

학교에서는 Number Theory나 정수론 과목에서 배우는 걸로 알고 있는데, 이 방식을 사용한다면 더욱 더 단 시간으로 해결할 수 있을 것 같다.

1. 모든 숫자를 나열

    ![]({{site.url}}/images/2025-02-08-unity-bonus-daythree/step1.png)

2. 일단 소수도, 합성수도 아닌 유일한 자연수 1을 제거

    ![]({{site.url}}/images/2025-02-08-unity-bonus-daythree/step2.png)

3. 2를 제외한 2의 배수를 제거한다.

    ![]({{site.url}}/images/2025-02-08-unity-bonus-daythree/step3.png)

4. 3을 제외한 3의 배수를 제거한다.

    ![]({{site.url}}/images/2025-02-08-unity-bonus-daythree/step4.png)

5. 4의 배수는 지울 필요 없다.(2의 배수에서 이미 지워졌기 때문에) 2, 3 다음으로 남아있는 가장 작은 소수, 즉 5를 제외한 5의 배수를 제거

    ![]({{site.url}}/images/2025-02-08-unity-bonus-daythree/step5.png)

6. 이런 식으로 남은 숫자 중에서 가장 작은 소수를 찾고, 그 값을 제외한 그 값의 배수들을 점점 지워나가보면 소수들만 남게 된다.

    ![]({{site.url}}/images/2025-02-08-unity-bonus-daythree/step6.png)


일종의 노가다 방식이라 상당히 무식한 방법이긴 하지만, 특정 범위가 주어지고 그 범위 내의 모든 소수를 찾아야 하는 경우, 아직까지 소수들 간의 연관성(=소수를 생성할 수 있는 공식)이 나오지 않았으므로 에라토스테네스의 체보다 빠른 방법이 없다. 
- 다만 에라토스테네스의 체는 '특정 범위 내의 소수'를 판정하는 데에만 효율적이다. 만약 주어진 수 하나가 소수인가? 만을 따지는 상황이라면 이는 소수판정법이라 해서 에라토스테네스의 체 따위와는 비교도 안 되게 빠른 방법이 넘친다.

에라토스테네스의 체를 이용해 $ 1 $ ~ $ n $까지의 소수를 알고 싶다면, $ n $까지 모든 수의 배수를 다 나눠 볼 필요는 없다. 만약 $ n $ 보다 작은 어떤 수 $ m $이 $ m = a*b $ 라면 $ a $와 $ b $ 중 적어도 하나는 $ n $ 이하이다. 즉 $ n $ 보다 작은 합성수 $ m $은 $ n $보다 작은 수의 배수만 체크해도 전부 지워진다는 의미이므로, $ n $ 이하의 수의 배수만 지우면 된다. 

>  1~144인 위의 경우, 사실은 11의 배수 중 남아있는 121(11*11), 143(11*13)만 더 지우면 끝난다. 만일 표에 132(169)보다 큰 수가 있다면 13를 제외한 13의 배수를 지워야 하는데, 이 과정에서 최초로 지워지는 수는 169(132)이다. 그런데 이는 주어진 범위를 초과하는 수이다.

### 과정의 차이

1. 조합을 전부 생성.
2. 굳이 모든 숫자마다의 에라토스테네스의 체를 실행시킬 필요가 있을까? → 가장 큰 값만 뽑아서 에라토스테네스의 체를 한번만 실행해주면 그 안에 나머지들은 존재할 것이다. n 이하의 소수는 하나도 빼놓지 않고 모두 살아남는 방법이기 때문.
3. 그러니 조합 중 max값을 구해서 그 Max값에 대한 에라토스테네스의 체를 한번만 실행해서, 이 체에 남아있는 것과 조합을 비교만 해주면 

끝

### Psudo Code

```python
from itertools import permutations

def eratos(limit):
    is_prime = [True] * (limit + 1)
    is_prime[0], is_prime[1] = False, False

    for num in range(2, int(limit ** 0.5) + 1):
        if is_prime[num]: 
            for multiple in range(num * num, limit + 1, num):
                is_prime[multiple] = False

    return is_prime

def solution(numbers):
    number_set = set()
    
    for length in range(1, len(numbers) + 1):
        for perm in permutations(numbers, length):
            num = int("".join(perm))
            number_set.add(num)

    max_number = max(number_set)
    prime_eratos = eratos(max_number)
    prime_count = sum(1 for num in number_set if prime_eratos[num])
    
    return prime_count
```