---
title: "Unity Troubleshooting - infinite"
layout: single
Typora-root-url: ../
categories: [TroubleShooting, Unity]
tag: Unity
use_math: true
---

하루종일 삽질한 플랫폼 크랙문제

## 문제의 시작.

`Magazine` 스크립트 폴더를 작성하면서 생긴 문제였다. 가상 공간에서 만든 총기거치대와, 총기 Grab과 관련된 상호작용을 건드리던 도중 발생하였다.

`Magazine` 스크립트는 총기를 놓게 되면, 스스로 총기 거치대로 돌아가고, 거치대에 돌아간 총은 전부 재장전되는 목적을 가지고 있었다. 

```c#
private int currentBullets;

private int currentBullets
{
    get => currentBullets;
    set
    {
        if(value < 0)
            currentBullets = 0;
        else if (value > maxBullets)
            currentBullets = maxBullets;
        else    
            currentBullets = value;

        OnBulletsChanged?.Invoke(currentBullets);
        OnChargeChanged?.Invoke((float)currentBullets / maxBullets);
    }
}
```
이 코드의 목적이 중요한 건 아니기 때문에 넘어가보자.

문제는, Unity Event를 활용해 코드를 짰기 때문에, Inspector에서 이벤트에 메서드를 연결하고, 코드로 이벤트 리스너를 등록한 후에 Game View를 play해보니 Unity 자체가 꺼지기 시작했다.

**SIBAL**

## 시도한 방법.

### 공식문서

[Unity 공식문서](https://docs.unity3d.com/kr/2022.3/Manual/TroubleShootingEditor.html) 를 참고하는 것을 우선으로 생각하기 때문에, 공식문서를 살펴보았다.

![]({{site.url}}/images/2025-01-24-infinite-trouble/docu.png)

재설치요??

하... 이건 진짜 마지막의 수단으로 가져가야 할 것 같았다. 그리고 다시 한번 느끼지만, 굉장히 윈도우 친화적인 설명들이 많다는 것을 다시 깨닫게 되었다. ~~하..윈도우 사고 싶다.~~

### 먼가 용량이 부족한 것인가..?

그래서 Library와 Temp 파일을 지우기 시작했다. 일단 뭔가 과부하가 걸린 거 같았다... 근데 뭔지는 알고 지워야겠지..?

- Library 폴더
Unity 에디터가 프로젝트를 관리하고 실행하기 위해 필요한 캐시 데이터를 저장한다고 하네요.
    + 텍스처를 압축하거나, 모델 데이터를 Unity가 읽을 수 있는 형식으로 변환
    + Unity가 컴파일한 C# 스크립트의 바이너리 데이터가 저장 + 프로젝트 실행 시 이 데이터를 사용해 빠르게 동작

- Temp 폴더
Unity 에디터가 실행 중에만 사용하는 임시 데이터를 저장하는 폴더
    + Unity 실행 중에도 필요하지 않은 데이터는 자동으로 삭제되는 경우가 있다고 한다.

즉, Library 폴더는 삭제되어도 다시 실행되면 알아서 다시 생성되는 폴더이고, Temp폴더는 말그대로 임시 폴더이기 때문에, 시간이 좀 지났기 때문에 많은 임시 데이터가 생겼구나 하고 삭제해도 괜찮다고 판단했다.

하지만 문제는 여전했다.

### Unity 크래쉬 버그 구글링

연관 검색어에 Unity Crash Bug 연관검색어가 있길래 일단 찾아보았었다. 허나, 같은 비정상적인 종료는 맞지만, 네트워크나 PC 사양과 같은 원인을 주로 나타나는 것을 확인하였다.

### 여기서 단서의 실마리 발견.

크래시 버그가 PC사양과 같은 하드웨어적인 문제가 아니라면, 오직, Magazine의 파일을 생성하면서 생긴 문제라고 문제의 요점을 파악하게 되었고,

그럼 코드에서 뭔가가 에러가 난 걸까? 라고 생각하기엔, Console에서 에러창이 1도 표시되지 않았다.

![]({{site.url}}/images/2025-01-24-infinite-trouble/chill.jpg){: .img-width-half}

**진짜 뭐지?**

나는 Chill Chill 맞은 Chill Guy니까 다시 한번 돌아보자.

```c#
private int currentBullets;

private int currentBullets{
    ...
}
```

뭔가 이상하지 않냐. 이쯤에서 Chill Guy 브금이 나와야 할 꺼 같다.

## 해결

currentBullets 프로퍼티와 필드 이름이 동일하기 때문에 무한 재귀 호출이 발생하면서 튕기면서 종료되는 거 였다.

get이나 set에서 currentBullets를 직접 참조하면 프로퍼티의 get이나 set이 다시 호출되고, 이 과정이 반복되어 스택 오버플로우가 발생하면서 유니티 프로그램이 강제로 종료되는 방식이었던 것이었다.


![]({{site.url}}/images/2025-01-24-infinite-trouble/gg.jpg){: .img-width-seventy} <br>
새벽 2시 16분 작렬히 전사...