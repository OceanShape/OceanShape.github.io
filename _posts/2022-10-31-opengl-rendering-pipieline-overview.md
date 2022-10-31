---
layout: post
title:  "Rendering Pipeline"
categories: opengl
---

- 이 글은 [Vulkan 튜토리얼](https://vulkan-tutorial.com/Drawing_a_triangle/Graphics_pipeline_basics/Introduction)의 그래픽 파이프라인 기본 사항 페이지를 정리한 내용이다

### 파이프라인 구조

- 녹색 부분은 고정 기능 단계(작동 방식이 정해져 있음)

<img src="https://user-images.githubusercontent.com/42532724/198971435-decab534-fc1b-4a11-a10d-ef162f62cad2.png" width="45%"/>
<img src="https://user-images.githubusercontent.com/42532724/198971440-fb416d2a-b89a-45f2-9473-6bc7be4f77a8.png" width="45%"/>

1. Input Assembler
    - 버퍼로부터 초기의 vertex 데이터를 모음
    - index buffer를 사용해 중복없이 요소 반복 가능
2. Vertex Shader
    - 모든 vertex에 대해 실행하여, 정점별 데이터를 출력
    - vertex position을 model space에서 screen space로 바꾸는 변환 적용
3. Tessellation
    - 특정 규칙에 따라 geometry를 세분화하여, mesh 품질을 높임
4. Geometry Shader
    - 모든 primitive(삼각형, 선, 점 등)에서 실행
    - 더 많거나 적은 primitive들을 출력
5. Rasterization
    - primitive들을 fragment들로 변환
        - fragment: framebuffer를 채울 pixel element
    - 화면 밖의 모든 fragment들은 폐기
    - vertex shader에서 출력된 attribute들은 fragment 전체에 걸쳐 보간
6. Fragment Shader
    - 살아남은 모든 fragment에서 실행
    - fragment의 색상, 깊이, 기록될 Framebuffer(들)의 위치 지정
    - Vertex Shader에서 보간된 데이터 활용
        - 이 데이터는 텍스처 좌표와 조명을 위한 법선까지도 포함할 수 있다
7. Color Blending
    - Framebuffer의 동일한 pixel에 매핑되는 서로 다른 fragment들을 합치는 과정
        - 덮어쓰거나, 투명도 기반 합산/혼합 가능

용어 간단 정리

- mesh: 다면체의 형태를 구성하는 다각형들과 정점들의 집합
- framebuffer: 화면에 나타날 영상 정보를 저장하는 기억장치
    - 프레밍 버퍼의 최소단위는 화면상의 픽셀과 대응
    - 화면 각 픽셀의 on/off 상태나 색을 비트맵으로 기억함