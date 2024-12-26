---
title: "OpenGL ES 3차원 컴퓨터그래픽스 Rasterizer"
layout: single
Typora-root-url: ../
categories: OpenGL
tag: Rasterizer
use_math: true
---

ⓒ 2019. [JungHyun Han](https://media.korea.ac.kr/people/jhan/) Korea University Seoul, All rights reserved.

<br/>


## Rasterizer

앞서 배운 vertex shader를 통해 삼각형이 그려질 것이고, 삼각형이 화면에 나타나면서 픽셀을 점유하게 될 것이다.
- 픽셀마다 필요한 색깔과 같은 데이터를 대입시켜주어야 한다.

1. Clipping
2. Perspective
3. Back-face culling
4. Viewport Transform
5. Scan conversion

### Clipping

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/clipping.png)

- t1의 경우 view frustum 밖에 있기 때문에 신경쓰지 않아도 된다.
- t2의 경우 view frustum안에 있기 때문에 구현해야겠다. GPU rendering pipeline 다음 단계로 넘어감.
- t3의 경우 view frustum 경계에 걸쳐있기 때문에 Clipping을 활용하여 처리를 해주어야겠다.

clipping의 결과물로 일부 vertex들은 무시하고 처리될 수 있지만, 잘라진 선을 기준으로 다른 vertex들이 대체되고, 새로움 삼각형을 구성할 수 있기 때문에, 잘 처리해야겠다.

### Perspective Division

Camera 공간에서 Clip공간으로 옮기는 과정을 projection transform이라 배웠었다. (Fovy, aspect, n, f)

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/projectionTransform.png)

기존의 projection 행렬에 camera space의 좌표 변환을 시키는데, 
- 변환이 완료되면 -z가 마지막 원소로 결과가 나오게 되고, 이를 모든 원소에 나눠주면 저런 식으로 최종 결과를 받아낼 수 있겠다.

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/perspectiveDivision.png)

- P1과 Q1의 점을 보면, 원점에서 멀리 떨어져 있기 때문에, projection transform을 거친 이후 나온 z값(w-value)이 2가 나왔음을 알 수 있다.  

- 이 때, P2와 Q2를 보면, 원점과 가깝기 때문에 변환 결과가 1인 걸 볼 수 있다.

- 이 w-value는 우리가 마지막에 이 z값으로 모든 원소들을 나눠주고 z값을 1로서 고정하는 부분을 통해 **z값이 크면 클수록 나누기한 값이 작아짐**을 확인해볼 수 있고, 이는 결국 **원근법**을 바탕으로 한 연산이 되는 것을 확인할 수 있다.

= Perspective Division이라고 이해하면 되겠다.

### Back-face Culling

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/backfaceCulling.png)

이 참고자료를 보면, t1삼각형의 경우 view frustum안에 들어와 있지만, 시선을 등지고 있음을 확인할 수 있다. 이런 삼각형들(object)들은 어차피 가려져서 보여지지 않기 때문에 구현하지 않아도 되는 것들이니까, 없애버리는 행위를 **culling**이라고 부른다.

- only frontface만 살린다.

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/universal.png)

각 삼각형이 위에서 쳐다보았을 때 1자로 보인다고 가정했을 때, 해당 normal벡터와 각 삼각형의 한 꼭짓점과 원점을 이은 벡터의 내적 값을 통해 이 삼각형면의 위치관계를 파악할 수 있다.
- NDC 공간으로 변환되었을 때는, projection line들이 모두 z축에 parellel하게 변환된다.


<br/>

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/cwCcw.png)

x,y 좌표계로 투영해본다는 과정을 생각해보자.(Universal projection line onto the xy-plane)
- 그런 다음 반시계방향(CCW) 꼭짓점 순서가 있는 2D 삼각형은 **front-face**이고 시계방향(CW) 꼭짓점 순서가 있는 2D 삼각형은 **back-face**라는 것을 알 수 있다.

컴퓨터 내부에서는? 원점과 각 정점을 이은 벡터들을 바탕으로 행렬 곱을 계산할 수 있는데, 
- If it is positive, CCW and so front face,
- If negative, CW and so back face.
- If 0, edge-on face.

<br/>

#### 반투명한 구?

hollow translucent sphere에서는 back-face를 culled하는 것이 아니라, 모든 face들이 다 표현되어야 할 것이다.

#### Front-face Culling?

반대로, front-face를 죽이게 된다면 결국 반구의 평면을 보는 것과 비슷한 장면을 볼 수 있을 것이다.

이런 다양한 상황을 유연하게 작동하기 위해서 API에서 여러가지 option들을 열어두고 있다.
```c
glEnable(GL_CULL_FACE);     //활성화
glDisable();    //활성화를 안시키겠다.

glCullFace(GL_FRONT);   //어떤 face를 날릴꺼야
glCullFace(GL_BACK);    //보통은 back이기 때문에 default임
glCullFace(GL_FRONT_AND_BACK);

glFrontFace(GL_CW)      //어디 방향을 front?
glFrontFace(GL_CCW)     //반시계?
```

<br/>

### Viewport

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/viewport.png)

실제 window안에서 내가 그림을 뿌릴 구역을 우리는 Viewport라고 한다.
- **Clip space → Screen space**로 변환하는 것이 Viewport Transform이라고 한다.

- 이때 screen-space rectangle로 window공간에 projection되는데, 이를 불러오는 viewport 객체는 이런 식으로 구성된다.

```c
void glViewport(GLint minX, GLint minY, GLsizei w, GLsizei h)

//꽉채우는 경우 minX, minY를 0으로 설정하면 됨.
```

실제현상과 비슷하게, 3D space로 생각을 해봐야하는데, 이는 viewport에서도 적용해서 생각해볼 수 있겠다.
- viewport의 깊이를 나타내는 z축을 하나 더 가상해보자.

```c
void glDepthRangef(GLclampf minZ, GLclampf maxZ)
```

<br/>

#### Viewprot Transform

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/viewportTransform.png)

- 2 x 2 x 2 공간의 clip공간에서 시작한다고 가정해보자.

- 이를 직사각형의 screen space로 옮길 때는, w/2, h/2, (maxZ-minZ)/2 만큼의 스케일링이 필요하다. (2 * w/2 = w 이기 때문) Scaling Factor를 이와 같이 잡아주면 될 것 같다.

- 물체의 중심이 원점에 있었던 것이 아니기 때문에, translation의 과정도 필요한데, 이때는 각 요소의 중점들을 찾아주면 되겠다. 너무 쉬운 작업.

이 모든 작업은 가상의 직사각형이나 정사각형에 적용되는 것이 아니라, polygon mesh 이 자체에 작용되는 것을 알아두자.

<br/>

### Scan Conversion

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/linearInterpolation.png)

앞에서 배운 linear interpolation의 원리 그대로 쓴다고 생각하면 되겠다.

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/scanConversion.png)

각각의 삼각형을 fragment세트로 쪼갤 수 있겠다. 각각의 정점들의 normal들과 texture coordinate들을 **interpolation**하는 작업을 Scan Conversion이라고 한다.

먼저 위에서 본 것처럼 각 삼각형의 선분에 대해서 Linear Interpolation이 진행된다.
- 하지만, 그 후에는 수평/수직적으로 나타나는 scan line이라는 것이 pixel을 규정하고 있는데, 각 scan line들이 선분과 만나는 그 좌표가 interpolation 계산에 필요할 것 같다.

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/bilinearInterpolation.png)

삼각형의 각 변에 대해 여러가지 기울기를 계산한다.
![]({{site.url}}/images/2024-09-29-opengl-rasterizer/calDep.png)

각 변에 대해 선형 보간을 통해 정점별 속성을 구한다. 이를 scan line인 삼각형 내의 y = k들이 각 변과 교차하는 교차점에 대해 적용하여 각 교차점 별 R, x 값을 구한다.

이후, 각 변의 정점들에 대해 얻어진 R, x 값을 이용해, 스캔 라인과 x = k들의 교차점들에 대해 R, x를 구한다.

(1) 각 **변에 대한 linear interpolation** + (2) **scan line에 따른 각 점에 대한 linear interpolation**

=>  **Bilinear Interpolation**

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/interpolationExample.png)
*이런 식으로 각 정점의 RGB값에 따라 색깔이 정해질 것이다.*

#### Normal Interpolation

normal도 동일하다. 3차원 좌표 (x, y, z)로 구성되어 있기 때문에, 위에서 RGB로 연산했던 것과 동일하게 진행하면 되겠다.

![]({{site.url}}/images/2024-09-29-opengl-rasterizer/normalInterpolation.png)
*각 픽셀의 위치마다 normal이 보간된다.*

실제 Scan Conversion 과정에서는 색상 값뿐만 아니라 normal, texture Coordinate, 깊이 등 모든 vertex 속성을 보간하여 각 픽셀별 fragment를 생성하게 된다. 이제 rasterization 단계가 완료되고 실제 색상을 결정하는 fragment 처리 단계로 넘어가게 된다.


<br/>


```toc
```