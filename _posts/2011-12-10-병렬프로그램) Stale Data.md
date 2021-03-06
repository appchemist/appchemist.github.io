---
layout: post
title: 병렬프로그램) Stale Data
date: '2011-12-10T08:47:51+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118200817646/ebb391eba0aced9484eba19ceab7b8eb9ea8-s
---
단일 스레드를 사용하는 환경에서 특정 변수를 저장하고,그 값을 읽으면 이전에 저장 했던 값을 가지고올수 있다. 하지만, 여러 스레드가 작동하는 환경에서는 변수에 값을 저장하고 읽어 오는 코드가 여러 스레드에 의해서 실행이 된다면 상황은 다르다.

```java
public class NoVisibility {
    private static boolean ready;
    private static int number;
    
    private static class ReaderThread extends Thread {
        public void run() {
            while(!ready)
                Thread.yield();
            System.out.println(number);
        }
    }
    
    public static void main(String[] args) {
        new ReaderThread().start();
        number = 43;
        ready = true;
    }
}
```

위의 소스코드를 본다면 43이 출력할 것이라고 생각할 수 있다.
하지만, ready 변수의 값을 ReaderThread에서 영원히 false로 확인을 해서 무한 루프에 돌 수도 있다.
아니면, ready의 값을 먼저 읽어들여 number의 값이 지정이 안 되어있을 수도 있다.
이러한 현상을 재배치라고 하며, 동기화를 하지 않는다면, 컴파일러나 프로세서, JVM이 실행되는 순서를 임의로 바꿔 실행하는 이상한 경우가 발생하기도 한다.