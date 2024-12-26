---
title: "OpenGL ES 3차원 컴퓨터그래픽스 Output Merger"
layout: single
Typora-root-url: ../
categories: OpenGL
tag: OutputMerger
use_math: true
---

ⓒ 2019. [JungHyun Han](https://media.korea.ac.kr/people/jhan/) Korea University Seoul, All rights reserved.

<br/>


## Color and Depth Buffers

![]({{site.url}}/images/2024-10-07-opengl-output-merger/wrapup.png)

**Viewport** : 실제적으로 스크린에 보여줄 영역이다. 이때 그 영역을 잠시 보관하고 있는 공간을 **Buffer**라고 한다.

- Color Buffer : 스크린에 나타날 픽셀들을 잠시 저장하는 곳 (WxH)
- Depth Buffer : Color Buffer에 저장된 픽셀들의 Z-value들을 저장하는 곳이라 생각하면 되겠다.  

![]({{site.url}}/images/2024-10-07-opengl-output-merger/buffer.png)

Fragment Shader에서 계산된 normal을 이용해서 Lighting을 진행하고, 계산된 texCoord를 사용해서 texturing을 진행해서 결국 output merger에 RGB값을 전달하게 된다.

<br/>

### Z-buffering

![]({{site.url}}/images/2024-10-07-opengl-output-merger/zBuffering.png)

color buffer에 저장하게 되는데, 각 fragment 모두마다 z-value를 비교를 해서 나타나게 된다.

- 예를 들어, 1,0으로 주어진 default 값에 비해 더 z값이 작기 때문에 해당 값의 RGB가 fragment를 채우게 된다.

- 파란색 fragment와 겹치는 자리는 z값을 비교해보면 파란색의 z값이 더 작기 때문에 파란색 fragment로 지정이 된다고 생각하면 되겠다.

- Rendering-Order는 상관없이 동일한 결과를 얻게 된다.

### Z-buffering in GL

```c
glClearDepth(1.0f);     //initialized with depth 1.0
glClearColor(1.0f, 1.0f, 1.0f);     // initialized with white
glClear(GL_DEPTH_BUFFER_BIT | GL_COLOR_BUFFER_BIT);     // clear buffers
```

```c
glEnable(GL_DEPTH_TEST);
glDepthFunc(GL_LESS);   // not need
```

<br/>

## RGB "A" → Alpha blending

alpha channel은 [0,1] 범위 안에서 투명도를 계산해서 보여준다고 생각하면 되겠다.
- 0 denotes "fully transparent"
- 1 denotes "fully opaque"

![]({{site.url}}/images/2024-10-07-opengl-output-merger/formula.png)
*기존의 픽셀 색깔과 fragment색깔을 블랜딩한다는 의미*

z-buffering 알고리즘은 렌더링 순서와 상관없이 어떤 순서로 하든 동일한 결과물을 제공한다.

- 그래서 alpha 값이 1인 경우와, 그렇지 않은 경우를 구분해야하는데,
- 이때 alpha값이 1이 아닌 경우에는 뒤에서부터 앞의 z값을 비교해서 처리하는 back-to-front order를 가져가야 한다. 그래야 z-buffering test결과를 보장할 수 있고, 위 식에도 정확한 값이 나올 것이다.
- 그래서 sorting이 필수 불가결하겠다.

```C
glEnable(GL_BLEND);
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
glBlendEquation(GL_FUNC_ADD);
```

- 아까 본 formula 그대로 적용하면 된다고 생각하면 편할 듯?
- fragment = source라고 생각하고 그대로 대입하면 된다.
- 이외에도 다양한 value들이 있어서 참고하면 되겠다!

<br/>

![]({{site.url}}/images/2024-10-07-opengl-output-merger/alphaBlending.png)

![]({{site.url}}/images/2024-10-07-opengl-output-merger/fog.png)
*alpha blending을 잘하면 사용할 수 있는 예시들*


<br/>


```toc
```