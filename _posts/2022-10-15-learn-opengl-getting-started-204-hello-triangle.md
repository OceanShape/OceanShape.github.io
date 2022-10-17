---
layout: post
title:  "Learn OpenGL Getting started 2-4. Hello Triangle"
categories: opengl
---

- 그래픽 파이프라인에 대한 정리는 별도의 페이지로 만들 예정

### Vertex Input
  - 컴퓨터 그래픽스에서의 vertex는 위치 정보뿐만 아니라, 구현하 목적에 맞도록 부가적인 정보를 추가하여 정의
  - NDC(normailzed device doordinates) 범위 안에 float배열로 정의하여 전달
### Vertex Shader/Fragment Shader
  - GPU에게 정점 데이터를 어떻게 처리해야 하는지 지시
  - Vertex Shader
      - 순서
          1. GPU에 정점 데이터를 저장할 공간의 메모리를 할당
          2. OpenGL이 어떻게 메모리를 해석할 것인지 구성하고 데이터를 어떻게 그래픽 카드에 전달할 것인지에 대해 명시
          3. vertex shader가 명시한 만큼의 정점들을 메모리에서 처리
          - VBO(vertex buffer objects)라고 불리는 것을 통해 이 메모리를 관리
              - 가능한 많은 데이터를 한 번에 보내, 처리 속도를 올리기 위해
      
      ```glsl
      #version 330 core
      layout (location = 0) in vec3 aPos;
      
      void main()
      {
          gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
      }
      ```
      
  - Fragment Shader
      - 픽셀의 출력 컬러 값을 계산하는 것에 관한 쉐이더
      
      ```glsl
      #version 330 core
      out vec4 FragColor;
      
      void main()
      {
          FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
      }
      ```
      
  - 이후 두 셰이더 프로그램을 연결
  
    ```cpp
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    ```
      
### Linking Vertex Attributes

- OpenGL은 메모리 상의 정점 데이터를 어떻게 해석해야하는지 모른다
    - 정점 데이터를 vertex shader의 속성들과 어떻게 연결해야 하는지도 모름
    - OpenGL에게 이러한 것들을 알려주어야 한다
- Vertex shader는 모든 입력을 정점 속성으로 지정할 수 있도록 해줌
    - 따라서 입력 데이터의 어느 부분이 vertex shader의 어떠한 정점 속성과 맞는지 직접 지정해야 한다
    
    ![vertex_attribute_pointer](https://user-images.githubusercontent.com/42532724/196231020-f6b7ac73-5825-42bc-a0f3-2cc082a4f835.png)
    
    ```cpp
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    ```
    
    - glVertexAttribPointer()
        - 설정할 vertex 속성
            - Vertex shader에서 `layout (location = 0)` 코드를 사용했으므로,  vertex 속성의 location을 0으로 지정
        - vertex 속성의 크기
            - `vec3` 타입이므로 `3`개의 값
        - 데이터의 타입
            - GLSL에서 `vec`는 실수형 점으로 이루어진다
        - 데이터를 정규화 여부
            - GL_TRUE로 설정하면 `0`(부호를 가진 데이터라면 `-1`)과 `1` 사이에 있지 않는 값들의 데이터들이 그 사이의 값들로 매핑
        - stride라고도 불리며 연이은 vertex 속성 세트들 사이의 공백
            - 다음 포지션 데이터의 세트는 정확히 `float` 타입 3개의 크기 뒤에 떨어져 있습니다. 즉, 여기서는 `3 * sizeof(float)`을 stride로 지정
            - 배열이 빽빽히 채워져 있어, 다음 vertex 속성 값 사이에 공백이 없다면, stride를 `0`으로 지정 가능
                - OpenGL이 알아서 stride를 지정하게 할 수 있다
                - 만약 또다른 vertex 속성들이 있다면 공간을 정의해야 한다
        - 버퍼에서 데이터가 시작하는 위치의 offset
            - `void*` 타입이므로 형변환이 필요
            - 현재는 위치 데이터가 데이터 배열의 시작 부분에 있기 때문이 `0`으로 지정
- OpenGL에게 vertex 데이터 해석방법을 지정한 후에는,  `glEnableVertexAttribArray()` 의 파라미터로 vertex 속성 location를 전달하고 호출하여 vertex 속성을 사용할 수 있도록 함
    - Vertex 속성은 기본적으로 사용하지 못하도록 설정되어 있다
- 각 vertex 속성은 VBO에 의해 관리되는 메모리로부터 데이터를 받음
    
    ![vertex_array_objects](https://user-images.githubusercontent.com/42532724/196231015-d589b531-e701-498d-87b7-3dcdd119ac11.png)
    
- OpenGL의 오브젝트를 그리는 것은 다음과 같은 형식을 취한다
    
    ```cpp
    // 0. vertex가 저장된 배열을 OpenGL이 사용할 수 있도록 복제
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    // 1. vertex attribute pointer 설정
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);  
    // 2. 셰이더 프로그램 사용을 설정한 뒤 도형 렌더링
    glUseProgram(shaderProgram);
    ```
    
### Element Buffer Objects
  - vertex의 중복을 막을 수 있음
  - 핵심: 삼각형 두 개로 사각형을 그리는 경우 → 4개의 정점만 저장
      - 즉, 고유한 정점만 저장하고, 그리는 순서를 지정하면 된다
  - EBO는 VBO처럼 버퍼
  - EBO에는 그리는 순서를 저장(indices)
      - 여전히 VBO에 vertex를 저장한다
  
  ```cpp
  // ..:: 초기화 코드 :: ..
  // 1. Vertex Array Object 바인딩
  glBindVertexArray(VAO);
  // 2. OpenGL이 사용하기 위해 vertex 리스트를 vertex 버퍼에 복사
  glBindBuffer(GL_ARRAY_BUFFER, VBO);
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
  // 3. OpenGL이 사용하기 위해 인덱스 리스트를 element 버퍼에 복사
  glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
  glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
  // 4. 그런 다음 vertex 속성 포인터를 세팅
  glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
  glEnableVertexAttribArray(0);  
  
  // ...
    
  // ..:: 드로잉 코드 (렌더링 루프 내부) :: ..
  glUseProgram(shaderProgram);
  glBindVertexArray(VAO);
  glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0)
  glBindVertexArray(0);
  ```

출처

- [https://learnopengl.com/Getting-started/Hello-Window](https://learnopengl.com/Getting-started/Hello-Window)