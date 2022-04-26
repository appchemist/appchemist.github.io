---
layout: post
title: SICP 연습문제 1.6
date: '2011-12-10T08:51:45+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118201986351/sicp-ec97b0ec8ab5ebacb8eca09c-1-6
---
제곱 근을 구하기 위해서 뉴튼 법을 사용한다. 여기에 그 예제 소스가 있다.


```lisp

(define (sqrt-iter guess x)
(if (good-enough? guess x)
guess
(sqrt-iter (improve guess x)
x)))

(define (improve guess x)
(average guess (/ x guess)))

(define (average x y)
(/ (+ x y) 2))

(define (good-enough? guess x)
(&lt; (abs (- (square guess) x)) 0.001))

(define (sqrt2 x)
(sqrt-iter 1.0 x))

(define (square x)
(* x x))

```

여기에서 if문을 cond로 만든 프로시저로 대체 하면 어떤 결과가 나오는가 하는게 문제이다.


```lisp

(define (new-if predicate then-clause else-clause)
(cond (predicate then-clause)
(else else-clause)))

```

제시한 프로시저는 다음과 같다. 그리고 if를 new-if로 대체 후, 실행한다면 프로시저는 무한 루프에 빠진다. applicative order evaluation이기 때문에 프로시저는 각 인자의 값을 먼저 계산하러 들어 갈 것이다.
여기에서 new-if를 사용하는 sqrt-iter는 재귀 호출로 자신의 sqrt-iter을 계속 해서 호출하게 된다. 즉, if문으로 good-enough?의 결과에 따라 sqrt-iter이 계산이 안 되던 것이 sqrt-iter을 계속해서 계산하여 들어가게 되어 무한 루프에 빠지게 된다.