---
layout: post
title:  "렌더링 API 및 구조(OpenGL 2.0)"
categories: opengl
---

- 개선점
    - 이전: glVertexPointer/glColorPointer(fixed function pipiline)과 VBO를 사용
    - 2.0부터는 fixed pipeline대신 shader를 통해 사용자가 직접 렌더링 구현 가능
        - 현재는 대부분 shader 사용
    - OpenGL에서는 GLSL(OpenGL Shading Language)언어로 shader을 구성함
- `Vertex Shader` , `Fragment Shader`
    - 변수 키워드
        - `attribute`/`varying` , `in`/`out`
            - `attribute`: 밖에서 전달받는 값임을 나타냄
            - `varying`: vertex shader에서 fragment shader로 전달되는 값
            - GLSL 130부터는 `in`/`out` 키워드로 대체
                - `in`/`out`는 모든 shader에서 입력값/출력값의 의미
                - 두 셰이더 사이에 geometry shader를 추가할 수 있게 되면서, 이 키워드로 대체
        - `gl_Position` , `gl_FragColor`
            - 기본으로 정의되어 있는 변수
            - 여기에 값을 대입하면, shader가 자동으로 position과 color 정보로 인식
    - position에서 `vec4`를 사용하는 이유
        - 변환에 유리하기 때문(그래픽스에서는 행렬을 곱하는 방법을 자주 사용함)
        - w=0은  vector, w=1은 point로 생각하면 된다
            - w가 0도 1도 아니라면, x,y,z값을 w로 나누어주면 된다
- 작성 순서
    1. 셰이더 프로그램을 관리할 Shader클래스 생성
    2. .txt 파일로 작성해 둔 shader 프로그램을 Shader에 등록
        - Vertex Shader code
            
            ```glsl
            #version 110
            
            // vertex 속성
            attribute vec3 vertexData;
            attribute vec3 colorData;
            
            // vertex shader -> fragment shader로 전달되는 값
            varying vec3 vertexColor;
            
            void main()
            {
                vertexColor = colorData;
                gl_Position = vec4(vertexData.xyz, 1.0);
            }
            ```
            
        - Fragment Shader code
            
            ```glsl
            #version 110
            
            // vertex shader로부터 값을 전달받음
            varying vec3 vertexColor;
            
            void main()
            {    
                gl_FragColor = vec4(vertexColor, 1.0);
            }
            ```
            
    3. vertex attribute 값을 `glVertexAttributePointer` 를 사용하여 전달
        
        ```cpp
        glVertexAttribPointer(0, 3, GL_FLOAT, false, 0, 0);
        glEnableVertexAttribArray(0);
        // {{렌더링 루프}}
        glDisableVertexAttribArray(0);
        ```
        
        - 사용법은 `glVertexPointer` 과 동일하나, 미리 생성하여 데이터를 담아 둔 VBO를 사용하거나, vertex를 담은 배열을 전달 가능
            - `pointer` 매개변수(맨 마지막)의 값이 0일 경우, 현재 bind된 VBO를 `vertex`, `color` 값이 들어 있는 buffer의 offset으로 간주
        - 전달한 이후에는 `glEnableVertexAttribArray(index)` 를 사용하여 활성화 시켜야 한다
            - 루프가 끝나면  `glDisableVertexAttribArray(index)` 를 사용하여 비활성화
    4. 렌더링 루프 수행
        - Shader 프로그램을 매번 시작/종료시키면서 렌더링 수행
        
        ```cpp
        // shader 프로그램 시작
        shader.start();
        	
        // vertex attribute 활성화
        glEnableVertexAttribArray(0);
        glEnableVertexAttribArray(1);
        	
        // vertex 정보 bind
        glBindBuffer(GL_ARRAY_BUFFER, vertexVBO);
        glVertexAttribPointer(0, 3, GL_FLOAT, false, 0, 0);
        	
        // color 정보 bind
        glBindBuffer(GL_ARRAY_BUFFER, colorVBO);
        glVertexAttribPointer(1, 3, GL_FLOAT, false, 0, 0);
        	
        // {{렌더링 루프 내부}}
        glDrawArrays(GL_TRIANGLES, 0, 3);
        	
        // 언바인드 및 해제
        glBindBuffer(GL_ARRAY_BUFFER, 0);
        glDisableVertexAttribArray(0);
        glDisableVertexAttribArray(1);
        	
        // shader 프로그램 종료
        shader.stop();
        ```