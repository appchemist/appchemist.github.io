---
layout: post
title: Overloading Typecasts
date: '2016-07-31T10:04:45+09:00'
tags:
- c++
- operator overloading
- type casting
- 형 변환
tumblr_url: https://appchemist.tumblr.com/post/148246261836/overloading-typecasts
---
C++은 기본 타입에 대한 형 변환을 어떻게 수행할지 알고 있다. 하지만 사용자 정의 클래스는 어떻게 형 변환을 수행할지 알지 못 한다. 아래의 클래스를 살펴보자.


```cpp
class Cent {
private:
    int m_cents;
public:
    Cent(int cents = 0) : m_cents(cents) {}

    int get_cents() { return m_cents; }
    void set_cents(int cents) { m_cents = cents; }
};
```

Cent 클래스는 m_cents라는 int 형을 가지고 있고, 접근 연산자를 가지고 있다. 또한 int 형을 어떻게 Cent 클래스로 변환할지 알 수 있는 생성자를 가지고 있다.
즉, int 형에서 Cent 형으로 변환이 가능하다. 그렇다면, Cent 형에서 int 형으로 변환할 수 있을까? 어떤 경우에는 안 되지만, 아래와 같이 변환이 가능하다.


```cpp
void print_ints(int value) {
    std::cout << "print_ints : " << value << std::endl;
}

int main()
{
    Cent cents(7);
    print_int(cents.getCents()); // print 7
 
    return 0;
}
```

다음과 같이 접근 연산자를 통해서 가능하다. 하지만, 더 간단하게 암시적 타입 변환 혹은 기본 연산자와 같이 형 변환을 수행할 수 없을까?


```cpp
class Cent {
private:
    int m_cents;
public:
    Cent(int cents = 0) : m_cents(cents) {}

    // Overloaded int cast
    operator int() { return m_cents; }

    int get_cents() { return m_cents; }
    void set_cents(int cents) { m_cents = cents; }
};
```

위와 같이 우리는 operator int() 함수를 통해서 Cent 클래스를 int 형으로 형 변환이 가능하다.
### Overloading Typecast
- int 형 변환을 위해서 operator int() 함수를 오버로딩 한다. 여기에서 operator 다음 공백을 주고 변환할 형을 지정한다.
- 형 변환 연산자는 반환값을 지정하지 않는다. 대신 C++ 컴파일러는 사용자가 정확한 형을 반환한다고 가정한다.


```cpp
int main()
{
    Cent cents(7);
    print_ints(cents);
 
    return 0;
}
```

컴파일러는 print_ints 함수가 int 형을 받는 것을 알고 있다. 하지만, cents 변수는 int 형이 아닌 Cent 클래스이다. 결국 컴파일러는 Cent 클래스가 int 형으로 변환할 방법을 제공하는지 확인하는데, 해당 방법이 operator int() 함수이다. 컴파일러는 operator int() 함수를 통해 print_ints 함수에 cents 변수를 int 형으로 변환하여 전달하게 된다.
이번에는 명시적 형 변환을 확인해보자. 이미 형 변환 방법은 제공했고, 기본 타입의 형 변환과 동일하게 수행할 수 있다.
아래 코드를 보자.


```cpp
Cent cents(7);
int iCent = static_cast<int>(cents);
```

이와 같은 방법으로 사용자 정의 클래스를 기본 타입과 같이 형 변환을 할 수 있다. 위는 클래스에서 기본 타입에 대한 형 변환을 확인해 봤다. 또한 기본 타입이 아닌 사용자 정의 클래스 간의 형 변환에서도 사용할 수 있다.