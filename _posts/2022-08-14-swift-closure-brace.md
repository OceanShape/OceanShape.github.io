---
layout: post
title:  "클로저로 저장 프로퍼티를 정의할 때, ()를 붙이는 이유"
categories: Swift
---

클로저는 Swift에서 중요한 개념인 만큼, 다양한 곳에서 쓰이고 문법도 많은 편이다.
그중 뷰 컨트롤러에 UI 요소를 추가할 때, 자주 사용하는 구문이 하나 있다.

```swift
let myUIButton : UIButton = { return UIButton() }()
```

돌아보니 문법을 쓰면서도 정작 원리에 대해 생각해본 적이 없었다. 마침 stackoverflow에서 나랑 비슷한 의문을 가진 사람을 찾을 수 있었고, 답변을 통해 왜 이렇게 사용하는지 알 수 있었다.

[[stackoverflow] what do parenthesis do after lazy var definition?](https://stackoverflow.com/questions/35237786/what-do-parenthesis-do-after-lazy-var-definition)
```swift
let test1 : Any = { return "Test" }
let test2 : Any = { return "Test" }()
print(type(of: test1))  // () -> String
print(type(of: test2))  // String
```
답변을 코드로 요약하면 위와 같다. ```Any``` 타입 프로퍼티로 타입 추론을 시도할 경우, ()를 붙이지 않으면 ```() -> String 형태의 함수```(클로저)로, ()를 붙이면 반환값(```String```)으로 타입이 정해진다는 사실을 알 수 있다.

```swift
let test = { return "Test" }
// let test : String = { return "Test" } // 컴파일 에러
```
**()를 붙이지 않을 경우**, 이 코드는 단순히 클로저를 대입하는 구문이다. 즉, ```test```는 ```() -> String``` 형태의 클로저 타입이 된다. 이렇게 대입이 가능한 이유는 클로저가 프로퍼티에 저장할 수 있는 일급 객체이기 때문이다.

```swift
let test = { return "Test" }()
```
**()를 붙일 경우**, ```test```는 클로저 내부 코드의 반환값인 "Test", 즉 ```String``` 타입이 된다. 이는 ()가 해당 클로저, 즉 코드의 실행을 의미하기 때문이다.

```swift
let test = { print("Test") }
test() // "Test" 출력
```
왜 ()가 클로저의 실행을 의미하도록 설계했는지까지는 알 수 없었다. 하지만, 대부분의 언어가 ()를 사용하여 함수를 호출하고, Swift에서의 클로저는 함수이기 때문에 이런 문법을 설계했다고 생각한다.
