---
layout: post
title:  "Codable 정리"
categories: Swift
---

## Encodable/Decodable

자신을 변환하거나 외부표현으로 변환할 수 있는 타입

- 변환의 의미에 대해 더 생각해 볼 것

### 구조

```swift
typealias Codable = Decodable & Encodable
```

- Encodable: 외부 표현으로 인코딩할 수 있는 타입
- Decodable: 외부 표현에서 디코딩할 수 있는 타입
    - 외부 표현(external representation)은 JSON으로 이해해도 무방
- Codable은 Class, Struct, Enum에서 모두 채택 가능

### 사용법

- encoding/decoding중에는 에러를 발생시킬 수 있기때문에 반드시 try와 함께 사용

```swift
import Foundation
struct Person: Codable {
    var name : String
    var age : Int
}

let ocean = Person(name: "Ocean", age: 30)

// -------------encoding-------------
// 1. encoder 선언 및 포맷 설정
let encoder = JSONEncoder()
encoder.outputFormatting = [.sortedKeys, .prettyPrinted]

// 2. encode() 메소드를 사용하여, Person 인스턴스를 Data로 encoding
let jsonData = try? encoder.encode(ocean)

// 3. Data를 String으로 변환
if let jsonString = String(data: jsonData!, encoding: .utf8) {
    print(jsonString)
//    {
//      "age" : 30,
//      "name" : "Ocean"
//    }
}

// -------------decoding-------------
let jsonString = """
{
"name" : "ocean",
"age" : 30,
}
"""

// 1. decoder 선언
let decoder = JSONDecoder()

// 2. String을 Data로 변환
let data = jsonString.data(using: .utf8)

// 3. Data를 Person 인스턴스로 decoding
if let myProfile = try? decoder.decode(Person.self, from: data!) {
    print(myProfile.name)
    print(myProfile.age)
//    ocean
//    30
}
```

## CodingKey

CodingKey: 인코딩/디코딩을 위한 키로 사용할 수 있는 타입

- 선언으로 미리 매칭해놓은 CodingKey를 실제 struct/class/enum의 key처럼 사용할 수 있다

### 사용법

- String도 같이 채택한 이유: Raw Value타입으로 String을 선택
    - JSON에서의 Key는 항상 String이기 때문

```swift
// -------------CodingKey-------------
struct Animal: Codable {
    var name : String
    var age : Int
    var height : Int

    enum CodingKeys : String, CodingKey {
        case name
        case age
        case height = "tall"
    }
}

// CodingKey로 정한 이상, JSON에서 원래의 key(여기서는 height)를 사용할 수 없다
// "height" : 55로 작성시 decoding 실패
let jsonCodingKeyString = """
{
"name" : "crepe",
"age" : 3,
"tall" : 55
}
"""

let decode = JSONDecoder()

// String을 Data로 변환 후, Animal 인스턴스로 decoding
if let data = jsonCodingKeyString.data(using: .utf8), let petProfile = try? decoder.decode(Animal.self, from: data) {
    print(petProfile.name)
    print(petProfile.age)
    print(petProfile.height)
    // print(petProfile.tall) // 컴파일 에러
}
```

출처: [https://zeddios.tistory.com/373](https://zeddios.tistory.com/373)
