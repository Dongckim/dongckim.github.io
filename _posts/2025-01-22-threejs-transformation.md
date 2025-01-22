---
title: "Three.js로 3D 플랫폼 개발 Transformation"
layout: single
Typora-root-url: ../
categories: Three.js
tag: [Object3D, Transform]
use_math: true
---

Three.js를 이용한 3D 플랫폼 개발 개념공부

## Transformation

Transformation에는 `Position`, `Scale`, `Rotation` 이 있다.

**움직임을 표현하는 각도 단위는 라디안임을 꼭 알고 있어야 한다.** → 그래서 `THREE.Mathutils.degToRad()`를 꼭 작성해서 적용해주어야 한다.

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
앞에서부터 x, y, z 요소가 되겠다.

- `<mesh>` 요소 안에 property로 작성하면, 이는 default값으로 적용이 된다.
- useframe 안에 적용되면, 이는 애니메이션 효과처럼 적용된다고 생각하면 된다.

```jsx
const boxRef = useRef<THREE.Mesh>(null);
const groupRef = useRef<THREE.Mesh>(null);

useFrame((state, delta) => {
    boxRef.current.rotation.x += delta;
    boxRef.current.position.x += 0.01;
    boxRef.current.scale.x += 0.01;
})
```

**useRef 훅을 이용해서 Three.js의 Mesh 객체를 참조하고, useFrame 훅을 통해 애니메이션 프레임마다 Mesh의 회전, 위치, 크기를 동적으로 업데이트가 가능해질 수 있는 것이다.**

## Object3D(Scene,Group,Mesh)
- Scene
- Group
- Mesh
    - Geometry
    - Material

전체적인 구조는 이렇다.
![]({{site.url}}/images/2025-01-22-threejs-transformation/image.png)

사진에서 보듯이, Scene은 가장 큰 개념으로 `Group`과`Mesh` 모두의 Transformation을 변형시킬 수 있는 권한이 있다.
- 마찬가지로 `Group`은 속해 있는 `Mesh` 모두의 Transformation을 변형시킬 수 있는 권한이 있다.

```jsx
const { size, gl, scene, camera } = useThree();
const groupRef = useRef<THREE.Mesh>(null);

useFrame((state, delta) => {
    scene.rotation.x += 0.01; // Scene
    groupRef.current.rotation.x += delta; // Group 
})

scene.rotation.x = THREE.MathUtils.degToRad(45) // Scene
```

### Geometry & Material?
애네는 상속의 개념이 아닌, 구성 요소라고 보면 되겠다.
구성요소는 즉, 같은 렌더러(Renderer)의 개념이 아닌, Mesh가 구성되는 최소조건 정도로 보면 되겠다.(완벽한 정의가 아님 주의!)

**즉, Position, Rotation, Scale과 관련 없는 속성들이다 라는 것을 알고 있어야 하는 것이다.**

## World & Local좌표계

Scene이 곧 World 좌표계라고 생각하면 된다. Threejs에서 가장 큰 좌표계이기 때문임.
- 그 외에 그룹이나, 메쉬는 다 local 좌표계라고 생각하면 된다.

`<axisHelper>`를 바탕으로 그 차이를 확인해보자.
- 원래는 `App.tsx`에 작성되어 있는 `<axisHelper>`를 가장 최상단이 아닌, Mesh나 Group에도 적용이 될 수 있다.
- 아래 코드를 통해 확인해보자.

```jsx
//App.tsx

function App() {

  return(
    <>
        <Canvas
          camera={{
            near: 1,
            far: 100,
            fov: 70,
            position: [5, 5, 5]
          }}
        >
          <color attach="background" args={["white"]}/>
          <OrbitControls/>
          <axesHelper args={[6]}/>  // ← 얘
          <gridHelper args={[10, 10]}/>
          <ThreeElement/>
        </Canvas>
    </>
  )
}

export default App
```

```jsx
// ThreeElement.tsx

return(
    <>
        <directionalLight position={[5,5,5]}/>
        <group
            ref={groupRef}
            position={[0,0,3]}
            rotation={[
                THREE.MathUtils.degToRad(0), 
                THREE.MathUtils.degToRad(0),
                THREE.MathUtils.degToRad(0)
            ]}
        >
            <axesHelper args={[3]}/>
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
                <axesHelper args={[3]}/>
                <boxGeometry />
                <meshStandardMaterial color="red"/> 
            </mesh>
        </group>
```

![]({{site.url}}/images/2025-01-22-threejs-transformation/local.png)
- 기본적으로 Scene에 배치된 XYZ축 이외에도, Group별, mesh별로도 존재한다.