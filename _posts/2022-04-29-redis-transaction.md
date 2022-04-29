---
layout: post
title: Redis - transaction
categories: [Study]
tags: [redis, transaction]
---

# Table of Contents

1.  [사용법](#orgc99462e)
2.  [Transaction에서의 Error](#org742f413)
3.  [왜 Redis는 Roll back을 지원하지 않나?](#org7138805)
4.  [Discarding the command queued](#org3f5f2b3)
5.  [Optimistic locking using check-and-set](#org17d112d)
6.  [WATCH explained](#org1b47c02)
7.  [Using WATCH to implement ZPOP](#orga2ff3c9)
8.  [Origin Document](#orgd95bc65)

MULTI, EXEC, DISCARD, WATCH 명령어가 Transation 관련 기반이되는 명령어들이다.
Transaction은 여러 명령으로 구성된 그룹의 실행을 하나의 단계와 같이 실행해준다.
그리고 다음 2가지를 보장한다.

-   Transaction 안에 모든 명령어들은 순서가 있으며, 순차적으로 수행된다.
    Redis Transaction의 수행 중에 다른 client로 부터 발생한 요청을 처리하지는 않는다.
    즉, 명령어들을 하나의 동릭된 Operation으로서 실행된다.
-   모두 실행되거나 하나도 수행되지 않는다. 그래서 Redis Transation은 원자성을 가진다.
    EXEC 명령이 Transation 안의 모든 명령을 수행을 시작하도록 한다.
    MULTI를 수행하기 전에 Transaction context 안에서 client가 server와의 연결이 끊긴다면 모든 Operations은 수행되지 않는다.
    하지만, EXEC 명령이 호출된다면, 모든 Operations은 수행된다.
    append-only 파일을 사용할 때, Redis는 Transaction을 disk로 쓰기위해서 단일 write syscall을 사용한다.
    하지만, 만약에 Redis 서버가 크래쉬가 되거나, 관리자에 의해서 종료된다면, Operations 중의 일부만이 실행될 수 있다.
    Redis는 이러한 상태(Operation이 일부만 실행된 상태?)를 재시작시에 발견할것이고 에러와 함께 종료할것이다.
    redis-check-aof tool을 사용해서 서버를 다시 재시작 가능한데, 이는 부분적 transation을 제거해서 append only 파일을 고치는 것이다.

버전 2.2에서 부터 Redis는 위의 두가지에 대해서 추가적인 보장을 제공한다. check-and-set(CAS) Operation과 매우 유사한 방법으로 Optimistic locking의 형태를 제공한다.
해당 내용은 뒤에서 다루도록 한다.


<a id="orgc99462e"></a>

# 사용법

Reids Transaction은 MULTI 명령어를 사용해서 진입한다. 해당 명령어는 항상 OK로 대답한다. 
이 때 사용자는 여러 명령어를 호출할 수 있다. 호출한 명령어들은 실행되는 대신에 해당 명령어들을 queue에 담아둔다.
모든 명령어들은 EXEC 명령어가 호출되면 수행된다.
EXEC 명령어 호출하는 대신 DISCARD 명령어를 호출하면, 해당 queue를 비우고 해당 Transaction은 종료된다.
다음 예제는 foo와 bar 키를 원자적으로 증가 시킨다.

    > MULTI
    OK
    > INCR foo
    QUEUED
    > INCR bar
    QUEUED
    > EXEC
    1) (INTEGER) 1
    2) (INTEGER) 1

위의 예제에서 보시다시피, EXEC는 결과들을 돌려준다. 각 결과들은 transaction안의 단일 명령의 결과 값이다. 결과들은 명령이 호출된 순서와 동일하다
Redis connection이 MULTI 명령어 요청의 context 안에 있다면, 모든 명령어들은 QUEUED 문자열로 응답한다.
Queue에 들어간 명령어들은 EXEC 명령어가 호출됐을 때, 실행되기 위해서 예약된 상태이다.


<a id="org742f413"></a>

# Transaction에서의 Error

Transaction 동안에 2가지 종류의 Command Error가 발생할 수 있다.

-   Command가 Queue에 넣는것이 실패할수 있으며, 그럴 경우에 EXEC가 호출되기 전에 Error가 발생한다.
    예를들어서 Command가 문법적으로 틀렸거나 메모리 부족과 같은 심각한 상태 때문일수도 있다.
-   Command가 EXEC를 호출하고 실해할수 있다. 예를들어서 Key에 대해서 잘못된 Value와 함께 실행시킨 경우이다.

EXEC 호출 전에 일어나는 첫번째 종류의 Error는 Queue넣는 Command의 결과값을 확인하므로써 확인할수 있을 것이다.
만약에 해당 Command가 QUEUED로 응답한다면, 이것은 정확히 Queue에 들어간것이다. 반대의 경우네는 Error를 반환한다.
Command를 Queue에 넣는 동안에 Error가 발생한다면, 대부분의 클라이언트들은 에러가 발생한 Transaction을 종료할것이다. 
하지만 Redis 2.6.5에서부터 서버는 Commands를 쌓는 동안에 발생한 Error를 기억한다. 그리고 EXEC가 수행되는 동안에 해당 Transaction이 실행되는 것을 막고, 해당 Transaction을 자동적으로 폐기 처리한다.
Redis 2.6.5 이전에는 바로 전에 발생한 Error에도 불구하고 EXEC를 호출하면 성공적으로 Queue에 쌓여있는 Command들의 부분집합만 가지고 정상적으로 Transaction을 실행한다.
이러한 새로운 행동은 Pipeline으로 Transaction을 섞어 사용하는것을 보다 간단하게 만들어준다. 그래서 이후에 전체 결과를 확인할 필요없이 전체 Transaction을 한번에 보낼수 있게 됐다.
대신에 EXEC 이후에 발생한 Error들은 특별한 방법으로 다뤄지지 않는다: 어떤 Command가 Transaction 동안 싫패하더라도 그외 다른 Command들은 모두 실행된다.
이것이 Protocol 단계에서 보다 명확하다. 다음 예제는 문법이 맞는데도 불구하고 하나의 명령이 실행중에 실패한다.

    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    MULTI
    +OK
    SET a abc
    +QUEUED
    LPOP a
    +QUEUED
    EXEC
    *2
    +OK
    -ERR Operation against a key holding the wrong kind of value

EXEC는 2가지 요소의 Bulk string reply를 반환했다. 하나는 OK 다른 하나는 -ERR 응답. 
이렇게 에러를 센스있게 사용자에게 보여주는것은 전적으로 client library의 몫이다.
Command가 실패하더라도 Queue에 있는 모든 다른 명령들이 수행된다는 것을 알아두는것이 매우 중요하다.
Redis는 Command들을 처리하는것을 멈추지 않는다.
또 다른 예제는 문법 에러가 어떻게 바로 보여지는지 보여준다.

    MULTI
    +OK
    INCR a b c
    -ERR wrong number of arguments for 'incr' command

이번에는 문법문제로 인해서 INCR Command는 Queue에 들어가지 않는다.


<a id="org7138805"></a>

# 왜 Redis는 Roll back을 지원하지 않나?

Redis는 Transaction 중에 명령이 실패할수 있지만, Redis는 Rollback 대신에 나머지 Transaction을 실행한다는 사실이 RDB에 대한 지식이 있는 사람의 입장에서는 매우 이상하게 보일수가 있다.
하지만 이러한 행동에 대한 2가지 매우 좋은 의견이 있다.

-   Redis Commands는 잘못된 문법으로 호출됐을 경우나 Key에 대해서 잘못된 Data Type을 가진 경우에만 실패할수 있다(그리고 그 문제는 Command가 Queue에 들어가는 동안에는 발견한수 없다) : 이것은 실제로 Command가 실패하는 것은 프로그래밍 Error의 결과 이고 이런 종류의 에러는 개발 단계에서 발견하기 매우 쉬워진다.
-   Redis는 내부적으로 간단하고 빠르다. 이는 Redis가 Rollback을 하지 않아도 되서 가능하다.

Redis에 대한 논쟁들은 위의 Bugs(위의 2가지는 Redis의 스펙 같으나, 해당 부분을 이해하지 못 하고 발생했을때 Bug라고 지칭한듯)들이 발생했을 경우들인다.
하지만, 일반적인 경우에서는 Roll back이 Programming Error로부터 당신을 구해줄수 없다는 것을 알아야 한다.
예를들어서 만약 Query가 Key를 1이 아닌 2로 증가시킨다거나 잘못된 Key를 증가시킨다면 여기에서는 Rollback이 할수 있는 일이 없다.
프로그래머의 실수를 해결해줄수 없고, Redis Command가 실패해야하는 종류의 Error들이 제품으로 나가길 바라지 않는다는 것을 고려한다면, 우리는 Error에서 Rollback을 지원하지 않는 대신에 간단하고 빠른 접근을 선택했다.


<a id="org3f5f2b3"></a>

# Discarding the command queued

DISCARD는 Transaction을 종료하기위해서 사용된다. 이런 경우에 어떤 명령도 실행되지 않고 연결 상태도 정상으로 복구된다.

    > SET foo 1
    OK
    > MULTI
    OK
    > INCR foo
    QUEUED
    > DISCARD
    OK
    > GET foo
    "1"


<a id="org17d112d"></a>

# Optimistic locking using check-and-set

WATCH는 Redis Transaction에서 check-and-set(CAS)를 제공한다.
WATCHed keys는 해당 값의 변경사항을 발견하기 위해서 모니터링된다.
만약에 적어도 하나의 Key가 EXEC Command가 수행되기전에 변경이된다면. 전체 Transaction은 종료된다. 그리고 EXEC는 NULL 결과를 반환해서 Transaction이 실패했음을 알려준다.
예를들어서 어떤 키의 값을 자동적으로 1씩 증가시켜야한다고 상상해보자(Redis가 INCR이 없다고 가정한다면)
우리는 다음과 같이 할 것이다.

    val = GET mykey
    val = val + 1
    SET mykey $val

이것은 우리가 하나의 client가 해당 시간에 작업을 수행한다면 믿을수 있는 작업을 수행할 것이다.
만약에 여러 client가 key의 값을 수정하기 위해서 같은 시간에 작업을 한다면, race condition 상태가 될것이다.
예를들어서 Client A와 B는 옛날 값을 읽을것이다. 예를들어서 10. 두 Client에 의해서 값은 11로 증가될것이다.
그리고 최종적으로 Key의 값으로 SET을 수행할것이다. 그러면, 최종값은 12가 아닌 11이 된다.
우리는 이런 문제를 잘 해결해준 WATCH에 감사해야 한다.

    WATCH mykey
    val = GET mykey
    val = val + 1
    MULTI
    SET mykey $val
    EXEC

위의 코드를 사용하면 Race Condition이 발생하고, WATCH와 EXEC 호출 사이에 다른 Client가 val의 결과를 수정하다면, Transaction은 실패할것이다.
우리는 이번에는 다른 Race Condition이 발생하지 않기를 바라면서 이 작업을 다시 수행하면된다.
이러한 형태의 locking은 optimistic locking이라고 부른다. 이러한 형태는 매우 강력한 형태의 locking이다.
많은 UseCases에서 여러 Client는 서로 다른 Key에 접근할 것이다. 그래서 충돌은 일반적이지 않아서 해당 Operation을 반복할 필요는 없을것이다.


<a id="org1b47c02"></a>

# WATCH explained

그래서 WATCH는 뭘까? 이것은 EXEC를 조건적으로 수행하게해주는 Command이다: 우리는 Redis에게 WATCHed Key가 수정된것이 없다면 Transaction을 수행하라고 Redis에게 요청하는것이다.
(하지만, 같은 client의해서 Transaction 안에서 수정되는 경우에는 종료하지 않는다 [more on this](https://github.com/antirez/redis-doc/issues/734)) 다른 경우에는 Transaction은 전혀 진입되지 않는다.
(Volatile key를 WATCH를 걸고 Redis가 해당 키를 만료시킨다면, EXEC는 여전히 작동한다. 해당 내용은 [more on this](https://code.google.com/archive/p/redis/issues/270))
WATCH는 여러번 호출할 수 있다. 간단하게 모든 WATCH 호출은 호출 시점부터 EXEC가 호출되는 시점까지 모든 변화에 대한 감시한다.
또한 단일 WATCH 호출에 여러개의 Key들을 보낼수 있다. 
EXEC가 호출됐을때, 모든 Keys가 UNWATCHED 상태라면, Transaction이 종료되던 말던 상관하지 않는다.
또한 Client의 연결이 끊겼을때, 모든것은 UNWATCHed 상태가 된다.
또한 UNWATCH Command를(Arguments 없이) 모든 watched keys를 정리하기위해서 사용할수도 있다.
때때로는 몇몇 Keys를 Optimistically lock을 할때 매우 유용하다. 이러한 keys을 변경하기 위해서 Transaction을 수행할 필요가 있다. 하지만 현재 keys의 내용을 읽고나서 처리를 하고 싶지 않다
이러한 상황일때, 우리는 UNWATCH만 호출하면 해당 Connection은 새로운 Transation을 위해서 자유로워진다.


<a id="orga2ff3c9"></a>

# Using WATCH to implement ZPOP

WATCH를 설명하기 위해서 Redis에서 제공하는 Atomic Operation 중에서 WATCH를 사용해서 만들수 있는 좋은 예제는 ZPOP이다. 
해당 Command는 Atomic한 방법으로 정렬된 set에서 lower score의 요소를 배출하는 명령이다. 
아래가 가장 간단한 구현이다.

    WATCH zset
    element = ZRANGE zset 0 0
    MULTI
    ZREM zset element
    EXEC

만약에 EXEC가 실패한다면, Operation을 반복하면 된다.


<a id="orgd95bc65"></a>

# Origin Document

<https://redis.io/docs/manual/transactions/>

