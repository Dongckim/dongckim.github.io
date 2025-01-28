---
title: "Three.js로 3D 플랫폼 개발 Light"
layout: single
Typora-root-url: ../
categories: Three.js
tag: light
use_math: true
---

Three.js를 이용한 3D 플랫폼 개발 개념공부

## Light
앞서 배운 내용을 좀 정리해볼까?

Light에 영향을 받는 mesh에는 다음과 같은 것들이 있었다.
- meshLambertMaterial
- meshPhongMaterial
- meshStandardMaterial
- meshPhysicalMaterial
- meshToonMaterial

총 5가지가 있었다.
이 외에는 사실 빛이 어떻게 쬐는지에 상관업이 렌더링되기 때문에 테스트하지 않아도 될 꺼 같다.

Light에는 총 6가지와, Environment와 같은 속성까지 7가지를 살펴보면 될 꺼 같다.
- AmbientLight
- HemisphereLight
- DirectionalLight
- PointLight
- SpotLight
+ Environment

하나씩 살펴보자.

### AmbientLight
```jsx
<ambientLight color={'blue'} intensity={5}/>
```
- AmbientLight는 다른 조명과 달리 방향성이 없으며, scene 전체에 균일하게 적용됨. 즉 그림자가 생기지 않음.
- intensity로 빛의 세기를 조절하고 color로 빛의 색상을 결정.
- 즉, **주변광이자 자연광**이라고 생각하면 제일 좋다.

![]({{site.url}}/images/2025-01-28-threejs-light/ambient.png)
- 그림에서와 같이 그림자가 보이질 않는 걸 확인할 수 있다.

-----

### useHelper
```jsx
const dLight = useRef<THREE.DirectionalLight>(null!);
useHelper(dLight, THREE.DirectionalLightHelper);


const sLight = useRef<THREE.SpotLight>(null!);
useHelper(sLight, THREE.SpotLightHelper);
```

후술될 광원들은 광원의 위치가 중요하기 때문에, 이를 눈으로 시각화하는걸 도와주는 툴을 사용해야겠다.

useHelper는 광원 useRef로 참조하고, useHelper로 light를 불러온다고 생각하면 될 것 같다.

----

### DirectionalLight
```jsx
<directionalLight 
    color={'#fff'} 
    position={[0,5,-5]} 
    intensity={5}
    ref = {dLight}
    target-position={[0,0,2]}
    castShadow
    // shadow-camera-top={10}
    // shadow-camera-bottom={-10}
    // shadow-camera-left={-10}
    // shadow-camera-right={10}
    shadow-mapSize={[5120,5120]}
/>
```
- 무한히 떨어지는 평행한 빛을 표현하는 light이다. 쉽게 말하면 태양광이라고 생각하면 좋다.
- intensity로 빛의 세기를 조절하고 color로 빛의 색상, position으로 빛의 위치을 결정.
- 주석처리된, `shadow-camera-*`속성은 DirectionalLight와 같은 평행광에서 그림자 범위를 설정할 때 주로 사용된다고 생각하면 되겠다.(빛의 카메라에 의해 정의된 공간만 렌더링)
    - shadow-camera-top: 그림자 맵의 위쪽 경계를 설정.
    - shadow-camera-bottom: 그림자 맵의 아래쪽 경계를 설정.
    - shadow-camera-left: 그림자 맵의 왼쪽 경계를 설정.
    - shadow-camera-right: 그림자 맵의 오른쪽 경계를 설정.
- `castShadow` → 해당 객체가 그림자를 생성할 수 있는지 여부를 결정

![]({{site.url}}/images/2025-01-28-threejs-light/directional.png)
-----

### spotlight
```jsx
<spotLight
    color={'#fff'} 
    position={[0,5,0]} 
    intensity={300}
    distance={10}
    angle={THREE.MathUtils.degToRad(45)}
    target-position = {[0,0,0]}
    penumbra={0.5}
    ref = {sLight}
    castShadow
/>
```
한 지점에서 빛을 방출하여 방출된 빛이 일직선으로 나아가다가 일정 각도 이내의 객체에만 빛이 비추는 조명
- 우리가 흔히 알고 있는 **스포트라이트**로 이해하면 편하다.
- position으로 빛의 위치, angle로 빛의 각도, penumbra로 부분 그림자의 강도를 설정
- penumbra는 감쇄율이라고 생각하면 된다. 경계를 확실히 할지, 흐리게 할지.

![]({{site.url}}/images/2025-01-28-threejs-light/spot.png)

----

### pointlight
```jsx
    <pointLight
        color={'#fff'} 
        position={[0,0, -2]} 
        intensity={50}
        distance={5}
        castShadow
        shadow-camera-top={10}
        shadow-camera-bottom={-10}
        shadow-camera-left={-10}
        shadow-camera-right={10}
        shadow-mapSize={[5120,5120]}
    />
```
하나의 지점에서 모든 방향으로 빛을 쐬주는 light. 
- 특히 pointlight는 빛이 어디에서 시작했는지 모르기 때문에 빛을 쏘는 방향은 없다고 생각하면 된다. 
- 전구를 생각하면 편하다.

![]({{site.url}}/images/2025-01-28-threejs-light/spot.png)

----

### Environment
```jsx
<Environment
    files={'./img/hdri.hdr'}
    background
    blur={0}
/>
```
3D 그래픽 작업을 하다 보면 사실적인 조명과 텍스처를 구현하는 것이 중요해진다. 
- 이 과정에서 **HDRI(High Dynamic Range Imaging)**와 같은 기술은 빛의 디테일과 정확한 조명 효과를 표현하는 데 큰 도움을 줌.

Poly Haven은 이런 리소스를 무료로 제공하는 플랫폼이다.
- [https://polyhaven.com/ko/hdris](https://polyhaven.com/ko/hdris)
-  모든 콘텐츠를 CC0 라이선스(퍼블릭 도메인)로 제공하기 때문에, 상업적 프로젝트에도 제한 없이 사용가능.

폴리해븐의 hdris에서 본인이 원하는 environment를 다운받은 후, hdr파일을 지정해 불러와주면 된다.

![]({{site.url}}/images/2025-01-28-threejs-light/environment.png)

