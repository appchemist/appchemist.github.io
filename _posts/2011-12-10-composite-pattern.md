---
layout: post
title: Composite Pattern
date: '2011-12-10T08:37:59+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/124891483606/composite-pattern
---
Composite Pattern은 하나의 객체와 같이 동일한 방법으로 다양한 객체 그룹들을 다룰 수 있다.

이러한 Composite Pattern의 목적은 객체들을 Tree 구조로 part-whole hierarchy를 표현하는 것이다.

이러한 목적을 달성하기 위해서 그룻(composite)와 내용물(leaf)을 동일시해서 재귀적인 구조를 구성을 한다.
사실 일반적인 Tree 구조를 다루다 보면, Programmer는 leaf node와 branch를 구분을 해야하만 하는 경우가 많다. 이것은 코드를 더욱 복잡하게 만들고, Error를 발생 시키기 쉽다.

이러한 문제의 해결법은 그릇과 내용물을 동일한 방식으로 다룰 수 있는 interface이다. 그릇과 내용물이 interface를 구현하므로써, 모두 동일한 Operation을 가지고 있다.
When to use
- 계층 구조(Part-Whole Hierarchy)를 사용해야 하는 경우
- Composition Object와 각 Object의 차이를 무시해야 하는 경우
- 각 Object를 동일한 code로 컨트롤해야 하는 경우
일반적인 구조
<a href="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/composite-pattern.png"><img class="aligncenter size-full wp-image-664" alt="composite pattern" src="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/composite-pattern.png?resize=720%2C465" data-recalc-dims="1"/></a>


```java
abstract class Entry {
    public abstract String getName();
    public abstract int getSize();
    public Entry add(Entry entry) throws FileTreatmentException {
        throw new FileTreatmentException();
    }
    public void printList() {
        printList("");
    }
    protected abstract void printList(String prefix);
    public String toString() {
        return getName() + " (" + getSize() + ")";
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
    protected void printList(String prefix) {
        System.out.println(prefix + "/" + this);
    }
}

class Directory extends Entry {
    private String name;
    private ArrayList directory = new ArrayList();
    public Directory(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
    public int getSize() {
        int size = 0;
        Iterator it = directory.iterator();
        while(it.hasNext()) {
            Entry entry = it.next();
            size += entry.getSize();
        }
        return size;
    }
    public Entry add(Entry entry) {
        directory.add(entry);
        return this;
    }
    protected void printList(String prefix) {
        System.out.println(prefix + "/" + this);
        Iterator it = directory.iterator();
        while(it.hasNext()) {
            Entry entry = it.next();
            entry.printList(prefix + "/" + name);
        }
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
