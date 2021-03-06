---
layout: post
title: ThreadLocal
date: '2011-12-10T08:49:15+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118200677481/threadlocal
---
스레드 내보의 값과 값을 가지고 있는 객체를 연결해 스레드 한정 기법을 적용할수 있도록 도와주는 형식적인 방법으로 ThreadLocal이 있다.
이 ThreadLocal은 호출하는 스레드마다 다른 값을 사용할 수 있도록 관리해준다.
즉, ThreadLocal의 get 메소드를 호출하면 현재 실행 중인 스레드에서 최근에 set 메소드로 저장했던 값을 가지고 온다.

이러한 ThreadLocal은 여러 스레드에서 변수가 임의로 공유되는 상황을 막기 위해서 자주 사용이 된다.

예를 들어 데이터베이스에 접속할 때 매번 Connection 인스턴스를 생성하는 부담을 줄이고자, 시작 시점에 Connection 인스턴스를 전역 변수에 넣어두고 계속해서 사용하는 방법을 사용하기도 한다. 하지만 JDBC 연결은 스레드에 안전하지 않기 때문에 적절한 동기화 없이 사용할 수 없다.
다음은 이러한 상황에서 ThreadLocal을 이용해서 해결한 예이다.


```java
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {
    public Connection initialValue() {
        return DriverManager.getConnection(DB_URL);
    }
};

public static Connection getConnection() {
    return connectionHolder.get();
}
```

개념적으로 본다면, ThreadLocal<T> 클래스는 Map<Thread, T>라는 자료 구조로 구성되어 있다고 생각할 수 있다. 하지만, 실제로 이렇게 구성된 것은 아니다. 결과적으로 ThreadLocal은 스레드 안전성을 보장할 수 있다.
ThreadLocal을 사용하면 하나의 스레드에서 동일한 값을 공유하면서, 스레드 한정을 유지할 수 있어서 편리하지만, 이것을 사용하는 코드는 해당 프레임웍에 대한 의존성을 갖게 된다. ThreadLocal 은 전역변수는 아니지만 전역 변수처럼 동작하기 때문에 코드의 재사용성을 떨어트리고 SideEffect를 유발할 수 있다.