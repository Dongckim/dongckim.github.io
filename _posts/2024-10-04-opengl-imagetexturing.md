---
title: "OpenGL ES 3차원 컴퓨터그래픽스 이미지 텍스쳐링"
layout: single
Typora-root-url: ../
categories: OpenGL
tag: imageTexturing
use_math: true
---

ⓒ 2019. [JungHyun Han](https://media.korea.ac.kr/people/jhan/) Korea University Seoul, All rights reserved.

<br/>


## Fragment Shader

- Lighting
- Texturing

### Texel

텍스쳐의 구성요소를 texture element의 약자인 texcel이라고 부른다.
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/texcels.png)

Texture Coordinate는 polygon mesh의 각 vertex들이 계산된 값을 의미한다.
- Texture Coordinates (s, t)는 texture space에 투영될 fragment를 구성하기 위해서 interpolation이 되는데, 이는 fragment의 color를 결정하게 된다.

- s와 t는 각각 해상도가 계산되어서 최종 결정되어버린다.

- parameter space가 Normalized 되어있기 때문에, Texture Coordinate도 정규화된다는 걸 알 수 있다.

- 정규화된 texture  coordinate는 특정 해상도에 의존하지 않으며 다양하게 연결할 수 있다.

<br/>

### Surface Parameterization
모델링 단계에서 어떻게 정점마다 (s,t)를 할당할까?

![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/surface.png)

얼굴과 같은 3D 작업물을 잘 펴는 방식이 매우 중요하다고 한다. 다만, 이렇게 피는 과정에는 이미 잘 만들어진 알고리즘이 있으니 해당 알고리즘을 사용해서 평평하게 피는 작업을 할 수 있고, 이를 바탕으로 좌표계를 설정할 수 있다고 한다.

![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/patch.png)

- 하나의 물체를 전부 평평하게 만든다기 보다는, Patch라는 하위 단위로 나누어 각각을 평평하게 만들기 시작한다.

- 복잡한 polygon mesh들은 여러 개의 patch들, 그리고 이를 평평하게 한 Chart로 구성된다. 

- 복잡한 차트들을 하나로 모아서 관리하는데 이를 Atlas라고 부른다.

![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/summary.png)

## Texturing in GL

```C
struct Texel {
    GLubyte r;
    GLubyte g;
    GLubyte b;
    GLubyte a;
}

struct TexData {
    std::vector<Texel> texels;
    GLsizei width;
    GLsizei height;
}

TexData texData;

GLuint texture;
glGenTexture(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, texData.wdth, texData.height,
        0, GL_RGBA, GLUNSIGNED_BYTE, texData.texels.data());

```
이전에 `glBundBuffer()`을 배웠던 것처럼, 똑같은 방식으로 Generate → Bind → Data 이 순서로 코드를 작성해나가면 되겠다.

- `glGenTextures()`를 불러와서 texture 오브젝트를 생성해야겠다.

- `glBindTexture()`해당 texture 오브젝트를 불러와서 Bind시킨다.

- 해당 2차원 텍스쳐에 실제 데이터를 넣어주는 방식을 사용한다. `glTexImage2D()`

<br/>

### Texture Wrapping
꼭 [0,1] 범위 내에만 있을 필요는 없다. texture wrapping 모드는 범위를 벗어난 (s,t) 핸들을 처리하게 된다.
- Clamp-to-edge : [0,1] 범위를 벗어난 좌표들을 다시 범위로 맞추는 작업
- repeat : 하나의 페턴으로 작은 텍스쳐가 넓은 면적을 커버할 수 있을 때 반복작업을 실행한다.
- Mirrored-Repeat : 정수경계에서 너무나 경계가 확실히 보이기 때문에 반전시켜서 아주 부드럽게 이미지가 이어지게 만든다.

```C
glTexParameteri(GLenum target,
                GLenum pname,
                Glint param);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT)
```

<br/>

## Texture Magnification & Minification

만약 texture Coordinate가 Texcel 중앙에 떨어지지 않으면 어떡할까? 사실 중앙에 떨어지는 경우가 더 적을 것이다.
- Screen-space quad가 texture보다 크게 나타나 quad에 맞게 Magnified한다고 하자. Texcel보다 더 많은 fragment(pixcel)가 생성될 것이다.
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/magnification.png)

- 반대로, Texcel의 개수가 fragment(pixcel)의 개수보다 적다면 이를 Minification했다고 한다. 말그래도 scale을 줄인 것.
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/minification.png)

### Filtering for Magnification

실제로 RGB값을 어떻게 받아올까?

1. option1: 가장 가까운 texcel을 가져오는 sampling
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/nearest.png)

- 4개의 픽셀은 동일한 텍스쳐값을 가져오게 되어서 blocky image 즉, 모자이크 영상이 만들어진다.

2. option2: Bilineaer Interpolation(선호)
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/bilnear.png)

- (s', t')가 어떻게 계산이 되건, 한 픽셀을 둘러싼 텍셀은 4개가 된다. 각각의 텍셀은 중심점이 있고, 픽셀의 투영 좌표를 이용하면 투영된 픽셀의 상대적인 위치가 계산될 수 있다.

- 길이가 1인 두 텍셀 사이의 상대적인 위치를 볼 수 있다. 프로젝트 된 픽셀의 상대적인 위치

- p하고 1-p가 주어졌을 때, C1과 C2는 선형 interpolation을 할 수 있겠다. 모자이크 방지 가능.

### Filtering for Minification

pixcel 대비 texcel이 너무 많다. 갯수 조정이 필요할 것이다.
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/miniPro.png)

- 높은 해상도의 이미지를 낮은 해상도로 줄일 때에는 픽셀들의 sampling을 했을 때 오류가 날 수 밖에 없다. (Nearest Sampling이든, Bilinear Interpolation이든)

해결책 : Mipmap - down sampling
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/mipmap.png)

- 한 단계씩 줄여가면서 8x8 에서 4x4, 2x2에서 1x1까지 점점 줄여가는걸 의미한다.

- 이때 해상도가 (2^l x 2^l)이라고 가정했을 때, 해당 피라미드는 (l+1)번의 단계를 거쳐서  완성하게 된다.

- Mipmap에서 특정 레벨을 잘 선택하는게 중요한데, 이때 하나의 Level을 Lambda라고 한다.

- 해당 레벨은 texcel카운트와 pixcel카운트가 비슷해질 때 만족하는 Lambda를 고르면 될꺼 같다.

![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/lambdachose.png)

- log2(4) = 2, 2번 거치면 됨.

만약, 완벽하게 나눠떨어지지 않는 경우엔 어떨까?
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/log3.png)

- log2(3)을 하면 답이 1.585가 나온다. 이 때 두가지 옵션을 챙길 수 있는데

1. nearest level로 그냥 sampling하는 방법 (Floor or Ceiling)
2. Floor 랑 ceiling 둘 다를 linear interpolation을 하는 방법. (trilinear Interpolation)
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/trilinear.png)

- Ceiling했을 때 Level1에서의 Bilinear interpolation으로 나오는 C1값과 Floor했을 때 Level2에서의 Bilinear interpolation으로 나오는 C2의 값을 다시 또 interpolation하여서 그 중간의 real c값을 찾는 방식이다.

- 이때 비율은 소숫점 자릿수가 그 비율의 중추가 되겠다.

<br/>

## Texture Filtering in GL

- screen-space pixcel이 주어지면, 컴퓨터는 알아서 footprint를 계산하고, 이를 Magnified할지, Minified할지 정하게 된다.

- Magnification일 때, Minification일 때 각각의 경우 어떻게 필터링할지 정해주기만 하면 된다.

```c
glTexParameteri(GLenum target, GLenum pname, GLint param);
```
이 함수에서 pname만 신경쓰면 되는데,
- `GL_TEXTURE_MAG_FILTER`
- `GL_TEXTURE_MIN_FILTER`
- `GL_TEXTURE_WRAP_S`
- `GL_TEXTURE_WRAP_T`

만약, GL_TEXTURE_MAG_FILTER 이걸 선택했다면, param쪽 파라미터를
- `GL_NEAREST`
- `GL_LINEAR `

이렇게 두개의 파라미터를 받을 수 있다.

만약, GL_TEXTURE_MIN_FILTER 이걸 선택했다면, param쪽 파라미터를,
- `GL_NEAREST` → mipmap을 사용하지 않고 그냥 쌩으로...
- `GL_LINEAR` → mipmap을 사용하지 않고 그냥 쌩으로...
- `GL_NEAREST_MIPMAP_NEAREST` → 가까운 level을 택하는 방식
- `GL_LINEAR_MIPMAP_NEAREST` → 가까운 level을 택하는 방식
- `GL_NEAREST_MIPMAP_LINEAR` → 두가지 level을 택하는 방식
- `GL_LINEAR_MIPMAP_LINEAR` → 두가지 level을 택하는 방식

이렇게 6가지 파라미터를 받을 수 있다. 

## Fragment Shader

`V_normal`과  `V_texCord`를 입력으로 받아서 실행이 된다.
![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/fragmentShader.png)

- vertex shader에서 나온 `V_normal`과 `V_texCord`는 rasterizer에 의해서 interpolation이 될 것이고, 이 후에 나온 output이 똑같이 fragment shader로 입력되는 것이다.

- texturing을 진행하는 fragment shader는 모든 fragment의 texture는 동일하기 때문에 `uniform`이라고 하는 형태로 제공이 된다.

- 결국, v_normal과 v_texCord를 바탕으로 색상을 딱 정해져서 texturing이 진행된다.

```c
#version 300 es

precision mediump float;
uniform sampler2D colorMap;
in vec2 v_texCoord;
layout(location = 0) out vec4 fragColor;

void main(){
    fragColor = texture(colorMap, v_texCoord);

}

```
- precision(정밀도) 
- 2차원 텍스쳐를 의미한다   //line2
- 입력으로 v_texCoord를 받는다  //line3
- 어찌됐든 fragColor 하나를 출력한다.   //line4

v_normal은 왜 안보이지? 사실 안쓰고 texturing만 수행해도 된다. 사실 v_normal은 lighting을 수행할 때 쓰이게 된다. 차이는 다음과 같다.

![]({{site.url}}/images/2024-10-04-opengl-imagetexturing/lighting.png)

 

<br/>


```toc
``` 