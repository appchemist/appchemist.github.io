---
layout: post
title: SICP 연습문제 1.10
date: '2012-04-28T12:52:34+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118201724526/sicp-ec97b0ec8ab5ebacb8eca09c-1-10
---
다음은 애커만 함수를 나타낸 프로시저이다.


```lisp

(define (A x y)
(cond ((= y 0) 0)
((= x 0) (* 2 y))
((= y 1) 2)
(else (A (- x 1)
(A x (- y 1))))))

&amp;nbsp;

(define (f n) ( A 0 n))

(define (g n) (A 1 n))

(define (h n) (A 2 n))

```

다음 f, g, h 프로시저의 기능을 수학으로 정의하라.

f 프로시저는 2 * n
g 프로시저는 2 ^ n
h 프로시저는 2 ^ 2 ^ n
