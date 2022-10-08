---
layout: post
title:  "렌더링 API 및 구조(OpenGL 3.0)"
categories: opengl
---

- 개선점
    - OpenGL 2.0에서는 VAO(Vertex Array Object)가 하나
        - VAO를 통해 Vertex Attribute를 관리함
    - 즉, 여러 개의 물체를 그릴 때에는 각 물체의 바인딩을 반복해주어야 한다
        
        ![1.png](https://user-images.githubusercontent.com/42532724/194720623-cdc4d61e-7aa5-455b-92b1-8b4681242e40.png)
        
        ```cpp
        // 이 과정을 모든 물체마다 반복
        glBindBuffer(positionVBO);
        glVertexAttribPointer(0,...);
        glBindBuffer(colorVBO);
        glVertexAttribPointer(1,...);
        glDrawArray();
        ```
        

→ 물체별로 바인딩 함수를 한 번만 호출할 순 없을까?

- 사용자가 VAO를 만들 수 있게 하여, 여러 개의 VAO를 두자
    - 렌더링 루프에서 수행하는 바인딩의 횟수가 줄어듦
    - 초기화 단계에서 각 물체마다 VAO를 만들어 속성을 미리 바인딩
        - 이후 각 물체를 그릴 때마다 VAO만 바인딩하여 그리면 된다
    
    ![2.png](https://user-images.githubusercontent.com/42532724/194720626-af829d69-3cd9-4234-b41c-84638a005303.png)
    
    - 코드 로직
        
        ```
        초기화:
        	모든 물체에 수행
        		특정 물체를 위한 VAO 생성->저장->바인딩
        		한 번 그리는 데에 필요한 모든 버퍼(VBO)를 바인딩
        		VAO 바인딩 해제
        
        렌더링 루프:
        	모든 물체에 수행
        		특정 물체 VAO 바인딩
        		glDrawArry(...) 혹은 glDrawElements(...)를 호출하여 드로잉
        		VAO 바인딩 해제
        ```
        
- 출처: [https://stackoverflow.com/questions/8923174/opengl-vao-best-practices](https://stackoverflow.com/questions/8923174/opengl-vao-best-practices)
- 출처: [https://m.blog.naver.com/cjdeka3123/220947181159](https://m.blog.naver.com/cjdeka3123/220947181159)
