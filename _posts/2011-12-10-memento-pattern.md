---
layout: post
title: Memento Pattern
date: '2011-12-10T08:44:03+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512507006/memento-pattern
---
Memento Pattern 은 객체의 이전 상태를 저장하기 위한 패턴이다.

객체의 상태를 저장하거나 복원하기 위해서는 객체 내부의 정보를 자유롭게 액세스할 수 있어야 한다.

특히 복원하기 위해서는 Visibility가 객체 밖으로 공개되지 않은 Attribute는 쓸수가 없으므로 복원할 수가 없다. 그렇다고 Visibility를 넓힌다면 캡슐화를 약하게 만드는 요인이 된다.
Memento Pattern은 객체의 캡슐화를 강화시키면서 상태 값을 저장하고 복원할 수 있도록 해준다.
Memento Pattern은 세 가지 역활이 등장한다.

Originator : State를 가지고 있는 객체로 자신의 상태를 저장하기 위해서 Memento를 사용한다.

Memento : Originator의 상태를 저장하지만, 내부 정보를 외부에 공개하지 않는다. Originator의 내부 클래스로 구현할 수도 있다.

CareTaker : Originator의 저장된 상태인 Memento를 저장하여 관리해 준다. 하지만 Memento의 내부 정보를 액세스할 수 없다.
** 아래의 예제 소스에서는 Main이 Originator의 역활이고 Memento와 Originator는 다른 페키지에 존재한다. 편집 상 패키지가 표시되어 있지 않다.

일반적인 구조
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/memento.png"><img src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/memento.png?resize=720%2C250" alt="memento" class="aligncenter size-full wp-image-637" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
public class MementoPattern {
    public static void main(String[] args) {
        Gamer gamer = new Gamer(100);
        Memento memento = gamer.createMemento();
        for(int i = 0; i &lt; 100; i++) {
            System.out.println("==== " + i);
            System.out.println("현상:" + gamer);
           
            gamer.bet();
           
            System.out.println("소지금은" + gamer.getMoney() + "원이 되었습니다.");
           
            if (gamer.getMoney() &gt; memento.getMoney()) {
                System.out.println(" (많이 증가했으므로 현재의 상태를 저장하자)");
                memento = gamer.createMemento();
            } else if (gamer.getMoney() &lt; memento.getMoney() / 2) {
                System.out.println(" (많이 감소했으므로 이전의 상태로 복원하자)");
                gamer.restoreMemento(memento);
            }
           
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }
            System.out.println("");
        }
    }
}

public class Memento {
    int money;
    ArrayList&lt;String&gt; fruits;
   
    public int getMoney() {
        return money;
    }
   
    Memento(int money) {
        this.money = money;
        this.fruits = new ArrayList&lt;String&gt;();
    }
   
    void addFruit(String fruit) {
        fruits.add(fruit);
    }
   
    List getFruits() {
        return (List)fruits.clone();
    }
}

public class Gamer {
    private int money;
    private ArrayList&lt;String&gt; fruits = new ArrayList&lt;String&gt;();
    private Random random = new Random();
    private static String[] fruitsname = {
        "사과", "포도", "바나나", "귤",
    };
    public Gamer(int money) {
        this.money = money;
    }
    public int getMoney() {
        return money;
    }
    public void bet() {
        int dice = random.nextInt(6) + 1;
        if(dice == 1) {
            money += 100;
            System.out.println("소지금이 증가했습니다.");
        } else if (dice == 2) {
            money /= 2;
            System.out.println("소지금이 절반이 되었습니다.");
        } else if (dice == 6) {
            String f = getFruit();
            System.out.println("과일(" + f + ")을 받았습니다.");
            fruits.add(f);
        } else {
            System.out.println("변한 것이 없습니다.");
        }
    }
   
    public Memento createMemento() {
        Memento m = new Memento(money);
        Iterator&lt;String&gt; it = fruits.iterator();
        while(it.hasNext()) {
            String f = (String)it.next();
            if (f.startsWith("맛있는 ")) {
                m.addFruit(f);
            }
        }
        return m;
    }
   
    public void restoreMemento(Memento memento) {
        this.money = memento.money;
        this.fruits = (ArrayList)memento.getFruits();
    }
   
    public String toString() {
        return "[money = " + money + ", fruits = " + fruits + "]";
    }
   
    private String getFruit() {
        String prefix = "";
        if (random.nextBoolean()) {
            prefix = "맛있는 ";
        }
        return prefix + fruitsname[random.nextInt(fruitsname.length)];
    }
}
```
