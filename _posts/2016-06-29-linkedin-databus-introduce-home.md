---
author: springloops
comments: true
date: 2016-06-29
layout: post
title: 'Linkedin Databus Introduce - Home'
categories:
- OPEN SOURCE/Linedin Databus
tags:
- LinkedIn
- Databus
- tools
- opensource
- CDC
---

Linkedin Databus Wiki Home 을 발번역 하였습니다.

언제나 그렇듯, 원문을 참고하세요. [Linkedin Databus Wiki](https://github.com/linkedin/databus/wiki)

번역이라기 보단 영어를 잘못하기 떄문에, 읽으면서 적은 내용 입니다.

## 개요

Databus 는 링크드인의 데이터 처리 파이프라인의 일부로 낮은 지연 변화 캡쳐 시스템이다.

다음같은 기능을 제공한다.

1. 소스와 소비자 사이를 격리시킨다.
2. 순서와 최소 한번 전송과 고가용성을 보장한다.
3. 전체 데이터와 임의 시점의 데이터 모두 소비한다.
4. 분할 소비
5. 소스 일관성 유지

## 구성

중요 구성은 다음과 같다.

### Databus Relays

1. Databus 소스로 부터 소스 데이터베이스의 변경된 rows 를 읽어와, 메모리 버퍼에 Databus의 데이터 변경 이벤트로 직렬화 한다. ( Avro 이용 )
2. Databus Client 로부터 요청을 듣고, 새로운 Databus 데이터 변화 이벤트로 변환한다.
3. 더 자세한 내용 -> [Databus 2.0 Relay](https://github.com/linkedin/databus/wiki/Databus-2.0-Relay)

### Databus Clients

1. Relay 에서 새로운 데이터 변경 이벤트를 체크하고 명시적인 콜백 비즈니스 로직을 실행한다.
2. 만약 relay 에서 부터 너무 멀리 떨어질 경우, 부트스트랩 서버의 격차해소(catchup) 쿼리를 실행한다.
3. 새로운 Databus Client 는 부트스트랩 서버의 부트스트랩 쿼리를 수행하고, 최신 데이터 변경 이벤트를 위해 Relay 를 변경한다.
4. 하나의 클라이언트는 전체 데이터 버스 스트림을 처리 할 수도 있고, 클러스터의 일부만 소비할 수도 있다.
5. 더 자세한 내용 -> [Databus 2.0 Client](https://github.com/linkedin/databus/wiki/Databus-2.0-Client)

### Databus Bootstrap Producers
1. 특별한 종류의 Databus Client
2. Relay 에서 새로운 데이터 변경 이벤트를 체크한다.
3. Mysql 에 이벤트를 저장한다.
4. Mysql은 Client 가 Bootstrap 과 격차해소(catchup)를 하기 위해 사용된다.

### Databus Bootstrap Servers
1. Databus Client 로 부터의 요청(request)을 듣고, 시작(bootstrapping) 과 격차해소(catchup)을 위해 긴 look-back 데이터 변환 이벤트를 반환한다.
