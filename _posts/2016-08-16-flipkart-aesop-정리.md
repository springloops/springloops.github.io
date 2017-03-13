---
author: springloops
comments: true
date: 2016-08-16
layout: post
title: 'Flipkart Aesop 대충 정리'
categories:
- OPEN SOURCE/Flipkart Aesop
tags:
- flipkart
- Aesop
- tools
- opensource
- CDC
---

Linkedin 의 Databus 를 wrapping 한 Flipkart 의 aesop 문서를 읽으며 적은 글입니다.
번역이 아닙니다. 원문을 참고하세요.

# introduce
[Flipkart/aesop Change Propagation Approach](https://github.com/Flipkart/aesop/wiki/Change-Propagation-Approach)

aesop 은 링크드인 [Databus](/archivers/linkedin-databus-introduce-home) 처럼 변경된 데이터를 발견하여 로그를 수집하는 방식을 사용한다.

또한 변경 이벤트를 제공하기 위해 데이터버스의 인프라 구성요소를 사용한다.

Event Producer, Relay, Event Buffer, Bootrap, Event Consumer 와 System Change Number(SCN) 은 매우 매력적이고, Aesop 에서 대부분 사용된다.

Aesop 은 NGDATA Hbase-sep 기반으로 Event Producer 를 구현하여, HBase 데이터 저장소의 변화 탐지에 대한 지원을 확장한다.

MySQL의 경우, Aesop은 데이터 버스에서 사용할 수 있는 producer 구현을 생성한다.

Aesop 은 Push Producer, Pull Producer 를 제공하는데,

Push Producer 는 변경을 감지, Relay (Databus 로 데이터를 push 하는 것이고,

The log mining approach has limitations if data is distributed across database tables (when updates are not part of a single transaction) where-in it is hard to correlate multiple updates to a single change. It also has limitations when data is distributed across types of data stores - for e.g. between an RDBMS and a Document database. Aesop addresses this problem by implementing a "Pull" producer that uses an application-provided "Iterator" API to periodically scan the entire datastore and detect changes between scan cycles. This implementation is based off Netflix Zeno.

# 디자인 원칙

Aesop은 스트리밍 ETL 의 능력을 가진 시스템이다.

ETL 프로세스의 각 부분을 정의하는 유연함을 제공한다.

Change Propagation 시스템처럼, 데이터베이스 데이터 변경을 감지하기 위해 ETL 기능을 확장한다.

Aesop 디자인의 원칙:

* 신뢰할수 있는 변화 감지 머신. 프로세스 실패 및 복구 및 재개 할 수 있는 능력으로 생존 해야한다.
* 변경 이벤트의 최소 한번 전송됨을 보장으로 부터 최종 일관선을 가능하게 한다.
* 어디서 어떻게 생성되는 이벤트 소스이던 상관없이, 변경 이벤트 소비자를 위한 통합 인터페스.
* Horizontally scalable event sourcing and consumption
* 수평적 확장 이벤트 생성 및 소비
* 변경 전파(change propagation) 파이프라인의 다양한 컴퍼넌트를 위한 고가용성
* 메트릭과 모니터링

# Aesop 아키텍쳐의 중요 컴포넌트 

| 컴포넌트 | 기능 설명 | 구현 노트 |
|--------|---------|--------|
| Runtime container | 컴포넌트를 배포하고, 배포된 컴포넌트의 생명주기를 관리한다.| 스프링 어플리케이션 컨텍스트를 제공하는 DI 를 제공하는 스프링 어플리케이션 컨텍스트를 관리하는 Trooper Runtime|
| Module management | 독립적으로 다양한 구성 요소를 배포하고 관리할 수 있는 능력.| LinkedIn Databus ServerContainer 인스턴스의 Runtime 등록기, 그것은 Spring 설정 파일로 부터 만들어진다. 또한 classloader 의 격리, ServerContainer 인스턴스를 독립적으로 시작/중지 할 수 있는 능력을 제공한다.|
| Relay | 호스트들의 이벤트 생성자와 소비자들을 등록하고 이벤트롤 소비할 수 있게 한다.| LinkedIn Databus Relay. Aesop 에서 설정, 지표 수집과 Admin console를 위해 강화 되었다. |
| Event Producer | 소스 데이터 저장소로 부터 소스 이벤트 | LinkedIn Databus 구현을 이식하고 Aesop에서 강화했다. Push 와 Pull 메카니즘을 지원한다. Pull 은 데이터 스냅샷을 Trooper 배치, netflix Zeno 를 이용해서 생성된다.|
| Client & Event Consumer | Relay 또는 Bootstrap server 에 연결하여 변경 이벤트를 듣는다. | LinkedIn Databus 컨슈머 구현. bootstrapping, 설정과 admin console 을 위해 Aesop 에서 향상된 기능.|
| Bootstrap producer | Relay client, 그것은 이벤트를 소비하고 스냅샷 데이터베이스에 저장한다. | mysql 데이터베이스에 데이터를 저장하는 LinkedIn Databus Bootstrap producer. 설정을 위해 Aesop 에서 향상됬다.|
| Bootstrap server | 새로운 이벤트 컨슈머 bootstrapping 을 위한, Relay 로부터 떨어진 서버 | LinkedIn Databus Bootstrap server. 설정을 위해 Aesop 에서 강화됨 |

# Aesop 논리적 아키텍쳐
![Aesop logical archtecture](https://github.com/Flipkart/aesop/raw/master/docs/Overall_architecture.png)

일반적으로 파이프라인에서 변화 전파 흐름은 :

* Aesop Relay 는 하나 이상의 설정된 Event Producer 와 시작된다. Relay 와 Event producer 의 단순 설정은 여기있다 : [Sample Relay Configuration.](https://github.com/Flipkart/aesop/blob/master/samples/sample-hbase-relay/src/main/resources/external/spring-relay-config.xml). 설정 파일의 이름은 spring-relay-config.xml 처럼 고정되어 있다. 또한 배포 디렉토리에 이러한 여러 파일을 갖을 수 있다.
Trooper runtime 은 이 파일들에 정의된 Relay 와 Event Producer 인스턴스를 선택하고 로드한다.
각 설정 파일은 Trooper 에 의해 분리된 배포 모듈로 관리된다.
	* 각 Relay의 설정은 `RelayConfig` 의 인스턴스에 의해 정의된다. `RelayConfig`는 listening port, 이벤트 버퍼 할당 정책, checkpoing 저장 위치등 자세한 정보를 담고 있다.
	* Event Producer 는 Relay 가 사용하는 소스 설정과 관련있다. 즉, `PhysicalSourceConfig` 가 포함하는 하나 이상의 `LogicalSourceConfig` 정의.<br/>
	Relay 구성 파일의 정의가 포함되어 있다. [어렵다.. Sample Relay Configuration xml 파일을 보자;;;]
	* Relay 인스턴스는 Factory 에 의해 생성된다. 즉 `DefaultRelayFactory`. 이 Factory 는 relay 설정과 `ProducerRegistration` 의 하나 이상의 인스턴스를 획득한다. 그것은 Event Producer 와 source configuration 을 연결한다.<br/>
	[Sample Relay Configuration.](https://github.com/Flipkart/aesop/blob/master/samples/sample-hbase-relay/src/main/resources/external/spring-relay-config.xml) 과 같이 보자;;;
* 각 Event Producer 는 마지막 바라본 SCN(System Change Number) 를 사용하여 초기화 된 후 Relay에 의해 시작된다. 각 Event Producer 구현은 event sourcing/production 의 주기로(Cycle) 실행하고 각 주기의(Cycle) 끝에서 마지막 바라본 SCN 을 checkpoint 한다. <br/> Cycle 은 위 그림에서 확인!!
* Aesop Client 는 하나 이상으로 설정된 Event Consumer 와 시작된다. Client 와 Event Consumer 의 단순 설정은 여기 있다: [Sample Client Configuration](https://github.com/Flipkart/aesop/blob/master/samples/sample-client/src/main/resources/external/spring-client-config.xml). 설정 파일의 이름은 `spring-client-config.xml` 처럼 고정되어 있다. 또한 배포 디렉토리에 이러한 여러 파일을 갖을 수 있다. Trooper runtime 은 이 파일들에 정의된 Relay 와 Event Producer 인스턴스를 선택하고 로드한다. 각 설정 파일은 Trooper 에 의해 분리된 배포 모듈로 관리된다.
	* 각 Client 의 설정은 `ClientConfig` 의 인스턴스에 의해 정의된다. `ClientConfig` 는 listening ports, data puller 속성, checkpoint를 저장하는 위치 등 이러한 자세한 정보를 담고 있다.
	* Consumer에 관심이 있는 Event Consumer 는 실행중인 Relay 와 Avro 변경 이벤트의 유형을 가리키는 `RelayClientConfig` 를 사용하여 Client와 연결되어 있다. Client 설정 파일은 그 정의들을 담고 있다.  [어렵다.. Sample Client Configuration xml 파일을 보자;;;]
	* Client 인스턴스는 Factory 로부터 생성된다. 즉 `DefaultClientFactory`. 이 Factory 는 client 설정과 하나 이상의 `ConsumerRegistration` 의 인스턴스를 보유하고, source 구성과 Event consumer 를 연결한다.
	* Client 설정은 선택적으로 `BootstrapClientConfig` 를 갖을 수 있다. client 의 cold start 나 relay 로 부터 떨어졌을때, 이 설정은 Bootstrap Server 를 참조한다. client 는 Bootstrap server 로 부터 Relay 를 따라 잡기 위해 snapshot 을 소비하고, 온라인 소비모드로 변경한다.
* 각 Event Consumer 는 마지막 바라본 SCN(System Change Number)을 사용하여 초기화 된 후 Client 에 의해 시작된다.
각 Event Consumer 는 Relay 또는 Bootstrap Server 를 소비하는 각 변경 이벤트에 대해 Callback 이벤트를 수신한다. 이벤트 payload 는 Avro 변경 이벤트 타입니다.
* 각 consumer 는 변경 이벤트를 다르게 처리 할 수 있다. 변경 전파 파이프라인 안의 실제 Consumer 는 downstream 데이터 저장소에 쓸 수 있다. log 파일 또는 검색이 가능한 보조인덱스에..
* Bootstrap producer 는 변경 이벤트를 Mysql snapshot 데이터베이스에 작성하는 Event Consumer 의 특별한 종류다.
이 데이터베이스는 위에 서술한 것 처럼 소비자에게 데이터를 제공하기 위한 Bootstrap server 에 의해 사용된다.
* Aesop 과 Databus 는 통계 수집은 통계를 수집하고 JMX 를 통해 노출한다.
dashboard 는 실시간 변경 뷰를 제공하기 위해 이 집계데이터를 사용한다. Aesop 의 특정 사용자 또한 JMX 메트릭을 직접 사용하고, StatsD / Graphite 같은 time series database 로 넣을 수 있고, Skyline 같은 알림 툴을 사용 할 수 있다.
JMX 모니터 컴포넌트는 곧 오픈소스화 될 것이다. 대안으로 jmxtrans 같은걸 포함.


