---
title: "Three.js로 3D 플랫폼 개발 Interaction & Event"
layout: single
Typora-root-url: ../
categories: Three.js
tag: light
use_math: true
---

Three.js를 이용한 3D 플랫폼 개발 개념공부

## Interaction 

유저가 하는 행동들에 의해서 어떤 변화가 일어나는 것을 의미한다.
- onClick
- Mouse Hover
- Move
...

이런 것들을 어떤 방식으로 진행되는지 알아보자.

### mesh의 속성

r3f의 인터렉션으로 여러가지를 제공하는 걸 볼 수 있는데, 

```jsx
<mesh
    onClick={(e) => clickFunc(e)}
    onContextMenu={(e) => console.log('context menu')}
    onDoubleClick={(e) => console.log('double click')}
    onWheel={(e) => console.log('wheel spins')}
    onPointerUp={(e) => console.log('up')}
    onPointerDown={(e) => console.log('down')}
    onPointerOver={(e) => overFunc(e)}
    onPointerOut={(e) => outFunc(e)}
    onPointerEnter={(e) => console.log('enter')}
    onPointerLeave={(e) => console.log('leave')}
    onPointerMove={(e) => console.log('move')}
    onPointerMissed={() => console.log('missed')}
    onUpdate={(self) => console.log('props have been updated')}
    position={[-2,0,0]}
>
    <boxGeometry/>
    <meshStandardMaterial/>
</mesh>
```

이런 식으로 속성으로 확인해볼 수 있겠다.

각각의 속성 함수에는 또 따로 함수를 달 수 있는데, 예를 들면 이런식이 될 것 같다.

```jsx
function clickFunc(e:any){
    // e.stopPropagation();
    console.log("clickFunc e:", e)
    e.object.material.color = new THREE.Color('green');
}

<mesh
    onClick={(e) => clickFunc(e)}
    position={[0,0,0]}
>
    <boxGeometry/>
    <meshStandardMaterial/>
</mesh>
```

이런 식으로 `clickFunc` 이라는 함수를 다시 명명하여 사용하면서, mesh를 클릭하였을 때 원하는 작동 방식을 커스터마이즈할 수 있다는 것이다. 

특히 위의 예시에서는 e.object.material.color로, material의 색깔을 가져와 변경해주는 방식을 보여주고 있다.

```jsx
function overFunc(e:any){
    console.log("overFunc e:", e)
    e.object.scale.set(2,2,2);
}
```
이런 방식을 이용해서, scale도 조정해볼 수도 있는 걸 확인할 수 있다.

### Event Bubbling

이상한 현상을 볼 수 있다.

상자를 클릭했는데, 이와 함께 뒤쪽에 있는 mesh에도 이벤트가 전달되어지는 현상을 확인해볼 수 있다. 이는 프론트엔드 개발을 하다보면 많이 마주한 현상인데, event bubbling을 여기서도 확인할 수 있었다.

> 한 요소에 이벤트가 발생하면, 이 요소에 할당된 핸들러가 동작하고, 이어서 부모 요소의 핸들러가 동작. → 가장 최상단의 조상 요소를 만날 때까지 이 과정이 반복되면서 요소 각각에 할당된 핸들러가 동작하게 됨.

DOM element의 현상 중 하나이다.

### 오브젝트 클릭 이벤트 전달 막기

`e.stopPropagtaion()`를 이용하면 해당 현상을 막을 수 있다.

## Raycast

이떄 Raycast의 개념을 짚고 넘어갈 필요가 있다.

어떤 한 광선이 쭉 지나가면서 마주하는, 혹은 투과하는 것을 raycast라고 일컫는다.

![]({{site.url}}/images/2025-02-04-threejs-interaction/ray.png){: .img-width-seventy}

위 화면처럼 핑크색 광선이 지나가면서 투과하는 mesh를 카메라와 가까운 쪽부터 0, 1, 2번째로 지정하여 raycast가 진행이 된다. 

DOM에서 발생하는 Children을 클릭했을 때 Parent까지 이벤트가 쭉 전달되는것처럼, 가까운 쪽부터 먼 쪽까지 쭉 연결되는 걸 의미한다.

### eventObject VS object
- eventObject의 경우, 해당 event를 달아놓은 mesh라고 생각하면 될 것 같다. 
- event가 달려있지 않더라고 어떤 행위가 일어난 mesh가 있을 것이다. 그게 Object라고 생각하면 되겠다.

예를 들면, group에 onClick을 달았다고 생각해보자.
- eventObject는 group이고, object는 직접 행위가 작동된 Mesh가 될 것이다.

### Raycaster만들기
```Jsx
raycaster.setFromCamera(pointer, camera);
const intersect = raycaster.intersectObject(e.eventObject, true);
if(intersect.length > 0){
    const mesh = intersect[1].object as any;
    mesh.material.color = new THREE.Color('red')
}
```

- 내가 클릭하는 포인터에서 내가 보고있는 카메라를 에서의 레이케스터를 만든다.
- 이 광선에서 검사하는 어떤 오브젝트 그룹에 대한 가장 상위 parent를 넣으면 됨 
    - true → 모든 children을 다 검사하겠다는 뜻이 되겠다.
    - Scene을 넣으면, Scene 안에 모든 children을 검사하겠다는 뜻이다.

위 코드는 e.eventObject를 상위 parent로 지정하였기 때문에, event가 달려있는 mesh (ex. group)이 가장 상위 parent로 그 안에 intersect되는 모든 것을 확인해볼 수 있게 되는 것이다.
