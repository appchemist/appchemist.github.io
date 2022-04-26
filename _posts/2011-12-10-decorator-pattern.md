---
layout: post
title: Decorator Pattern
date: '2011-12-10T08:38:43+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512474711/decorator-pattern
---
새로운 기능을 추가하귀 위해서는 SubClass를 많이 이용을 한다. SubClass는 Compile Time에 기능을 추가할 수 있다.

그리고 추가된 기능은 Original Class의 모든 객체들에게 적용이 된다.

사실 설계시에 추후에 어떤 기능과 기능들의 조합이 더 필요할지 알기는 힘들다.

이 말은 추후에 새롭게 필요로 하는 기능과 그 기능들의 조합을 모두 구현해야 한다는 말이다.

이것은 OOP(Object Oriented Programming)에서 DRY(Don’t repeat yourself)에 위반된다고 볼 수 있다.
그렇다면, Decorator Pattern에 대해서 알아보자.

Decorator Pattern은 어떤 객체에 Runtime에 기능을 추가할 후 있다. 그리고 여러개의 Decorator를 추가 할 수 있는데, 추가된 Decorator는 Stack에 쌓이듯이 Override 된 Decorator Method들이 호출이 된다.

즉, 새롭게 필요로 하는 기능을 기존의 기능과 더하여 추가할 수 있다.

기존 JDK의 API 중에서 Decorator Pattern으로 구성이 된 것은, I/O관련 Library에서 찾아 볼 수 있다. (InputStream, OutputStream, Reader, Writer 등)
기본적인 구조
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/decorator.png"><img src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/decorator.png?resize=720%2C570" alt="decorator" class="aligncenter size-full wp-image-657" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
abstract class Display {
    public abstract int getColumns();
    public abstract int getRows();
    public abstract String getRowText(int row);
    public final void show() {
        for(int i = 0; i &lt; getRows(); i++) {
            System.out.println(getRowText(i));
        }
    }
}

class StringDisplay extends Display {
    private String string;
    public StringDisplay(String string) {
        this.string = string;
    }
   
    @Override
    public int getColumns() {
        return string.getBytes().length;
    }
   
    @Override
    public int getRows() {
        return 1;
    }
   
    @Override
    public String getRowText(int row) {
        if(row == 0) {
            return string;
        } else {
            return null;
        }
    }
}

abstract class Border extends Display {
    protected Display display;
    protected Border(Display display) {
        this.display = display;
    }
}

class SideBorder extends Border {
    private char borderChar;
   
    public SideBorder(Display display, char ch) {
        super(display);
        this.borderChar = ch;
    }
   
    @Override
    public int getColumns() {
        return 1 + display.getColumns() + 1;
    }
   
    @Override
    public int getRows() {
        return display.getRows();
    }
   
    @Override
    public String getRowText(int row) {
        return borderChar + display.getRowText(row) + borderChar;
    }
}

class FullBorder extends Border {
    public FullBorder(Display display) {
        super(display);
    }
   
    @Override
    public int getColumns() {
        return 1 + display.getColumns() + 1;
    }
   
    @Override
    public int getRows() {
        return 1 + display.getRows() + 1;
    }
   
    @Override
    public String getRowText(int row) {
        if(row == 0) {
            return "+" + makeLine('-', display.getColumns()) + "+" ;
        } else if(row == display.getRows() + 1) {
            return "+" + makeLine('-', display.getColumns()) + "+" ;
        } else {
            return "|" + display.getRowText(row - 1) + "|";
        }
    }
   
    private String makeLine(char ch, int count) {
        StringBuffer buf = new StringBuffer();
        for(int i = 0; i &lt; count; i++) {
            buf.append(ch);
        }
        return buf.toString();
    }
}
```
