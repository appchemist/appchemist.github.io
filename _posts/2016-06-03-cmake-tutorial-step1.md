---
layout: post
title: "[CMake] Tutorial Step1"
date: '2016-06-03T01:45:14+09:00'
tags:
- build
- cmake
- tutorial
tumblr_url: https://appchemist.tumblr.com/post/145343031481/cmake-tutorial-step1
---
항상 필요할때 필요한 부분만 찾아서 사용하던 CMake. 이번에 공부를 해보고자 CMake 사이트의 Tutorial을 번역하면서 예제를 따라 해보기로 했다.
A Basic Starting Point(Step1)
소스 코드로 부터 실행 파일을 만들수 있는 간다한 프로젝트에서 CMakeLists 파일에 단 2줄만 필요로 한다고 해보자. 이것은 우리 튜토리얼의 시작부분이다. CMakeLists 파일은 아래와 같다.


```cpp
cmake_minimum_required(VERSION 2.6)
 project(Tutorial)
 add_executable(Tutorial tutorial.cpp)
```

해당 예제는 CMakeLists 파일에서 소문자 명령을 사용하고 있다. CMake에서는 대문자 소문자 대소문자 명령을 모두 지원한다. tutorial.cpp를 위한 간단한 예제는 아래와 같다.


```cpp
#include <stdio.h>
 #include <stdlib.h>
 #include <math.h>
  int main(int argc, char *argv[]) 
{ 
    if (argc < 2)     
    {
        fprintf(stdout, "Usage: %s number\n", argv[0]); 
        return 1; 
    } 
    double inputValue = atof(argv[1]); 
    double outputValue = sqrt(inputValue); 
    fprintf(stdout, "The square root of %g is %g\n", inputValue, outputValue);
    return 0; 
}
```

Adding a Version Number and Configured Header File

우리가 처음에 추가한 기능은 실행 파일과 프로젝트에 버전 정보를 추가하는 것이다. 버전 정보를 소스코드 안에서 제공할수 있지만, 반면 CMakeLists 파일은 좀 더 유연함을 제공할 수 있다. 버전 정보를 추가하기 위해서 CMakeLists 파일을 아래와 같이 수정해야 한다.


```bash
cmake_minimum_required(VERSION 2.6)
project(Tutorial)

# The version number.
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
        "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
        "${PROJECT_BINARY_DIR}/TutorialConfig.h"
)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}")

# add the executable
add_executable(Tutorial tutorial.cpp test.cpp test.h)
```

Configured file은 binary tree에 쓰여지기 때문에 include files을 찾기 위한 경로 목록에 디렉토리를 추가해줘야 한다. 그리고 TutorialConfig.h.in 파일을 source tree에 아래의 내용으로 생성한다.


```cpp
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

 

CMake가 이 해더 파일을 설정했을 때, @Tutorial_VERSION_MAJOR@와 @Tutorial_VERSION_MINOR@의 값이 CMakeLists 파일의 값으로 수정이 된다. 다음으로 tutorial.cpp이 configured 파일을 포함하도록 수정하고 해당 버전 정보를 사용하도록 수정하겠다. 결과 코드는 아래와 같다.


```cpp
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include "TutorialConfig.h"


int main(int argc, char *argv[])
{
    if (argc < 2)
    {
        fprintf(stdout, "%s Version %d.%d\n", argv[0], Tutorial_VERSION_MAJOR, Tutorial_VERSION_MINOR);
        fprintf(stdout, "Usage: %s number\n", argv[0]);
        return 1;
    }
    double inputValue = atof(argv[1]);
    double outputValue = sqrt(inputValue);
    fprintf(stdout, "The square root of %g is %g\n", inputValue, outputValue);

    return 0;
}
```

주요한 변경은 TutorialConfig.h 해더 파일 포함과 버전 정보 출력이다.