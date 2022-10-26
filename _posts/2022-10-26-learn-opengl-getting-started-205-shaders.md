---
layout: post
title:  "Learn OpenGL Getting started 2-5. Shaders"
categories: opengl
---

- Shader 클래스 코드는 생략

### Shader
  - GPU에 의존하는 작은 프로그램
  - 입력을 출력으로 변환하며, 서로 통신 불가능
      - 유일한 통신은 입출력을 통해서만 가능

### GLSL 코드 구조

- 선언 작성 순서
    1. 버전 설명
    2. 입출력 변수 목록
    3. uniform 변수
    4. 진입점 함수(main())
- 진입점 함수 내부에서, 출력 변수의 값을 대입함으로써 출력값 전달
- OpenGL의 경우, 최소 16개의 4성분 정점 속성 사용이 보장된다

### 자료형

- 주로 사용하는 자료형
    - C에서 사용하는 기본 자료형: `int`, `float`, `double`, `uint`, `bool`
    - 컨테이너: `vector`, `matrix`(뒷장에서 다룸)
- `vector`
    - 요소의 개수별(2, 3, 4), 자료형별로 `vec`를 선언할 수 있다
        - vecn(float), bvecn(bool), ivecn, uvecn, dvecn…
        - 대체로 기본인 vecn(float)을 사용
    - 구성 요소는 `.x`, `.y`, `.z`, `.w`로 접근 가능
        - 이를 활용하여 색상, 텍스처 좌표에서도 응용 가능
    - Swizzling
        
        ```glsl
        vec2 someVec;
        vec4 differentVec = someVec.xyxx;
        vec3 anotherVec = differentVec.zyw;
        vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
        vec2 vect = vec2(0.5, 0.7);
        vec4 result = vec4(vect, 0.0, 0.0);
        vec4 otherResult = vec4(result.xyz, 1.0);
        ```
        

### Uniform: 전역변수

- 데이터를 CPU 상의 application에서 GPU 상의 shader로 옮기는 또다른 방법
- 전역변수이므로, 모든 shader program 별로 동일
    - shader program 에 같이 연결되어 있는 shader에 한해 전역변수로 사용 가능
- 주의) GLSL 코드에서 쓰이지 않는 uniform 변수를 선언하면 컴파일러가 자동으로 제거

```glsl
#version 330 core
out vec4 FragColor;
  
uniform vec4 ourColor; // 이 변수를 OpenGL 변수에서 선언해야 함

void main()
{
    FragColor = ourColor;
}
```

```glsl
// OpenGL 코드 일부
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

- Uniform 함수는 자료형별로 이름이 다르다
    - OpenGL은 C언어 기반이라 함수 오버로딩이 불가능하기 때문

### 예제 코드(속성 추가)

```cpp
float vertices[] = {
    // positions         // colors
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // bottom right
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // bottom left
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // top 
};
```

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;   // position attribute
layout (location = 1) in vec3 aColor; // color attribute
  
out vec3 ourColor; // output a color to the fragment shader

void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor; // set ourColor to the input color we got from the vertex data
}
```

- 위치 메타데이터로 입력 변수를 지정해, CPU에서 정점 속성을 구성할 수 있다: `layout (location = 0)`
    - `glGetAttributeLocation()`에 비해 이해가 쉽고 작업량을 줄일 수 있음

```glsl
#version 330 core
out vec4 FragColor;  
in vec3 ourColor;
  
void main()
{
    FragColor = vec4(ourColor, 1.0);
}
```

![vertex_attribute_pointer_interleaved](https://user-images.githubusercontent.com/42532724/198049585-335f7493-60d7-48af-b2f7-8976c0d32bf1.png)

```cpp
// 현재 형태에 맞게 속성 수정
// position attribute
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
// color attribute
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3* sizeof(float)));
glEnableVertexAttribArray(1);
```
