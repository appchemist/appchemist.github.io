---
layout: post
title: SICP 연습문제 1.11
date: '2012-04-28T12:53:15+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118201647951/sicp-ec97b0ec8ab5ebacb8eca09c-1-11
---
n < 3 일 때 f(n) = n이고, n >= 3 일 때 f(n) = f(n-1) + 2f(n-2) + 3f(n-3)으로 정의한 함수가 있다.
f의 프로시저를 되도는 프로세스를 만들고 반복 프로세스를 만들어 내는 프로시저도 만들어라.

Recursive Process


```lisp

(define (f n)
(if (&lt; n 3)
n
(+ (f (- n 1)) (* 2 (f(- n 2))) (* 3 (f(- n 3))))))

```

Iterative Process


```lisp

(define (f2 n)
(define (f-iter first second third counter)
(if (= counter 3)
(+ third (* 2 second) (* 3 first))
(f-iter second third (+ third (* 2 second) (* 3 first)) (- counter 1))))
(if (&lt; n 3)
n
(f-iter 0 1 2 n)))

```
