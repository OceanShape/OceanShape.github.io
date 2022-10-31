---
layout: post
title:  "Simple Rendering Loop(code)"
categories: opengl
---

- OpenGL 3.3 버전의 기본 코드를 정리한 페이지(수정 혹은 삭제 예정)

```cpp
int main()
{
  glfwSetErrorCallback(showGlfwError);

  // GLFW 초기화
  if (!glfwInit()) {
    std::cerr << "GLFW 초기화 실패" << '\n';
    exit(-1);
  }

  glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
  glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
  glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
  glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);

  GLFWwindow* window = glfwCreateWindow(
    1280, // width
    720, // height
    "OpenGL Example",
    NULL, NULL);

  if (!window)
  {
    std::cerr << "윈도우 생성 실패" << '\n';
    glfwTerminate();
    exit(-1);
  }

  glfwMakeContextCurrent(window);
  glfwSetWindowSizeCallback(window, windowResized);
  glfwSetKeyCallback(window, keyPressed);
  glfwSetCursorPosCallback(window, mouseMoved);
  glfwSwapInterval(1);

  glewExperimental = GL_TRUE;
  GLenum err = glewInit();
  if (err != GLEW_OK) {
    std::cerr << "GLEW 초기화 실패 " << glewGetErrorString(err) << '\n';
    glfwTerminate();
    exit(-1);
  }

  std::cout << glGetString(GL_VERSION) << '\n';

  // VAO, VBO 선언
  GLuint VAO, VBO;

  glGenVertexArrays(1, &VAO); // VAO 생성
  glBindVertexArray(VAO); // VAO 연결

  glGenBuffers(1, &VBO); // VBO 생성
  glBindBuffer(GL_ARRAY_BUFFER, VBO); // VBO 연결

  // 정점 좌표를 VBO에 저장
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
  // VAO 해석 방식을 VAO에게 전달
  glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);

  // VAO 사용 허용
  glEnableVertexAttribArray(0);
  // VBO 수정 종료 및 연결 초기화
  glBindBuffer(GL_ARRAY_BUFFER, 0);

  glBindVertexArray(0); // VAO 해제

  glViewport(0, 0, 1280, 720);
  // clear시 초기화할 화면 색상 지정(루프 밖에 작성해도 된다)
  glClearColor(0, 0, 1, 1);

  while (!glfwWindowShouldClose(window)) {
    // 이벤트(키보드 입력이나 마우스 이동 이벤트)가 발생하였는지 확인
    // 윈도우 상태 업데이트 및 등록한 콜백 함수 호출
    glfwPollEvents();

    // 배경색상 초기화
    glClear(GL_COLOR_BUFFER_BIT);

    // VBO에 있는 데이터 연결
    glBindVertexArray(VBO);
    // 데이터를 바탕으로 그리기
    glDrawArrays(GL_TRIANGLES, 0, 3);
    // 데이터 연결 해제
    glBindVertexArray(0);

    // 더블 버퍼링 수행
    glfwSwapBuffers(window);
  }

  glfwDestroyWindow(window);
  glfwTerminate();
  return 0;
}
```
- 함수 추가 설명
    - ```glDrawArrays(GLenum  mode, GLint   first, GLsizei count)```
        - 데이터를 바탕으로 그리는 함수
        - ```mode```: 렌더링할 기본 형식의 종류
        - ```first```: 활성화된 배열의 시작 인덱스
        - ```count```: 렌더링할 인덱스 수

출처: https://learnopengl.com/
