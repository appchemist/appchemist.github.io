---
layout: post
title: Prototype Pattern
date: '2011-12-10T08:30:24+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512454701/prototype-pattern
---
이 패턴은 생성할 객체들의 타입이 Prototype인 객체로 부터 결정이 된다.

그렇다면 언제 사용하면 좋을까?
- 종류가 너무 많아 클래스로 정리되지 않는 경우

다루는 Object의 종류가 너무 많아 각가의 클래스로 만들어 다수의 소스 파일을 작성해야하는 경우
- 클래스로 부터 인스턴스 생성이 어려운 경우

생성 과정이 복작한 작업이 많아 클래스로부터 인스턴스 생성이 어려운 경우
- framework와 생성할 인스턴스를 분리하고 싶은 경우
- 객체 생성 비용이 클 경우

객체는 일반적인 방법(예를 들어, new를 사용해서라든지)으로 객체를 생성하는 고유의 비용이 주어진 응용 프로그램 상황에 있어서 불가피하게 매우 클 때, 이 비용을 줄여준다.
일반적 구조
<a href="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/prototype.png"><img src="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/prototype.png?resize=720%2C158" alt="prototype" class="aligncenter size-full wp-image-684" data-recalc-dims="1"/></a>

예제 코드(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
import java.util.HashMap;


public class PrototypePattern {
    public static void main(String[] args){
        Manager manager = new Manager();
        UnderlinePen upen = new UnderlinePen('~');
        MessageBox mbox = new MessageBox('*');
        MessageBox sbox = new MessageBox('/');
        manager.register("strong message", upen);
        manager.register("warning box", mbox);
        manager.register("slash box", sbox);
        
        Product p1 = manager.create("strong message");
        p1.use("Hello, world");
        Product p2 = manager.create("warning box");
        p2.use("Hello, world");
        Product p3 = manager.create("slash box");
        p3.use("Hello, world");
    }
}

interface Product extends Cloneable {
    public abstract void use(String s);
    public abstract Product createClone();
}

class Manager {
    private HashMap&lt;String, Product&gt; showcase = new HashMap&lt;String, Product&gt;();
    public void register(String name, Product proto) {
        showcase.put(name, proto);
    }
    public Product create(String protoname){
        Product p = (Product)showcase.get(protoname);
        return p.createClone();
    }
}

class MessageBox implements Product {
    private char decochar;
    
    public MessageBox(char decochar) {
        this.decochar = decochar;
    }
    
    public void use(String s){
        int length = s.getBytes().length;
        for(int i = 0; i &lt;length + 4; i++) {
            System.out.print(decochar);
        }
        System.out.println(" ");
        System.out.println(decochar + " " + s + " " + decochar);
        for(int i = 0; i &lt; length + 4; i++) {
            System.out.print(decochar);
        }
        System.out.println(" ");
    }
    
    @Override
    public Product createClone() {
        Product p = null;
        try {
            p = (Product)clone();
        }catch(CloneNotSupportedException e){
            e.printStackTrace();
        }
        return p;
    }
}

class UnderlinePen implements Product {
    private char ulchar;
    
    public UnderlinePen(char ulchar) {
        this.ulchar = ulchar;
    }
    
    public void use(String s) {
        int length = s.getBytes().length;
        System.out.println(""" + s + """);
        System.out.print(" ");
        for(int i = 0; i &lt; length; i++) {
            System.out.print(ulchar);
        }
        System.out.println("");
    }
    
    @Override
    public Product createClone() {
        Product p = null;
        try {
            p = (Product)clone();
        }catch(CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return p;
    }
}
```
