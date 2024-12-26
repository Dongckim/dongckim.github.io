---
title: "OpenGL ES 3차원 컴퓨터그래픽스 Introduction"
layout: single
Typora-root-url: ../
categories: OpenGL
# tag: shader
use_math: true
---

ⓒ 2019. [JungHyun Han](https://media.korea.ac.kr/people/jhan/) Korea University Seoul, All rights reserved.

<br/>


## 3차원 컴퓨터그래픽스의 정의

3차원으로 표현된 물체를 입력으로 받아서 2차원 영상을 출력하는 작업.
이를 프레임이라고 부르는데, 이 프레임을 얼마나 빠르게 변환할 수 있는지에 따라
- 실시간 그래픽스(real-time grapics)와,
- 비-실시간 그래픽스(visual effects)로 분류할 수 있다.
초당 30개 이상(fps)을 만들어내는 대표적인 예로는 게임, 가상현실(VR), 증강현실(AR)이 있다.

OpenGL ES를 이용해서 실시간 그래픽스의 기본적인 알고리즘을 이해하는 것이 주 목적이 될 것이다.

<br/>



## Computer Grapics Production
- Modeling
- Rigging
- Animation
- Rendering
- Post-Processing

### Modeling
가상의 그래픽스 환경을 구성하는 각각의 물체를 컴퓨터가 처리할 수 있는 방식으로 표현한 것이 모델(Model)이라고 한다.
- 많은 경우 다각형 메쉬를 사용해서 표현한다 (polygon meshes)
- 그래픽 아티스트(graphic Artist)가 오프라인으로 이미지 텍스쳐를 생성해서 폴리곤 매쉬 표면에 입히는 작업을 거쳐야 한다.(texturing)

### Rigging

![]({{site.url}}/images/2024-09-21-opengl-introduction/rigging.png)

움직임을 표현하기 위해, 뼈대 및 골격을 만들어서 폴리곤 매쉬 안에 입힌 후에,
- 폴리곤 매쉬들의 일부 꼭짓점(vertex)과 뼈대와의 상관관계를 잘 정의해서 대입한다면, 행동에 맞춰서 폴리곤 매쉬가 자연스럽게 움직이는 애니메이션을 생성할 수 있다.

### Animation

![]({{site.url}}/images/2024-09-21-opengl-introduction/animation.png)

- skeleton motion이다. 

### Rendering

![]({{site.url}}/images/2024-09-21-opengl-introduction/rendering.png)

3차원 광경으로부터 2차원 영상을 만들어 내는 작업을 일컫는다.
- 텍스쳐링이 기본적으로 진행되어야하고,
- 라이팅(lighting) 작업도 기본적으로 알아두어야 한다.

### Post-processing

![]({{site.url}}/images/2024-09-21-opengl-introduction/postprocessing.png)

카메라 노출시간이 아무리 짧더라도, 노출시간동안 물체가 고정이 된 것이 아니라 움직이기 때문에 흐릿하게 보이는 것이다.
- Motion Blur : 사실성이 굉장히 높아질 것이다.
- 렌더링과 다르게 필수적인 작업은 아님. 선택적.


<br/>

## Grapics API

![]({{site.url}}/images/2024-09-21-opengl-introduction/table3.png)

게임엔진(Game Engine)은 개발도구 중에 하나라고 보면 됨. 애니메이션, 렌더링, 후처리(postprocessing) 개념이 다 포함된 도구이다. 
- 유니티(Unity), 언리얼(Unreal)..

이런 게임엔진 기반에는 바로, 그래픽스 인터페이스가 있다고 생각하면 되겠다.
- Direct 3D(Microsoft), OpenGL(Khronos)

모바일과 같은 임베디드 시스템과 같은 경우, OpenGL의 일부를 따온 OpenGL ES를 사용한다.

OpenGL ES 기반에는 또 GPU라는 하드웨어가 있다.
- Graphics Processing Unit

게임엔진이나 그래픽스 애플리케이션이 OpenGL ES를 호출을 하게 되면, 이는 GPU를 가동하게 된다.
- 따라서 **그래픽스 API는 GPU에 대한 소프트웨어 인터페이스**라고 이해하면 된다.

## VR
VR headset is a head-mounted Display (HMD) that shows stereo images
- Virtual Reality (VR) = tracking + rendering
- VR + motion simulator
![]({{site.url}}/images/2024-09-21-opengl-introduction/VRmotionSimulator.png)
*IEEE VR 2017, 2019; ACM CHI 2018*

## AR
A spectrum of mixed reality or extended reality (XR)
- Microsoft HoloLens 1 and 2
- **AR = Tracking + 3D Reconstruction + Registration + Rendering**

![]({{site.url}}/images/2024-09-21-opengl-introduction/IndustrialAR.png)
*Industrial AR Application*

### Next-generation AR 
Tracking + Dynamic 3D Reconstruction + Registration + Rendering + Physics Simulation





```toc
```