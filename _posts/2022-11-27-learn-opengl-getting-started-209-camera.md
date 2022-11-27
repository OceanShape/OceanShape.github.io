---
layout: post
title:  "Learn OpenGL Getting started 2-9. Camera"
categories: opengl
---

- 실제 코드는 간단하게 작성
- 예시 코드의 “마우스 조작으로 시점을 회전하는 카메라 만들기” 부분에서, 삼각법의 도입부 내용은 제외했다. 혼동의 여지가 있다는 댓글이 많았고, 개인적으로도 잘 이해되지 않았기 때문이다. 그래서 도움이 되었던 댓글의 이미지를 가져와 설명을 대체한다.

### View(Camera) Space 정의하기

**OpenGL 자체는 카메라의 개념과 친숙하지 않다**

- 하지만, scene의 모든 오브젝트를 반대 방향으로 이동시켜, 착시를 일으킬 순 있다.
    <p align="center"><img src="https://user-images.githubusercontent.com/42532724/203885469-188ee312-b399-468d-90ec-d1c40c5d3261.png">

    - 카메라를 정의하기 위해서는 아래의 정보가 필요
        - 카메라의 위치(World Space 기준)
        - 바라보고 있는 방향
        - 카메라의 오른쪽을 가리키는 벡터
        - 카메라의 위쪽을 가리키는 벡터

    ⇒ 카메라의 위치를 원점으로 하고, 3개의 수직인 축을 가진 좌표계를 만듦

    - 코드에서는 사용 방법이 약간 다르므로, 정의가 가능하다는 사실만 기억하자

- 정의하는 과정

    1. 카메라 위치

        - 카메라 위치는 World Space의 벡터

        ```cpp
        // z+방향으로 이동시켜 위치를 뒤로 옮김
        glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
        ```

    2. 카메라 방향

        - **카메라가 z+방향을 바라보도록 만들기**
            - `(카메라 위치 벡터) - (화면의 원점 벡터)`

        ```cpp
        glm::vec3 cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
        glm::vec3 cameraDirection = glm::normalize(cameraPos - cameraTarget);
        ```

        - 주의! cameraDirection은 실제로 카메라가 보는 방향의 반대 방향을 가리킴

    3. 오른쪽 축

        - **카메라가 x+방향을 나타내는 오른쪽 벡터 구하기**
            1. World Space에서 위쪽을 가리키는 벡터 지정
            2. 위쪽을 가리키는 벡터에 cameraDirection을 외적
            3. 외적의 결과는 두 벡터와 수직인 벡터 → x+방향의 벡터를 구할 수 있음
                - 외적은 오른손 엄지로 계산하는 것임을 기억할 것

        ```cpp
        glm::vec3 up = glm::vec3(0.0f, 1.0f, 0.0f);
        glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDirection));
        ```

    4. 위쪽 축

        - **카메라가 y+방향을 나타내는 위쪽 벡터 구하기**
            - cameraDirection에 cameraRight을 외적

        ```cpp
         glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight);
        ```

### Look At

- 행렬을 사용하는 이유
    - n(예를 들어 3)개의 직각(또는 비선형)인 축으로 좌표 space를 만들 경우
    - n개의 축과 이동 벡터가 한 번에 포함된 데이터(행렬) 생성 가능
    - 어떠한 벡터든지 이 행렬과 곱하여 이 좌표 space로 변환 가능
<p align="center"><img src="https://user-images.githubusercontent.com/42532724/203885472-9bf29b2b-2616-48ca-88f0-560f5bcf6095.jpg">

- R은 오른쪽 벡터, U는 위쪽 벡터, D는 방향 벡터, P는 카메라의 위치 벡터
    - 위치 벡터가 반대로 되어 있는 이유
        - World를 우리가 원하는 방향과 반대로 이동시켜야 하기 때문
- GLM에는 세 가지 정보만 주면, view행렬을 만들어 주는 `lookAt()`함수 존재

```cpp
glm::mat4 view;
// 순서대로 위치, 목표물, 위쪽 벡터 입력(셋 모두 World Space 기준)
view = glm::lookAt(glm::vec3(0.0f, 0.0f, 3.0f),
       glm::vec3(0.0f, 0.0f, 0.0f),
       glm::vec3(0.0f, 1.0f, 0.0f));
```

### 예시 코드

- 원을 그리며 돌기

    ```cpp
    glm::mat4 view{ 1.0f };

    float radius = 10.0f;
    float camX = sin(glfwGetTime()) * radius;
    float camZ = cos(glfwGetTime()) * radius;
    view = glm::lookAt(glm::vec3(camX, 0.0f, camZ), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f));

    shader.setMat4("view", view);
    ```

- 키보드 조작으로 이동하는 카메라 만들기

    1. 카메라 시스템 세팅

        - 프로그램 맨 위에 카메라 변수 정의

        ```cpp
        glm::vec3 cameraPos   = glm::vec3(0.0f, 0.0f,  3.0f);
        glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
        glm::vec3 cameraUp    = glm::vec3(0.0f, 1.0f,  0.0f);
        //...
        // glm::lookAt()에서의 사용법
        view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
        ```

        - 두 번째 매개변수가 `cameraPos` + `cameraFront` 인 이유
            - 매개변수 “바라보는 위치”를 의미
            - 카메라의 앞을 바라보는 위치로 설정하면, 카메라의 위치를 따라감
        - 우리는 이미 GLFW의 키보드 입력을 관리하기 위해 processInput 함수를 정의하였습니다. 그래서 확인할 새로운 키 커맨드를 추가해봅시다.

    2. 카메라 키보드 세팅

        - `void process Input(GLFWwindow *window)` 내부에 조작 관련 코드 추가

        ```cpp
        float cameraSpeed = 0.05f;
        if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
            cameraPos += cameraSpeed * cameraFront;
        if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
            cameraPos -= cameraSpeed * cameraFront;
        if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
            cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
        if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
            cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
        ```

    3. 이동속도

        - 하드웨어마다 프로세싱 파워가 다르다
            - 성능이 좋을 경우, 다른 사람보다 더 많은 수행이 일어남
        - 시간당 처리하는 프레임의 수를 하드웨어에 관계없이 일정하게 만들 필요가 있다 ⇒ `deltaTime` 개념 사용
            - `deltaTime`은 마지막 프레임을 렌더링하는 데에 걸리는 시간
            1. 프레임의 렌더링 속도가 느릴 경우, `deltaTime`값도 커짐
            2. `deltaTime`을 곱해주면, 카메라 이동속도가 빨라진다

        ```cpp
        // 전역 변수 정의
        float deltaTime = 0.0f;
        float lastFrame = 0.0f; // 마지막 프레임의 시간

        //...

        // deltaTime = 마지막 프레임과 현재 프레임 사이의 시간
        float currentFrame = glfwGetTime();
        deltaTime = currentFrame - lastFrame;
        lastFrame = currentFrame;

        //...

        // cameraSpeed에 적용
        float cameraSpeed = 2.5f * deltaTime;
        ```

- 마우스 조작으로 시점을 회전하는 카메라 만들기
    <p align="center"><img src="https://user-images.githubusercontent.com/42532724/204139410-7fce775e-8298-495f-b868-57b7a2010661.png">

    - Euler angles(오일러 각)은 3D상에서의 모든 회전을 나타낼 수 있는 3개의 값
        - `pitch`, `yaw`, `roll` 존재(단, 여기서는 `roll`값을 다루지 않음)
            - 오일러 각의 요소는 하나의 값으로 나타냄
    - 유도 과정
        <p align="center"><img src="https://user-images.githubusercontent.com/42532724/204139407-1da417e4-2b66-4786-a81f-aabcaf2d28e3.png">

        1. yaw

            ```cpp
            glm::vec3 direction;
            direction.x = cos(glm::radians(yaw));
            direction.z = sin(glm::radians(yaw));
            ```

        2. pitch

            ```cpp
            direction.y = sin(glm::radians(pitch));
            ```

        3. 두 공식 합치기
            - 실제 단위 벡터는 r이기 때문에, yaw/pitch의 빗변의 길이는 1이 아님

                ⇒ r에서 계산한 빗변 p의 값을 x/z에 곱해주어야 한다
        <p align="center"><img src="https://user-images.githubusercontent.com/42532724/204139413-cdbcc542-8030-4d06-a672-747f7cb6395e.png" height="400">

        - 최종 결과

            ```cpp
            direction.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
            direction.y = sin(glm::radians(pitch));
            direction.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
            ```

            - 카메라로 보는 공간은 -z방향에 위치하므로, yaw의 초기값은 -90.0f이다
    - 마우스 입력/줌 확대 축소(코드 생략)
        - 마우스 움직임을 통해 yaw/pitch 값 도출
            - yaw: 마우스 수평 움직임
            - pitch: 마우스 수직 움직임
            - 콜백 함수에서의 ypos 매개변수는 OpenGL 좌표축과 방향이 반대이므로, 부호를 뒤집어줘야 한다

                ```cpp
                float xoffset = xpos - lastX;
                float yoffset = lastY - ypos;
                ```

        - 줌 확대 축소 구현
            - FOV의 값이 작아지면 scene projectes space가 작아지므로, zoom in의 효과를 준다

---

- deltaTime에 대한 정보가 필요한 경우, “키보드 조작으로 이동하는 카메라 만들기 - 이동속도”를 찾아볼 것
- Euler angles에 대한 정보가 필요할 경우, “마우스 조작으로 시점을 회전하는 카메라 만들기”를 찾아볼 것
