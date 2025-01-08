---
title: "Unity XR Interaction Toolkit Locomotion 파트"
layout: single
Typora-root-url: ../
categories: Unity
tag: locomotion
# use_math: true
---
Unity 개념학습 다섯 번째 포스팅

## Locomotion System
Locomotion System은 XR Origin을 이동하거나 회전시킬 수 있는 기능을 제어하는 컴포넌트이다.
- Scene 어딘가에 하나만 있어야 하며, 
- Teleportation Provider등 여러가지 Provider들과 함께 사용된다.
- Timeout은 연결된 XR Origin을 베타적으로 제어하는 제한시간입니다.
- 이 Locomotion System이 XR Origin을 제어하는 동안 XR Origin의 위치나 방향을 수정하면 안됨.
- XR Origin Parameter는 Locomotion System을 이용해 제어할 XR Origin 컴포넌트임.
    - 비워둘 경우 자동으로 찾아서 연결.


## Locomotion Provider
![]({{site.url}}/images/2025-01-08-xr-locomotion/map.png)
Locomotion Provider는 이동시키거나 회전시키는 방법에 대해 정의되어 있는 Provider들의 기본 부모 추상 클래스이다.
- Snap Turn Provider, Continuous Move Provider, Continuous Turn Provider, Teleportation Provider가 상속하고 있다.
- 각각의 Provider들은 Locomotion System 컴포넌트와 같은 게임 오브젝트에 있어야 할 필요는 없지만, 일반적으로 같이 위치하는 편이다.
- 유일한 파라미터인 Locomotion System 파라미터는 Locomotion Provider들을 작동시킬 Locomotion System을 연결할 파라미터임.    
    - 비워 둘 경우 자동으로 찾아서 연결함.

## Teleportation Provider
Teleportation Provider는 Teleportation Area와 Teleportation Anchor가 작동하게 하는 컴포넌트이다. 
- Locomotion System 컴포넌트가 있어야 작동하며, 별다른 파라미터는 없다.

## Snap Turn Provider
Snap Turn Provider는 정해진 수치만큼 좌우로 회전하거나 180도 뒤로 작동시키는 기능을 한다.
- 주로 스틱 키를 좌우로 조작하거나 뒤로 조작하면 작동하도록 설정된다.
    - Devised-Based
    - Action-Based → Input System으로 상세하게 연결이 가능한 버전이 있음.
- Turn Amount는 좌우로 회전하는 각도
- Debounce Time은 뒤로 회전할 때 걸리는 시간이다. 
- Enable Turn Lef† Right옵션을 활성화하면, 좌우로 회전할 수 있음.
- Enable Turn Around가 활성화되어 있으면, 180도 뒤로 회전할 수 있습니다.
- Left Hand Snap Turn Action과 Right Turn Snap Turn Action을 이용하면 왼손과 오른손 컨트롤러에서 어떤 액션을 취해야 회전할 지를 선택할 수 있음.

## Continuous Move Provider
Continuous Move Provider는 지속적인 이동을 할 수 있게 해주는 기능을 작동시킴.
- 주로 스틱 키로 조작하도록 설정하며, 해당 액션을 하고 있는 동안 지속적으로 이동함.
- 기본적으로 이동은 현재 쳐다보고 있는 방향 기준으로 XZ평면에 평행하도록 이동하며 Y축으로는 이동하지 않음.
    - Device Based Version
    - Acrion Based Version
- 마찬가지로 Locomotion System컴포넌트가 있어야 작동함.
- Move Speed 파라미터는 매 초마다 이동하는 거리이다. 
- 예를 들어 1로 설정한 경우, 매초마다 유니티 단위로 1씩 이동.
- Enable Strafe가 활성화되어 있으면, 좌우로도 이동할 수 있음.
- Use Gravity가 활성화되어 있으면, 이동할 때 중력의 영향을 받게 할 수 있음.
- Gravity Application Mode를 설정하여 중력을 계산하는 시점을 결정할 수 있음.
    - Attemping Move로 설정할 경우, 이동할 때만 중력을 적용함.이동하지 않을 때는 바닥에 닿을 때까지만 마지막 속도를 계속 적용함.
    - Immediately는 이동하지 않을 때에도 중력을 적용함
- Left Hand Move Action과 Right Hand Move Action을 이용해 어떤 액션을 해야 이동을 할 지를 선택할 수 있음.

## Continuous Turn Provider
Continuous Turn Provider는 지속적인 회전을 할 수 있게 해주는 기능을 작동시킨다.
- 주로 스틱 키로 조작하도록 설정하며 해당 액션을 하고 있는 동안 지속적으로 Y축으로 회전한다.
    - 동일하게 Device Based version
    - Action Based Version이 있음.
- Locomotion System 컴포넌트가 있어야 작동하는 것도 동일하다.
- Turn Speed는 초당 회전하는 속도이다. 
    ex) 60으로 설정한다면, 매 초마다 60도 만큼 회전한다. 
- Left Hand Turn Action과 Right Hand Turn Action으로 왼손과 오른손 컨트롤러에서 어떤 액션을 해야 회전할 지를 선택할 수 있음.