---
title: "Unity 2D 프로젝트 트러블슈팅 [Collider]"
layout: single
Typora-root-url: ../
categories: Unity
tag: Collider
# use_math: true
---

Unity에서 Collider를 처음 만들어보면서 공부한 내용을 기록해둔다.

## Collider란?

Collider는 Unity에서 물리적 충돌을 감지하는 컴포넌트이다. Mesh Renderer나 Sprite Renderer 같은 그래픽 요소와는 다르게, Collider는 오브젝트의 충돌 판정을 담당하는 보이지 않는 형태(히트박스) 를 정의함. 
- Collider 자체는 물리적인 충돌만 감지할 뿐이지, 실제로 오브젝트가 움직이거나 반응하려면 Rigidbody와 함께 사용해야 한다.

### 언제 Collider를 사용할까?

Collider는 주로 충돌 감지, 캐릭터 이동 제한, 피격 판정 등 다양한 용도로 사용됨.

1. 오브젝트 간 충돌 감지
- 캐릭터가 벽을 뚫고 지나가지 않도록 하기 위해
- 적이 플레이어에게 부딪혔을 때 데미지를 주기 위해

2. 트리거(Trigger) 이벤트 처리
- 특정 지역에 들어가면 이벤트 발생 (ex. 문이 열림, 보스전 시작 등)
- 아이템을 줍거나 목표 지점에 도달했을 때 감지

3. 물리적 상호작용
- Rigidbody와 함께 사용하여 중력, 충돌 반응 적용
- 공이 벽에 튕기는 효과 구현

4. 이동 제한
- 캐릭터가 특정 경계를 벗어나지 않도록 하기 위해
- 바닥 Collider를 두어 낙사를 방지


| **Collider 종류**       | **특징**                          | **사용 예시**                 |
|------------------------|--------------------------------|-----------------------------|
| **Box Collider**       | 박스 형태의 충돌 영역           | 벽, 상자, 바닥               |
| **Sphere Collider**    | 구형 충돌 영역                  | 공, 폭탄, 구체 형태의 캐릭터  |
| **Capsule Collider**   | 캡슐 형태의 충돌 영역            | 캐릭터 몸통, 기둥            |
| **Mesh Collider**      | 메쉬 형태 그대로 충돌 영역 생성   | 복잡한 지형, 건물 구조물      |
| **Polygon Collider 2D** | 2D 오브젝트에 사용, 다각형 형태   | 2D 캐릭터, 장애물            |
| **Circle Collider 2D**  | 2D 원형 충돌 영역               | 2D 공, 코인                 |
| **Edge Collider 2D**    | 선 형태의 충돌 영역              | 2D 벽, 플랫폼 가장자리       |
| **Terrain Collider**    | 지형(Terrain)과 충돌 영역 생성   | 오픈 월드 지형              |


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

