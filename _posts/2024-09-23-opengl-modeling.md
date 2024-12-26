---
title: "OpenGL ES 3차원 컴퓨터그래픽스 모델링"
layout: single
Typora-root-url: ../
categories: OpenGL
tag: modeling
use_math: true
---

ⓒ 2019. [JungHyun Han](https://media.korea.ac.kr/people/jhan/) Korea University Seoul, All rights reserved.

<br/>


## Modeling

모델링이란, 그래픽스에서 렌더링할 물체를 만들어내는 것을 모델링이라고 한다.
- 우리가 흔히 아는 것은, 음함수 구의 방정식을 이용해서 구를 모델링하는 것이다.
- 문제는, GPU는 음함수를 처리하는데에 매우 적합하지 않게 설계되어있다. (추후에 이유는 알아보자.)

![]({{site.url}}/images/2024-09-23-opengl-modeling/modeling.png)

구의 방정식으로 모델링하는 방법 대신, 구 위의 여러가지의 정점들을 샘플링을 한다. 
- 해당 정점들을 다각형으로 이루어져서 만들어 낸 것을 polydon mesh라고 한다.
- GPU는 반대로, 이런 폴리곤 매쉬를 처리하는데에 매우 적합하다고 하자 (왜 적합할까?)

<br/>

## 1. Polygon Mesh
- 가장 간단한 polygon mesh는 삼각형으로 이루어진 triangle mesh이다.
- 우리가 배우는 OpenGL ES에서는 삼각형의 triangle mesh만 처리를 한다.

삼각형 메쉬는 보통 꼭짓점 갯수의 2배만큼의 삼각형이 생성된다. (오일러 공식 증명)

![]({{site.url}}/images/2024-09-23-opengl-modeling/triangleToQuad.png)

<br/>

### 정점의 갯수? 
polygon mesh에서의 정점의 갯수를 우리는 해상도(resolution)라고 일컫는다. = Level of Detail (LOD)
- Resolution 높아지면 당연히 원래 주어진 표현과 가깝게 묘사되겠지만, 처리되는데에 시간이 오래걸린다 (적당한 trade off 필요)


![]({{site.url}}/images/2024-09-23-opengl-modeling/resolution.png)
*LOD 컨트롤은 그래픽스에서 중요한 주제 중에 하나라고 볼 수 있음.*

<br/>

### Polygon Mesh의 원리

![]({{site.url}}/images/2024-09-23-opengl-modeling/2dpolygon.png)

- t1 삼각형의 세 꼭짓점을 배열로 나타낸다. [(0,0), (1,0), (0,1)]
- 이런 Vertex Array는 중복된 값이 많아져서 매우매우 비효율적이고 낭비가 심하다. 
- 정점을 중복없이 나열할 수 있는 방법은 없을까?

**Index Array**

![]({{site.url}}/images/2024-09-23-opengl-modeling/indexarray2.png)

- vertex 정보를 담은 셀은 중복없이 하나의 세트로 array를 구성하고, 해당 인덱스를 가지고 index array로 polygon mesh의 삼각형의 구성정보를 표현하는 방식이 기본적인 원리이다.

- index array에는 각 cell마다 2byte나 4byte정도 밖에 안되는 integer값이 들어가 있는 반면,

- vertex array에는 각 꼭짓점의 위치정보 외에도 구체적인 다양한 정보들이 들어가 있기 때문에, cell 하나하나가 매우 무겁다고 생각하면 될 것 같다. 그래서 중복을 기피하는 듯?

<br/>


## 2. Surface Normals(법선 벡터)

각 삼각형마다의 법선벡터는 그래픽스에서 중요한 역할을 한다.

< p1, p2, p3 > (counter-clock wise) 라고 한 삼각형의 꼭짓점이 주어졌다고 가정해보자.
- p1 에서 p2로 가는 벡터
- p1 에서 p3로 가는 벡터

![]({{site.url}}/images/2024-09-23-opengl-modeling/crossproduct.png)

두가지 벡터를 설정하고, **Cross Product**를 이용해서 수직인 벡터를 구할 수 있고,
- 삼각형의 두 변과 수직이면 삼각형 평면과도 수직으로 정의할 수 있기 때문에, 이를 Surface Normals로 정의한다.

* 컴퓨터 그래픽스에서는 모든 Surface Normals를 길이가 1인 단위벡터로 설정하도록 한다. 따라서 자기 **자신의 길이로 나누는 Normalization 과정**을 거쳐서 마무리지어주어야 한다.

- 컴퓨터 그래픽스에서는 Surface Normals를 **무조건 물체 바깥을 향하도록** 하는 것을 원칙으로 하기 때문에, 삼각형의 **점 3개를 counter-clock wise 방향**으로 설정하는 것을 주의해야겠다.

- 시작 vertex는 딱히 중요하지 않다. 어떤 것을 설정하든, 3개의 vertex를 counter-clock wise 방향으로 나열하기만 한다면, 모두 유효할 것이다.
 
<br/>


### Vertex Normals

정점 법선 벡터라는게 사실 말이 안되긴 한다.

- 샘플링 된 폴리곤 메쉬 구 말고 원래의 부드러운 구라고 생각해보면, Tangent Plane을 생각해볼 수 있다. 해당 Tangent Plane과 구가 만나는 유일한 한 점을 생각해보면, Tangent Plane의 법선 벡터가 결국 Vertex Normal이라고 정의할 수 있겠다.

하지만, Sampling된 Polygon mesh 구에서는 어떻게 구해야 할까? (Smooth하지 않음.)
![]({{site.url}}/images/2024-09-23-opengl-modeling/vertexnormal.png)

- 해당 정점을 공유하는 모든 샘플링된 triangle들의 Surface Normal을 모두 구해서 평균을 내는 방식을 사용한다고 한다. (정규화과정은 필수)


## .obj 파일

모델링 된 파일을 유니티와 같은 게임엔진에서 import하기 위해서는 .obj파일을 알아야 한다.

![]({{site.url}}/images/2024-09-23-opengl-modeling/obj.png)

1.  v 테이블을 보면, 앞에서 x,y,z 좌표를 나타낸다. (26개)

2.  vn 테이블을 보면 앞에서부터 벡터값을 x,y,z로 나타낸다. (26개)

> vertex normal과 vertex의 갯수가 같다?
- "구" 라는 특별한 케이스이기 때문

- 평면을 생각해보면 여러지점에서 **같은 법선벡터가 생성되어 중복**되기 때문에, 갯수 자체는 더 적다. 

3. f (face) 테이블을 보면, `(vertex position // vertex normal position)` 의 형식으로 요소가 이루어져있다.


mesh는 해당 .obj 파일을 export해서 유니티와 같은 게임엔진이 import받았을 때, vertex array와 index array를 구성하게 된다.

![]({{site.url}}/images/2024-09-23-opengl-modeling/unityimport.png)

- vertex array에서는 중복된 position값을 가져오지 않아, 26개의 cell을 구성하게 된다.

- index array는 각각의 모든 삼각형의 구성요소를 표현하기 때문에 144 (48 * 3)개의 cell을 가지게 된다.


<br/>


```toc
```