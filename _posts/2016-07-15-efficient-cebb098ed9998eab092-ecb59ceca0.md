---
layout: post
title: EFFICIENT C++_반환값 최적화
date: '2016-07-15T04:05:16+09:00'
tags:
- c++
- Efficient C++
- Performance
- Return Value Optimization
- RVO
tumblr_url: https://appchemist.tumblr.com/post/147437208136/efficient-cebb098ed9998eab092-ecb59ceca0
---
앞에서 불필요한 객체의 생성과 소멸을 제거할 때 마다 성능이 좋아지는 것을 보았다. 이번에는 속도를 위해서 컴파일러가 수행하는 Return Value Optimization을 살펴 보겠다.
**값으로 반환의 동작 원리**
아래는 Complex 클래스로 복소수를 표현하고 있다.


```cpp
class Complex {
    friend Complex operator+(const Complex&, const Complex&);
public:
    Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) {}

    Complex(const Complex& c) : real(c.real), imag(c.imag) {}

    Complex&operator=(const Complex& c);

    ~Complex() {}
private:
    double real;
    double imag;
};
```

더하기 연산자는 Complex 객체를 값으로 반환한다.


```cpp
Complex operator+(const Complex& lhs, const Complex& rhs)
{
    Complex retVal;

    retVal.real = lhs.real + rhs.real;
    retVal.imag = lhs.imag + rhs.imag;

    return retVal;
}
```

다음과 같은 연산을 수행한다.


```cpp
   Complex c1(1, 1), c2(2, 2), c3;

    c3 = c1 + c2;
```

c1 + c2의 값을 어떻게 c3로 대입할 수 있을까? 컴파일러가 사용하는 유명한 방법은 임시 __tempResult 객체를 생성하여 Complex::operator+()의 첫 번째 인자로 전달하는 것이다. 이때 이 임시 객체는 참조로 전달된다. 그래서 컴파일러는 위의 operator+ 함수를 아래와 같이 변경한다.


```cpp
void Complex_Add(Complex& __tempResult,
                 const Complex& lhs,
                 const Complex& rhs)
{
…
}
```





```cpp
c3 = c1 + c2;
```

위의 원래 소스 구문은 아래와 같은 의사 코드로 변환된다.


```cpp
struct Complex __tempResult;
Complex_Add(__tempResult, c1, c2);
c3 = __tempResult;
```

값으로 반환 구현은 지역객체 retVal(oeprator+ 함수 안에 있는)을 없애고, 반환 값을 __tempResult 임시 객체로 직접 계산하는 형태로 최적화할 수 있다. 이것이 반환 값 최적화이다.
**반환 값 최적화(RVO)**
어떠한 최적화도 없다면 Complex_Add()에 대해서 컴파일러가 생성한 의사 코드는 다음과 같다.


```cpp
void Complex_Add(Complex& __tempResult,
                 const Complex& lhs,
                 const Complex& rhs)
{
    struct Complex retVal;
    retVal.Complex::Complex();                  // Construct retVal

    retVal.real = lhs.real + rhs.real;
    retVal.imag = lhs.imag + rhs.imag;

    __tempResult.Complex::Complex(retVal);      // Copy-Construct __tempResult

    retVal.Complex::~Complex();                 // Destroy retVal
    return;
}
```

컴파일러는 지역 객체 retVal을 없애고, 이것을 __tempResult로 교체하여 Complex_Add()를 최적화할 수 있다. 이것이 반환 값 최적화이다. 아래는 최적화한 의사 코드이다.


```cpp
void Complex_Add(Complex& __tempResult,
                 const Complex& lhs,
                 const Complex& rhs)
{
    __tempResult.Complex::Complex();
    __tempResult.real = lhs.real + rhs.real;
    __tempResult.imag = lhs.imag + rhs.imag;
    return;
}
```

operator+ 함수를 이용해서 RVO가 적용된 버전과 적용되지 않은 버전의 성능을 비교해 보자.
측정 코드는 아래와 같다.


```cpp
TEST(Chapter4, Check_Baisc_Complex_Operator_Plus)
{
    Complex a(1.0), b(2.0), c;

    struct timeval start, end, running;
    gettimeofday(&start, NULL);

    for (int i = 0; i < 90000000; i++) {
        // 수행 명령
    }

    gettimeofday(&end, NULL);
    if (end.tv_usec <= start.tv_usec) {
        end.tv_sec--;
        end.tv_usec += 1000000;
    }
    running.tv_usec=end.tv_usec-start.tv_usec;
    running.tv_sec=end.tv_sec-start.tv_sec;

    printf("Running Time : %d.%06d\n", running.tv_sec, running.tv_usec);
}
```

위의 측정 코드를 통해서 앞에서 이야기 했던 반환 값 최적화 의사 코드와 최적화 하지 않은 일반 의사 코드의 성능을 측정해 보면 아래와 같다.

최적화가 없는 경우 : 약 1.7초

최적화가 된 경우 : 약 0.5초
반환 값 최적화가 진행된 경우가 약 3배 정도 더 빠르다는 것을 알 수 있다.
의사코드가 아닌 실제적으로 컴파일러가 최적화를 진행한 경우와 최적화를 진행하지 않은 경우에 대해서 확인해보자.


```cpp
Complex operator+(const Complex& lhs, const Complex& rhs)
{	// operator+ 버전1
    Complex retVal;

    retVal.real = lhs.real + rhs.real;
    retVal.imag = lhs.imag + rhs.imag;

    return retVal;
}
```





```cpp
Complex operator+(const Complex& lhs, const Complex& rhs)
{	// operator+ 버전2
    double r = lhs.real + rhs.real;
    double i = lhs.imag + rhs.imag;

    return Complex(r, i);
}
```




```cpp
Complex operator+(const Complex& lhs, const Complex& rhs)
{	// operator+ 버전3
    Complex retVal(lhs.real + rhs.real, lhs.imag + rhs.imag);

    return retVal;
}
```





```cpp
Complex operator+(const Complex& lhs, const Complex& rhs)
{	// operator+ 버전4
    return Complex(lhs.real + rhs.real, lhs.imag + rhs.imag);
}
```

위와 같이 4가지 버전에 대해서 RVO가 발생하는지 stdout을 통하여 확인을 해보았다. 나의 환경에서는 위에서 말했던 의사코드와 같은 최적화는 발생하지 않았다.
단, -fno-elide-constructors 컴파일러 옵션을 사용한 경우 위의 4가지 버전 모두 Copy Construction이 추가적으로 발생했고, 속도가 이전보다 떨어지는 것을 확인했다.<span class="Apple-converted-space">  </span>최적화가 책에서 말하는 부분과는 다소 달리 발생했고, 책을 기준으로 버전2~4가 최적화가 발생했지만, Copy Constructor의 발생을 기준으로 본다면 위의 4가지가 모두 동일하게 최적화가 발생했다. 아쉽게도 위에서 이야기된 최적화 의사코드와는 다른 최적화가 발생했다.
**연산 생성자**


```cpp
Complex operator+(const Complex& lhs, const Complex& rhs)
{
    return Complex(lhs, rhs);
}
```





```cpp
Complex(const Complex& lhs, const Complex& rhs) : real(lhs.real + rhs.real), imag(lhs.imag + rhs.imag) { }
```

위와 같이 연산을 생성자에서 수행하는 것이 연산 생성자다. 위와 같은 경우 RVO를 적용할 확률이 높다고 한다. 실험 결과 책에서 말한 최적화는 발생하지 않았다. 하지만 추가적인 Copy Constructor는 발생하지 않았다. 즉 위와 동일한 최적화가 발생했다. 하지만 측정한 속도에서는 위의 버전1~4 보다 약 20%가 더 빠르게 측정됐다.
**키 포인트**
- 객체를 값으로 반환해야 하는 경우, 반환 값 최적화를 사용해 지역 객체가 생성되고 소멸되는 것을 방지하여 성능이 좋아질 수 있다.
- 컴파일러 구현에 따라 RVO의 적용 사례가 달라진다. 사용하는 컴파일러 문서를 보거나 직접 실험해 보아야 한다. 
- 연산 생성자를 사용하여 RVO가 적용될 확률을 높일 수 있다.
