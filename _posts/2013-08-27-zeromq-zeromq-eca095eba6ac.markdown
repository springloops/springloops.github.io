---
author: springloops
comments: true
date: 2013-08-27 07:53:21+00:00
layout: post
link: https://springloops.wordpress.com/2013/08/27/zeromq-zeromq-%ec%a0%95%eb%a6%ac/
published: false
slug: zeromq-zeromq-%ec%a0%95%eb%a6%ac
title: '[ZeroMQ] ZeroMQ 개념 정리'
wordpress_id: 93
categories:
- AMQP
tags:
- 0mq
- amqp
- zeromq
- zeromq 정리
---

ZeroMQ

  


ZeroMQ 경량 메시지 커널은 메시징 미들웨어 제품에서 제공하는 기능과 기본적인 소켓 인터페이스를 확장한  라이브러리

- ZeroMQ 소켓은 비동기 메시지 큐의 추상화를 제공

- 멀티 메시지 패턴

- 메시지 필터링 (구독)

- 여러 전송 프로토콜 등에 대한 원활한 액세스 제공

  


Context

ZeroMQ 라이브러리 함수를 호출하기 전에 ZeroMQ context를 반드시 초기화해야한다.

  


Initialise : zmq_init();

Terminate : zmq_term();

  


Thread safety

  


ZeroMQ context 는 쓰래드에 안전하다.

호출하는 곳에서  lock 없이 필요한 만큼 어플리케이션에서 스래드간 공유할 수 있다.

  


  


하나의 어플리케이션에서 여러 context를 갖을 수 있다.
