---
layout: post
title: "[CMAKE] TUTORIAL STEP2"
date: '2016-06-07T02:21:20+09:00'
tags:
- build
- cmake
- tutorial
tumblr_url: https://appchemist.tumblr.com/post/145546954146/cmake-tutorial-step2
---
이제 우리 프로젝트에 라이브러리를 추가해보자. 해당 라이브러리는 우리가 앞 서 구현한 제곱근을 계산하는 라이브러리다. 그러면 컴파일러에서 제공하는 기본 제곱근 계산 함수 대신에 이 라이브러리를 사용할 수 있다. 해당 튜토리얼을 위해서 이 라이브러리는 MathFunctions라는 디렉토리에 두도록 한다. 그리고 이 라이브러리의 CMakeLists에는 아래의 한 줄을 추가해야 한다.


```bash
add_library(MathFunctions mysqrt.cpp)
```

mysqrt.cpp 는 mysqrt 라는 컴파일러의 sort 함수와 유사한 기능을 제공하는 함수를 가지고 있다. 새 라이브러리를 빌드하여 사용하기 위해서 add_subdirectory 함수를 최상위 CMakeLists 파일에 추가해줘야 한다. 또한 Function Prototype을 찾기 위해서  MathFunctions/mysqrt.h 해더 파일 위치를 지정하기 위해서 include directory를 추가해줘야 한다. 최종적으로 아래와 같이 수정되어야 한다.


```bash
include_directories("${PROJECT_SOURCE_DIR}/MathFunctions")
 add_subdirectory(MathFunctions)
  # add the executable
 add_executable(Tutorial tutorial.cpp)
 target_link_libraries(Tutorial MathFunctions)
```

자, 이제 MathFunctions 라이브러리를 선택적으로 사용해보자. 해당 튜토리얼에서는 선택적으로 사용할 이유는 없다. 하지만 제 3의 코드에 의존적인 거대한 라이브러리나 여러 라이브러리를 사용해야 할 수도 있다. 최상위 CMakeLists 파일에 선택적으로 라이브러리를 추가하기 위한 첫 과정은 다음과 같다.


```bash
option(USE_MYMATH         "Use tutorial provided math implementation" ON)
```

CMake GUI에 기본값인 ON이 노출되며, 사용자가 원하는 값으로 변경 가능하다. 여기서 설정한 값은 캐쉬에 저장되며 다음에 사용자가 다시 설정할 필요가 없다. MathFunctions 라이브러리를 선택적으로 빌드하고 링킹하기 위한 다음 변경사항은 최상위 CMakeLists 파일을 수정할 것이며 다음과 같다.


```bash
# add the MathFunctions library?
if (USE_MYMATH)
    include_directories("${PROJECT_SOURCE_DIR}/MathFunctions")
    add_subdirectory(MathFunctions)
    set(EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
include_directories(${CMAKE_BINARY_DIR})

# add the excutable
add_executable(Tutorial tutorial.cpp)
target_link_libraries(Tutorial ${EXTRA_LIBS})
```

MathFunctions를 컴파일하고 사용할지는 USE_MYMATH의 설정을 따른다. EXTRA_LIBS 변수는 이후 추가적인 선택적 라이브러리를 모아서 실행파일에 링크하기 위함이다. 큰 프로젝트에서 많은 선택적 컴포넌트들을 사용하기 위한 일반적 접근방법이다. 위의 대응하는 소스코드의 변화는 매우 직관적이다.


```cpp
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include "TutorialConfig.h"
#ifdef USE_MYMATH
#include <mysqrt.h>
#endif

int main(int argc, char *argv[])
{
    if (argc < 2)
    {
        fprintf(stdout,"%s Version %d.%d\n", argv[0],
                Tutorial_VERSION_MAJOR,
                Tutorial_VERSION_MINOR);
        fprintf(stdout,"Usage: %s number\n",argv[0]);
        return 1;
    }

    double inputValue = atof(argv[1]);
#ifdef USE_MYMATH
    double outputValue = mysqrt(inputValue);
#else
    double outputValue = sqrt(inputValue);
#endif
    fprintf(stdout, "The square root of %g is %g\n", inputValue, outputValue);

    return 0;
}
```

소스코드에서 USE_MYMATH를 또 사용하게 된다. 앞에서 TutorialConfig.h.in을 통해서 설정했듯이 USE_MYMATH는 CMake에서 소스코드로 제공된다. TutorialConfig.h.in에 아래 한줄을 추가해주면 된다.


```bash
#cmakedefine USE_MYMATH
```
