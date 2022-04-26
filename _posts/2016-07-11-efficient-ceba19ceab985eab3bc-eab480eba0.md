---
layout: post
title: Efficient C++_로깅과 관련된 성능 이야기
date: '2016-07-11T05:45:08+09:00'
tags:
- c++
- Efficient C++
- Performance
tumblr_url: https://appchemist.tumblr.com/post/147231771621/efficient-ceba19ceab985eab3bc-eab480eba0
---
소프트웨어를 유지보수를 하다보면 로깅이 매우 중요하다.

로깅을 통해서 문제 상황 추적 및 프로그램의 실행 흐름을 파악하는데 많은 도움이 된다.
이런 로깅과 관련된 코드는 성능과 관련된 여러가지 문제들을 만나기 쉽다.
로깅과 관련된 성능 최적화의 극단적인 방법은 추적 호출을 #ifdef 블록 안에 내장하여 성능 부하를 제거하는 것이다.


```cpp
#ifdef TRACE
    Trace t("myFunction");
    t.debug("Some message");
#endif
```

하지만 #ifdef는 로깅을 켜고 끌때마다 컴파일을 새로 해야 한다는 단점이 있다.
이를 보완하기 위해서 로깅을 남길지 여부를 로깅을 하기 전에 확인하도록 Trace 클래스를 구현할 수 있다.


```cpp
void Trace::debug(const string &msg)
{
    if (traceIsActive) {
        // 로그 메시지 기록
    }
} 
```

해당 코드는 로깅을 수행 중일때의 성능을 고려하지 않은 코드이다.

코드가 최고의 성능을 낼 수 있도록 일반적인 동작을 수행하는 동안에는 기본적인 로깅을 꺼야하고, 로깅 오버헤드는 최소가 되어야 한다. 일반적인 로깅 구문은 다음과 같은 형식이 될 것이다.


```cpp
t.debug("x = " + itoa(x));
```

해당 구문은 성능 이슈가 몇 가지 존재한다.

- “x = “에서 임시 string 객체를 만든다.
- itoa(x)를 호출한다.
- itoa()가 반환한 char 포인터로 부터 임시 string 객체를 만든다.
- 세 번째 임시 string 객체를 만들어 위 두 string 객체를 연결한다.
- debug() 호출이 반환한 직후 세 string 임시 객체는 모두 소멸한다.

string 객체와 Trace 객체를 생성하고 소멸하는 오버헤드는 많으면 수백 명령까지 차이가 날수 있다.
초기 로깅 구현

로깅 성능 비교를 위한 간단한 테스트 함수를 작성해보자


```cpp
int addOne(int x)
{
    return x + 1;
}
```

해당 함수를 약 1000000회 정도 수행하니 3 millisecond가 걸렸다.
아래는 로깅을 위한 함수를 간단하게 작성한것이다.


```cpp
class Trace {
public:
    inline Trace(const string& name) : theFunctionName(name)
    {
        if (traceIsActive) {
            cout << "Enter Function " << name << endl;
        }
    }
    inline ~Trace()
    {
        if (traceIsActive) {
            cout << "Exit Function " << theFunctionName << endl;
        }
    } 
    inline void debug(const string &msg)
    {
        if (traceIsActive) {
            cout << msg << endl;
        }
    }
    static bool traceIsActive;
private:
    string theFunctionName;
};
```

Trace 클래스를 삽입한 결과 아래와 같다.


```cpp
int addOne(int x)
{
    string name = "addOne";
    Trace t(name);

    return x + 1;
}
```

위와 동일하게 addOne을 수행할 경우, 52 millisecond가 걸렸다. Trace 클래스를 삽입한 결과로 17배 정도 성능 저하가 발생한 것이다.
C++ 성능과 관련해 각기 다른 의견이 있을 수 있지만, 모두가 동의하는 기본 법칙이 있다.

- I/O는 부하를 많이 주는 작업이다.
- 함수 호출도 오버헤드의 원인 중 하나이므로 자주 호출되는 함수는 인라인으로 작성한다.
- 객체를 복사하는 것은 부하를 많이 주는 작업이다. 값 전달 보다는 참조로 전달하는 방식을 사용하자.

하지만 위의 법칙들 모두 Trace 클래스에서 모두 준수하고 있는 상황이다.
위의 addOne 함수를 보면 로깅을 사용하지 않는 상황에서는 실제로 사용하지 않는 객체를 생성하고 소멸시키는 부분이 있다.

이 부분만 아니면 속도는 더 빨라질수 있다.
다음 버전으로 불필요한 객체 생성을 제거하여 보자.


```cpp
int addOne(int x)
{
    char* name = "addOne";
    EnhancedTrace t(name);

    return x + 1;
}
```

일단 addOne에서 불필요하게 생성 소멸된 string 객체인 name을 char*로 변경하였다.

그리고 로깅이 켜져 있는 경우에만 해당 name 문자열을 담을 메모리를 할당하도록 수정했다.


```cpp
class EnhancedTrace {
public:
    inline EnhancedTrace(const char * const name) : theFunctionName(NULL)
    {
        if (traceIsActive) {
            cout << "Enter Function1 " << name << endl;
            theFunctionName = (char*)malloc(sizeof(char) * strlen(name));
        }
    }
    ~EnhancedTrace()
    {
        if (traceIsActive) {
            cout << "Exit Function1 " << theFunctionName << endl;
            free(theFunctionName);
        }
    }
     void debug(const string &msg)
    {
        if (traceIsActive) {
            cout << msg << endl;
        }
    }
    static bool traceIsActive;
private:
    char* theFunctionName;
};
```

아까와 동일하게 addOne 함수를 수행한 결과 12 millisecond가 걸린것을 확인했다.

즉 17배의 성능 저하에서 4배의 성능 저하로 많은 오버헤드가 줄어든것을 확인할 수 있다.
키포인트

- 객체를 정의하면 생성자와 소멸자의 형태로 조용한 실행이 발생한다. 대게의 경우 생성자와 소멸자는 부하가 아니기 때문에 조용한 실행이라고 한다. 하지만, 위의 예제에서와 같이 현저한 오버헤드를 발생하는 경우도 존재한다.
- 객체를 참조로 전달하는 것이 무조건적인 성능을 보장하는 것은 아니다. 일단 불필요한 객체의 생성과 소멸은 막을 수 있다면 성능에 도움이 된다.
- char 포인터는 가끔 string에 비해서 더 효율적이면서도 단순한 작업을 잘 수행한다.
- 인라인. 작은 함수를 자주 호출하면 인라인을 통해서 함수 호출 오버헤드를 줄일수 있다.