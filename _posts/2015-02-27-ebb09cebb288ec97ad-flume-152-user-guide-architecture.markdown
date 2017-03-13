---
author: springloops
comments: true
date: 2015-02-27 05:00:34+00:00
layout: post
link: https://springloops.wordpress.com/2015/02/27/%eb%b0%9c%eb%b2%88%ec%97%ad-flume-152-user-guide-architecture/
slug: '%eb%b0%9c%eb%b2%88%ec%97%ad-flume-152-user-guide-architecture'
title: '[발번역] Flume 1.5.2 User Guide - Architecture'
wordpress_id: 112
categories:
- Open Source/Apache Flume
tags:
- apache flume
- 번역
- flume
- flume ng
- 플럼
- 플럼 ng
- 아파치 플럼
- NG
---

역시 공부하느라 발번역한 문서 공유임..

  


  


[원문] [http://flume.apache.org/FlumeUserGuide.html](http://flume.apache.org/FlumeUserGuide.html)

  





# Flume 1.5.2 User Guide







## Introduction







### Overview







  





Apache Flume is a distributed, reliable, and available system for efficiently collecting, aggregating and moving large amounts of log data from many different sources to a centralized data store.




플럼은 분산되고, 신뢰할 수 있는, 그리고 수집, 집계, 그리고 많은 다른 소스로부터 중앙 데이터 저장소로, 로그 데이터의 대용량 이동에 효과적인 시스템이다.




  





The use of Apache Flume is not only restricted to log data aggregation.




아파치 플럼의 사용은 오직 로그데이터 집계에 제한되지 않는다.




Since data sources are customizable, Flume can be used to transport massive quantities of event data including but not limited to network traffic data, social-media-generated data, email messages and pretty much any data source possible.




데이터 소스는 맞춤형이기 때문에, 플럼은 이벤트 데이터의 방대한 양을 수송하는데 이용되지만, 네트워크 트래픽 데이터에 국한되지 않고, 소셜미디어가 생성한 데이터, 이메일 메시지 그리고 거의 모든 데이터 소스가 가능하다.




  





  





Apache Flume is a top level project at the Apache Software Foundation.




아파치 플럼은 아파치 탑 레벨 프로젝트다.




  





There are currently two release code lines available, versions 0.9.x and 1.x.




사용가능한 두개 출시 코드라인은 0.9 버전, 1.x 버전이다.




  





Documentation for the 0.9.x track is available at [the Flume 0.9.x User Guide](http://archive.cloudera.com/cdh/3/flume/UserGuide/).




0.9 를 위한 문서는 플럼 0.9.x 추적은 사용자 가이드를 사용할 수 있다.




  





This documentation applies to the 1.4.x track.




이문서는 1.4.x 를 에대한 문서다.







New and existing users are encouraged to use the 1.x releases so as to leverage the performance improvements and configuration flexibilities available in the latest architecture.










### System Requirements




  1. _Java Runtime Environment - Java 1.6 or later (Java 1.7 Recommended)_
  2. Memory - Sufficient memory for configurations used by sources, channels or sinks
  3. Disk Space - Sufficient disk space for configurations used by channels or sinks
  4. _Directory Permissions - Read/Write permissions for directories used by agent_









### Architecture










#### Data flow model







  





A Flume event is defined as a unit of data flow having a byte payload and an optional set of string attributes. A Flume agent is a (JVM) process that hosts the components through which events flow from an external source to the next destination (hop).




플럼 이벤트는 데이터 흐름이 가지는 바이트 수익과 문자열 속성의 선택 셋의 단위로 정의 되어졌다.




플럼 에이전트는 (JVM) 호스트 컴포넌트를 통한 외부 소스에서 부터 다음 목적지로 이벤트 흐름 프로세스다.







![](https://flume.apache.org/_images/UserGuide_image00.png)

  


  








  





A Flume source consumes events delivered to it by an external source like a web server.




플럼 소스 소비자 이벤트들은 웹 서버 같은 외부 소스로부터 배달되어진다.




The external source sends events to Flume in a format that is recognized by the target Flume source.




외부 소스는 목적지 플럼 소스로 부터 받아들여지는 포멧안의 플럼으로 이벤트를 보낸다.




For example, an Avro Flume source can be used to receive Avro events from Avro clients or other Flume agents in the flow that send events from an Avro sink.




예를 들어, 에이브로 플럼 소스는 에이브로 클라이언트 또는 에이브로 싱크로 부터 이벤트를 보내는 다른 흐름에 있는 플럼 에이전트로부터 이벤트를 받을수 있다.




A similar flow can be defined using a Thrift Flume Source to receive events from a Thrift Sink or a Flume Thrift Rpc Client or Thrift clients written in any language generated from the Flume thrift protocol.




비슷한 흐름은 정의할수 있다 쓰리프트 플럼 소스는 이벤트를 받는 것을, 쓰리프트 싱크 또는 플럼 쓰리프트 Rpc 클라이언트 또는 쓰리프트 클라이언트가 쓴 다른 언어의 일반적인 스리프트 프로토콜.




  





When a Flume source receives an event, it stores it into one or more channels.




플럼 소스가 하나의 이벤트를 받았을때, 그것은 하나 또는 어러 체널 안으로 저장된다.




The channel is a passive store that keeps the event until it’s consumed by a Flume sink.




채널은 그것이 플럼 싱크 Sink 로부터 소비되어지는 동안 이벤트를 유지하는 수동적 저장소이다.




The file channel is one example – it is backed by the local filesystem.




파일 채널은 하나의 예제이다 - 그것은 로컬 파일시스템에 의해 뒷받침된다.




The sink removes the event from the channel and puts it into an external repository like HDFS (via Flume HDFS sink) or forwards it to the Flume source of the next Flume agent (next hop) in the flow.




Sink 는 이벤트를 채널로부터 삭제하고 HDFS 같은 외부의 저장소 안으로 집어 넣거나, 흐름안의 다음 플럼 에이전트의 플럼 Source 로 전송한다.




The source and sink within the given agent run asynchronously with the events staged in the channel.




채널 안에서 이벤트 단계적으로 주어진 에이전트의 source 와 sink 는 비동기적으로 실행된다.




  








Complex flows 복잡한 흐름







Flume allows a user to build multi-hop flows where events travel through multiple agents before reaching the final destination.




플럼은 이벤트가 최종 목적지에 도달하기 전에 여러 에이전트를 통해 사용자에게 multi-hop 흐름을 구축하는것을 허용한다. (interceptor 를 말하는듯)




It also allows fan-in and fan-out flows, contextual routing and backup routes (fail-over) for failed hops.




또한, fan-in 과 fan-out 흐름을 허용하고, 실패 단계를 위한 문맥상 라우팅과 백업 라우팅을 허용한다.




fan-in 예를 들어 3개 input 을 하나의 out으로 방출. fan-out 반대.[http://en.wikipedia.org/wiki/Fan-in](http://en.wikipedia.org/wiki/Fan-in)




  








Reliability 신뢰도







The events are staged in a channel on each agent.




이벤트는 각 에이전트의 채널 안에서 단계적이다.




The events are then delivered to the next agent or terminal repository (like HDFS) in the flow.




이벤트는 흐름의 다음 에이전트 또는 종단의 저장소로 배달된다.




The events are removed from a channel only after they are stored in the channel of next agent or in the terminal repository.




이벤트는 다음 에이전트의 채널이나 종단 저장소에 저장된 후에만 채널에서 지워진다.




This is a how the single-hop message delivery semantics in Flume provide end-to-end reliability of the flow.




이것은 플럼에서 제공하는 흐름의 종단간의 single-hop 메시지 전달 의미 신뢰 방법이다.




  





Flume uses a transactional approach to guarantee the reliable delivery of the events.




플럼은 이벤트의 신뢰성있는 전달을 보장하기 위해 트랜젝션 방식을 사용한다.




The sources and sinks encapsulate in a transaction the storage/retrieval, respectively, of the events placed in or provided by a transaction provided by the channel.




sources 와 sinks 는 저장/복구 티랜젝션에서, 각각 채널에의해 제공된 또는  트랜젝션에서 캡슐화 된다.???




This ensures that the set of events are reliably passed from point to point in the flow.




이것은 일련의 이벤트가 흐름속에서 확실히 전달되는 것을 보장한다.




In the case of a multi-hop flow, the sink from the previous hop and the source from the next hop both have their transactions running to ensure that the data is safely stored in the channel of the next hop.




multi-hop 의 경우, 이전 hop 과 sink와 source 는 트랜젝션 데이터를 안전하게 다음 홉의 채널에 안전하게 저장되는 것을 보장하기 위해 실행된다.???




  








Recoverability 회복







The events are staged in the channel, which manages recovery from failure.




이벤트는 채널에서 단계적이고, 메시지는 실패로 부터 복구된다.




Flume supports a durable file channel which is backed by the local file system.




플럼은 중복 파일 채널을 제공하는데 그것은 로컬 파일 시스템으로 부터 뒷받침된다.




There’s also a memory channel which simply stores the events in an in-memory queue, which is faster but any events still left in the memory channel when an agent process dies can’t be recovered.




이것은 또한 인메모리 큐에 단순 이벤트 저장소인 메모리 체널이고, 그것은 더 빠르지만 어떤 이벤트는 에이전트 프로세스가 복구되지 못하고 죽었을때, 여전히 메모리 채널에 남게된다.













  








  

