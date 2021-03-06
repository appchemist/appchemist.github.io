---
layout: post
title: Observer Pattern
date: '2011-12-10T08:43:17+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512500281/observer-pattern
---
Observer Pattern은 말 그대로 무언가 관찰 하는데 유용한 패턴이다.

Subject라는 자신의 상태 값을 가지고 있는 객체가 존재 한다. 그리고 Observer라는 Subject의 상태 값을 관찰하는 개체가 존재한다.

Observer가 Subject 객체를 관찰한다고 생각할 수 있지만, 실제로는 Subject가 자신의 상태가 변경이 되면 Observer에게 통보를 한다. 이렇게 통보를 받은 Observer는 Subject의 상태가 변경됐다는 것을 알고 Subject의 상태를 읽어올 수 있게 된다.
일반적으로 분산 이벤트 처리를 위해서 많이 사용된다고 한다.
Java API에서 Oserver 패턴을 지원한다.

Observer : java.util.Observer

Subject   : java.util.Observable
Java API를 사용하여 구현 시 참고사항

    1. java.util.Observer

    -update(Observable o, Object arg) :

    Observable은 어떤 Subject에서 상태가 변했는지 알 수 있다.

    Object는 변경 된 상태라고 생각할 수 있다.

    2. java.util.Observable

    -addObserver(Obserber o) :

    이 메소드를 통해서 이 subject를 관찰할 Observer를 등록

    -setChanged() :

    이 메소드를 통해서 현제 subject의 상태가 변경 됐다는 것을 표시, 호출하지 않으면 알려지지 않음

    -notifyObserver() :

    이 메소드를 통해서 등록된 모든 Observer에게 통보함
일반적인 구조
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/observer.png"><img src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/observer.png?resize=720%2C298" alt="observer" class="aligncenter size-full wp-image-640" data-recalc-dims="1"/></a>

예제소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
public class ObserverPattern {
    public static void main(String[] args) {
        NumberGenerator generator = new RandomNumberGenerator();
        Observer observer1 = new DigitObserver();
        Observer observer2 = new GraphObserver();
        generator.addObserver(observer1);
        generator.addObserver(observer2);
        generator.execute();
    }
}

interface Observer {
    public void update(NumberGenerator generator);
}

abstract class NumberGenerator {
    private List&lt;Observer&gt; observers = new ArrayList&lt;Observer&gt;();
    public void addObserver(Observer observer) {
        observers.add(observer);
    }
    public void deleteObserver(Observer observer) {
        observers.remove(observer);
    }
    public void notifyObservers() {
        Iterator&lt;Observer&gt; it = observers.iterator();
        while(it.hasNext()) {
            Observer o = it.next();
            o.update(this);
        }
    }
    public abstract int getNumber();
    public abstract void execute();
}

class RandomNumberGenerator extends NumberGenerator {
    private Random random = new Random();
    private int number;
    public int getNumber() {
        return number;
    }
    public void execute() {
        for(int i = 0; i &lt; 20; i++) {
            number = random.nextInt(50);
            notifyObservers();
        }
    }
}

class DigitObserver implements Observer {
    public void update(NumberGenerator generator) {
        System.out.println("DigitObserver:" + generator.getNumber());
        try {
            Thread.sleep(100);
        } catch(InterruptedException e) {
        }
    }
}

class GraphObserver implements Observer {
    public void update(NumberGenerator generator) {
        System.out.print("GraphObserver:");
        int count = generator.getNumber();
        for(int i = 0; i &lt; count; i++) {
            System.out.print("*");
        }
        System.out.println("");
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {   
        }
    }
}
```
