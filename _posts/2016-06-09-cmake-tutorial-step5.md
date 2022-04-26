---
layout: post
title: "[CMAKE] TUTORIAL STEP5"
date: '2016-06-09T10:34:02+09:00'
tags:
- build
- cmake
- tutorial
tumblr_url: https://appchemist.tumblr.com/post/145659935926/cmake-tutorial-step5
---
이번에는 생성된 소스 파일을 어플리케이션을 빌드하는 과정에 어떻게 넣을수 있는지 보자.

이번 예제에서는 빌드 과정의 한 부분으로서 미리 계산된 제곱근의 테이블을 생성하고, 해당 테이블을 우리 어플리케이션에 컴파일해보자. 이를 위해서는 일단 테이블을 생성하는 프로그램이 필요하다. MathFunctions 의 하위 디렉토리에 이런 작업을 하는 MakeTable.cpp 파일을 생성하자


```cpp
//
// Created by appchemist on 2016. 6. 8..
//

#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main(int argc, char *argv[])
{
    int i;
    double result;

    // make sure we have enough arguments
    if (argc < 2)
    {
        return 1;
    }

    // open the output file
    FILE *fout = fopen(argv[1], "w");
    if (!fout)
    {
        return 1;
    }

    // create a source file with a table of square roots
    fprintf(fout, "double sqrtTable[] = {\n");
    for (i = 0; i < 10; ++i)
    {
        result = sqrt(static_cast<double>(i));
        fprintf(fout, "%g,\n", result);
    }

    // close the table with a zero
    fprintf(fout, "0};\n");
    fclose(fout);
    return 0;
}
```

테이블은 C++ 코드로 생성된다. 결과 파일 이름은 argument로 전달된다. 다음으로는 MathFunctions CMakeLists 파일에 MakeTable	을 빌드하기 위한 명령을 추가하고, 빌드 과정의 부분으로 이것을 실행할 것이다. 명령 몇개가 필요한데, 아래를 확인해보자.


```bash
# first we add the executable that generates the table
add_executable(MakeTable MakeTable.cpp)

# add the command to generate the source code
add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
        COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
        DEPENDS MakeTable
)

# add the binary tree directory to the search path for
# include files
include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_library(MathFunctions mysqrt.cpp ${CMAKE_CURRENT_BINARY_DIR}/Table.h)

install (TARGETS MathFunctions DESTINATION bin)
install (FILES mysqrt.h DESTINATION include)
```

먼저 다른 executable처럼 MakeTable도 추가를 했다. 그리고 사용자 정의 명령을 추가하여 MakeTable을 실행하여 Table.h를 생성하도록 했다. 다음으로 CMake에 mysql.cpp가 Table.h를 필요하다는 것을 알려줬고 이러면 MathFunction 라이브러리를 위해서 생성한 Table.h를 추가하는 작업은 끝이난다. 그리고 mysqrt.cpp가 Table.h를 찾고 include를 할 수 있도록 현재 바이너리 디렉토리 위치를 include directories에 추가해줘야 한다. 이 프로젝트를 빌드하면, 먼저 MakeTable가 빌드된다. 그리고 MakeTable을 실행하여 Table.h를 생성한다. 마지막으로 mysqrt.cpp를 컴파일하는데, mysqrt.cpp는 Table.h를 포함하고 MathFunctions 라이브러리를 생성한다. 현재까지의 내용은 아래와 같다.


```bash
include(CTest)

set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

find_package(PkgConfig REQUIRED)
include(CheckFunctionExists)
set(CMAKE_REQUIRED_INCLUDES math.h)
set(CMAKE_REQUIRED_LIBRARIES m)
check_function_exists(log HAVE_LOG)
check_function_exists(exp HAVE_EXP)
set(CMAKE_EXTRA_INCLUDE_FILES)
set(CMAKE_REQUIRED_LIBRARIES)


option(USE_MYMATH
        "Use tutorial provided math implementation" ON)

configure_file(
        "TutorialConfig.h.in" # Input
        "${PROJECT_BINARY_DIR}/TutorialConfig.h"    # Output
)

include_directories(${CMAKE_BINARY_DIR})

# add the MathFunctions library?
if (USE_MYMATH)
    include_directories("${PROJECT_SOURCE_DIR}/MathFunctions")
    add_subdirectory(MathFunctions)
    set(EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

# add the excutable
add_executable(Tutorial tutorial.cpp)
target_link_libraries(Tutorial ${EXTRA_LIBS})

# add the install targets
install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
        DESTINATION include)


#does the application run
add_test(TutorialRuns Tutorial 25)

# does it handle negative numbers
add_test (TutorialNegative Tutorial -25)
set_tests_properties (TutorialNegative
        PROPERTIES PASS_REGULAR_EXPRESSION "-25 is 0")

# does it handle small numbers
add_test (TutorialSmall Tutorial 0.0001)
set_tests_properties (TutorialSmall
        PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 is 0.01")

# does the usage message work?
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage
        PROPERTIES
        PASS_REGULAR_EXPRESSION "Usage:.*number")

# define a macro to simplify adding test, then use it
macro(do_test arg result)
    add_test (TutorialComp${arg} Tutorial ${arg})
    set_tests_properties(TutorialComp${arg}
            PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro(do_test)

# do a bunch of result based tests
do_test (25 "25 is 5")
do_test (-25 "-25 is 0")
```
