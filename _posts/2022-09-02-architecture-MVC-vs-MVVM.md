---
layout: post
title:  "Swift에서의 MVC/MVVM 비교"
categories: architecture
---

## MVC: M ← C ↔ V
**장점**
- Model과 View를 확실하게 분리
- Model 테스트 용이(어디에도 종속되지 않음)

**단점**
- View와 Controller 계층이 제대로 분리되지 않음
    - ViewController의 코드가 길어짐
    - Controller와 View가 깊이 종속되어 테스트가 어려움
    - View가 변경되면 Controller도 변경 필요
- Controller가 커지고 시간이 지나면 유지보수가 어려움

## MVVM: V → VM ↔ M
**장점**
- View Model이 View에 대해서 전혀 모르며 테스트가 용이
- View 또한 개별적인 테스트 용이
- MVP보다 깔끔하며 코딩 양이 적다.
- View는 View Model과의 바인딩을 통하여 Model의 값의 변경을 쉽게 추적 및 업데이트 할 수 있다.

**단점**
- 기본 MVP의 View보다 MVVM의 View는 책임의 범위가 더 크다.
- Reactive 프레임워크를 주로 사용하기 때문에 경우에 따라서 디버깅이 어려울 수 있다.
- 간단한 앱의 경우 MVC가 오히려 관리가 편하고 코드가 적다.

참고링크
- MVC/MVP/MVVM 장단점: [https://joycestudios.tistory.com/18](https://joycestudios.tistory.com/18)
- MVC/MVVM 비교(feat. SwiftUI): [https://mac-user-guide.tistory.com/135](https://mac-user-guide.tistory.com/135)
