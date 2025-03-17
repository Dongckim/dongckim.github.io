---
title: "Unity 2D 프로젝트 트러블슈팅 [Game Manager와 Singleton패턴]"
layout: single
Typora-root-url: ../
categories: Unity
tag: GameManager
# use_math: true
---

Unity에서 Game Manager를 처음 만들어보면서 공부한 내용을 기록해둔다.

##  Game Manager란?

GameManager는 현재 내 게임의 상태를 관리하는 걸 의미한다. 여러 씬이나 게임 오브젝트 간에 공유되어야 하는 데이터를 유지하는 역할을 할 때 사용. 
- Singleton 패턴을 활용하여 구현된 클래스이다.
- 게임의 흐름을 조정하고, 점수 스테이지 관리, UI업데이트 생성 및 제거 등의 역할을 수행한다.

### Singleton 패턴이란?

상황에 맞게 사용될 수 있는 문제들을 해결하는데에 쓰이는 서술이나 템플릿인 **소프트웨어 디자인 패턴** 안에 있는 생성 패턴 중 하나이다.

생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴라는 방식을 의미한다.
- 해당 객체가 단 하나임을 보장한다는 의미

![]({{site.url}}/images/2025-02-15-unity-gamemanager/singleton.png){: .align-center}

#### PROS
- 객체를 생성하는 것은 비용이 드는 행위다. 기본적인 연산, 객체가 생성되는 힙 영역의 메모리, 그리고 소요 시간까지.
- 하지만 어플리케이션을 구성하는데 있어서 객체가 단 하나만 있어도 되는 경우는 꽤 많으며 객체가 싱글톤임을 보장한다면 위에서 말한 많은 비용을 줄일 수 있게 된다.
- 또한 싱글턴 객체는 전역에서 접근가능한(global) 객체이므로, 해당 객체에 상태(field)가 존재하면 이를 공유하기 매우 편해진다.

#### CONS
- 동시성 문제등을 고려할때, 공유 상태를 유지(stateful)하는 설계는 바람직하지 못하다. 즉 싱글톤객체를 설계할 때는 무상태(stateless)를 지향하여 설계해야만 한다.
- 여기서 무상태란
    - 싱글톤 객체는 특정 클라이언트에 의존적인 필드가 있으면 안된다.
    - 특정 클라이언트가 값을 변경할 수 있는 필드가 있어서는 안된다.
    - 가급적 읽기만 가능해야 한다.
- 만약 싱글톤 객체가 공유필드를 갖고있고, 이를 수정가능할경우 멀티쓰레드 환경에서 동시성 문제가 발생할 확률이 높다.
- 또한 일반적으로 기본 생성자를 private으로 닫아두게 되는데, 이러면 상속이 불가능해진다.

#### Game Manager 사례로 Singleton 구현
```c#
using UnityEngine;

public class GameManager : MonoBehaviour
{
    // 싱글톤 인스턴스
    public static GameManager Instance { get; private set; }

    // 게임 상태 변수 예시
    public int score = 0;
    
    private void Awake()
    {
        // 싱글톤 보장 (중복 인스턴스 제거)
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);  // 씬 변경 시에도 유지
        }
        else
        {
            Destroy(gameObject);  // 기존 인스턴스가 있으면 새로운 객체 삭제
        }
    }

    // 예제: 점수 추가 메서드
    public void AddScore(int points)
    {
        score += points;
        Debug.Log("현재 점수: " + score);
    }
}

```

- `public static GameManager Instance { get; private set; }`
    - Instance를 통해 어디서든 GameManager에 접근 가능하게 만듦.
    - private set;을 사용하여 외부에서 변경할 수 없도록 제한.
- `Awake()`에서 싱글톤 보장
    - `if (Instance == null)` → 처음 생성된 객체를 Instance로 설정.
    - `DontDestroyOnLoad(gameObject);` → 씬 변경 시에도 GameManager 유지.
    - `else { Destroy(gameObject); }` → 기존 Instance가 존재하면 새 객체 삭제.
- `AddScore(int points)` 메서드 추가
    - `GameManager.Instance.AddScore(10);` 같은 방식으로 쉽게 접근 가능.

## Game Manager 사용방식


