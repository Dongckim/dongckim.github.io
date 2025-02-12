---
title: "Unity 2D 프로젝트 트러블슈팅 [Prefabs]"
layout: single
Typora-root-url: ../
categories: Unity
tag: Prefabs
# use_math: true
---

Unity에서 Prefab을 처음 만들어보면서 공부한 내용을 기록해둔다.

## Prefab이란?

Prefab이란, 게임 오브젝트의 템플릿이나 청사진 역할을 하는 개념이다. 즉, 자주 사용하는 오브젝트를 미리 설정해두고, 이를 여러 번 재사용할 수 있도록 해주는 역할이라고 생각하면 될 것 같다.

### 언제 Prefab을 사용할까?

1. 동일한 구조를 유지해야하는 오브젝트가 있을 때, 한 번 설정해 둔 구조를 일관되게 유지 가능하게 할 수 있다.

2. 여러 개의 오브젝트를 동시에 수정해야 할 때, Prefab을 수정하면 해당 Prefab을 사용하는 모든 오브젝트가 자동으로 업데이트된다.

3. 동적으로 오브젝트를 생성해야 할 때, Instantiate()로 Prefab을 쉽게 생성 가능해진다.

4. 성능 최적화를 위해 오브젝트 풀링을 사용할 때 사용한다고 한다. 예를 들어, 총알이나 파티클을 계속 생성하고 제거하는 대신, 미리 생성해 둔 오브젝트를 재사용할 수 있는 방식이겠다.

## Prefab을 사용해야겠다고 결정한 부분

![]({{site.url}}/images/2025-02-08-unity-prefab/prefab.png)

- 캐릭터가 계속 달려가면서 빌딩과 배경이 움직여야하는데, 말그대로 진짜 캐릭터가 움직이는 것이 아니라 배경과 빌딩이 캐릭터가 움직이는 반대 방향으로 움직이는 것이라고 보면 된다.

- 이때 캐릭터가 피해야 할 장애물도 캐릭터가 뛰는 방향의 반대쪽에서 나타나야 하는데, 이 모든 것들이 거의 비슷한 메커니즘으로 움직이는 것을 알 수 있다. 세부적으로는 다를지 몰라도, 움직임 하나만큼은 비슷하기 때문에 Prefab으로 묶어도 될 듯하다.

- 추가로, 장애물의 경우는 `spawner`을 활용해서 장애물을 랜덤 시간 간격을 기준으로 생성해내는데, 이때 생성되는 각각의 장애물도 다 동일하게 움직이는 속성을 가져야하기 때문에 Prefab으로 지정해두면 이점이 많아 보인다.

### 저 빌딩을 예시로 스크립트 먼저 짜볼까?

화면의 오른쪽에서 Spawn해서 왼쪽으로 이동한 후 (1), 왼쪽끝에 닿았을 때 destroy되는(2) 총 2가지 스크립트가 필요할 것 같다.

### Mover 

```c#
using UnityEngine;

public class Mover : MonoBehaviour
{
    public float moveSpeed;
    void Start()
    {

    }
    void Update()
    {
        transform.position += Vector3.left * moveSpeed * Time.deltaTime;
    }
}
```

간단할 것 같다. transform property에 바로 access해서 해당 위치값을 초당 변화값으로 주어지면 될 것 같다.


### Destroyer

```c#
using UnityEngine;

public class Destroyer : MonoBehaviour
{
    void Start()
    {

    }
    void Update()
    {
        if(transform.position.x < -15)    // -15는 scene 마지막 부분을 의미한다.
        {
            Destroy(gameObject);
        }
    }
}
```

## 이제 두 스크립트를 넣어볼까?

![]({{site.url}}/images/2025-02-08-unity-prefab/script.png){: .img-width-half .align-center}

컴포넌트에 두 스크립트를 삽입해서 작성하면 될 것 같다. 이제 이 컴포넌트를 Prefab으로 돌려서 sprite형태만 변화시키면 다양한 형태의 빌딩을 구현해볼 수 있겠다.

![]({{site.url}}/images/2025-02-08-unity-prefab/prefablist.png)

- 모든 빌딩에는 동일한 스크립트(mover와 destroyer)이 포함된 상태로 움직일 것이다.

## Spawner

```c#
using UnityEngine;

public class Spawner : MonoBehaviour
{
    [Header("Settings")]
    public float minSpawnDelay;

    public float maxSpawnDelay;

    [Header("Reference")]

    public GameObject[] gameObjects;

    void Start()
    {
        Invoke("Spawn", Random.Range(minSpawnDelay, maxSpawnDelay));
    }
    void Spawn(){
        GameObject randomObject = gameObjects[Random.Range(0, Range(0, gameObjects.Length))];
        Instantiate(randomObject, transform.position, Quaternion.identity);
        Invoke("Spawn", Random.Range(minSpawnDelay, maxSpawnDelay));
    }
}

```
- 아래에서의 사진처럼 gameobject를 자유롭게 추가하는 칸을 만들 수 있게 된다. minimum과 maximum을 정해서 그 사이 Range아네서 random으로 배열 안의 object를 마구 생성할 수 있게 된 것이다.


![]({{site.url}}/images/2025-02-08-unity-prefab/spawner.png)

## 아이템들도?

아이템도 동일하지만, collider가 추가되어야 할 것 같다. 이 이야기는 다음 포스팅에서 이어서 해보자.