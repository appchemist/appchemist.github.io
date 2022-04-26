---
layout: post
title: SICP 연습문제 1.9
date: '2012-04-28T12:51:28+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118201833996/sicp-ec97b0ec8ab5ebacb8eca09c-1-9
---
다음 두 프로시저가 반복하는 프로세스인지 되도는 프로세스인지 구별하고 각 프로시저의 프로세스 과정을 밝혀라.

프로시저 1


```lisp

(define (+ a b)

(if (= a 0)

b

(inc (+ (dec a) b))))

```


프로시저 2


```lisp

(define (+ a b)

(if (= a 0)

b

(+ (dec a) (inc b))))

```

우선, 프로시저 1의 프로세스 과정을 보면


```lisp

(+ 4 5)

(inc (+ 3 5))

(inc (inc (+ 2 5)))

(inc (inc (inc (+ 1 5))))

(inc (inc (inc (inc (+ 0 5)))))

(inc (inc (inc (inc (5)))))

(inc (inc (inc 6)))

(inc (inc 7))

(inc 8)

9

```

결과 적으로 프로시저 1은 되도는 프로세스 과정이다.

다음으로 프로시저 2의 프로세스 과정을 보면


```lisp

(+ 4 5)

(+ 3 6)

(+ 2 7)

(+ 1 8)

(+ 0 9)

9

```

결과 적으로 프로시저 2는 반복하는 프로세스 과정이다.