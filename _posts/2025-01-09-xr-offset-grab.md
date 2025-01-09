---
title: "Unity XR Interaction Toolkit 이외에 기타 등등 ②"
layout: single
Typora-root-url: ../
categories: Unity
tag: [simulator, Grab]
# use_math: true
---
Unity 개념학습 마지막 포스팅

## XR Grab Offset Interactable
XR Grab Interactable을 확장하여 Select할 때 Interactor와 Attach Point로 Interactable을 끌어들이지 않고, Select하는 순간 그 거리를 유지해주는 컴포넌트라고 생각하면 편하다.
- XR Interaction Toolkit에서 제공하는 기능은 아니지만, 다양한 VR 컨텐츠를 사용할 때 유용하게 사용하는 기능이다.

### 원리
XR Grab Interactable의 기능을 확장하여 Select Enter하여 Interactor의 Attach Point를 Interactable의 월드 포지션과 방향으로 설정하고, Select Exit할 때 원상 복구 시키는 방법을 사용함.
- 자연스럽게 Interactor와 Interactable의 상대적인  위치와 방향을 유지할 수 있게 됨.