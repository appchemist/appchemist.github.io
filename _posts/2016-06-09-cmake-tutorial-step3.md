---
layout: post
title: "[CMAKE] TUTORIAL STEP3"
date: '2016-06-09T10:27:12+09:00'
tags:
- build
- cmake
- tutorial
tumblr_url: https://appchemist.tumblr.com/post/145659714436/cmake-tutorial-step3
---
다음 과정으로 설치 규칙과 테스트 관련 정보를 프로젝트에 추가해보자. 설치 규칙은 매우 직관적이다. 아래의 두줄을 라이브러리의 CMakeLists 파일에 다음 두줄을 추가해서 MathFunctions 라이브러리와 헤더 파일을 설치할 수 있다.


```bash
install (TARGETS MathFunctions DESTINATION bin)
 install (FILES mysqrt.h DESTINATION include)
```

최상위 CMakeLists 파일에 다음 두 줄을 추가해서 실행파일과 설정 헤더 파일을 설치할 수 있다.


```bash
# add the install targets install (TARGETS Tutorial DESTINATION bin) 
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" 
        DESTINATION include)
```

이제 Tutorial을 빌드하고, make install 을 실행하면 헤더 파일과 라이브러리, 실행 파일이 설치 된다. CMAKE_INSTALL_PREFIX 변수는 해당 파일들이 설치될 root 디렉토리를 지정한다.

Test를 추가하는 것도 매우 직관적이다. 최상위 CMakeLists 파일에 어플리케이션을 확인하는 여러 테스트 코드들을 추가할 수 있다.


```bash
include(CTest)  
#does the application run 
add_test(TutorialRuns Tutorial 25)  

# does it sqrt of 25 
add_test(TutorialComp25 Tutorial 25)  
set_tests_properties(TutorialComp25 PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")  

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
```

빌딩 이후에 테스트를 실행하기 위해서 ctest 커맨드라인 툴을 실행한다.

첫 테스트는 간단하게 어플리케이션이 동작하는지 segfault 또는 크래쉬가 발생하는지 그리고 0을 반환하는 값을 가지고 있는지 확인한다. 이것은 CTest 테스트의 간단한 형태이다. 다음 몇몇 테스트는 테스트의 결과 문자열의 특정 문자열을 포함하고 있는지 확인하기 위해서 PASS_REGULAR_EXPRESSION 테스트 속성을 사용한다. 이 경우에는 제곱근의 결과가 무엇이 나와야 하는지 그리고 잘못된 값을 넣을때 나오는 사용 메시지를 확인한다. 만약에 다른 입력값에 따른 많은 테스트를 추가하고 싶다면 메크로 사용해보자.


```bash
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
