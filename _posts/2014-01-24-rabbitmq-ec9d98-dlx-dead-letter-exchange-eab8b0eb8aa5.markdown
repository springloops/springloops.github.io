---
author: springloops
comments: true
date: 2014-01-24 08:35:24+00:00
layout: post
link: https://springloops.wordpress.com/2014/01/24/rabbitmq-%ec%9d%98-dlx-dead-letter-exchange-%ea%b8%b0%eb%8a%a5/
slug: rabbitmq-%ec%9d%98-dlx-dead-letter-exchange-%ea%b8%b0%eb%8a%a5
title: RabbitMQ 의 DLX (Dead Letter eXchange) 기능
wordpress_id: 98
categories:
- Open Source/Rabbitmq
tags:
- dead letter exchange
- dlx
- rabbitmq
---

RabbitMQ 에 DLX
(Dead Letter eXchange) 기능이 있네요.

Storm Spout를 만들다가
rejected 메시지를 어떻게 처리 할까 하다가 찾은 방법입니다.

exchange 는 일반 적으로 동일하고, queue 를 생성 할떄 argument를 아래와 같이 주시면 됩니다.

x-dead-letter-exchange =  exchange 명

생성한 queue를 exchange에
바인딩 하면 reject 된 메시지의 경우 header 값에

x-death
키로
rejected 정보가 reject횟수만큼 담겨 옵니다.

rabbitmq
web ui 에서 테스트 해 볼 때,

데이터를 exchange를 통해서 publish 해야만 정상 동작하고, queue에 직접 데이터를 넣을 경우 추척하지 않고

그냥 없어집니다.…. 이것 때문에 엄청 삽질 했네요..

마지막으로 reject된 메시지의 조건은 아래 세가지 입니다.

(application에서 reject, nack 할때 requeue 는 false로 해야 다시exchange를 통해 전달됩니다…)

· 
The message is rejected (basic.reject or basic.nack) with **requeue=false**,

· 
The TTL for the message
expires; or

· 
The queue length limit is
exceeded.

추가 적으로 x-dead-letter-routing-key
옵션이 있는데, 

해당 키로 등록된 queue로 전달 되는 routing-key를 등록하는 것이구요.

설정 하지 않을 경우 기본 routing-key로 다시 publish 됩니다.





































































URL
: [http://www.rabbitmq.com/dlx.html](http://www.rabbitmq.com/dlx.html)
