---
layout: post
title: Adapter Pattern
date: '2011-12-10T08:07:15+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512406816/adapter-pattern
---
Adapter Pattern은 “이미 제공되고 있는것”과 “제공할것” 사이의 차이를 없애주는 Pattern이다.
기존의 Interface를 사용하는 interface를 client에 제공하는 방법을 통해서, Adapter Pattern은 일반적으로 interface 호환성 때문에 함께 사용할 수 없는 class들을 함께 작동하게 한다.

adapter는 새 interface의 호출을 기존의 interface 호출로 해석해준다. 그리고 Data를 자신이 원하는 적절한 형태로 변경도 할 수 있다.
Adapter Pattern은 구현을 크게 두가지 방법으로 나눌 수 있다.
- 상속을 통한 구현
- 위임을 통한 구현
-상속을 통한 구현
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/adapter-pattern.png"><img src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/adapter-pattern.png?resize=546%2C381" alt="adapter pattern" class="aligncenter size-full wp-image-695" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
public class AdapterPattern {
    public static void main(String[] args) {
        Print p = new PrintBanner("Hello");
       
        p.printStrong();
        p.printWeak();
    }
}

adaptee
class Banner {
    private String string;
    public Banner(String string) {
        this.string = string;
    }
   
    public void showWithParen() {
        System.out.println("(" + string + ")");
    }
   
    public void showWithAster() {
        System.out.println("*" + string + "*");
    }
}

target
interface Print {
    public void printWeak();
    public void printStrong();
}

adapter
class PrintBanner implements Print {
    private Banner banner;
    public PrintBanner(String string) {
        banner = new Banner(string);
    }
   
    public void printWeak() {
        banner.showWithParen();
    }
   
    public void printStrong() {
        banner.showWithAster();
    }
}
```
