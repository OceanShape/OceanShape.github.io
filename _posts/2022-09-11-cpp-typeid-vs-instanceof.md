---
layout: post
title:  "typeid(C++)/instanceof(Java) 차이점"
categories: C++
---

Java의 instanceof 연산자처럼, C++에서도 typeid 연산자를 사용하면 반환된 type_info객체를 통해 타입을 알 수 있다.
게임 프로젝트를 위해 두 함수를 비교하던 도중, 서로 다르게 동작하는 부분이 있어 이 글을 작성한다.

Java에서는 B 클래스가 A 클래스를 상속받을 경우,
객체 b에 대한 instanceof 연산자는 부모 클래스와 자식 클래스 모두 true를 반환한다.

```cpp
// package와 main함수는 생략
class A{}
class B extends A{}

A a = new A();
B b = new B();

// b instanceof A: true
// b instanceof B: true
```

C++의 typeid 연산자는 완전히 똑같은 클래스여야만 true를 반환한다.

```cpp
#include <iostream>
#include <string>
#include <typeinfo>

class Animal
{};

class Dog : Animal
{};

int main()
{
	Animal Cat;
	Dog Maltese;

	if ((typeid(Cat) == typeid(Animal)) == true)
	{
		std::cout << "Cat type: Animal" << std::endl;
	}

	if ((typeid(Maltese) == typeid(Animal)) == true)
	{
		std::cout << "Maltese type: Animal" << std::endl;
	}

	if ((typeid(Maltese) == typeid(Dog)) == true)
	{
		std::cout << "Maltese type: Dog" << std::endl;
	}
    
	// 출력 결과
	// Cat type: Animal
	// Maltese type: Dog

	return 0;
}
```