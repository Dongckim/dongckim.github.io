---
title: "Three.js로 3D 플랫폼 개발 [Camera/Scene/Renderer]"
layout: single
Typora-root-url: ../
categories: Three.js
tag: [Dom, SVG, Canvas]
# use_math: true
---

Three.js를 이용한 3D 플랫폼 개발 개념공부

## 카메라가 무대를 찍고 있다?

![]({{site.url}}/images/2025-01-16-threejs-renderer/camera.jpg){: .align-center}

우리가 알아야 할 개념은 이게 전부라고 봐도 무방하다.

- Scene: 무대 그 자체를 의미한다.
- Camera: 무대를 촬영하고 있는 카메라를 의미한다.
- Renderer: 카메라에 비춰지는 무대의 모습을 의미한다.

## useThree라는 훅

useThree 훅은 `React Three Fiber` 라이브러리에서 제공하는 훅으로, Three.js의 렌더러와 관련된 컨텍스트를 가져오는 데 사용된다고 한다.

특히 이 훅은, `Canvas` 안에서만 존재하는 Element로서만 존재할 수 있다.

useThree에는 여러가지 객체와 속성들을 포함하고 있는데,
- gl: WebGLRenderer(three.jsdml 렌더러 객체)
- scene: 현재 렌더링 중인 Three.js의 씬
- camera: 활성화 된 카메라 객체
- size: 캔버스의 크기(width와 height 포함)
- viewport: 현재 뷰포트의 크기 및 비율 정보
- setDefaultCamera: 기본 카메라를 설정하는 함수

이 객체들을 console에 찍어보면서 해당 값들을 확인해볼 수 있을 것이다.

그럼 이제 각각을 자세히 살펴보자.

---
### What is Renderer?

Renderer은 크게 useFrame()이라는 Hook 정도만 알아두면 이해하기 쉬울 것 같다. 

Three.js의 애니메이션 루프에 코드를 주입할 수 있게 함.
- Three.js의 requestAnimationFrame과 유사한 역할을 하며, 매 프레임마다 특정 작업을 실행할 수 있도록 도움.

1. 프레임 기반 업데이트
- useFrame은 애니메이션 루프에서 매 프레임마다 호출됩니다. 이를 통해 카메라, 객체, 상태 등을 매 프레임마다 업데이트.

2. 매개변수
- useFrame은 콜백 함수에 다음과 같은 매개변수를 제공:
- `state`: React Three Fiber의 상태 객체로, Three.js의 주요 요소(gl, scene, camera, clock 등)에 접근.
- `delta`: 이전 프레임과 현재 프레임 사이의 시간 간격(초 단위)입니다. 애니메이션의 속도를 조정할 때 유용.

#### 코드로 구현?
```js
const boxRef = useRef<THREE.Mesh>(null);

useFrame((state, delta) => {
	boxRef.current.rotation.x += delta;
	boxRef.current.position.y -= 0.01;
	boxRef.current.scale.z += 0.01;
}
)
```

useRef() 리액트 훅을 이용해서 .current 프로퍼티로 전달된 인자(initialValue)로 초기화된 변경 가능한 ref 객체를 반환하는데, 이 값들을 변형 시키면서, useFrame 훅 안에서 프레임 단위로 작업을 실행하는 코드를 보여준다.

### What is Camera?

사실 뭐 카메라를 모르는 사람은 없을 것 같다. 다만, 3D 플랫폼에서 카메라라고 하면 여러가지를 의미할 수 있는데,
- perspective Camera 원근
- orthographic Camera 직각/수직

perspective 카메라는 우리가 흔히 보는 시야와 같은 의미의 카메라이다. 반면 orthograpthic은 거리와 상관없이 직각 투영으로 원근법이 없는 2D 화면에서 주로 사용하게 된다.

#### 코드로?

![]({{site.url}}/images/2025-01-16-threejs-renderer/canvas.png){: .img-width-seventy}

보통 이런식으로  Canvas 요소 안에 넣어놓곤 한다.

- near (가까운 절두체 평면, Near Clipping Plane): 카메라에서 얼마나 가까운 객체까지 렌더링할지 결정합니다.
- far (먼 절두체 평면, Far Clipping Plane): 카메라에서 얼마나 먼 객체까지 렌더링할지 결정합니다.
- fov (Field of View, 시야각): 카메라의 수직 시야각(Vertical Field of View)을 정의합니다.

### What is Scene?

React Three Fiber에서는 `<Canvas>` 컴포넌트 내부가 자동으로 scene으로 처리가 된다.
- 씬 안에 배치된 모든 객체는 렌더링 대상이 되는 것이다.

#### Mesh
mesh는 3D객체의 가장 기본적인 구성요소로, 씬에 배치되는 실제 객체를 의미한다.
- Geometry: 객체 자체의 구조를 정의한다.
- Material: 객체의 표면을 렌더링하는 방법(재질)을 정의한다.

![]({{site.url}}/images/2025-01-16-threejs-renderer/mesh.png){: .img-width-seventy}

## OrbitControls

**OrbitControls**는 Three.js 및 React Three Fiber에서 카메라를 마우스나 터치로 제어할 수 있도록 해주는 컨트롤러이다. 이를 통해 사용자는 3D 씬(scene) 내에서 카메라를 회전, 줌, 이동(pan)할 수 있다.
- React Three Fiber에서는 `drei 라이브러리`의 OrbitControls 컴포넌트를 사용하여 간단히 구현할 수 있다.

```jsx
<OrbitControls
	maxPolarAngle={Math.PI / 2}  // 위로 90도까지만 회전 가능
	minPolarAngle={Math.PI / 4}  // 아래로 45도까지만 회전 가능
	maxAzimuthAngle={Math.PI / 2}  // 왼쪽으로 90도까지만 회전 가능
	minAzimuthAngle={-Math.PI / 2} // 오른쪽으로 90도까지만 회전 가능
/>
```
- AzimuthAngle: 왼쪽 오른쪽 각도를 의미함.
- PolarAngle: 위 아래 각도를 의미함.


## AxesHelper
**AxesHelper**는 Three.js에서 제공하는 유틸리티 객체로, 3D 공간의 축을 시각적으로 표시하는 데 사용됨.
- 단위는 미터이다!!

## GridHelper
**GridHelper**는 Three.js에서 제공하는 유틸리티 객체로, 3D 씬에서 격자(grid)를 시각적으로 표시하는 데 사용됨.
- 단위는 미터이다!!

## Leva
**Leva** 는 React 기반의 UI 라이브러리로, 주로 3D 그래픽을 다루는 애플리케이션에서 사용되는 컨트롤러와 슬라이더를 생성하는 데 유용하다.
- Three.js와 같은 3D 라이브러리와 함께 사용하여 실시간으로 3D 객체나 씬의 속성을 변경할 수 있는 인터페이스를 제공.


## 이번 주 구현 결과

![]({{site.url}}/images/2025-01-16-threejs-renderer/result.png){: .img-width-seventy}

여기 사진에서 상단 오른쪽에 있는 GUI가 **Leva**이다.

마음 같아선 gif나 영상을 올리고 싶지만, 용량 문제로 그냥 가볍게 감상만 하자.