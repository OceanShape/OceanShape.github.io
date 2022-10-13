---
layout: post
title:  "Learn OpenGL Getting started 2-1. OpenGL"
categories: opengl
---

### OpenGL이란

- OpenGL은 그래픽 및 이미지를 조작하는 데 필요한 기능을 제공하는 API(Application Programming Interface)
    - OpenGL 자체는 API가 아니라 설명서
        - 즉, 각 함수의 출력이 무엇이고 어떻게 수행되어야 하는지 정의
            - 구현 세부 사항을 제공하지 않으므로 OpenGL의 실제 개발된 버전은 함수의 구현이 다를 수 있다
        - 구현(*implementing*)하는 개발자는 해당 함수가 어떻게 작동해야하는지에 대한 솔루션을 제시하여야 한다
    - 실제 OpenGL 라이브러리는 일반적으로 그래픽 카드 제조사에서 개발
        - 각 업체의 그래픽 카드는 해당 카드(시리즈) 용으로 특별히 개발 된 OpenGL의 특정 버전 지원
        - 라이브러리 대부분은 제조사에서 구현하므로, 버그가 발견되어도 일반적으로 드라이버를 업데이트하면 해결됨
            - 드라이버에는 카드가 지원하는 가장 최신 OpenGL 버전을 포함

### Immediate mode(=fixed function pipeline) vs Core-profile

- 옛날에는 상대적으로 쉬운 Immediate mode로 개발
    - 장점: OpenGL이 수행한 실제 작업을 추상화했기에,  배우기 쉽다
    - 단점: 사용과 이해가 쉽지만 매우 비효율적
        - 하지만 기능 대부분이 라이브러리에 숨겨져 있어, 작동 방법을 건드리기 어려웠다
        - 개발자들은 더 많은 유연성을 요구
- 버전 3.2부터는 imiediate mode를 더 이상 사용하지 않고, OpenGL의 사양에 따라 core-profile 모드로 개발하도록 유도
    - 시간이 지나면서 사양이 유연해지고, 개발자들이 그래픽을 보다 잘 제어할 수 있게 되었기 때문
    - 이제는 core-profile을 사용할 때에도, 더이상 쓰이지 않는 함수(deprecated)를 호출하면 오류를 발생시키고 그리기를 중지시킴
- 방향: 현대적인 접근법이 조금 어려울지라도 개발자가 OpenGL과 그래픽 프로그래밍을 진정으로 이해할 수 있어야 한다
    - 더 유연하고 효율적이며, 그래픽 프로그래밍을 제대로 이해하는 방법
    - 이 강좌가 Core-Profile OpenGL 버전 3.3에 맞추어져있는 이유

### 왜 더 최신 버전을 두고 OpenGL 3.3버전을 배우는가?

1. OpenGL 3.3 버전 이후 모든 개념과 기술이 동일
    - 같은 작업에 대해 약간의 효율적이거나 유용한 방법이 몇 가지 추가되었을 뿐
    - OpenGL 3.3으로 많은 경험을 해본다면, 최신 버전의 OpenGL을 사용하는 데에도 어려움이 없다
2. 최신 버전의 기능은 가장 최신 그래픽 카드만 응용 프로그램 실행 가능
    - 보통 더 높은 버전의 기능을 선택적으로 활용하는 이유

### OpenGL의 특징

- **Extensions**
    - 그래픽 회사에서 렌더링을 위한 새로운 기술 또는 최적화가 등장할 때마다 드라이버에서 그 기술이 확장되어 구현됨
        - 기능이 많이 쓰이거나 매우 유용하다고 생각되면 결국엔 추후 OpenGL 버전의 일부가 된다
        - 이를 통해 확장에서 제공하는 기능을 사용하여 고급 그래픽 또는 효율적인 그래픽 사용 가능
    - 개발자는 이러한 확장들이 사용가능한지 여부를 확인하거나, 하나의 확장 라이브러리를 사용하여야 한다
        
        ```cpp
        if(GL_ARB_extension_name)
        {
            // 하드웨어가 지원하는 새롭고 현대적인 기능을 수행
        }
        else
        {
            // 확장 미지원: 예전의 방법으로 기능을 수행
        }
        ```
        
- **State machine**
    - OpenGL은 하나의 state machine
        - state machine: 동작을 정의하는 변수들의 집합
        - `state`를 `OpenGL` `context`라고도 부른다
    - 일부 옵션과 버퍼를 조작한 다음, 현재 context를 사용하여 렌더링함으로써 state를 변경
        - 예시) 삼각형 대신 선을 그려야 한다고 전달할 때마다 OpenGL이 그려야하는 방식을 설정하는 context 변수를 변경하여 OpenGL의 state를 변경
            - OpenGL이 선을 그어야한다고 말하여 상태를 변경하자마자, 다음 드로잉 명령은 이제 선을 그리는 명령이 됩니다.
    - OpenGL에서는 아래 두 종류의 함수를 사용하여 작업
        - `state-setting`: context(= state)를 변경
        - `state-using`: 현재 상태를 기반으로 일부 작업을 수행
- Objects
    - OpenGL은 객체 개념이 존재함
        - OpenGL은 다른 언어의 많은 파생을 허용하지만, 결국 핵심은 C 라이브러리
        - C 언어의 언어 구조가 고급언어와 잘 맞지 않으므로, 객체를 비롯한 추상화를 염두하여 개발됨
    - OpenGL에서의 객체(object): OpenGL 상태의 하위 집합을 나타내는 옵션들의 모음
        - 예시) 드로잉 창의 설정을 나타내는 객체를 가짐으로써, 창의 크기/지원하는 색상의 범위 등을 설정할 수 있다
            - 객체는 C와 같은 구조체로 시각화
            
            ```cpp
            struct OpenGL_Context {
              	//...
              	object* object_Window_Target;
              	//...  	
            };
            ```
            
    - **객체를 활용한 OpenGL의 일반적인 작업 흐름 예시: 화면 설정**
        
        ```cpp
        // 1.
        unsigned int objectId = 0;
        glGenObject(1, &objectId);
        // 2.
        glBindObject(GL_WINDOW_TARGET, objectId);
        // 3.
        glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
        glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
        // 4.
        glBindObject(GL_WINDOW_TARGET, 0);
        ```
        
        1. 객체 생성 & 참조를 id로 저장
            - 실제 데이터는 다른 위치에 저장됨
        2. 객체를 context에 바인딩
            - 예제에서 윈도우 객체 대상의 위치는 `GL_WINDOW_TARGET`으로 정의됨
        3. `GL_WINDOW_TARGET`에 현재 바인딩 된 객체의 옵션 설정
        4. context 대상을 다시 기본값(0)으로 설정
        - 결과적으로 설정한 옵션값은 objectId가 참조하는 객체에 저장됨
            - 이후 이 객체에 다시 바인딩하면 곧바로 값을 가져올 수 있다
    - 이 방식의 장점
        - 응용 프로그램에서 둘 이상의 객체를 정의하고 옵션을 설정할 수 있다
            - 여러 객체의 옵션을 일일이 설정할 필요 없이, 그리기 전에 바인딩만 하면 됨
        - OpenGL의 상태를 사용하는 작업을 시작할 때마다 객체를 원하는 설정으로 바인딩할 수 있다
