---
layout: post
title: "Learn OpenGL Getting started 2-8. Coordinate Systems"
categories: opengl
---

-   Learn OpenGL은 접근 방식에 맞게, 각 좌표계를 기준으로 변환 단계를 설명한다. 개인적으로 변환의 과정과 입/출력을 파이프라인처럼 정리하는 게 편해서, 각 단계의 동작에 초점을 맞춰보았다.

-   Vertex Shader 이후의 가정: 모든 점은 NDC(normalized device coordinates)에 존재
    -   `x`, `y`, `z` 모두 `-1.0` ~ `1.0` 범위 내에 있는 점만 보임

### 오른손 좌표계

<p align="center"><img src="https://user-images.githubusercontent.com/42532724/203482439-7b4878e6-8088-45bf-aa85-dc0e75275933.png" width="40%">
<img src="https://user-images.githubusercontent.com/42532724/203482444-63b093a7-6b04-45ca-b623-8b03347991c9.png" width="50%"></p>

-   관례상 OpenGL은 오른손 좌표계를 사용
    -   Z축이 반대 방향인 왼손 좌표계는 DirectX에서 사용한다
-   NDC에서는 왼손 좌표계를 사용한다(projection matrix가 변경해준다)

### 좌표 공간

-   일부 작업을 특정 좌표계에서 보다 이치에 맞추거나, 쉽게 쓰기 위해 여러 개의 공간으로 나눔
    1. Local space(Object space): 각 오브젝트만의 local 좌표 공간
    2. World space: 모든 오브젝트를 배치한 전체 공간
    3. View space(Eye space): 카메라 기준의 공간
    4. Clip space: 화면에 출력할 vertex를 결정하는 좌표계(범위: `-1.0` ~ `1.0`)
    5. Screen space: 화면에 출력

![coordinate_systems](https://user-images.githubusercontent.com/42532724/201527497-393c1432-8b56-412b-b732-c0588f8024a2.png)

### 좌표 변환 단계

-   `Vertex Shader: 1~3단계` , `rasterization: 4단계`

1. Model transform: Local Space → World Space
2. View transform: World Space → View Space

    - 카메라 전면을 보도록 만드는 과정에서, translate/rotate의 조합을 사용하여 변환

3. Projection transform: View Space → Clip Space(정규화까지 끝난 NDC)

    - NDC로 변환하는 과정
        1. orthographic 혹은 perspective projection matrix 정의
            - matrix는 각자의 특성에 따른 절두체를 생성함
            - 이 절두체가 좌표의 범위 역할을 함
        2. 절두체 밖 모든 좌표 clipping(제거됨)
        3. 절두체 내부의 모든 좌표를 NDC로 변환

    1. orthographic projection
        - 내부 좌표를 NDC로 직접 매핑
            - 2D평면에 똑바로 매핑하여, 원근감이 없음
        - perspective division은 w가 `1.0`일 경우 좌표를 수정하지 않음
            - 즉, orthographic에서도 수행은 되지만 의미가 없음(전후 동일)
    2. perspective projection 행렬이 하는 일

        1. 주어진 절두체를 clip된 공간에 매핑
            - 좌표들이 clip space로 변환되고 나면 그들은 `-w` ~ `w` 범위 안
        2. perspective division ⇒ `-1.0` ~ `1.0` 좌표(NDC)
            - `perspective division은 vertex 좌표의 각 요소를 w로 나누는 과정`
                - 4D clip space를 3D NDC로 변환

    - 여기까지가 Vertex Shader에서 하는 일
        - vertex shader의 최종 출력은 `-1.0`와 `1.0` 범위 안의 좌표만 존재하는 NDC가 됨

4. Viewport transform: Clip Space → Screen Space
    - `glViewPort()` 파라미터를 사용하여 NDC좌표를 screen좌표에 매핑

### In practice

-   `model`, `view`, `projection` 행렬을 합성하면, 3D좌표를 2D좌표(NDC)로 만들 수 있다
    <p align="center"><img src="https://user-images.githubusercontent.com/42532724/201527516-82d3af3a-3e06-43be-955f-9c60547c9dca.png"></p>

    1. model matrix
    2. view matrix

    3. projection matrix

        - orthographic projection matrix

            ![orthographic_frustum](https://user-images.githubusercontent.com/42532724/201527502-85d19b38-f133-482a-a336-ecf62927d002.png)

            `detail::tmat4x4<T> glm::ortho(left, right, bottom, top, zNear, zFar)`

            - 매개변수 자료형 전부 `T const &`
                - `left, right`: 좌/우 좌표 지정
                - `bottom, top`: 아래/위의 좌표 지정
                - `zNear, zFar`: 가까운 평면/먼 평면과의 거리

        - perspective projection matrix

            ![perspective_frustum](https://user-images.githubusercontent.com/42532724/201527507-36744ab4-4b2b-4a90-aa7c-ded1af7bb105.png)

            `detail::tmat4x4<T> glm::perspective(fovy, aspect, near, far)`

            - 매개변수 자료형 전부 `T const &`
                - `fovy`: field of view(view space의 크기를 각도로 설정)
                - `aspect`: 화면 비율(width/height)
                - `near/far`: 가까운 평면/먼 평면과의 거리

-   만든 matrix를 uniform으로 선언하여 vertex shader에 전달

    ```cpp
    // uniform location 지정. 다양한 방법을 사용할 수 있다
    int modelLoc = glGetUniformLocation(ourShader.ID, "model");
    glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));

    int viewLoc = glGetUniformLocation(ourShader.ID, "view");
    glUniformMatrix4fv(viewLoc, 1, GL_FALSE, &view[0][0]);

    ourShader.setMat4("projection", projection);
    ```

    ```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;
    //...
    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 projection;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);
        //...
    }
    ```

    -   일반적으로 projection matrix는 거의 변하지 않기 때문에, 경우에 따라 렌더링 루프 바깥에 두는 것도 좋은 방법이다.

-   z-buffer
    <p align="center"><img src="https://user-images.githubusercontent.com/42532724/203484122-06a9237d-4744-4c3c-a778-4690001ed2ff.png" width="40%"></p>

    -   지금까지의 결과물: 일부가 덮혀서 어색함. 왜 이상하게 그려질까?
        → OpenGL이 삼각형 단위로 그리기 때문에 발생하는 문제
        -   이미 픽셀이 그려져 있는 위치에 픽셀을 또 그린다
            ⇒ 해결 방법: z-buffer(depth buffer)
    -   OpenGL은 z-buffer에 모든 깊이 정보를 저장
        -   GLFW가 버퍼를 자동으로 생성함
        -   깊이 정보는 각 fragment의 z값으로 저장
    -   픽셀 위에 그릴지 여부를 결정할 때 사용
        -   OpenGL이 `depth testing`을 할 수 있도록 설정 가능
    -   `depth testing`
        -   current fragment의 출력을 시도할 때마다, z-buffer에 들어있는 깊이 값과 비교
            -   기존 fragment보다 뒤에 있을 경우 ⇒ 무시됨
            -   앞에 있을 경우 ⇒ 기존 fragment에 덮어 씌움(z-buffer도 갱신)
    -   사용법

        1. depth testing enable 설정(초기에 한 번): `glEnable(GL_DEPTH_TEST)`
        2. depth buffer 비우기(매 루프마다): `glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)`
         <p align="center"><img src="https://user-images.githubusercontent.com/42532724/203484882-c0885fc8-8487-4225-9b68-4eaf8b2467a8.jpg" width="40%"></p>
