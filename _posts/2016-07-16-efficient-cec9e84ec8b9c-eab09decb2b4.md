---
layout: post
title: EFFICIENT C++_임시 객체
date: '2016-07-16T08:52:44+09:00'
tags:
- c++
- Efficient C++
- Performance
- temporary
- Temporary object
tumblr_url: https://appchemist.tumblr.com/post/147492238871/efficient-cec9e84ec8b9c-eab09decb2b4
---
임시 객체 생성은 성능적으로 작은 영향을 주는 부분은 아니다. 즉, 임시 객체의 기원, 그것의 비용 그리고 임시 객체를 제거할 수 있는 방법을 알지 않고서는 효율적인 코드를 작성하기 힘들다. 

임시 객체는 소스 코드에 존재하지 않고 컴파일러가 조용히 객체를 만들어 낸다. 컴파일러가 어떤 코드에서 임시 객체를 만들어 내는지 찾아내려면 숙련된 관찰력이 필요하다.

이번에는 컴파일러가 어떤 경우에 임시 객체를 생성하는지 확인해보자.
**객체 정의**


```cpp
class Rational {
    friend Rational operator+(const Rational&, const Rational&);
public:
    Rational(int a = 0, int b = 1) : m(a), n(b) { printf("Rational"); }

private:
    int m;
    int n;
};
```





```cpp
    Rational r1(100);			// 1
    Rational r2 = 100;			// 2
    Rational r3 = Rational(100);	// 3
```

컴파일러 구현과 무관하게 Rational r(100) 구문만이 임시 객체를 생성하지 않는다고 보장할 수 있다.

만약 2, 3의 형태를 사용한다면 컴파일러 구현에 따라 임시 객체를 생성할지도 모른다. 3의 형태를 사용한다고 하면  컴파일러가 이 형태를 만나면 Rational::Rational(int, int) 생성자를 사용하여 정수 100을 가지고 Rational 형식의 임시 객체를 생성한다. 그런 다음 복사 생성자를 사용해 임시 객체로부터 r3을 초기화한다.

의사 코드로 작성하면 아래와 같다.


```cpp
{    
    Rational r3;
    Rational _temp;
    _temp.Rational::Rational(100,1);    // Construct the temporary
    r3.Rational::Rational(_temp);       // Copy-construct r3 
    _temp.Rational::~Rational();        // Destroy the temporary 
    ...
}
```

실제로는 대부분의 컴파일러는 최적화를 통해서 임시 객체를 없애기 때문에, 3가지 형태 모두 임시 객체가 생성되지 않았다.
**형식 불일치**
형식 불일치의 일반적인 경우에는 형식 X의 객체가 필요할 때마다 어떤 다른 형식이 제공되는 것이다. 컴파일러는 이때 제공된 형식을 형식 X의 객체로 변환한다. 이 과정에서 임시객체는 생겨난다.


```cpp
{
    Rational r;
    r = 100;
    ... 
}
```

위의 Rational 클래스는 정수 매개 변수를 받는 대입 연산자를 선언하지 않았다. 기본적으로 생성되는 대입 연산자로 Rational은 우변에 Rational 객체를 기대한다. 위에서 정의한 생정자가 정수 매개 변수를 처리하는 방법을 알고 있다. 결과적으로 처리되는 순서를 의사 코드로 표현하면 아래와 같다.


```cpp
Rational _temp;                     // Place holder for temporary
_temp.Rational::Rational(100,1);    // Construct temporary
r.Rational::operator=(_temp);       // Assign temporary to r
temp.Rational::~Rational();         // Destroy the temporary
```

위와 같이 컴파일러가 중간에 임시 객체를 생성을 하고 r객체에 해당 값을 할당한다. 이러한 부분은 프로그래밍에는 편리하지만 불필요한 객체가 생성되고 소멸되는 과정이 임의적으로 생성된 것이다. 성능 관점에서는 좋지 않다.

이런 임의의 변환을 컴파일러가 수행하지 못 하도록 제한할 수 있다. 생성자에 explicit 키워드를 사용하면 된다. 이는 개발자가 명시적으로 생성자를 사용하지 않는다면 컴파일러가 임의로 사용할 수 없도록 제한하는 것이다. 

또 다른 방법으로는 operator=() 함수를 오버로드하여 이 형식의 임시 객체를 제거할 수 있다.


```cpp
class Rational {
public:
    ...  // as before
    Rational& operator=(int a) {m=a; n=1; return *this; } 
};
```

그리고, 해당 책에서 형식 불일치 임시 객체 생성을 해결할 수 있는 방법을 제시하고 있다. 다음 예제를 보자.


```cpp
    Complex a, b;

    for (int i = 0; i < 100; i++) {
        a = b + 1.0;
    }
```

a = b + 1.0 구문이 반복문으로 실행될 때 마다 계속해서 임시 객체가 생성된다. 즉 100회의 추가적인 임시객체가 생성된다. 이 경우 생성되는 임시객체를 제거해 보자.


```cpp
    Complex a, b, one(1.0);

    for (int i = 0; i < 3; i++) {
        a = b + one;
    }
```

처음의 1.0 상수 값을 Complex를 one이라는 객체로 변경했고, 그로인해서 1.0이 Complex 임시 객체가 생성되는 것을 방지했다.
**값으로 전달**
객체를 값으로 전달할 때,  다음과 같이 형식 매개변수를 실제 매개변수로 초기화 한다.


```cpp
T formalArg = actualArg;
```





```cpp
void g(T formalArg) {
    ...
}
T t;
g(t);
```

다음과 같이 g 함수가 존재하고 g를 위와 같이 호출을 했다. 이런 경우 컴파일러는 형식 T의 임시 객체를 생성한 다음 t를 입력 인자로 사용한다. 의사 코드로 표현하면 다음과 같다.


```cpp
T _temp;
_temp.T::T(t);  // copy construct _temp from t
g(_temp);       // pass _temp by reference
_temp.T::~T();  // Destroy _temp
```

임시 객체를 생성하고 소멸하는 과정은 부담이 간다. 가능하면 객체는 포인터나 참조로 전달하여 임시 객체 생성을 방지하는 것이 좋다.
**값으로 반환**
임시 객체가 생성될 수 있는 또 하나의 경우는 함수가 값을 반환할 때이다. 

다음 예제를 통해서 확인해보자.


```cpp
string f() {
    string s;
    ... // Compute "s"
    return s;
}

String p;
...
p = f();
```

f()의 반환 값은 string 형식의 객체이다. 결과적으로 반환 값을 저장하기 위해서 임시 객체가 생성된다. 위의 예제를 보면 f()의 반환 값은 임시 객체로 전달 되고 좌변 객체로 대입된다. 
좀 더 자세한 예제로 string operator+를 가지고 확인해보자. 아래는 string operator+ 연산자의 직관적 해석으로 구현한 것이다.


```cpp
string operator+(const string& lhs, const string& rhs)
{
    char *buffer = new char[lhs.length() + rhs.length() + 1];

    strcpy(buffer, lhs.c_str());
    strcat(buffer, rhs.c_str());

    string result(buffer);
    delete buffer;

    return result;
}
```

다음 코드는 string operator+를 사용하는 예제이다.


```cpp
    string s1 = "Hello";
    string s2 = "World";
    string s3;

    s3 = s1 + s2;
```

위에서 s3 = s1 + s2 구문은 아래와 같은 연산을 포함하고 있다.
- operator+(const string&, const string&); 
- string 생성자가 opertor+ 연산자내에서 string result(buffer)를 실행
- operator+()의 반환 값을 저장하기 위해 임시 객체가 필요, 복사 생성자를 이용해서 operator+의 결과 string을 복사하여 임시 객체 생성
- operator+() 함수가 종료하기 전에 지역 범위 객체인 result 객체가 소멸
- 대입 연산자를 사용하여 operator+가 생성한 임시 객체를 좌변 s3에 대입한다.
- 반환 값을 저장하기 위해서 사용한 임시 객체가 소멸
이렇게 s3 = s1 + s2 하나의 구문을 수행하기 위해서 6개의 연산이 수행된다. 앞에서 이야기 했던 RVO가 적용되면 result 객체를 제거하여 생성자 소멸자를 수행하지 않을 수 있다. 

그렇다면 임시 객체도 제거할 수 있을까? 임시 객체를 제거하면 두 가지의 함수 호출을 더 제거할 수 있다.

s3 = s1 + s2 구문에서 왜 임시 객체를 생성할까? 이유는 우리에게 string s3의 이전 내용을 지우고 s1 + s2의 내용으로 덮어쓸 수 있는 자유가 없기 때문이다. 대입 연산자는 string s3의 이전 내용을 새 내용으로 덮어쓰는 역활을 한다. 컴파일러는 operator=을 수행하지 않을 권한이 없기 때문에 임시 객체는 필수이다. 

그렇다면 s3이 이전 내용을 가지지 않은 완전 새로운 객체라면 어떠할까? 이 경우 결과가 직접 복사 생성 되므로 임시 객체가 필요 없게 된다.


```cpp
{
    string s1 = "Hello";
    string s2 = "World";
    string s3 = s1 + s2;  // No temporary here.
    ...
 }
```





```cpp
{
    string s1 = "Hello";
    string s2 = "World";
    string s3;
    s3 = s1 + s2;	// Temporary generated here.

    ... 
}
```

위의 형태가 임시 객체가 생성되지 않아 아래의 형태보다 좋다.
**op=()를 사용해 임시 객체 제거**


```cpp
{
    string s1,s2,s3;
    ...
    s3 = s1 + s2;
    ...
}
```

아까 이야기 했던 방법을 이용하면 위와 같은 상황도 임시 객체를 제거할 수 있다.
operator+ 대신에 operator+=를 사용하면 된다. 


```cpp
s3 = s1 + s2;	// Temporary generated here
```

를 다음과 같이 수정하면 된다.


```cpp
s3 = s1;	// operator=(). No temporary.
s3 += s2;	// operator+=(). No temporary.
```


두 코드는 논리적으로 동일하지만 전자는 임시 객체를 생성하지만 후자는 생성하지 않는다. 즉, 후자가 더 효율적인 코드이다.
**키 포인트**
- 임시 객체는 생성자와 소멸자 실행으로 성능 부하가 생긴다.
- 생성자에 explicit 키워드를 사용하면 컴파일러가 해당 생성자를 사용해 형식 변환을 할 수 없다.
- 컴파일러는 형식 불일치를 해결하기 위해서 임시 객체를 생성
- 가능하면 객체 복사를 방지하고 참조나 포인터로 전달하고 반환하자.
-  op=연산자를 사용해 임시 객체를 제거할 수 있다. op는 +, -, *, /이 될 수 있다.
