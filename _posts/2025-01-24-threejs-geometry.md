---
title: "Three.js로 3D 플랫폼 개발 Geometry"
layout: single
Typora-root-url: ../
categories: Three.js
tag: geometry
use_math: true
---

Three.js를 이용한 3D 플랫폼 개발 개념공부

## Geometry

삼각형은 가장 간단한 다각형이다. 3D 형태는 어떤 형태든 삼각형으로 분해할 수 있기 때문에, 삼각형을 사용하면 복잡한 형상을 쉽게 표현 가능.
- 점 3개만 있다면 면을 구성할 수 있기 때문이다.

다만 삼각형의 갯수에 따라서 많은 고민이 생기기 마련이다.
1. 삼각형을 많이 세팅하여 더 세부적으로 구현할 수 있겠지만 용량이 커지기 마련이다.
2. 삼각형을 적게 세팅하게 되면 형체 자체가 불확실해질 수 있다는 것이 리스크이다.

개발자는 이 사이에서 많은 고민을 할 수 밖에 없다고 한다.

blender나 maya라는 툴로 3D 모델링을 직접 제작도 가능하다. 아마 이래서 디자이너가 매우 필요한 분야 중 하나이지 않을까 싶다.

## R3F에서 `<mesh>`를 추가하는 방법

### `<mesh>`를 바탕으로 HTML처럼 추가하는 방식
지금까지 해온 방식이 이런 방식이 될 것 같다.

```jsx
<mesh 
    ref={boxRef}
    position={[0,0,0]}
    scale={[1,1,1]}
    rotation={[
        THREE.MathUtils.degToRad(0), 
        THREE.MathUtils.degToRad(0),
        THREE.MathUtils.degToRad(0)
    ]}
>
    <boxGeometry />
    <meshStandardMaterial color="red"/> 
</mesh>
```

### `<mesh>`를 scene.add를 통해 추가하는 방식
scene이라는 것은 이 공간에서 가장 최상단에 있는 컴포넌트이기 때문에, 여기에 mesh를 임의로 추가하는 방식도 가능하다.
```jsx
const geometry = new THREE.BoxGeometry(1,1,1);
const material = new THREE.MeshBasicMaterial({color: 0x00ff00});
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);
```

### `<mesh>` Property
mesh 요소의 프로퍼티를 geometry로 받아서 정의해도 가능하다. 근데 제일 불편해보이긴 해서 비추!
```jsx
<mesh geometry={new THREE.BoxGeometry(1,1,1)}>
    <meshStandardMaterial color={"blue"}/>
</mesh>
```

### Drei를 이용하면 가능해진다.

```jsx
import { Box, Sphere, Cone } from '@react-three/drei';

<Box position={[-2,0,0]}>
    <meshStandardMaterial color={"green"}/>
</Box>
<Sphere position={[-2,0,0]}>
    <meshStandardMaterial color={"green"}/>
</Sphere>
<Cone>
    <meshStandardMaterial color={"green"}/>
</Cone>
```
보다 편리하게 불러올 수 있다. 다양한 형태도 지원하는 걸 볼 수 있다.


## Wireframe

![]({{site.url}}/images/2025-01-24-threejs-geometry/wireframe.png)

Geometry의 형태가 모두 보인 상태로 보여지는 걸 볼 수 있다. 다만, 이 뼈대와 함께 색상도 집어넣어야 우리가 원하는 형태가 될 것이다. 

mesh를 두 개를 넣어야 하는 걸까? 그것보다는, 두 개의 `<mesh>`가 서로 속성과 데이터를 공유하거나 참조를 통해 연결된 상태를 만들 수 있어야 할 것 같다.

이때 활용 될 수 있는 것이 `useRef()` 훅을 사용하면 되겠다.

```jsx
useEffect(()=>{
    boxCopyRef.current.geometry = boxRef.current.geometry;
},[boxControl])
```
위 코드처럼 boxCopyRef가 boxRef의 geometry를 공유하게 설정되면, 두 `<mesh>`는 같은 기하학적 데이터를 사용하는 형태임.
- boxRef와 boxCopyRef라는 두 개의 ref가 각각 `<mesh>`에 연결됨. React에서는 ref를 사용해 DOM 요소나 Three.js 객체를 직접 참조하는데,
- 만약 boxCopyRef가 boxRef를 참조하거나, 두 ref가 동일한 데이터를 기반으로 동작한다면, 두 `<mesh>`는 연결된 상태라고 하면 된다.

boxControl이 저 변수자리에 들어가게 된 이유는, 

```jsx
import {useControls} from 'leva';

const boxControl = useControls({
    height: {value: 1, min: 0.1, max: 10, step:0.1},
    depth: {value: 1, min: 0.1, max: 10, step:0.1},
    widthSeg: {value: 1, min: 1, max: 10, step: 1},
    heightSeg: {value: 1, min: 1, max: 10, step: 1},
    depthSeg: {value: 1, min: 1, max: 10, step: 1}
})

return (
    <mesh 
        ref={boxRef}
        position={[0,0,0]}
    >
        <boxGeometry args={[
            boxControl.width, 
            boxControl.height, 
            boxControl.depth, 
            boxControl.widthSeg, 
            boxControl.heightSeg, 
            boxControl.depthSeg
            ]} />
        <meshStandardMaterial wireframe/> 
    </mesh>
)
```

`leva`를 통해서 boxControl을 바탕으로 boxGeometry의 속성을 조율할 수 있다는 것이다. UI를 바탕으로 조율하는 값들을 계속 지켜보고 리스닝하고 있어야 하기 때문에, 변수자리에 boxControl자체를 넣어서 변화를 확인하는 방식이라고 생각하면 된다.

![]({{site.url}}/images/2025-01-24-threejs-geometry/share.png)

**편-안**

## 이외의 Geometry

이외에도, 
- plane
- box
- circle
- sphere
- cylinder
- torusKnot
등과 같은 geometry등을 지원하고 자유롭게 사용할 수 있을 것 같다.