---
layout: post
title: Builder Pattern
date: '2011-12-10T08:30:44+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512465321/builder-pattern
---
Builder Pattern은 객체 생성을 위한 Design Pattern이다.

이 Pattern의 목적은 다른 구현을 통해서 다른 형태의 Object를 조립 과정을 추상화 시키는 것이다.
종종 Design은 Factory Method(덜 복잡하고, 더욱 보편적이다.)에서 시작을 해서, Abstract Factory, Prototype, Builder와 같은 Pattern으로 변화시킬수 있다.
- 복잡한 객체를 생성하는 과정에 사용될 수 있다.
- 종종 Composite Pattern을 조립하는데 사용
- Method Chaining을 이용하여 객체를 구성하는데 유용하다.
Method Chaining을 이용한 객체 생성 예)



```java
em.createNamedQuery("Student.findByNameAgeGender")
             .setParameter("name", name)
             .setParameter("age", age)
             .setParameter("gender", gender)
             .setFirstResult(1)
             .setMaxResults(30)
             .setHint("hintName", "hintValue")
             .getResultList();
```


<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/builder-pattern.png"><img src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/builder-pattern.png?resize=533%2C240" alt="builder pattern" class="aligncenter size-full wp-image-681" data-recalc-dims="1"/></a>

예제 코드(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;


public class BuilderPattern {
    public static void main(String[] args) {
        TextBuilder textBuilder = new TextBuilder();
        Director director = new Director(textBuilder);
        director.construct();
        String result = textBuilder.getResult();
        System.out.println(result);
       
        HTMLBuilder htmlBuilder = new HTMLBuilder();
        director = new Director(htmlBuilder);
        director.construct();
        String filename = htmlBuilder.getResult();
    }
}

abstract class Builder {
    public abstract void makeTitle(String title);
    public abstract void makeString(String str);
    public abstract void makeItems(String[] items);
    public abstract void close();
}

class Director {
    private Builder builder;
   
    public Director(Builder builder) {
        this.builder = builder;
    }
   
    public void construct() {
        builder.makeTitle("Greeting");
        builder.makeString("아침과 낮에");
        builder.makeItems(new String[] {
                "좋은 아침입니다.",
                "안녕하세요.",
        });
       
        builder.makeString("밤에");
        builder.makeItems(new String[] {
                "안녕하세요.",
                "안녕히 주무세요.",
                "안녕히 계세요."
        });
        builder.close();
    }
}

class TextBuilder extends Builder {
    private StringBuffer buffer = new StringBuffer();
   
    @Override
    public void makeTitle(String title) {
        buffer.append("======================n");
        buffer.append("(" +title + ")n");
        buffer.append("n");
    }
   
    @Override
    public void makeString(String str) {
        buffer.append('*' +str + "n");
        buffer.append("n");
    }
   
    @Override
    public void makeItems(String[] items) {
        for(int i = 0; i &lt; items.length; i++) {
            buffer.append(" -" + items[i] + "n");
        }
        buffer.append("n");
    }
   
    @Override
    public void close() {
        buffer.append("=======================n");
    }
   
    public String getResult() {
        return buffer.toString();
    }
}

class HTMLBuilder extends Builder {
    private String filename;
    private PrintWriter writer;
   
    @Override
    public void makeTitle(String title) {
        filename = title + ".html";
        try {
            writer = new PrintWriter(new FileWriter(filename));
        }catch(IOException e) {
            e.printStackTrace();
        }
        writer.println("&lt;html&gt;&lt;head&gt;&lt;title&gt;" + title + "&lt;/title&gt;&lt;/head&gt;&lt;body&gt;");
        writer.println("&lt;h1&gt;" + title + "&lt;/h1&gt;");
    }
    @Override
    public void makeString(String str) {
        writer.println("&lt;p&gt;" + str + "&lt;/p&gt;");
    }
    @Override
    public void makeItems(String[] items) {
        writer.println("&lt;ul&gt;");
        for(int i = 0; i &lt; items.length; i++) {
            writer.println("&lt;li&gt;" + items[i] + "&lt;/li&gt;");
        }
        writer.println("&lt;/ul&gt;");
    }
    @Override
    public void close() {
        writer.println("&lt;/body&gt;&lt;/html&gt;");
        writer.close();
    }
   
    public String getResult() {
        return filename;
    }
}
```
