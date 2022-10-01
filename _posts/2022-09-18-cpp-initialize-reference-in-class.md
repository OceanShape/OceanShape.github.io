---
layout: posts
title:  "클래스에서의 참조형 변수 초기화"
categories: C++
---

인스턴스를 포함하는 클래스를 만드는 상황이 있다. 이럴 경우 데이터의 용량이 크면 참조를 멤버 변수로 사용하여야 한다. 내용물을 일일이 복사하는 과정에서 시간이 많이 걸리기 때문이다.

이때 **참조형 멤버 변수는 초기화 리스트를 통해서만 초기화가 가능하다는 점**을 주의할 필요가 있다.
```cpp
class test
{
	int& a;
public:
	test(int& inputA)
	{
		a = inputA;
	}
};
```

위 코드에 대해, Visual Studio에서는 다음과 같은 오류가 발생한다.

![img](https://user-images.githubusercontent.com/42532724/190864786-c8069617-54e8-4cc8-86fb-4c0d005fab6a.png)

이러한 오류가 발생하는 이유는 생성자도 하나의 함수이기 때문이다.
즉, 대상이 처음부터 정해져야 하는 참조형 변수는 생성자에서 초기화가 불가능하다.

따라서 참조형 변수를 초기화하기 위해서는 초기화 리스트를 사용할 수밖에 없다.
```cpp
class test
{
	int& a;
public:
	test(int& inputA) : a(inputA)
	{}
};
```
