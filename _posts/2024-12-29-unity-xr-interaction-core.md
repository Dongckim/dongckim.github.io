---
title: "Unity XR Interaction Toolkit TroubleShooting"
layout: single
Typora-root-url: ../
categories: Unity
tag: XR
# use_math: true
---
![Alt text]({{site.url}}/images/2024-12-29-unity-xr-interaction-core/Unity.png)

## XR Interaction Toolkit 개념과 핵심 기능

XR Interaction Toolkit은 VR 및 AR 경험을 제작하기 위한 고수준 구성 요소 기반 상호작용 시스템으로, Unity 입력 이벤트를 활용해 3D 및 UI 상호작용을 지원하는 프레임워크를 제공한다.

### 핵심 구성 요소
1. Interactor와 Interactable 구성 요소.
2. 두 구성 요소를 연결하는 Interaction Manager.
3. 이동 및 시각적 피드백을 위한 추가 구성 요소.

### 주요 기능
1. HMD, Controller Tracking
2. Custom Controller Model
3. Interactor & Interactable 
4. Events
5. Locomotion System, Provider
6. Teleportation
7. UI Interaction
8. Haptic
9. XR Socket Interactor

### Trouble Shootings

-----
#### a. XR origin안에 Controller가 설정되지 않는 문제.

&emsp; &emsp; **AS IS** : XR origin을 설치하고 하위 계층에 `Main Camera`와 함께 Left, Right Controller가 생기지 않은 문제가 발생

![]({{site.url}}/images/2024-12-29-unity-xr-interaction-core/XR_origin.png)

&emsp; &emsp; **To BE** : 

1. XR Toolkt 버전 재설치 시도
2. Start Asset import한 이후에 Preset 파일 안에 XRI Default right/XRI Default left를 재추가
3. XR Controller 생성 완료.


-----
#### b. Cube 오브젝트를 만들어서 허공에 생성했더니 지 맘대로 움직임

&emsp; &emsp; **AS IS** : Cube를 설치했는데 자기 맘대로 허공에서 움직이는 현상을 발견함

&emsp; &emsp; **To BE** : is Kinetic + Use Gravity 버튼을 취소해주면 물리법칙을 따르지 않으면서 움직이지 않는 현상.

![]({{site.url}}/images/2024-12-29-unity-xr-interaction-core/iskinetic.png){: .img-width-seventy .align-center}

-----
#### c. Interaction Layer Mask 설정 신경써야함.

&emsp; &emsp; **AS IS** : Grab 작동 아예 안되는 문제 발생

&emsp; &emsp; **To BE**  :

1. XR Direct Interactor 쪽에는 Layer Mask가 everything으로 설정되어 있어야하고, 
2. XR Grab Interactable 쪽에는 Default로 Mask가 설정되어있음.

이 두개의 layer mask가 서로 겹쳐야 서로 인터랙션이 되어지는 상태가 된다.


----
#### d. Hide Controller On Select
&emsp; &emsp; **AS IS** : Cube를 집었을 때, 컨트롤러의 시각적 표현(모델)을 숨기고 싶다.

&emsp; &emsp; **To BE** :

`Hide Controller On Select`는 XR Interaction Toolkit에서 사용되는 요소로, 컨트롤러와 상호작용하는 동안 컨트롤러의 시각적 표현(모델)을 숨길 때 사용됨.

ex) VR 게임에서 무기를 잡을 때 컨트롤러를 숨기고 무기만 보이도록 처리하고 싶을 때 이렇게 처리함.

----
#### e. Device 컨트롤에 관해

&emsp; &emsp; **AS IS** : Device를 설정하는 법에 대해 조금 더 알아보자

&emsp; &emsp; **To BE** : Grip 버튼을 눌러 OnSelect를 활성화 시키고, 이 상태에서 Trigger버튼을 눌러 OnActivated를 하게 한다. 이 상황이 VR게임의 예시에서 총에 손을 들고 총알을 발사하는 상황인 것이다.

----
#### f. 물체 (Interactable Event)

&emsp; &emsp; **AS IS** : Interactor와 Interactable에 대해 구분해서 알아보면 좋을 것 같다.

&emsp; &emsp; **To BE** : 

![]({{site.url}}/images/2024-12-29-unity-xr-interaction-core/interactorEvent.png){: .img-width-seventy .align-center}

XR Direct Interactor VS LeftHand Controller

**Lefthand Controller**
- 컨트롤러의 위치, 회전, 입력 데이터를 처리하는 기본 GameObject
- 주로 XR Controller가 포함되어 있음.
- 물리적 컨트롤러와 Unity의 가상 컨트롤러 간의 입력 매핑을 함.

**LeftHand Direct Interactor**
- XR Interaction Toolkit의 상호작용 시스템의 일부로, Direct Interaction(직접 상호작용)을 처리
- 컨트롤러가 가까운 거리에서 오브젝트를 잡거나(interact) 터치하는 동작
- XR Direct Interactor 컴포넌트 -> 상호작용 가능한 오브젝트(예: XR Grab Interactable)를 탐지

결론적으로, LeftHand Controller는 입력 처리와 컨트롤러 위치/회전 추적에 중점인 반면, LeftHand Direct Interactor는 오브젝트와의 직접 상호작용(잡기, 터치 등)을 처리한다.

----
#### g. Locomotion System

&emsp; &emsp; **AS IS** : Locomotion System이 뭐지

&emsp; &emsp; **To BE** :

**Locomotion System**

1. Continuous Movement
- 사용자가 컨트롤러의 조이스틱이나 방향 입력을 사용해 부드럽게 이동

2. Teleportation
- 사용자가 특정 위치를 지정한 후, 해당 위치로 즉시 이동
- Teleportation Provider : 순간 이동을 처리하는 컴포넌트, Line Renderer 또는 Arc Ray를 사용해 목표 지점을 시각적으로 표시

3. Snap Turn & Smooth Turn
- 사용자가 컨트롤러를 사용해 특정 각도로 빠르게 회전

-----
#### h. XR Ray Interactor

&emsp; &emsp; **AS IS** : XR Ray Interactor이 뭐지

&emsp; &emsp; **To BE** :

**XR Ray Interactor**

1. Object Interaction (객체 상호작용)
- Ray가 닿는 객체와 상호작용 ex) 버튼 누르기, 객체 잡기(Grab), 메뉴 선택 등.

2. UI Interaction (UI 상호작용)
- XR 환경에서 Unity의 Canvas UI와 상호작용 ex) VR 환경에서 UI 버튼 클릭.

3. Ray Visualization (레이 시각화)
- Line Renderer나 Curve Renderer를 사용해 레이를 시각적으로 표현 ex) 직선형 또는 곡선형 레이로 구성할 수 있습니다.

4. Raycast Configuration
- Raycast 설정을 통해 레이의 거리, 방향, 상호작용 가능 객체 등을 정의.

![]({{site.url}}/images/2024-12-29-unity-xr-interaction-core/raycast.png){: .img-width-seventy .align-center}
특히,  Line Type을 통해 레이의 시각적 표현 방식을 설정, (Straight Line: 직선 레이.Projectile Curve: 곡선 레이 (예: 포물선 형태).Bezier Curve: 부드러운 곡선 형태.)
