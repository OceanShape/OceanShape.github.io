---
layout: post
title:  "Swift 공식 구현 코드 찾기"
categories: tips
---

새로운 언어를 공부할 때, 그 언어의 공식 라이브러리 헤더에서 도움을 많이 받을 수 있다. 언어 설계자들이 프로토타입(매개변수와 반환값, 이름)과 주석을 통해 사용법을 상세하게 써놓기 때문이다.
`Swift`의 경우, `Xcode`에서 제공하는 `Jump to Definition`(메서드 우클릭)기능을 통해 라이브러리를 확인할 수 있다.

<center><img width="400" alt="Jump to Definition1" src="https://user-images.githubusercontent.com/42532724/186222727-bf518094-8f28-4c3a-87d7-ae12861d4318.png">
<img width="400" alt="Jump to Definition2" src="https://user-images.githubusercontent.com/42532724/186222823-ded3ba42-32e8-4822-88b9-7b9e92b6791a.png">
</center>

하지만 프로토타입뿐만 아니라, 메서드를 어떻게 구현했는지까지도 알고 싶을 때가 있다.
이 때는, `github` 리포지토리에서 애플의 공식 라이브러리의 코드를 확인할 수 있다.

https://github.com/apple/swift

특히 `swift/stdlib/public/core` 디렉터리에서는, 핵심 라이브러리의 구현 코드를 볼 수 있다. 예를 들어, '==='연산자의 공식 구현 코드는 아래와 같다.

<center>
<img width="500" alt="===implement" src="https://user-images.githubusercontent.com/42532724/186232301-1e59ce41-f3f1-432c-9d9d-7f4731de5e59.png">
</center>

