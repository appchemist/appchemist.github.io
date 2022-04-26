---
layout: post
title: Java I/O Stream
date: '2011-12-12T03:22:08+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118512518046/java-io-stream
---
# I/O를 처리하는 방법

## Byte Streams

InputStream과 OutputStream은 8-bit로 입력과 출력을 한다

| 생성 | 입력 및 출력 | 자원 반환 |
|-----|-----|-----|
| FileInputStream in = null;<br />FileOutputStream out = null;<br />in = new FileInputStream(“example1.txt”);<br />out = new FileOutputStream(“example2.txt”); | int c = in.read();<br />out.write(); | in.close();<br />out.close(); |

## Character Streams

JDK 1.1 이후에 16-bit가 필요한 Unicode가 소개 되었지만, 기존의 InputStream과 OutputStream 그리고 그 subclass들은 8-bit의 byte stream만 지원을 했다. 이에 JDK 1.1에서 java.io.Reader와 java.io.Writer 그리고 그 subclass들이 추가 되었고 Character Stream을 지원을 한다.

| 생성 | 입력 및 출력 | 자원 반환 |
|-----|-----|-----|
| FileReader in = null;<br />FileWriter out = null;<br />in = new FileReader(“example1.txt”);<br />out = new FileWriter(“example2.txt”); | int c = in.read();<br />out.write(); | in.close();<br />out.close(); |

## Stream Chaining

Stream Chaining은 I/O stream class의 장점이다.

Stream Chaining은 각 stream class에 연결을 하여 데이터를 얻는 방법이다.

각 class는 정해진 일을 수행하고, 다음 연결된 class에게 forward한다.
예를 들어서 입력 받은 Data를 압축하고, 암호화해서 파일에 저장한다고 하자.
<a href="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput11.png"><img class="alignnone size-medium wp-image-134" title="javainputoutput1" src="http://i1.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput11.png?resize=381%2C134" alt="" data-recalc-dims="1"/></a>

그림과 같이 작업은 수행이 된다.

그리고 Stream Chaining을 한 코드는 이렇다.

GZIPOutputStream gos = new GZIPOutputStream(new CryptOutputStream(new FileOutputStream(“example.txt”)));
위의 작업을 수행하는 코드는 이렇다.

gos.write(“저장할 내용”);

# Design Pattern

## Stream Chaining은 어떻게 가능할까?

Stream Chaining 기법은 Input, OuputStream과 Reader, Writer와 subclass 간의 class 관계에서 가능하게 된다.

이러한 관계를 Decorator Pattern이라고 한다.
- 일반적 Decorator Pattern의 구조
<a href="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput2.png"><img class="alignnone size-medium wp-image-135" title="javainputoutput2" src="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput2.png?resize=300%2C217" alt="" data-recalc-dims="1"/></a>
- InputStream에서 Decorator Pattern 구조

 Reader도 FilterInputStream과 동일하게 FilterReader와 BufferedReader가 Abstract Decorator이다.
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput3.png"><img class="alignnone size-medium wp-image-136" title="javainputoutput3" src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput3.png?resize=376%2C239" alt="" data-recalc-dims="1"/></a>
- Decorator Pattern의 Forward 과정
<a href="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput4.png"><img class="alignnone size-medium wp-image-137" title="javainputoutput4" src="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput4.png?resize=300%2C214" alt="" data-recalc-dims="1"/></a>

### 결과

InputStream에서 Buffer가 달린 InputStream과 각 라인에 0에서부터 카운트하는 LineNumberInputStream 등의 기능들을 InputStream이라는 객체에 각 기능을 장식(Decorator)을 달아서 자신이 원하는 기능의 InputStream을 사용할 수 있다.

각 Concrete Decorator Class들은 InputStream의 기능을 사용하여 추가할 기능만 추가 구현된 Class이다.

결과적으로 코드 재사용성을 높일 수 있다.
## 기존 Byte Stream과 Character Stream의 차이

A. 문제점

Java의 Standard Input과 output은 System.in과 System.out으로 미리 정의되어 있다.

하지만 System.in과 System.out은 InputStream과 OutputStream 객체이다. 그러면, Byte Stream인데 Character Stream과 다루는 bit의 양이 각 8-bit와 16-bit로 차이가 난다.

게다가, 암호화를 위해서 제공되는 CipherInputStream과 압축을 위해서 제공되는 ZipInputStream도 Byte Stream이다.

하지만, 기존의 이러한 Byte Stream을 두고 Character Stream을 위해서 Standard Input과 압축, 암호화를 위한 Class를 제공하지 않는다.

B. 해결책

이러한 차이점이 있지만, 기존의 Class를 버리지 않았다. 오히려 그 Byte Stream Class를 재사용하는 방법으로 Character Stream에도 다양한 기능을 제공한다.

그 해결책으로 Bridge Pattern으로 되어있다.

일반적 Bridge Pattern의 구조
<a href="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput5.png"><img class="alignnone size-medium wp-image-138" title="javainputoutput5" src="http://i2.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput5.png?resize=377%2C181" alt="" data-recalc-dims="1"/></a>

Writer의 Bridge의 구조
<a href="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput6.png"><img class="alignnone size-medium wp-image-139" title="javainputoutput6" src="http://i0.wp.com/appchemist.net/wp-content/uploads/2011/12/javainputoutput6.png?resize=352%2C184" alt="" data-recalc-dims="1"/></a>
C. 결과

기존의 Byte Stream인 InputStream을 StreamEncoder에 composition을 하고, 그 StreamEncoder를 OutputStreamWriter에 composition을 하였다. Writer와 그 subclass는 이 Bridge Class인 OutputStreamWriter를 인자로 받아서 입/출력 처리를 한다.

결과적으로 기존의 Byte Stream과 Character Stream의 차이를 극복하게 됐다.