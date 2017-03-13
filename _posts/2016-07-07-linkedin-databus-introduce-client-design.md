---
author: springloops
comments: true
date: 2016-07-07
layout: post
title: 'Linkedin Databus Introduce - Client Design'
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

언제나 그렇듯, 원문을 참고하세요. [Linkedin Databus Wiki - Client Design](https://github.com/linkedin/databus/wiki/Databus-2.0-Client-Design)

번역이라기 보단 영어를 잘못하기 떄문에, 읽으면서 적은 내용 입니다.

# Databus 2.0 Client Design

## Introduction
* Databus 클라이언트는 Databus Relay 로 부터 이벤트를 소비한다. 
* Databus client library 라고 불리는 library 와 통합해야한다
* Databus client library 는 변화 스트림을 선택하기 위한 API 를 제공한다.

## Architecture

![Databus Client Architecture](https://camo.githubusercontent.com/001a97537d221955a0dce04a2afcfe0cd3676b24/68747470733a2f2f7261772e6769746875622e636f6d2f6c696e6b6564696e2f646174616275732f6d61737465722f646f632f656e67696e656572696e675f646f63732f636c69656e742d6c6962726172792d617263682e706e67)

다음 모듈이 중요해 보인다.

* Relay Puller 
* Bootstrap Puller
* Relay Events Dispatcher
* Bootstrap Events Dispatcher
* Multi-threaded callback Driver
* Checkpoint Persistence Provider

뭐... 전부인가..

1. Relay Puller 가 Relay로 부터 데이터를 가져오고,
2. Relay Events Dispatcher 가 처리해야할 Event 를 컨트롤 해서 callback driver 에게 넘기면,
3. 등록된 callback, 사용자 프로그램이 실행.
4. Relay Events Dispatcher 는 callback(사용자 프로그램) 의 수행 결과에 따라 Checkpoint provider 에게 처리한 데이터 정보 저장을 위힘하면,
5. checkpoint provider 가 저장하는 형태. 
로 보인다. (Bootstrap 역시 마찬가지..)

## Relay Connection

connection은 [relay HTTP interface](https://github.com/linkedin/databus/wiki/Databus-2.0-protocol) 를 사용하여 Relay 로 부터 실시간 변경 스트림을 획득하기 위해 사용된다.
이벤트 소비는 [online stream consumption state machine](https://github.com/linkedin/databus/wiki/Databus-2.0-Client-State-Machine#REQUEST_STREAM_RELAY) 을 따른다.


## Bootstrap Connection

connection은 [bootstrap HTTP Interface](https://github.com/linkedin/databus/wiki/Databus+v2.0++Protocol#Databusv2.0Protocol-Databus2Bootstrap) 를 사용하는 bootstrap server 로 부터, long look-back update를 획득하기 위해 사용된다.
이벤트 소비는 [bootstrap consumption state machine](https://github.com/linkedin/databus/wiki/Databus+Client+State+Machine#DatabusClientStateMachine-Bootsrapconsumption) 을 따른다.
지속성은 로컬 파일이나 zookeeper 를 이용한 공유 중 하나다.

## Dispatcher

dispatcher 는 online streams 또는 bootstrap 으로 부터 들어오는 이벤트들을 읽고, consumer callback 코드를 실행한다.
주요 책임은 

1. 올바른 callback 을 결정한다.
2. error 와 timeout 을 모니터링한다.
3. 소비 이벤트의 소비자 진행 사항을 유지하는 것을 보장한다.

dispatcher 는 상태 머신 ([state machine](https://github.com/linkedin/databus/wiki/Databus+Client+State+Machine#DatabusClientStateMachine-DISPATCHEROVERVIEW)) 을 따른다.

## Consumer Code Callbacks

[Consumer callback API](https://github.com/linkedin/databus/wiki/Chapter+II+-+Event+Model+and+Consumer+API#ChapterII-EventModelandConsumerAPI-DatabusConsumersAPI) 를 구현한 소비자 코드다.

Callback 은 [다음 실행 모델](https://iwww.corp.linkedin.com/wiki/cf/display/ENGS/Chapter+II+-+Event+Model+and+Consumer+API#ChapterII-EventModelandConsumerAPI-ExecutionModel)을 사용하여 실행된다.

[Databus migration wiki](https://iwww.corp.linkedin.com/wiki/cf/download/attachments/41117293/databus2_migration.pptx?version=1&modificationDate=1321480026000) 의 callback 섹션을 보시라.

## Checkpoint Persistence

checkpoint 는 소바자(consumer)에 의한 업데이트 스트림의 소비점에 대한내부 표현이다.

기본 형식은 Checkpoint 객체의 JSON 표현이다.

```json

{"windowOffset":-1,"snapshot_offset":-1,"prevScn":-1,"windowScn":5984508975840,"consumption_mode":"ONLINE_CONSUMPTION"}

```

## Bootstrap checkpoint

```json

{"snapshot_offset":0,"prevScn":5984488321377,"windowScn":5984488321377,"consumption_mode":"BOOTSTRAP_SNAPSHOT","windowOffset":-1,"bootstrap_since_scn":5984488321377,"bootstrap_start_scn":-1,"bootstrap_target_scn":-1,"bootstrap_snapshot_source_index":0,"bootstrap_catchup_source_index":0,"bootstrap_server_info":null,"snapshot_source":"com.linkedin.databus.example.Person"}

```