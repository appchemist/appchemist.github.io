---
layout: post
title: 클린 아키텍처 - 클린 아키텍처
categories: [Study, Book]
tags: [clean architecture, architecture]
---

# Table of Contents

1.  [의존성 규칙](#org428bde2)
2.  [엔티티](#org34c075d)
3.  [유스케이스](#org53d0f9b)
4.  [인터페이스 어댑터](#orgc1429c9)
5.  [프레임워크와 드라이버](#org8b2d503)
6.  [원은 네 개여야만 하나?](#org2ca7467)
7.  [경계 횡단하기](#org5a62f3f)
8.  [경계를 횡단하는 데이터는 어떤 모습인가](#org5a8b7e7)
9.  [결론](#org76d6d7f)

지난 수십 년간 우리는 시스템 아키텍처와 관련된 여러 가지 아이디어를 봐왔다. 아래의 내용도 여기에 포함된다.

-   육각형 아키텍처(Hexagonal Architecture)
-   DCI(Data, Context and Interaction)
-   BCE(Boundary-Control-Entiry)

이 아키텍처들의 목표는 모두 같다. 바로 관심사의 분리(seperation of concerns)다. 이들은 모두 소프트웨어를 계층으로 분리함으로써 관심사의 분리라는 목표를 달성할 수 있었다.

이들 아키텍처들의 시스템은 다음과 같은 특징을 지니도록 만든다.

-   프레임워크 독립성
-   테스트 용이성
-   UI 독립성
-   DB 독립성
-   모든 외부 에이전시에 대한 독립성

아래의 다이어그램은 이들 아키텍처 전부를 실행 가능한 하나의 아이디어로 통합하려는 시도다.

![img](/assets/img/클린_아키텍처/2020-08-25_10-12-11_2020-08-25-22.png)


<a id="org428bde2"></a>

# 의존성 규칙

위의 다이어그램에서 각각의 동심원은 소프트웨어에서 서로 다른 영역을 표현한다.
보통 안으로 들어갈수록 고수준의 소프트웨어가 된다. 바깥쪽 원은 메커니즘이고, 안쪽 원은 정책이다.

이러한 아키텍처가 동작하도록 하는 가장 중요한 규칙은 의존성 규칙(Dependency Rule)이다.

**소스 코드 의존성은 반드시 안쪽으로, 고수준의 정책을 향해야 한다.**

내부의 원에 속한 요소는 외부의 원에 속한 어떤 것도 알지 못한다. 특히 내부의 원에 속한 코드는 외부의 원에 선언된 어떤 것에 대해서도 그 이름을 언급해서는 절대 안 된다.

같은 이유로, 외부의 원에 선언된 데이터 형식도 내부의 원에서 절대로 사용해서는 안 된다.
우리는 외부 원에 위치한 어떤 것도 내부의 원에 영향을 주지 않기를 바란다.


<a id="org34c075d"></a>

# 엔티티

엔티티는 전사적인 핵심 업무 규칙을 캡슐화한다. 엔티티는 메서드를 가지는 객체이거나 일련의 데이터 구조와 함수의 집합일 수도 있다.
기업의 다양한 애플리케이션에서 엔티티를 재사용할 수만 있다면, 그 형태는 그다지 중요하지 않다.

전사적이지 않은 단순한 단일 애플리케이션을 작성하고 있다면 엔티티는 해당 애플리케이션의 업무 객체가 된다.
이 경우 엔티티는 가장 일반적이며 고수준인 규칙을 캡슐화한다.
운영 관점에서 특정 애플리케이션에 무언가 변경이 필요하더라도 엔티티 계층에는 절대로 영향을 주어서는 안 된다.


<a id="org53d0f9b"></a>

# 유스케이스

유스케이스 계층의 소프트웨어는 **애플리케이션에 특화된 업무 규칙을 포함한다.**
또한 유스케이스 계층의 소프트웨어는 시스템의 모든 유스케이스를 캡슐화하고 구현한다.
유스케이스는 엔티티로 들어오고 나가는 데이터 흐름을 조정하며, 엔티티가 자신의 핵심 업무 규칙을 사용해서 유스케이스의 목적을 달성하도록 이끈다.

유스케이스 계층에서 발생한 변경이 엔티티에 영향을 줘서는 안 된다.
또한 외부 요소에서 발생한 변경이 이 계층에 영향을 줘서도 안 된다.
유스케이스 계층은 이러한 관심사로부터 격리되어 있다.

하지만 운영 관점에서 애플리케이션이 변경된다면 유스케이스가 영향을 받으며, 따라서 이 계층의 소프트웨어에도 영향을 줄 것이다.
유스케이스의 세부사항이 변하면 이 계층의 코드 일부는 분명 영향을 받을 것이다.


<a id="orgc1429c9"></a>

# 인터페이스 어댑터

인터페이스 어댑터 계층은 일련의 어댑터들로 구성된다.
어댑터는 데이터를 유스케이스와 엔티티에게 가장 편리한 형식에서 DB나 웹 같은 외부 에이전시에게 가장 편리한 형식으로 변환한다.
이 계층은, 예를 들어 **GUI의 MVC 아키텍처를 모두 포괄** 한다. 프레젠터, 뷰, 컨트롤러는 모두 인터페이스 어댑터 계층에 속한다.
**모델은 그저 데이터 구조 정도에 지나지 않으며**, 컨트롤러에서 유스케이스로 전달되고, 다시 유스케이스에서 프레젠터와 뷰로 되돌아 간다.

마찬가지로 이 계층은 데이터를 엔티티와 유스케이스에게 가장 편리한 형식에서 영속성용으로 사용 중인 임의의 프레임워크(즉, DB)가 이용하기에 가장 편리한 형식으로 변환한다.
이 원 안에 속한 어떤 코드도 DB에 대해 조금도 알아서는 안 된다.
예컨데 SQL 기반의 DB를 사용한다면 모든 SQL은 이 계층을 벗어나서는 안 된다. 특히 이 계층에서도 DB를 담당하는 부분으로 제한되어야 한다.

또한 이 계층에는 데이터를 외부 서비스와 같은 외부적인 형식에서 유스케이스나 엔티티에서 사용되는 내부적인 형식으로 변환하는 또 다른 어댑터가 필요하다.


<a id="org8b2d503"></a>

# 프레임워크와 드라이버

위의 다이어그램에서 가장 바깥쪽 계층은 일반적으로 DB나 웹 프레임워크 같은 프레임워크나 도구들로 구성된다.
일반적으로 이 계층에서는 안쪽 원과 통신하기 위한 접합 코드 외에는 특별히 더 작성해야 할 코드가 그다지 많지 않다.

프레임워크나 드라이버 계층은 모두 세부사항이 위치하는 곳이다.
우리는 이러한 것들을 모두 외부에 위치시켜서 피해를 최소화 한다.


<a id="org2ca7467"></a>

# 원은 네 개여야만 하나?

위의 다이어그램에 표시한 원들은 그저 개념을 설명하기 위한 하나의 예시일 뿐이다.
네 개보다 더 많은 원이 필요할 수도 있다. 항상 네 개만 사용해야 한다는 규칙은 없다.
하지만 어떤 경우에도 **의존성 규칙은 적용** 된다.
소스코드 의존성은 항상 안쪽을 향한다.
안쪽으로 이동할수록 추상화와 정책의 수준은 높아진다. 가장 바깥쪽 원은 저수준의 구체적인 세부사항으로 구성된다.
그리고 안쪽으로 이동할수록 소프트웨어는 점점 추상화되고 더 높은 수준의 정책들을 캡슐화한다.
따라서 **가장 안쪽 원은 가장 범용적이며 높은 수준** 을 가진다.


<a id="org5a62f3f"></a>

# 경계 횡단하기

![img](/assets/img/클린_아키텍처/2020-08-25_12-03-48_2020-08-25-23.png)
위의 다이어그램은 원의 경계를 횡단하는 방법을 보여주는 예시이다.
우선 제어흐름에 주목해 보자. 제어흐름은 컨트롤러에서 시작해서, 유스케이스를 지난 후, 프레젠터가 실행되면서 마무리된다.
다음은 소스코드 의존성도 주목해 보자. 각 의존성은 유스케이스를 향해 안쪽을 가리킨다.

이처럼 제어흐름과 의존성의 방향이 명백히 반대여야 하는 경우, 대체로 의존성 역전 원칙을 사용하여 해결한다.

동적 다형성을 이용하여 소스 코드 의존성을 제어흐름과는 반대로 만들수 있고, 이를 통해 제어흐름이 어느 방향으로 흐르더라도 의존성 규칙을 준수할 수 있다.


<a id="org5a8b7e7"></a>

# 경계를 횡단하는 데이터는 어떤 모습인가

경계를 가로지르는 데이터는 흔히 간단한 데이터 구조로 이루어져 있다.
기본적인 구조체나 간단한 데이터 전송 객체(data transfer object) 등 원하는 대로 고를 수 있다. 그게 아니라면 데이터를 해시맵으로 묶거나 객체로 구성할 수도 있다.
중요한 점은 **격리된 간단한 데이터 구조가 경계를 가로질러 전달** 된다는 사실이다.
엔티티 객체나 데이터베이스의 행을 전달하는 일은 원치 않는다.
데이터 구조가 어떤 의존성을 가져 의존성 규칙을 위배하게 되는 일은 바라지 않는다.

따라서 경계를 가로질러 데이터를 전달할 때, **데이터는 항상 내부의 원에서 사용하기에 가장 편리한 형태** 를 가져야만 한다.


<a id="org76d6d7f"></a>

# 결론

소프트웨어를 계층으로 분리하고 의존성 규칙을 준수한다면 본질적으로 테스트하기 쉬운 시스템을 만들게 될 것이며, 그에 따른 이점을 누릴 수 있다.
DB나 웹 프레임워크와 같은 시스템의 외부 요소가 구식이 되더라도, 이들 요소를 야단스럽지 않게 교체할 수 있다.

