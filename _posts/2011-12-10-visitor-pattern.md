---
layout: post
title: Visitor Pattern
date: '2011-12-10T08:41:09+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512482126/visitor-pattern
---
Visitor Pattern은 알고리즘이 처리하는 객체 집합에서 알고리즘은 분리한 Pattern이다.
알고리즘을 분리한 결과 새로운 Operation을 추가하기 위해서 객체 집합을 수정할 필요가 없어졌다.

즉, Open-Closed Principle을 만족할 수 있습니다. Open-Closed Principle은 처리 연산에 대해서는 확장할 수 있지만, 기존 클래스를 수정하지 않아도 된다.

장점이 있으면 단점이 있듯이 Visitor Pattern을 보면 처리의 흐름을 이해하기가 힘들다. 
그리고 Visitor Pattern에서 Visitor 역활의 클래스와 Element 역활의 클래스를 살펴보자.

Element 클래스는 Accept 함수를 가지고 Client에게서 작업 객체(Visitor)를 전달받는다.

여기에서 Accept 함수는 전달받은 객체를 다시 작업 수행을 요청한다.

여기에서 Visitor 객체와 Element 객체의 자료형에 따라서 수행되는 작업이 결정이 된다.

이것을 Double-Dispatch라 한다.
일반적인 구조
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/visitor.png"><img src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/visitor.png?resize=515%2C458" alt="visitor" class="aligncenter size-full wp-image-649" data-recalc-dims="1"/></a>

예제 소스(출처 : Java 언어로 배우는 디자인 패턴 입문. 영진닷컴)


```java
abstract class Visitor {
    public abstract void visit(File file);

    public abstract void visit(Directory directory);
}

interface Element {
    public abstract void accept(Visitor v);
}

abstract class Entry implements Element {
    public abstract String getName();

    public abstract int getSize();

    public Entry add(Entry entry) throws FileTreatmentException {
        throw new FileTreatmentException();
    }

    public String toString() {
        return getName() + " (" + getSize() + ")";
    }

    public Iterator&lt;Entry&gt; iterator() throws FileTreatmentException {
        throw new FileTreatmentException();
    }
}

class File extends Entry {
    private String name;
    private int size;

    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }

    public String getName() {
        return name;
    }

    public int getSize() {
        return size;
    }

    public void accept(Visitor v) {
        v.visit(this);
    }
}

class Directory extends Entry {
    private String name;
    private ArrayList&lt;Entry&gt; directory = new ArrayList&lt;Entry&gt;();

    public Directory(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public int getSize() {
        int size = 0;
        Iterator&lt;Entry&gt; it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = it.next();
            size += entry.getSize();
        }
        return size;
    }

    public Entry add(Entry entry) {
        directory.add(entry);
        return this;
    }

    public Iterator&lt;Entry&gt; iterator() {
        return directory.iterator();
    }

    public void accept(Visitor v) {
        v.visit(this);
    }
}

class ListVisitor extends Visitor {
    private String currentdir = "";

    public void visit(File file) {
        System.out.println(currentdir + "/" + file);
    }

    public void visit(Directory directory) {
        System.out.println(currentdir + "/" + directory);
        String savedir = currentdir;
        currentdir = currentdir + "/" + directory.getName();
        Iterator&lt;Entry&gt; it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = it.next();
            entry.accept(this);
        }
        currentdir = savedir;
    }
}

class FileTreatmentException extends RuntimeException {
    public FileTreatmentException() {

    }

    public FileTreatmentException(String msg) {
        super(msg);
    }
}
```
