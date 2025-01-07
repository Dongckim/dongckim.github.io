---
title: "Unity XR Interaction Toolkit Interactable파트"
layout: single
Typora-root-url: ../
categories: Unity
tag: interactable
# use_math: true
---
Unity 개념학습 네 번째 포스팅

## XR Base Interactable
![]({{site.url}}/images/2025-01-07-xr-interactable/xrinteractable.png){: .img-width-seventy .align-center}

Interactable의 공통 기능들을 모아놓은 기본 추상 클래스임. 
- Hover나 Select등의 이벤트들과 Interaction Manager 선택이나 Interaction Layer Mask 설정 등 Interactable들의 공통 기능들이 정의되어 있음.
- Base Teleportation Interactable을 거쳐, Teleportation Area와 Teleportation Anchor가 상속받아 사용하고 있고, XR Grab Interactable이 직접 상속받아 사용하고 있음.
- Interaction Manager는 Interactor와의 인터렉션을 주관할 Interaction Manager를 연결할 수 있음.
    - 마찬가지로, 비어있을 경우 Scene어딘가에 있는 Interaction Manager를 자동으로 찾아서 연결함.
- Interaction Layer Mask는 기존 유니티 레이어와는 별개의 레이어 마스크로 비교적 최신 버전의 XR Interaction Toolkit에서 추가된 기능임.
    - Interactor와 Interactable의 Interaction layer mask가 서로 겹쳐야 인터렉션이 허용됨.
- Collider는 인터렉션을 할 영역을 설정할 Collider 목록.
    - 비워둘 경우 Interactable 컴포넌트가 있는 게임오브젝트와 계층적으로 자식 모두에 있는 Collider들을 자동으로 찾아서 연결해줌.
- Custom Reticle을 사용할 경우, 이 Interactable을 연결할 경우 연결된 게임오브젝트를 추가로 시각화 해줌.
    - 일반적으로 XR Ray Interactor등으로 가리킬 때, 가리켜지는 지점에 Custom Reticle을 시각화할 때 사용.
- Select Mode를 사용하여 이 Interactable이 동시에 여러 Interactor가 Select할 수 있게 할 수 있음.
    - Single
    - Multiple 여러 개의 Interactor가 동시에 Select할 수 있음.
    - XR Grab Interactable의 경우, Multiple로 설정하게 되면 시각적으로 올바르지 않게 보이기 때문에 Single로만 설정할 수 있게 되어있음.

## XR Grab Interactable
Interactor을 통해 집을 수 있는 대상이 되는 컴포넌트이다. Collider와 Rigidbody를 같이 사용해야 하며, 집는 순간 interactor의 attach point로 빨려 들어가듯 이동되어지고 그 후에 Interactor의 포즈와 설정 값에 따라 동기화 됨.

- Movement Type 파라메터를 설정해 XR Grab Interactable이 Select되어 움직일 때의 방식을 설정할 수 있음.
- Velocity Tracking 타입은 Rigidbody의 Velocity와 Angular Velocity를 이용해 움직임. 일반적으로 이 오브젝트를 움직일 때 다른 collider에 의해 물리적으로 가로막히기를 원할 때 사용.
- Kinematic은 Kinematic으로 설정된 RigidBody를 물리적으로 움직임. 일반적으로 이 물체는 밀리지는 않지만, 다른 물리적인 물체들은 밀어내고 싶을 때 사용.
- Instantaneous는 Transform의 위치와 방향을 사용하여 움직임. 일반적으로 다른 Collider들의 영향을 받지 않고 움직이고 싶을 때 사용함.
- Retain Transform Parent를 활성화하면, Interactable의 원래 부모를 유지함. 이 파라미터를 비활성화 할 경우 항상 Root가 부모로 설정됨.
- Track Position과 Track Rotation을 활성화하면, Interactor가 Interactable을 Select했을 때 Interactor의 위치와 방향을 따라가게 할 수 있습니다.
- Smooth Position을 활성화하여 위치를 따라가게 할 수 있고, 
- Smooth Rotation을 활성화하여 방향을 따라게 할 수도 있음.
- Tighten Rotation과 Tighten Position은 Interactor에 얼마나 가깝게 유지되는지에 대한 비율이라고 보면 된다. 
- Throw On Detach를 활성화하면, 이 Interactable을 Deselect할 때 Interactor의 속도와 각속도를 적용합니다.
    - Movement Type에 영향을 받지 않고 아래 옵션들을 사용에 부드럽게 던지거나 더 빠르게 던질 수 있음.
    - Throw Smoothing Duration은 Interactable이 던져질 때 속도가 0에서부터 부드럽게 가속할 시간임.
    - Throw Smoothing Curve를 사용하여 Interactable이 던져질 때 시간의 흐름에 따라 스무딩 되는 정도를 설정할 수 있음.
    - Throw Velocity Scale과 Throw Angular Velocity scale은 Interactable이 던져질 때 Interactor의 속도나 각속도에 각 값을 곱해서 Interactable에 적용함. 더 빠르게 적용하고 싶으면 이 값을 1 이상의 숫자를 사용하면 됨.
- Force Gravity On Detach를 활성화하면, Interactable이 Deselect될 Rigidbody의 Use Gravity 파라미터를 활성화하게 함. → Interactable은 집었다가 놓으면 중력이 적용됨.
- Attach Transform을 설정하여 이 Interactable이 Select될 때의 기준점을 설정할 수 있음.
    ex) 총기의 경우 손잡이 부분에 Attach Transform을 설정하여 Interactable을 Select할 때 손잡이를 쥐도록 할 수 있다.
- Attach Ease In time는 Interactor에 Select될 때 이 값만큼의 시간이 더 걸리게 됨.

## Base Teleportation Interactable
Teleportation Area와 Teleportation Anchor가 상속받아 사용하고 있는 클래스이다.
- Teleportation Configuration으로 텔레포테이션과 관련된 기능들을 설정할 수 있음. 
- Match Orientation으로 텔레포테이션 직후로 XR Origin의 방향을 설정할 수 있음. 
- World Space Up으로 설정할 경우, XR Origin의 방향이 World Space Up 벡터를 기준으로 설정됨. 
- Target up으로 설정될 경우, XR Origin의 방향이 텔레포테이션 대상의 Up 벡터를 기준으로 설정됨.
- Target Up And Forward로 설정할 경우, XR Origin의 방향이 텔레포테이션 대상이 바라보는 방향을 기준으로 설정됨. → None으로 설정할 경우 XR Origin의 방향이 텔레포테이션 전 후에 동일하게 유지됨.

## Teleportation Area
내가 설정한 영역 아무 곳에서나 텔레포테이션을 할 수 있게 해주는 컴포넌트임.
- 텔레포테이션 영역은 3D Collider로 설정할 수 있고, 
- 모든 파라미터가 Base Teleportation Interactable에 구현되어 있음.

## Teleportation Anchor
특정지점으로 Teleportation 해주는 컴포넌트임.
- 모든 파라미터가 Base Teleportation Interactable에 구현되어 있음.
- Teleportation Anchor을 이용하여 텔레포트를 하면, Teleportation Anchor의 위치와 방향으로 텔레포테이션이 되지만
- Teleport Anchor Transform 파라미터를 연결하여 다른 위치에 텔레포트 되도록 설정할 수도 있음.
- 이 Parameter에 transform을 연결하면, 해당 Transform의 위치와 방향으로 텔레포테이션 됨.