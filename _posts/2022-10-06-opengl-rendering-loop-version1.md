---
layout: post
title:  "버전별 렌더링 루프(OpenGL1)"
categories: opengl
---

OpenGL의 렌더링 방식과 사용하는 API, 구조는 버전을 거치며 많은 변화가 이루어졌다. 모두 사용할 일은 없지만 적어도 레퍼런스의 버전은 알 수 있어야 한다고 생각했고, 렌더링 루프를 공부할 겸 버전별 변화와 추가된 개념을 정리하기로 했다.

- OpenGL버전별 코드 실행시 주의사항
    - 이전 버전의 렌더링을 테스트할 때에는, 아래 네 줄 코드를 반드시 지우고 진행할 것
    
    ```cpp
    // 윈도우는 생성되지만, 도형이 그려지지 않음
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    // 윈도우 생성 자체가 되지 않음
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
    ```
    - VBO 항목의 코드는 테스트 불가능함
      - 현재 모든 코드는 최신버전(4.6)에서 진행하고 있는데, API(예를 들면 `glBufferData()`)의 사용법이 바뀌어서라고 추측된다. 결국, 정확한 이유는 찾지 못했다.
      - 하지만, 정리의 목적이 외부 자료의 OpenGL버전을 알기 위함이고, 굳이 찾아낼 필요가 없을 정도로 충분히 옛날 버전이라고 보았다. 따라서, VBO 항목은 흐름을 정리만 해두고 넘어가기로 했다.
    
- glBegin/glEnd
    - 각 점의 좌표값과 색을 매 프레임마다 CPU에서 GPU로 넘겨주는 방식
        - 현재 기준으로는 비효율적인 방식
    - cull을 사용할 경우 우리를 향하지 않는 면은 그리지 않으므로 순서에 신경을 써야한다
        - 점 순서가 반시계 방향으로 돌아가는 게 우리를 향하는 삼각형
        - cull이 궁금하다면 [Face culling](https://heinleinsgame.tistory.com/27) 참고
    
    ```cpp
    // {{렌더링 루프 내부}}
    glBegin(GL_TRIANGLES);
    glColor3f(1f, 0f, 0f);
    glVertex3f(0, 0.5f, 0f);
    glColor3f(0f, 1f, 0f);
    glVertex3f(-0.5f, -0.5f, 0f);
    glColor3f(0f, 0f, 1f);
    glVertex3f(0.5f, -0.5f, 0f);
    glEnd();
    ```
    
    - glVertex, glColor를 매번 불러줘야 한다
- glDrawArray
    - 각 점의 좌표값/색에 대한 정보를 배열로 한 번에 보낸다.
    - 장점: glVertex, glColor의 호출 회수를 줄인다
    - 개선점: 여전히 매 프레임마다 CPU의 데이터가 GPU로 넘어가야 한다
      - 매 프레임마다 전달 받은 위치(포인터)를 기반으로 CPU에 저장된 데이터를 GPU로 옮김
    
    ```cpp
    // GL_VERTEX_ARRAY 와 GL_COLOR_ARRAY 활성화
    // 점과 색상 데이터를 넘겨줄 예정임을 미리 알림
    glEnableClientState(GL_VERTEX_ARRAY);
    glEnableClientState(GL_COLOR_ARRAY);
    
    // 점과 색상 데이터의 실제 위치를 포인터로 알려준다
    // vertices와 colors는 배열의 이름
    glVertexPointer(3, GL_FLOAT, 0, vertices);
    glColorPointer(3, GL_FLOAT, 0, colors);
    
    // {{렌더링 루프 내부}}
    // 데이터를 기반으로 그려냄
    glDrawArrays(GL_TRIANGLES, 0, vertices.length/3);
    ```
- VBO
    - 목적: 데이터가 그대로라면, 처음 한 번만 GPU에 가져다 놓고 계속 사용하자!
        - `glVertexPointer`, `glColorPointer`에 CPU상의 데이터 대신, 미리 GPU의 VBO에 넣어 놓은 데이터의 포인터를 전달
    - `VBO(Vertex Buffer Object)`
        - 위치/색상 등의 정보를 담고 있는 <u>GPU상의 메모리 버퍼 객체</u>
    
    ```cpp
    int vertexVBO, colorVBO;
    
    vertexVBO = glGenBuffers(); // VBO 생성
    glBindBuffer(GL_ARRAY_BUFFER, vertexVBO); // 버퍼에 타입 바인딩 bind
    // 버퍼에 데이터(여기서는 vertices) 담기
    glBufferData(GL_ARRAY_BUFFER, vertices, GL_STATIC_DRAW);
    glBindBuffer(GL_ARRAY_BUFFER, 0); // unbind
    
    colorVBO= glGenBuffers(); // VBO 생성
    glBindBuffer(GL_ARRAY_BUFFER, colorVBO); // bind
    glBufferData(GL_ARRAY_BUFFER, colors, GL_STATIC_DRAW); // 데이터 넣기
    glBindBuffer(GL_ARRAY_BUFFER, 0); // unbind
    ```
    
    - `glBindBuffer(GL_ARRAY_BUFFER, vertexVBO)`
        - `void glBindBuffer(GLenum target, GLuint buffer)`
        - GL_ARRAY_BUFFER 키워드는 Vertex attributes를 뜻함([https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBindBuffer.xhtml](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBindBuffer.xhtml) 참고)
    
    ```cpp
    glBindBuffer(GL_ARRAY_BUFFER, vertexVBO);
    // 맨 마지막 pointer가 0이면 현재 array buffer에 bind된 값을 기준으로 생각함.
    glVertexPointer(3, GL_FLOAT, 0, 0);
    		
    glBindBuffer(GL_ARRAY_BUFFER, colorVBO);
    glColorPointer(3, GL_FLOAT, 0, 0);
    		
    glDrawArrays(GL_TRIANGLES, 0, 3); // 3 == vertices.length / 3
    
    glBindBuffer(GL_ARRAY_BUFFER, 0); // unbind
    ```
    
    - `vertexVBO` , `colorVBO`를 다시 바인딩하는 이유
        - `glVertexPointer` , `glColorPointer`의 기능을 활용하여 데이터를 불러오기 위함
            - `pointer` 매개변수(맨 마지막)의 값이 0일 경우, 현재 array buffer에 bind된 VBO를 `vertex`, `color` 값이 들어 있는 buffer의 offset으로 간주
            - 즉, 현재 VBO의 값을 읽어들임
- 참고: [https://m.blog.naver.com/cjdeka3123/220947181159](https://m.blog.naver.com/cjdeka3123/220947181159)
