---
layout: post
title: Factory Method Pattern
date: '2011-12-10T08:10:34+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512445356/factory-method-pattern
---
Factory Method Pattern은Factory 개념을 구현한 Object-oriented design pattern이다.

다른 생성 패턴과 같이, 이것도 생성될 정확한 객체의 class를 명시하지 않으며, 이러한 객체의 생성 문제를 다룬다.

그렇다면, 기존의 객체 생성의 문제점은 무엇일까?
- 객체의 생성은 종종 많은 code의 중복을 야기한다.
- 객체를 구성하기 위해서 접근할 수 없는 정보를 요구하기도 한다.
- 충분한 추상 Level을 제공하지 못 하기도 한다.
- 생성하는 부분이 객체 생성과 무관한 부분이기도 하다.
그리고 객체 생성에서 어떤 객체의 생성과 객체의 Lifetime의 관리, build-up 관리, 객체의 tear-down 문제를 요구하기도 한다.
이러한 요구사항과 문제점들의 해결을 위해서, Factory Method pattern이 사용될 수 있으며, 이름에서 보듯이 객체의 생성을 위한 목적임을 외부에서 보더라도 한 눈에 확인 할 수 있는 장점이 생긴다.
기본 class diagram
<a href="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/Factory-method.png"><img src="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/Factory-method.png?resize=349%2C206" alt="Factory method" class="aligncenter size-full wp-image-690" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
import java.util.ArrayList;
import java.util.List;


public class FactoryMethodPattern {
    public static void main(String[] args) {
        Factory factory = new IDCardFactory();
        Product card1 = factory.create("길동");
        Product card2 = factory.create("철수");
        Product card3 = factory.create("영희");
        card1.use();
        card2.use();
        card3.use();
    }
}

abstract class Product {
    public abstract void use();
}

abstract class Factory {
    public final Product create(String owner) {
        Product p = createProduct(owner);
        registerProduct(p);
        return p;
    }
   
    protected abstract Product createProduct(String owner);
    protected abstract void registerProduct(Product product);
}

class IDCard extends Product {
    private String owner;
    IDCard(String owner) {
        System.out.println(owner + "의 카드를 만듭니다.");
        this.owner = owner;
    }
   
    public void use() {
        System.out.println(owner + "의 카드를 사용합니다.");
    }
   
    public String getOwner() {
        return owner;
    }
}

class IDCardFactory extends Factory {
    private List&lt;String&gt; owners = new ArrayList&lt;String&gt;();
    protected Product createProduct(String owner) {
        return new IDCard(owner);
    }
    protected void registerProduct(Product product) {
        owners.add(((IDCard)product).getOwner());
    }
    public List getOwners() {
        return owners;
    }
}
```
