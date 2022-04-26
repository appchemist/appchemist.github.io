---
layout: post
title: Boost.Preprocessor Library
date: '2016-08-02T04:21:32+09:00'
tags:
- boost
- library
- macro
- preprocessor
tumblr_url: https://appchemist.tumblr.com/post/148340227696/boost-preprocessor-library
---
코드를 읽다가 Boost에서 제공하는 Preprocessor 라이브러리가 있다는 사실을 알게 되었다. Preprocessor 라이브러리를 사용하여 함수의 원형을 작성 하고 여러개의 함수 인자를 받을 수 있도록 전처리기가 함수를 생성하여 오버로딩을 하는 코드였다. 일단, Preprocessor 라이브러리의 메크로로 생성한 코드는 컴파일 전처리 단계에서 코드가 생성이 된다. 이로 인해서 컴파일 시간이 늘어날 수 있겠지만, 실행 단계에서의 부하는 일반 함수를 호출하는 것과 동일한 부하가 생긴다는 것을 알 수 있다. 일단 내가 보고 있는 코드에서 사용하는 메크로들에 대해서 정리해보고자 한다.
# BOOST_PP_CAT

사용법은 BOOST_PP_CAT(a, b)와 같이 사용한다. 해당 메크로는 이름에서 느껴지듯 인자들을 연결 시킨다. 즉, 메크로가 모두 풀리면, ab와 같은 형태가 된다.

실제 예제를 보자. (컴파일 시, -save-temps 옵션을 주면 전처리가 수행된 결과 코드를 확인할 수 있다.)


```cpp
// 예제코드(BOOST_PP_CAT)
#include <boost/preprocessor.hpp>

#define check BOOST_PP_CAT(x, BOOST_PP_CAT(y, z))

int main(int argc, char* argv[])
{
    int check;
    return 0;
}
```

```cpp
// 전처리 후, 코드
int main(int argc, char* argv[])
{
    int xyz;	// check 상수가 xyz로 변경 됨.
    return 0;
}
```
# BOOST_PP_INC

사용법은 BOOST_PP_INC(a)와 같이 사용한다. 해당 메크로는 인자의 값을 1 증가 시킨다.

실제 예제를 보자.

```cpp
#include <iostream>
#include <boost/preprocessor.hpp>

using namespace std;

int main(int argc, char* argv[])
{
    cout << BOOST_PP_INC(1) << endl << BOOST_PP_INC(BOOST_PP_INC(5)) << endl;
    return 0;
}
```


```cpp
// 전처리 후, 코드
int main(int argc, char* argv[])
{
    cout << 2 << endl << 7 << endl;
    return 0;
}
```
# BOOST_PP_ENUM_PARAMS

사용법은 BOOST_PP_ENUM_PARAMS(count, param)와 같이 사용한다. 해당 메크로는 쉼표로 구분되는 파라미터들을 생성해준다. param##0, param##1, param##2, … 과 같은 형태로 count – 1까지 생성이 된다. 


```cpp
#include <boost/preprocessor.hpp>

void myFunc(BOOST_PP_ENUM_PARAMS(3, int i)) {
    return;
}

int main(int argc, char* argv[])
{
    myFunc(1, 2, 3);
    return 0;
}
```


```cpp
void myFunc( int i0 , int i1 , int i2) {	// myFunc의 인자 값이 메크로가 펼쳐진 결과이다.
    return;
}

int main(int argc, char* argv[])
{
    myFunc(1, 2, 3);
    return 0;
}
```
# BOOST_PP_ENUM_BINARY_PARAMS

사용법은 BOOST_PP_ENUM_BINARY_PARAMS(count, p1, p2)와 같이 사용한다. 해당 메크로는 쉼표로 구분되는 두개의 파라미터 리스트를 생성한다. p1##0 p2##0, p1##1 p2##1, … 과 같은 형태로 count – 1까지 생성된다. 


```cpp
#include <boost/preprocessor.hpp>

template<BOOST_PP_ENUM_PARAMS(3, typename T)>
void myFunc(BOOST_PP_ENUM_BINARY_PARAMS(3, T, t)) {
  return;
}

int main(int argc, char* argv[])
{
  myFunc(1, 2, 3);
  return 0;
}
```


```cpp
template< typename T0 , typename T1 , typename T2>
void myFunc( T0 t0 , T1 t1 , T2 t2) {		// myFunc의 인자 값이 BOOST_PP_ENUM_BINARY_PARAMS 메크로의 결과
  return;
}

int main(int argc, char* argv[])
{
  myFunc(1, 2, 3);
  return 0;
}
```
# BOOST_PP_REPEAT_FROM_TO

사용법은 BOOST_PP_REPEAT_FROM_TO(first, last, macro, data)와 같이 사용한다. 해당 메크로는 대상이 되는 메크로를 수평으로 반복하여 생성하는 메크로이다. 여기서 각 인자 값에 대해서 알아보자.

first : 메크로의 반복의 시작 값

last : (last – 1)안 반복의 긑 값

macro : macro(z, n, data) 형태의 메크로의 이름을 나타내고, BOOST_PP_REPEAT_FROM_TO 메크로로 반복하여 생성될 메크로이다. n이 현재 반복 값, data는 다음에 나올 BOOST_PP_REPEAT_FROM_TO의 data 인자 값

data : 생성될 메크로로 전달되는 보조 데이터


```cpp
#include <boost/preprocessor.hpp>

using namespace std;

#define DECL(z, n, text) BOOST_PP_CAT(text, n) = n;
BOOST_PP_REPEAT_FROM_TO(5, 10, DECL, int x)

#define TEMPLATE(Z, N, DATA)                                            \
  template <BOOST_PP_ENUM_PARAMS(N, typename A)>                        \
  void myFunc(                                                          \
      BOOST_PP_ENUM_BINARY_PARAMS(N, A, a))                             \
  {                                                                     \
        cout << a0 << endl;                                             \
        return;                                                         \
  }

BOOST_PP_REPEAT_FROM_TO(1, 11, TEMPLATE, _)
#undef TEMPLATE

int main(int argc, char* argv[])
{
  myFunc(1, 2, 3);
  return 0;
}
```

```cpp
using namespace std;

// BOOST_PP_REPEAT_FROM_TO(5, 10, DECL, int x) 의 결과
int x5 = 5; int x6 = 6; int x7 = 7; int x8 = 8; int x9 = 9;
# 18 "test_macro.cpp"
// BOOST_PP_REPEAT_FROM_TO(1, 11, TEMPLATE, _) 의 결과
template < typename A0> void myFunc( A0 a0) { cout << a0 << endl; return; } template < typename A0 , typename A1> void myFunc( A0 a0 , A1 a1) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2> void myFunc( A0 a0 , A1 a1 , A2 a2) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2 , typename A3> void myFunc( A0 a0 , A1 a1 , A2 a2 , A3 a3) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2 , typename A3 , typename A4> void myFunc( A0 a0 , A1 a1 , A2 a2 , A3 a3 , A4 a4) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2 , typename A3 , typename A4 , typename A5> void myFunc( A0 a0 , A1 a1 , A2 a2 , A3 a3 , A4 a4 , A5 a5) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2 , typename A3 , typename A4 , typename A5 , typename A6> void myFunc( A0 a0 , A1 a1 , A2 a2 , A3 a3 , A4 a4 , A5 a5 , A6 a6) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2 , typename A3 , typename A4 , typename A5 , typename A6 , typename A7> void myFunc( A0 a0 , A1 a1 , A2 a2 , A3 a3 , A4 a4 , A5 a5 , A6 a6 , A7 a7) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2 , typename A3 , typename A4 , typename A5 , typename A6 , typename A7 , typename A8> void myFunc( A0 a0 , A1 a1 , A2 a2 , A3 a3 , A4 a4 , A5 a5 , A6 a6 , A7 a7 , A8 a8) { cout << a0 << endl; return; } template < typename A0 , typename A1 , typename A2 , typename A3 , typename A4 , typename A5 , typename A6 , typename A7 , typename A8 , typename A9> void myFunc( A0 a0 , A1 a1 , A2 a2 , A3 a3 , A4 a4 , A5 a5 , A6 a6 , A7 a7 , A8 a8 , A9 a9) { cout << a0 << endl; return; }


int main(int argc, char* argv[])
{
  myFunc(1, 2, 3);
  return 0;
}
```
지금까지 Boost의 Preprocessor 라이브러리의 메크로 몇 가지를 확인해봤다. 직접 전처리기가 생성하는 코드도 확인해보면서 매크로의 결과를 직접 확인했다.
위와 같이 인자 값만 다르고 다른 부분이 모두 동일한 경우에 위와 같은 메크로들을 사용하여 중복 없이 여러 함수들을 전처리기를 통해서 생성할 수 있다. 매우 편리한 기능이다.
만약 인자 값만 다양하게 생성하고 싶다면, 위에서 살펴본 메크로 보다는 variadic template을 사용하는 방법이 더 직관적이면서 코드량도 줄어든다.
하지만, 내가 본 코드의 경우 Boost Preprocessor 라이브러리를 통하여 구현을 하였는데, 이유가 일단 C++ 11 이후에서 지원한다는 점이다. 추가적으로 variadic template과 lambda를 함께 사용할 수 없었던 버그 때문이였다. 현재는 해당 버그는 해결이 된 상황이다.
여튼 boost에서 전처리기를 위한 라이브러리를 따로 제공하고 있다는 점이 흥미로웠다.