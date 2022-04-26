---
layout: post
title: "[CMAKE] TUTORIAL STEP4"
date: '2016-06-09T10:30:04+09:00'
tags:
- build
- cmake
- tutorial
tumblr_url: https://appchemist.tumblr.com/post/145659803176/cmake-tutorial-step4
---
이번에는 대상 플랫폼이 지원하지 않는 기능에 의존적인 몇몇 코드들을 추가하는 상황에 대해서 알아보자.

이번 예제에서는 대상 플랫폼이 log와 exp 함수의 지원여부에 의존적인 코드를 추가할 것이다. 물론 거의 대부분의 플랫폼에서 해당 함수를 지원한다. 하지만 이번 튜토리얼에서는 대부분 지원하지 않는다고 가정한다. 만약 사용하는 플랫폼이 log 함수를 가지고 있고 우리가 만든 mysqrt 함수를 사용한다면, 우리는 먼저 최상위 CmakeLists 파일의 CheckFunctionExists.cmake 메크로를 사용해서 확인할 것이다.


```bash
# does this system provide the log and exp functions? 
include(CheckFunctionExists) 
check_function_exists(log HAVE_LOG) 
check_function_exists(exp HAVE_EXP)
```

다음으로 우리는 TutorialConfig.h.in 을 수정할 것이다. 만약 CMake 가 해당 플랫폼에서 log와 exp의 존재 여부를 알려주는 변수를 정의할 것이다.


```bash
#cmakedefine USE_MYMATH 
#cmakedefine HAVE_LOG 
#cmakedefine HAVE_EXP
```

TutorialConfig.h를 위한 configure_file 명령을 수행 전에 log와 exp 함수의 존재 여부를 확인하는 것이 중요하다. configure_file 명령은 현재의 CMake의 설정값을 사용하여 즉시 파일을 설정한다. 최종적으로 아래의 코드를 사용하여 log와 exp 함수를 사용해서 mysqrt 함수의 다른 구현을 만들수 있다.


```cpp
#if defined(HAVE_LOG) && defined(HAVE_EXP) 
    result = exp(log(x)*0.5); 
#else
```
