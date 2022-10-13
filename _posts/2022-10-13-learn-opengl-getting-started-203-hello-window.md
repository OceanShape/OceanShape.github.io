---
layout: post
title:  "Learn OpenGL Getting started 2-3. Hello Window "
categories: opengl
---

### 기본 화면을 출력하는 렌더링 루프

1. GLFW 초기화 및 옵션 설정
    
    ```cpp
    // GLFW 초기화
    glfwInit();
    // 3.3버전을 사용한다는 사실을 알려줌(해당 버전의 라이브러리가 없으면 멈춤)
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    // core-profile을 사용한다는 사실을 알려줌
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    //glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE); // for Mac
    ```
    
    - GLFW의 설정값: [https://www.glfw.org/docs/latest/window.html#window_hints](https://www.glfw.org/docs/latest/window.html#window_hints)
2. Window 객체 생성
    
    ```cpp
    // Window 객체 생성
    GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    // 
    glfwMakeContextCurrent(window);
    ```
    
3. GLAD 초기화
    - GLAD가 (OS/라이브러리 환경에 따라 다르게)구현된 OpenGL용 함수의 포인터를 관리
4. Viewport
    
    ```cpp
    void glViewport(GLintx, GLinty, GLsizeiwidth, GLsizeiheight);
    ```
    
    - viewport는 window와 별개의 개념
    - 왼쪽 아래를 기준으로, 화면에 display되는 영역 정의
        - viewport의 비율에 따라 출력되는 화면의 비율도 달라짐
        
        ```cpp
        glViewport(0,0, width, height);
        glViewport(width/2,height/2, width/2, height/2);
        ```
        
        ![1.png](https://user-images.githubusercontent.com/42532724/195541106-4a4ffb08-4f32-44fc-b041-e2bdd2bb70b9.png)
        
    - 화면 크기 바뀌었을 때 호출하는 콜백 함수 선언/정의/등록
        - `glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);`
5. 렌더링 루프 작성
    
    ```cpp
    // {{렌더링 루프}}
    while(!glfwWindowShouldClose(window))
    {
        processInput(window); // 입력 처리 함수
    
        // 렌더링 로직 작성
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f); // 지울 색상 설정
        glClear(GL_COLOR_BUFFER_BIT); // 화면 컬러 버퍼 삭제
    
        glfwSwapBuffers(window);
        glfwPollEvents();    
    }
    ```
    
    - `glfwWindowShouldClose()`
        - 각 루프가 시작될 때마다 GLFW가 종료하도록 지시되었는지 확인
        - 지시했을 경우, `true`를 반환하여 루프를 중지함
    - `glfwPollEvents()`
        1. 마우스/키보드 등의 이벤트가 발생하였는지 확인
        2. 윈도우 상태 업데이트
        3. 등록한 콜백 함수 호출
    - `glfwSwapBuffers()`
        - 컬러 버퍼 교체
            - 컬러 버퍼: GLFW 창 안의 각 픽셀들에 대한 컬러 값을 가지고 있는 큰 버퍼
            - 루프에서 이미지를 그리고 화면에 출력하는 기능
    - 입력 처리 함수(콜백 함수 아님) 추가
        - 내부에서 키보드, 마우스의 입력 상태에 따른 적절한 이벤트 처리
        
        ```cpp
        void processInput(GLFWwindow *window)
        {
            if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
                glfwSetWindowShouldClose(window, true);
        }
        ```
        
    - 화면 컬러 버퍼 삭제
        - `glClearColor()`: 지울 색상 설정. state-setting 함수
        - `glClear()`: 컬러 버퍼 삭제. state-using 함수
6. 렌더링 루프 종료시 리소스 해제: `glfwTerminate()`

- 전체 소스 코드: [https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/1.2.hello_window_clear/hello_window_clear.cpp](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/1.2.hello_window_clear/hello_window_clear.cpp)

---

- Double buffer
    - single buffer로 이미지를 그렸을 경우, 이미지가 깜박이는 문제가 발생할 수 있음
        - 이는 이미지가 한 번에 그려지는 것이 아니라 픽셀 하나씩 그리기 때문
            - 일반적으로 왼쪽 → 오른쪽/위쪽 → 아랫쪽 순으로 그려짐
        - 즉, 사용자에게 순간적으로 동시에 표시되지 않고 단계별로 보여지기 때문에 결함이 보일 수 있다
    - 이러한 문제를 피하기 위해 double buffer 렌더링을 적용
        - front buffer에는 최종 출력 이미지를 담으며, 모든 렌더링 명령은 back buffer에 그려짐
        - 모든 렌더링 명령이 완료되자마자 back buffer를 front buffer로 swap함으로써, 사용자에게 이미지를 즉시 표시

---

출처

- [https://emmadeveloper.tistory.com/10](https://emmadeveloper.tistory.com/10)
- [https://learnopengl.com/Getting-started/Hello-Window](https://learnopengl.com/Getting-started/Hello-Window)