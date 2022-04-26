---
layout: post
title: SICP 연습문제 1.8
date: '2011-12-10T08:52:54+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118201880806/sicp-ec97b0ec8ab5ebacb8eca09c-1-8
---
cube root 를 위한 Newton’s method는 아래의 수식을 기반으로 한다. y의 값은 x의 cube root의 근사 값이고 다음 수식을 수행할때 마다 더욱 근사치에 가까운 근사 값으로 다가간다.

위의 수식을 이용해서 cube-root를 구현해라, 단, square-root procedure와 유사하게 cube-root를 구현해라.

결과


```lisp

(define (cube-root x)
(cube-root-iter 1.0 x))

(define (cube-root-iter guess x)
(if (good-enough? guess x)
guess
(cube-root-iter (improve guess x)
x)))

(define (improve guess x)
(one-third (/ x (square guess)) (* 2 guess)))

(define (square x)
(* x x))

(define (cube x)
(* x x x))

(define (one-third x y)
(/ (+ x y) 3))

(define (good-enough? guess x)
(&lt; (abs (- (improve guess x) guess)) (* guess 0.000001)))

```
