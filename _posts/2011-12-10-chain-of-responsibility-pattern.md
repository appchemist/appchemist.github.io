---
layout: post
title: Chain Of Responsibility Pattern
date: '2011-12-10T08:41:44+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512488896/chain-of-responsibility-pattern
---
Chain Of Responsibility Pattern은 복수의 객체를 연결하여, 연결한 객체들을 차례로 돌며 처리하는 방법이다.

여기에서 연결된 객체들은 넘어오는 요구 객체를 처리할 수 있을 때 까지 다음에 연결된 객체에게 일을 넘깁니다.  즉, 요구 객체를 적절하게 처리할 수 있는 처리 객체가 적절하게 처리를 할 수 있습니다.

몇몇 경우에, 처리 객체가 상위의 처리 객체와 명령을 호출하여 작은 파트의 명령을 해결하기 위해 재귀적으로 실행되기도 한다. 이 경우에 재귀는 명령이 처리되거나 모든 트리가 탐색될때까지 진행이 됩니다. 예를 들어 XML 인터프리터를 들 수가 잇다.
즉, 요청하는 쪽과 처리하는 쪽의 연결을 우연하게 해서 각 객체를 부품으로 독립시킬 수 있습니다. 그리고 상황에 따라 처리할 객체가 변하는 프로그램도 작성가능합니다.
일반적인 구조
<a href="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/chainOfResponsibility.png"><img src="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/chainOfResponsibility.png?resize=458%2C273" alt="chainOfResponsibility" class="aligncenter size-full wp-image-645" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
class Trouble {
    private int number;
    public Trouble(int number) {
        this.number = number;
    }
   
    public int getNumber() {
        return number;
    }
   
    public String toString() {
        return "[Trouble " + number + "]";
    }
}

abstract class Support {
    private String name;
    private Support next;
   
    public Support(String name) {
        this.name = name;
    }
   
    public Support setNext(Support next) {
        this.next = next;
        return next;
    }
   
    public final void support(Trouble trouble) {
        if(resolve(trouble)) {
            done(trouble);
        } else if (next != null) {
            next.support(trouble);
        } else {
            fail(trouble);
        }
    }
   
    protected abstract boolean resolve(Trouble trouble);
    protected void done(Trouble trouble) {
        System.out.println(trouble + " is resolved by " + this + ".");
    }
    protected void fail(Trouble trouble) {
        System.out.println(trouble + " cannot be resolved");
    }
}

class NoSupport extends Support {
    public NoSupport(String name) {
        super(name);
    }
   
    protected boolean resolve(Trouble trouble) {
        return false;
    }
}

class LimitSupport extends Support {
    private int limit;
    public LimitSupport(String name, int limit) {
        super(name);
        this.limit = limit;
    }
   
    protected boolean resolve(Trouble trouble) {
        if(trouble.getNumber() &lt;  limit) {
            return true;
        } else {
            return false;
        }
    }
}

class OddSupport extends Support {
    public OddSupport(String name) {
        super(name);
    }
    protected boolean resolve(Trouble trouble) {
        if(trouble.getNumber() % 2 == 1) {
            return true;
        } else {
            return false;
        }
    }
}

class SpecialSupport extends Support {
    private int number;
    public SpecialSupport(String name, int number) {
        super(name);
        this.number = number;
    }
    protected boolean resolve(Trouble trouble) {
        if(trouble.getNumber() == number) {
            return true;
        } else {
            return false;
        }
    }
}
```
