---
layout: post
title:  "Learn OpenGL Getting started 2-7. Transformations"
categories: opengl
---

- 벡터와 행렬에 대한 기본 개념은 생략(변환부터 진행)

### Transformation

- Scaling
    
    ```
    S1  0  0  0    x    S1⋅x
     0 S2  0  0  ⋅  y =  S2⋅y
     0  0 S3  0    z    S3⋅z
     0  0  0  1    1      1
    ```
    
- Translation
    
    ```
    1  0  0  Tx    x    x+Tx
    0  1  0  Ty  ⋅  y =  y+Ty
    0  0  1  Tz    z    z+Tz
    0  0  0   1    1      1
    ```
    
    - Homogeneous coordinates: 동차 좌표계
        - 특정한 목적을 위해 특정 좌표에 한 차원을 추가하여 표현하는 좌표계(n → n+1)
        - 여기서 추가하는 벡터의 w 요소는 동차 좌표라고도 부른다
            - 동차 벡터에서 3d벡터를 얻기 위해선 x, y, z의 값을 w값으로 나눈다
                - 하지만 대부분 `w == 1`
        - 동차 좌표계의 장점
            1. 3D벡터에서 행렬 변환 수행 가능
            2. 3D perspective(원근)을 만들 수 있다
        - 동차 좌표가 `0` 일 경우, 일반적으로 방향 벡터라고 부른다(변환이 불가능하기 때문)
- Rotation
    - angle은 degree 혹은 radian으로 표현
        - 회전 함수에서는 radian을 많이 쓰지만, 아래 공식을 활용해 degree→radian 변환 가능
        
        ```
        angle in degrees = (angle in radians) * (180 / PI)
        angle in radians = (angle in degrees) * (PI / 180)
        PI ≒ 3.14159265359...
        ```
        
    - x, y, z 축 순서대로 결합하여 위치 벡터를 변환할 수 있다
    
        <center><img src="https://user-images.githubusercontent.com/42532724/199253270-7858b66a-4000-4919-bfae-0dcbc7c1369e.png"  width="60%"/></center>
    
    - 문제점: Gimbal lock이 발생할 수 있음
        - quaternions(쿼터니언)을 사용하면 해결할 수 있다(여기서는 다루지 않음)
    - 해결책(완벽하지 않음): 임의의 단위 축을 곱하여 한 번에 계산
        
        <center><img src="https://user-images.githubusercontent.com/42532724/199253284-6914a9a7-109b-4f2d-a7e8-f1422e9ff122.png"  width="80%"/></center>
        
- Combining matrices
    - 변환할 때 행렬을 사용하는 이유: 여러 변환을 하나의 행렬에 합칠 수 있음
    - 행렬 곱은 교환 법칙이 성립하지 않으므로, 순서가 중요(실제로 곱하는 순서는 반대)
        
        ```
        **Translation ⋅ Rotation ⋅ Scaling  ⋅ (target)**
        ```
        

### In practice

- OpenGL에서는 GLM 라이브러리를 통해 수학 개념을 간단하게 사용할 수 있다
    - GLM은 헤더만 있어, 별도의 컴파일과 링킹이 필요 없음
    - 대부분의 기능은 아래 3개의 헤더 파일에서 찾을 수 있다
    
    ```cpp
    #include <glm/glm.hpp>
    #include <glm/gtc/matrix_transform.hpp>
    #include <glm/gtc/type_ptr.hpp>
    ```
    
- 사용법 예시(항상 순서에 주의)
    1. 행렬 생성
        
        ```cpp
        glm::mat4 trans = glm::mat4(1.0f); // 초기화를 잊지 말자
        trans = glm::rotate(trans, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
        trans = glm::scale(trans, glm::vec3(0.5, 0.5, 0.5));
        ```
        
        - z축 방향으로 90도 회전, 0.5 스케일 적용
        - glm 함수가 행렬을 곱함으로써, 모든 변환이 조합된 변환 행렬 생성
        - 변환은 매번 실행되어야 하므로, 이 코드는 렌더링 루프 내부에 작성
        - 참고) scals의 vec요소가 음수일 경우, 해당 방향으로 뒤집힌 이미지가 출력됨
    2. vertex shader에 uniform mat4 타입을 사용하여 변환 행렬 전달
        
        ```glsl
        #version 330 core
        layout (location = 0) in vec3 aPos;
        layout (location = 1) in vec2 aTexCoord;
        
        out vec2 TexCoord;
          
        uniform mat4 transform;
        
        void main()
        {
            gl_Position = transform * vec4(aPos, 1.0f);
            TexCoord = vec2(aTexCoord.x, aTexCoord.y);
        }
        ```
        
        ```cpp
        // 행렬 변환 코드와 마찬가지로 렌더링 루프 내부에 작성
        unsigned int transformLoc = glGetUniformLocation(ourShader.ID, "transform");
        glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
        // 여기서의 trans는 &trans[0][0]와 같은 의미
        ```
        
        - `glUniformMatrix4fv()`
            
            
            | 파라미터, 리턴 | 설명 |
            | --- | --- |
            | GLuint location | 수정할 uniform 변수 위치 |
            | GLsizei count | 수정할 행렬 개수(2개 이상일 경우, 배열로 받음) |
            | GLboolean transpose | 행렬 전치 여부 |
            | const GLfloat *value | 행렬 데이터 |
            - `glm::value_ptr()` 사용하는 이유: glm의 행렬을 저장 방법을 특정할 수 없기 때문
