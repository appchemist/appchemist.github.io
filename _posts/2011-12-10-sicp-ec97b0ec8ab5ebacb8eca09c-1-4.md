---
layout: post
title: SICP 연습문제 1.4
date: '2011-12-10T08:50:30+09:00'
tags: []
tumblr_url: https://appchemist.tumblr.com/post/118202123456/sicp-ec97b0ec8ab5ebacb8eca09c-1-4
---
combination에  연산자 자리에 compound expression이 다시 와도 규칙에 따라 식을 구할 수 있다.
다음의 식은 연습문제에 나온 식이다.


```lisp

(define (a-plus-abs-b a b)
((if (&gt; b 0) + -) a b))

```

위의 식을 살펴 보면 ((if (> b 0) + -) a b)) 부분을 볼 수 있다. (if (> b 0) + -)를 연산자 자리에 위치하며, 이 식의 결과로 연산자가 +나 -를 정해서 a와 b의 값을 계산을 하게 된다.

그 결과로 a + |b|의 결과를 얻을 수 있다.

식의 연산자 자리의 복합 식을 사용할 수 있다는 것을 보여주는 흥미로운 예제이다.