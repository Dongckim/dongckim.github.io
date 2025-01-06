---
title: "Unity XR Interaction Toolkit Interactor파트"
layout: single
Typora-root-url: ../
categories: Unity
tag: interactor
# use_math: true
---
Unity 개념학습 세 번째 포스팅

## XR Base Interactor
![]({{site.url}}/images/2025-01-05-xr-interactor/baseinteractor.png){: .img-width-seventy .align-center}

Inteactor들의 공통적인 것을 모아놓은 추상 클래스이다.

XR Base Controller Interactor를 거쳐, XR Direct Interactor와 XR Ray Interactor가 상속 받아 사용되고 있고, XR Socket Interactor도 직접 상속 받아 사용중임.

- Interaction Manager는 Interactable과 인터렉션을 주관할 Interaction Manager을 연결할 수 있습니다.
    - 비어있을 경우, Scene 어딘가에 있는 Interaction Manager와 자동으로 연결됨.
- Interaction Layer Mask는 기존의 유니티 레이어와 별개의 마스크로 비교적 최신 버전의 XR Interaction Toolkit에서 추가된 기능이다.
    - Interactor와 Interactable의 Layer mask가 서로 겹쳐야 인터렉션이 허용됨.
- Attach Transform 파라미터를 사용하면, Interactable을 Select 할 때 Attach Transform의 포즈만큼 더해집니다. 즉, Select되어지는 기준 포즈를 설정할 수 있음.
    - Interactor의 위치나 방향에 관계없이 Interactable이 위치하기를 원한다면, Interactor게임 오브젝트와는 별개로 Attach Transform을 생성해 연결
    - Attach Transform을 설정하지 않는 경우, 자동으로 자식으로 빈 게임오브젝트가 생성되어 연결되기 때문에, 당연히 Interactor의 포즈와 Interactable의 포즈가 동기화 됨.
- Keep Selected Target Valid가 활성화 되어 있으면, 인터렉션 중인 대상이 유효해지지 않아도 인터렉션을 유지할 수 있음.
    - ex) 텔레포트를 할 때, 더 이상 Ray Interactor가 Teleportation Anchor나 Area를 가리키지 않는 경우 인터렉션 되지 않도록 할 때 사용.
- Starting Selected Interactable을 사용하여, 시작할 때 Select상태로 시작할 Interactable을 지정할 수 있음.
- Interactor Events는 Interactor에서 이벤트가 발생할 때, 실행할 기능을 연결할 수 있는 Unity Event.
    - Hover
    - Hover Exited
    - Select
    - Select Exited

## XR Base Controller Interactor
XR Base Controller Interactor는 XR Direct Interactor와 XR Ray Interactor가 동시에 상속받아 사용됨.
- Select Action Trigger 파라메터는 Select 액션을 처리하는 방식을 설정할 수 있는 파라미터이다.
    - State 타입은 버튼을 누르고 있는 동안만 Select상태를 유지함. (오버랩 전에 버튼을 누르고 있다가 오버랩을 해도, Select상태로 처리함.)
    - State Change는 버튼을 누르고 있는 동안만 Select상태를 유지함. 오버랩 전에 버튼을 누르고 있다가 오버랩을 하면 Select를 처리하지 않고 무시함.
    - Toggle은 오버랩 상태에서 버튼을 누를 때 Select되고 다시 버튼을 누를 때 Deselect가 됨.
    - Sticky는 오버랩 상태에서 버튼을 누를 때, Select되고, 다시 버튼을 눌렀다 땔 때 Deselect됨.
- Hide Controller On Select가 활성화되어 있으면, Interactor가 Select 상태에 들어갈 경우 컨트롤러 모델을 비활성화 합니다.
- Allow Hovered Activate를 활성화할 경우, 오버랩 후에 Select하지 않아도 Activate할 수 있게 됨.
    - ex) 총과 컨트롤러가 오버랩된 이후에 Trigger버튼만 눌러도 발사가 되는 현상임.
- Audio Events는 각 events에 맞췃 오디오를 재생하고 싶을 때 사용됨. → 재생할 Audio Clip을 연결할 수 있습니다.
- Haptic Events는 각 이벤트에 맞춰 Haptic을 재생하고 싶을 때 사용.(Unity Event타입은 아님.)

## XR Direct Interactor
직접적인 인터렉션을 시도해주는 컴포넌트임. 컨트롤러 대상에 가까이 대어 집거나 하는 등의 인터랙션을 시도할 수 있음.
- 당연히 3D Collider가 있어야 하며, Hover와 Select등 상황에 맞는 유니티 이벤트를 연결할 수도 있음.

## XR Ray Interactor
직선 또는 광선을 발사하여 부딪히는 대상에 원거리에서 인터랙션을 시도하는 컴포넌트. 일반적으로 XR Interactor Line Visual 컴포넌트를 사용하여, 광선을 시각화함. XR Base Controller Interactor의 자식 클래스로 대부분의 파라미터가 XR Base Interactor와 XR Base Controller Interactor에 구현되어 있음.
- Enable Interaction with UI GameObjects 옵션을 활성화할 경우, 유니티 UI와의 인터랙션을 허용
- Force Grab 옵션을 활성화할 경우, Interactable을 Select할 때 Interactor의 Attach Point로 당겨옵니다. 비활성화 되어 있을 경우, 광선의 끝에 Select하는 그 위치대로 유지됨.
- Anchor Control을 활성화할 경우, Select하고 있는 Interactable을 앞뒤로 움직이거나 회전시킬 수 있음.
- Rotate Reference Frame을 사용하여, 회전할 때의 기준 축을 설정할 수도 있음.
- Ray Origin Transform을 사용하여, Ray가 발사되는 시작지점의 위치와 방향을 변경할 수 있음.
- Raycast Configuration 토글을 열면, 레이캐스팅에 관련된 파라메터들을 설정할 수 있음.
    - Line Type → Straight Line은 이 Ray Interactor가 직선을 사용하게 함.
    - Max Raycast Distance 파라미터를 이용해 최대 Raycast거리를 설정할 수 있음.
    - Projectile Curve는 속력과 중력으로 샘플링 되는 곡선을 사용하게 함.
    - Reference Frame을 사용하면 기준 축을 설정하여, 레이의 시작지점과 레이가 나아가는 축을 변경할 수 있음.
    - Velocity는 Ray가 앞으로 뻗어 나가는 속도이고, Acceleration은 Ray가 Y축 아래 방향으로 받는 가속도임. 일반적으로는 중력가속도를 사용함.
    - Additional Ground Height를 사용하여, Ray가 부딪힌 지점의 높이를 설정할 수 있음. 이 값을 높이면, 레이의 끝점이 이 값만큼 아래로 내려감
    - Additional Flight Time은 Ray가 Ground Height에 도달 후, 추가적으로 이동할 거리임.
    - Sample Frequency는 이 Ray를 그려주는 점의 개수임. 높을수록 더 부드러운 곡선을 그리게 됨.
    - Beizer curve는 이 ray interactor가 Control Point와 End Point를 설정하여 베지어 커브로 그려지는 곡선을 사용하게 됨.
- Raycast Mask를 통해 Interactor가 인터랙션할 유니티의 레이어를 선택할 수 있음.
- Raycast Trigger Interaction은 Is Trigger가 체크되어 있는 Collider와 인터렉션을 할 지를 선택할 수 있는 파라메터임.
    - Collide로 설정하면, trigger모드인 Collider들도 Raycast를 시도함.
- Hit Detection Type을 선택하여 부피가 없는 광선으로 Raycasting을 하거나 일정 크기의 구를 발사해 부딪히는 Collider들을 찾아내는 방식의 Sphere Casting을 할 지를 선택할 수 있음.
- Hit Closest Only를 활성화하면, Raycast를 시도할 때 첫번째로 부딪히는 Collider만 찾아냄. 비활성화 한다면, 최대거리 안에 부딪히는 모든 Collider들을 찾아냄.
- Blend Visual Line Points를 활성화하면, VR Device로 플레이할 때 시각적으로 보여지는 라인을 실제 컨트롤러 포즈에 뒤쳐지지 않게 함.

## XR Socket Interactor
일반적으로 컨트롤러와 같이 사용하지 않고, 별도의 공간에 설치하는 식으로 사용하는, Interactable을 꽂았다가 뺏다가 할 수 있는 Interactor임.
- 다른 interactor로 Interactable을 Select하여 가까이 움직인 뒤에 Deselect하면, XR Socket Interactor가 자동으로 Select했다가 다시 다른 Interactor로 Select해서 빼내면 XR Socket Interactor는 Deselect되는 형태로 작동하게 됨. 
- 대부분의 파라미터가 XR base Interactor에 구현되어 있음.
- Show Interactable Hover Mesh를 사용하여, Interactable이 오버랩 되었을 때 시각적으로 표시할 수 있게 됨.
- 시각화 해주는 모양은, Interactable의 모양으로 고정되어 있고, Material과 크기만 바꿀 수 있음.
- Hover Mesh Material은 XR Socket Interactor가 인터렉션 할 수 있는 상태일 때 Interactable이 오버랩 되면 표시할 Material임.
- Can't Hover Mesh Material은 XR Socket Interactor가 다른 Interactable을 Select하고 있는 등, 인터렉션을 할 수 없는 상태일 때 Interactable이 오버랩되면 표시할 Material임.
- Socket Active가 비활성화 되어 있으면, XR Socket Interactor의 인터렉션 기능이 작동하지 않음.
- Recycle Delay Time은 Select 상태의 Interactable이 Deselect되는 순간부터 다시 인터렉션이 가능해지는 시간임. 0으로 설정한 경우 소켓에서 Interactable을 빼는 즉시 다시 꽂을 수 있음. 