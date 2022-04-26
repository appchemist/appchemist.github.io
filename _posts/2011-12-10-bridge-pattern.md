---
layout: post
title: Bridge Pattern
date: '2011-12-10T08:33:51+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118749321591/bridge-pattern
---
객체지향 설계는 기본적으로 인터페이스와 구현을 분리한 접근 방법이다. 이렇게 되면, 객체의 구현 방식이 바뀌더라도 그 객체를 사용하는 프로그램은 수정하지 않아도 되기 때문에 변경를 한정할 수 있다.
그렇다면, <font color="red">Bridge Pattern을 언제 사용</font>하면 좋을까?
- 새로운 ‘기능’을 추가하고 싶은 경우
- 새로운 ‘구현’을 추가하고 싶은 경우
기능을 추가한다는 것은 검색에서 본다면, 웹 검색 기능과 컴퓨터 내부 검색 등과 같이 여러 기능을 들 수가 있다. 이러한 기능들은 자신이 추가할려고 하는 기능의 목적과 가까운 클래스를 찾아 그 클래스의 하위 클래스를 만들면 된다.
구현을 추가한다는 것은 같은 검색을 한다고 하더라도 OS에 따라 검색 구현이 달라지게 된다. 즉, 실제적으로 작동하는 OS에 따라서 그 구현이 다른 것인데 이것을 따로 구현한다면, 수정과 추가 구현사항이 생긴다면 일은 배가 될 것이다. 그리고, OS가 바뀔 때 마다 프로그램은 수정이 되어야 한다.

하지만, 인터페이스를 만들고 그 하위 클래스 들이 추상 메소드들을 실제로 구현한다면 위와 같은 상황을 방지할 수 있다. 결과적으로 하위 클래스의 역할 분담에 의해서 교환 가능성을 높일 수 있다.
<font color="red">Bridge Pattern의 특징</font>은 ‘기능 클래스 계층’과 ‘구현의 클래스 계층’을 분리하는 것이다.

기능과 구현을 분리해 두면 각각의 계층을 독립적으로 확장할 수 있다. 즉, 기능을 추가하고 싶으면 기능 클래스 계층에 클래스를 추가하지만, 구현 클래스 계층은 전혀 수정할 필요가 없다.
<font color="red">Bridge Pattern vs Adapter Pattern</font>

Bridge Pattern과 Adapter Pattern이 비슷해 보일 수도 있다.

하지만, 두 Pattern은 서로 다른 목적을 가진다. Bridge Pattern은 위에서와 같이 기능과 구현을 나누고 결합시키는 패턴이지만, Adapter Pattern은 인터페이스가 다른 클래스를 결합시키는 패턴이다.
일반적인 Bridge Pattern의 구조
<a href="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/bridge-pattern.png"><img src="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/bridge-pattern.png?resize=720%2C360" alt="bridge pattern" class="aligncenter size-full wp-image-670" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
class Display {
    private DisplayImpl impl;
    public Display(DisplayImpl impl){
        this.impl = impl;
    }
   
    public void open() {
        impl.rawOpen();
    }
   
    public void print() {
        impl.rawPrint();
    }
   
    public void close() {
        impl.rawClose();
    }
   
    public final void display() {
        open();
        print();
        close();
    }
}

class CountDisplay extends Display {
    public CountDisplay(DisplayImpl impl) {
        super(impl);
    }
    public void multiDisplay(int times) {
        open();
        for(int i = 0; i &lt; times; i++) {
            print();
        }
        close();
    }
}

interface DisplayImpl {
    public void rawOpen();
    public void rawPrint();
    public void rawClose();
}

class StringDisplayImpl implements DisplayImpl {
    private String string;
    private int width;
    public StringDisplayImpl(String string) {
        this.string = string;
        this.width = string.getBytes().length;
    }
   
    public void rawOpen() {
        printLine();
    }
    public void rawPrint() {
        System.out.println("|" + string + "|");
    }
    public void rawClose() {
        printLine();
    }
   
    private void printLine() {
        System.out.print("+");
        for(int i = 0; i &lt; width; i++) {
            System.out.print("-");
        }
        System.out.println("+");
    }
}
```
