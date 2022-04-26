---
layout: post
title: Template Method Pattern
date: '2011-12-10T08:08:36+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512436541/template-method-pattern
---
Template Method는 algorithm에서 program skeleton을 정의합니다. 하나 이상의 algorithm 과정들을 subclass에서 그 행동을 재정의합니다. 반면 재정의된 algorithm의 단계를 구성하는 최상위 알고리즘은 그대로 따르게 됩니다.
우선 algorithm의 뼈대를 제공하는 첫 class를 생성합니다.

이 과정에서는 abstract method를 사용해서 algorithm 뼈대를 구현합니다. 이후에 subclass에서 abstract method의 실제 행동을 정의함으로써 그 내용을 변경할 수 있습니다.

즉, 일반적인 algorithm은 한 장소에서 보존하지만, 구체적인 과정은 subclass에서 변경이 될 수 있습니다.
이와 같이 상위 클래스에서 처리의 뼈대를 결정하고, 하위 클래스에서 그 구체적인 내용을 결정하는 디자인 패턴을 Template Method Pattern이라고 한다.
기본적인 Class Diagram
<a href="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/Template-Method.gif"><img src="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/Template-Method.gif?resize=536%2C265" alt="Template Method" class="aligncenter size-full wp-image-692" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
public class TemplateMethodPattern {
    public static void main(String[] args) {
        AbstractDisplay d1 = new CharDisplay('H');
       
        AbstractDisplay d2 = new StringDisplay("Hello, Template");
       
        d1.display();
        d2.display();
    }
}

abstract class AbstractDisplay {
    public abstract void open();
    public abstract void print();
    public abstract void close();
    public final void display() {
        open();
        for(int i = 0; i &lt; 5; i++) {
            print();
        }
        close();
    }
}

class CharDisplay extends AbstractDisplay {
    private char ch;
    public CharDisplay(char ch) {
        this.ch = ch;
    }
   
    public void open() {
        System.out.print("&lt;&lt;");
    }
   
    public void print() {
        System.out.print(ch);
    }
   
    public void close() {
        System.out.println("&gt;&gt;");
    }
}

class StringDisplay extends AbstractDisplay {
    private String string;
    private int width;
   
    public StringDisplay(String string) {
        this.string = string;
        this.width = string.getBytes().length;
    }
   
    public void open() {
        printLine();
    }
   
    public void print() {
        System.out.println("|" + string + "|");
    }
   
    public void close() {
        printLine();
    }
   
    public void printLine() {
        System.out.print("+");
        for(int i = 0; i &lt; width; i++) {
            System.out.print("-");
        }
       
        System.out.println("+");
    }
}
```
