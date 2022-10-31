---
layout: post
title:  "Learn OpenGL Getting started 2-6. Textures"
categories: opengl
---

- 텍스처는 개체의 세부 사항을 추가하는 데에 사용하는 2D 이미지
    - 많은 정점을 사용하지 않고도 사실감을 살릴 수 있다
- 이때, 모델에 텍스처를 입히는 작업을 텍스처 매핑이라고 한다

### Texture Sampling: 가져오기

- 각 정점(그리고 fragment)은 텍스처 이미지의 어떤 부분을 샘플링할지 지정하는 하나의 텍스처 좌표를 가져야 한다
    - fragment: 하나의 픽셀을 렌더링하기 위해 필요한 모든 데이터
    - 텍스처 매핑 해둔 정보를 바탕으로, 좌표값에 해당하는 텍셀의 정보를 읽어오는 과정을 **텍스처 샘플링**이라고 한다
        - 이후에는 Fragment Interpolation을 통해 나머지를 채움
- 텍스처 샘플링은 다양한 방법으로 수행 가능하기에, OpenGL에게 알려주어야 한다

### Texture Wrapping

- 텍스처 좌표인 (0, 0) ~ (1, 1)을 벗어나면 어떻게 될까?
    - 기본: OpenGL에서는 이미지를 반복한다(좌표의 정수 부분을 무시함)
    - 하지만 옵션에 따라 미러링, 패턴 늘이기, 테두리 지정 등의 방법도 있다
    
    ```cpp
    // s와 t축 모두에 대해 구성하여야 한다
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
    // GL_CLAMP_TO_BORDER일 경우: 테두리 설정
    float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
    glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
    ```
    
- 텍스처 축은 s, t, r(3차원일 경우)로 표현

### Texture Filtering(Sampling의 일부): 결합하기

- (샘플링 결과를 위해) 텍셀(혹은 텍셀 그룹)을 가져와 결합하는 과정 → 샘플링의 일부
- OpenGL은 특정 텍셀이 어떤 텍스처 좌표인지 찾아야 한다
    - 이유: 프래그먼트가 가지고 있는 텍스처의 좌표값은 픽셀의 중심점을 가지고 계산한, 넓이가 없이 위치만 있는 수학적 점 → 보통 정수가 아닌 소수의 형태
    - 반면 텍셀은 각각 넓이와 형태를 가지고 있고, 정수 좌표로 되어있기 때문에, 좌표값에 해당하는 텍셀을 곧바로 가져다 사용할 수 없다
        - 어떻게든 정수 좌표로 변환하거나, 아니면 보간을 통해 좌표값에 걸맞는 적절한 데이터를 만들어내서 프래그먼트 셰이더로 건네줄 필요가 있음
        - 이 방법을 바로 텍스처 필터링이라고 부른다
- 이 기능은 오브젝트가 크거나 이미지 해상도가 낮을 때 유효
- 대표적인 텍스처 필터링(GL_NEAREST, GL_LINEAR)
    - GL_NEAREST: 중심이 텍스처 좌표에 가장 가까운 텍셀 선택
        
        ![filter_nearest](https://user-images.githubusercontent.com/42532724/199012661-12e150c9-83e1-4510-b705-1a75ac96fcf2.png)
        
    - GL_LINEAR: 인접 텍셀에서 보간된 값을 가져옴
        - 중심이 텍스처 좌표와 가까운 텍셀일수록 색상 반영도가 큼
        
        ![filter_linear](https://user-images.githubusercontent.com/42532724/199012675-c9215d8e-5959-48f1-b0b1-ee3eaa2b37a1.png)
        
    - 둘의 사용상 차이점
        
        ![texture_filtering](https://user-images.githubusercontent.com/42532724/199012684-8bba08f5-052b-4380-b63f-cc89e84dd171.png)
        
        - 어떤 모양을 선호하는지에 따라 달라진다
- 사용법
    - 축소/확대된 텍스처별로 필터링 설정 가능
        - 두 옵션 모두에 대한 필터링을 지정해야 한다
    
    ```cpp
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    ```
    

### Mipmaps(밉맵스)

- 렌더링 속도를 향상시키기 위한 목적으로, 기본 텍스처와 이를 연속적으로 미리 축소시킨 텍스처들로 이루어진 비트맵 이미지의 집합
- 사용 이유: 성능 향상
    - 텍스처가 원래 크기보다 멀거나 작게 보일 경우, 축소된 텍스처를 렌더링에 대신 사용
        
        → 렌더링에 사용하는 텍셀의 수가 줄어듦
        
        - 샘플링에 사용하는 캐시 메모리를 줄일 수 있다
- OpenGL의 경우, 텍스처를 만든 후 `glGenerateMipmap()`를 호출해 밉맵을 만들 수 있다

- 밉맵 필터링 설정
    - 렌더링 도중 밉맵의 단계를 바꿀 때, atrifacts(시각적 결함)가 발생할 수 있다
        
        → 일반적인 texture filtering과 마찬가지로, 필터링 옵션를 적용하여 해결할 수 있음
        
    - 필터링 옵션
        - GL_NEAREST_MIPMAP_NEAREST, GL_LINEAR_MIPMAP_NEAREST, GL_NEAREST_MIPMAP_LINEAR, GL_NEAREST_MIPMAP_LINEAR
            - 앞의 GL옵션은 texture filtering과 동일
            - MIPMAP_NEAREST: 픽셀 크기와 가장 가까운 밉맵 사용
            - MIPMAP_LINEAR: 픽셀 크기와 가장 가까운 두 밉맵 사이를 선형으로 보간
        
        ```cpp
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        ```
        
    - 주의) 텍스처 확대시에는 밉맵을 쓰지 않으므로, 밉맵 필터링 옵션을 넣으면 오류 발생

### 텍스처 로드 및 생성

- 편리하게 로드할 수 있도록, `stb_image` 외부 라이브러리 사용
    - 아래 코드가 들어간 cpp파일 생성
    
    ```cpp
    #define STB_IMAGE_IMPLEMENTATION
    #include "stb_image.h"
    ```
    
    - `stb_image.h`인클루드하여 사용

```cpp
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
// 현재 바인딩된 텍스처에 대해 wrapping/filtering 옵션 설정
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);	
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

// 텍스처 로드 및 생성
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
if (data)
{
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
}
else
{
    std::cout << "Failed to load texture" << std::endl;
}
stbi_image_free(data);
```

- `glTexImage2D()`
    - 첫 번째: 텍스처 타겟
        - GL_TEXTURE_2D 의미: GL_TEXTURE_2D로 바인딩된 텍스처 객체에 텍스처 생성
            - GL_TEXTURE_1D나 GL_TEXTURE_3D로 바인딩된 객체는 영향 없음
    - 두 번째: 생성하는 텍스처의 mipmap 레벨을 수동으로 지정할 때 사용(아닐시 0)
    - 세 번째: OpenGL에게 저장하고 싶은 텍스처의 포맷을 전달
    - 네 번째, 다섯 번째: 결과 텍스처의 너비와 높이
    - 여섯 번째: 항상 0
    - 일곱 번째, 여덟 번째: 원본 이미지의 포맷과 데이터타입 지정
        - chars(bytes)는 GL_UNSIGNED_BYTE와 동일
    - 아홉 번째: 실제 이미지 데이터

### 텍스처 적용

```cpp
float vertices[] = {
     // 위치              // 컬러             // 텍스처 좌표
     0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // 우측 상단
     0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // 우측 하단
    -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // 좌측 하단
    -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // 좌측 상단
};
```

1. 텍스처 관련 vertex attribute 추가
    
    ```cpp
    glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
    glEnableVertexAttribArray(2);
    ```
    

![vertex_attribute_pointer_interleaved_textures](https://user-images.githubusercontent.com/42532724/199012695-da227cf0-d39c-4d2c-a79b-2fdeff43e9d0.png)

1. vertex shader 및 fragment shader 수정
    
    ```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aColor;
    layout (location = 2) in vec2 aTexCoord;
    
    out vec3 ourColor;
    out vec2 TexCoord;
    
    void main()
    {
        gl_Position = vec4(aPos, 1.0);
        ourColor = aColor;
        TexCoord = aTexCoord;
    }
    ```
    
    ```glsl
    #version 330 core
    out vec4 FragColor;
      
    in vec3 ourColor;
    in vec2 TexCoord;
    
    uniform sampler2D ourTexture;
    
    void main()
    {
        FragColor = texture(ourTexture, TexCoord);
    }
    ```
    
    - GLSL은 `sampler` 라는 텍스처 데이터 타입을 가지고 있음
        - `sampler2D`를 uniform으로 사용하여 fragment shader에 텍스처 전달
    - fragment shader에는 `texture()` 함수를 사용하여 텍스처를 샘플링 한다
        - 첫 번째: sampler
        - 두 번째: texture 좌표축
    - Fragment Shader 출력: (보간된) 텍스처 좌표의 위치에서 필터링 된 색상
2. `glDrawElements()` 함수 호출 전, 텍스처 바인딩
    - 이후 텍스처가 fragment sampler로 자동 할당됨

### 텍스처 여러 개 사용하기(Texture Units)

- 하나일 때에는 `glUniform()` 를 사용하지 않아도 `sampler2D` 변수를 `uniform` 으로 쓸 수 있었다
    - 기본 텍스처 유닛의 위치로써 0을 제공
        - 그래픽 드라이버에 따라, 제공하지 않아 렌더링이 안될 수도 있다
- 실제로 `glUniform1i()` 함수로 위치값을 `sampler` 에게 할당하면, Fragment Shader 내부에서 여러 개의 텍스처 사용 가능
    - 이 텍스처 위치값을 Texture Unit이라고 한다

- `sampler` 에 Texture Unit을 할당한 후, 활성화하면 여러 텍스처를 동시에 사용할 수 있다
    - OpenGL은 16개 이상의 Texture Unit을 가지고 있다
        - `GL_TEXTURE0` ~ `GL_TEXTURE15` 까지는 반드시 사용 가능
        - 순서대로 선언되어 있어 `GL_TEXTURE8` 를 `GL_TEXTURE0 + 8` 처럼 접근할 수 있다

```cpp
glActiveTexture(GL_TEXTURE0); // 텍스처를 바인딩하기 전에 먼저 텍스처 유닛을 활성화
glBindTexture(GL_TEXTURE_2D, texture);
```
