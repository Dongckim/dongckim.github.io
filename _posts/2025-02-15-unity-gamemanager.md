---
title: "Unity 2D 프로젝트 트러블슈팅 [Game Manager]"
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

### Game Manager 사용방식




###  isTrigger 옵션

Collider에는 `isTrigger` 옵션이 있는데, 이걸 켜면 실제 충돌은 발생하지 않고 오직 충돌 이벤트만 감지할 수 있다.

- `isTrigger`가 꺼져 있다면(default), 물리적으로 충돌이 발생할 때, 물체가 튕기거나 멈춘다.
- `isTrigger`가 켜져 있다면, 물리적 충돌 없이 겹쳐지지만 충돌 이벤트는 감지 가능하다.


## Collider을 사용해야겠다고 결정한 부분

아이템이 오른쪽에서 왼쪽으로 넘어오면서, 캐릭터와 맞닿으면 생명이 줄던가, 캐릭터가 먹었기 때문에 바로 사라지는 등의 현상을 구현하고 싶었다.
- isTrigger의 기능처럼 물리적 충돌 자체는 일어나지 않더라도 충돌 이벤트는 감지해야하는 상황인 것이다.

![]({{site.url}}/images/2025-02-12-unity-collider/collider.png)

Capsule Collider 2D를 생성해서 enemy로 지정되는 컴포넌트에 add시켜주면 될꺼 같다. Edit Collider를 통해 collider의 크기를 조절할 수 있다.
- 당연 isTrigger도 켜서 이벤트를 감지하도록 해야하겠다.

### Player.cs 스크립트 설정

```c#
using UnityEngine

public class Player : MonoBehaviour
{
    void OnCollisionEnter2D(Collision2D collision)
    {
        if(collision.gameObject.name == "Platform")
        {
            if(!isGrounded)
            {
                PlayerAnimator.SetInteger("state", 2);
            }
            isGrounded = true;
        }
    }

    void OnTriggerEnter2D(Collider2D collider)
    {
        if(collider.gameObject.tag == "enemy")
        {
            Hit();
        }
        else if(collider.gameObject.tag == "food")
        {
            Heal();
        }
        else if(collider.gameObject.tag == "golden")
        {
            StartInvinvible();
        }
    }
}
```

- PlayerAnimator의 SetInteger를 통해서 플레이어가 collision을 platform( = 땅 )과 하고 있다면, animation의 지정한 state로 유지하겠금 만들 수 있겠다.
- collider.gameObject를 통해 object의 정해진 tag에 따라서 원하는 행동을 하도록 지시할 수 있다.

