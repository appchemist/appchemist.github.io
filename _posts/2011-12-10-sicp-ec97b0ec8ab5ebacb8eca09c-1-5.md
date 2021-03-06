---
layout: post
title: SICP 연습문제 1.5
date: '2011-12-10T08:51:08+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118202065216/sicp-ec97b0ec8ab5ebacb8eca09c-1-5
---
applicative order evaluation과 normal-order evaluation의 차이를 묻는 문제이다.


```lisp

(define (p) (p))

(define (test x y)
(if (= x 0)
0
y))

```

위와 같이 2개의 프로시저를 정의한다.

(test 0 (p))

그리고 다음의 식 실행 시, applicative order evaluation과 normal-order evaluation의 각 결과는 무엇인지를 묻는 문제.

applicative order evaluation인 lisp의 경우에는 무한 루프에 빠지게 된다.
applicative order evaluation은 인자를 먼저 계산을 하는 계산법이기 때문이다.
즉, (test 0 (p))에서 test 프로시저 보다는 p 프로시저를 먼저 계산을 한다. 하지만 p 프로시저는 자기 자신을 종료 조건 없이 호출을 하므로, 무한 루프에 빠지게 된다.

nomal-order evaluation은 적합한 언어를 선택을 못 해서 돌려보지는 못 했지만, 보나 마나 결과는 0이 나온다.
이 계산법은 바깥 프로시저에서 안으로 모든 프로시저를 풀어 놓고 계산을 하는 계산법이다.
(test 0 (p))는 (if (= 0 0) 0 (p))와 동일 하다. if 문은 참이 되므로 0을 return 하게 된다. 즉, p를 계산하지 않는다.