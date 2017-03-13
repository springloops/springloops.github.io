---
author: springloops
comments: true
date: 2016-07-13
layout: post
title: 'Linkedin Databus Introduce - Databus 2.0 relay design'
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

언제나 그렇듯, 원문을 참고하세요. [Linkedin Databus Wiki - relay design](https://github.com/linkedin/databus/wiki/Databus-2.0-relay-design)

번역이라기 보단 영어를 잘못하기 떄문에, 읽으면서 적은 내용 입니다.

# Databus 2.0 Relay Design

## Introduction

Databus Relays 는 다음을 책임진다.

* 데이터베이스 소스(혹은 다른 relay chaine)의 Databus sources 로 부터 변경된 row 를 읽고, in-memory buffer 안에 Databus data change events 로 직렬화한다.
* Databus Client (Bootstrap Producers, 다른 relay chaine 혹은 event 소비자)로부터의 요청을 듣고, Client 에게 새로운 Databus data change event 를 스트림한다.

## Architecture

![linkedin relay architecture](https://camo.githubusercontent.com/6eb913fab28612fe9d0d4f5975063b107d7501c7/68747470733a2f2f7261772e6769746875622e636f6d2f6c696e6b6564696e2f646174616275732f6d61737465722f646f632f656e67696e656572696e675f646f63732f72656c61792d6172636869746563747572652e706e67)

* [mmap](http://www.joinc.co.kr/w/Site/system_programing/IPC/memory_map)
	* SCN index 저장용.
* Netty Container
	* RESTful Api 용.
* EventProducer
	* Database Event 생성용, 
	* 데이터베이스의 변화를 감지하여, Avro 형태로 데이터를 변환한다.
* Event Buffer
	* Event 저장용.
* Request Processor
	* Request 처리용.

relay 는 하나 이상의 circular event buffer 를 가지고, circular event buffer 에 시스템 변경 번호(SCNs) 가 증가하는 순서대로 databus event 를 저장한다.
이 버퍼 메모리는 직접적 또는 메모리 매핑된 파일 시스템에 의해 뒷받침 될 수 있다.
각 버퍼는 SCN 인덱스 이라고 불리는 메모리에 희소(sparse) 인덱스 및 MaxSCN reader/writer 를 갖는다.
(- mmap 을 사용하기 때문에 희소 인덱스 일 것이다. 파일 블락 크기로 메모리와 매핑 되므로, 단편화가 발생함.)

MaxSCN reader/writer 는 relay 에 들어간 이벤트안에서 볼 수 있는 가장 높은 SCN 인덱스의 값을 지속(persist) 시킨다.
(= relay 안의 이벤트 인덱스 관리자 정도 되는 듯)

relay 는 Netty 채널에 수신된 요청 프로세서(Requet Processor)를 통해 Databus Client 의 요청을 받아 들인다.
relay 는 evnet pull 과 relay manage 를 RESTful 인터페이스로 노출한다.

만약 이벤트 버퍼의 컨텐츠가 메모리 맵에 설정되면, relay 이벤트가 재시작에 의해 보존 되도록 SCN 인덱스 및 종료에 대한 메타 데이터를 저장합니다.

[Event Buffer Design](https://github.com/linkedin/databus/wiki/Databus%202.0%20Event%20Buffer%20Design) 에 대한 페이지를 보시라.

## Database Event Producer

Database Event Produver 는 정기적으로 Databus source 안의 기본 Oracle Database 변화를 조사한다. ( Mysql 은???? )
만약 변화를 발견하면, 소스안의 모든 변화된 rows 를 읽고 Avro 데이터로 변환한다.

JDBC RowSets 로 부터 변환된 AvroRecords 는 Schema Registry 안의 Avro Schemas를 기반으로 자동으로 저장을 수행한다.

Schemas 는 Databus sources에 대한 Oracle schema에서 자동으로 생성되어진다.
([linkedin databus test2 - relay ](/archivers/linkedin-databus-test2) 에서 실행한 결과를 보면, distributions/schemas_registry 에 자동으로 생성되어 있음.
com.linkedin.databus2.schemas package 확인해야함 !)

Avro 레코드는 데이터버스 관련 메타 데이터 및 avro 레코드의 바이너리 직렬화 데이터와 데이터 페이로드를 포함하는 Databus Event 로 직렬화된다. 

AbstractEventProducer class 는 event producer 개발을 위해 프레임웍을 제공한다.
txlog 테이블을 포함하도록 수정하고 txlog 테이블을 업데이트 트리거 된 Oracle 데이터베이스에서 읽는 동안 OracleEventProducer는 DbusEvents를 생성합니다.

## MaxSCN Reader/Writer

MaxSCN Reader/Writer 는 Database Event Producer 의 진행사항의 추적을 유지하기 위해 사용된다. (DBEP)
reader 는 DBEP 의 시작학하는 동안 사용된다.
시작 SCN이 지정되어 있지 않은 경우, 하나는 마지막으로 남은 곳에서 부터 DBEP 계속할 수 있도록 리더를 사용하여 읽을 수 있습니다.

업데이트의 각 배치는 DBEP 로 부터 읽고 relay event buffer 로 저장한 후, DBEP store 는 MaxSCN 을 이용하여 마지막 처리된 SCN 위치를 저장한다.

현재, MaxSCN reader/writer 는 SCN 로컬 파일에 저장한다.

```
5984592768094:Thu Dec 13 00:54:07 UTC 2012
```
또 다른 구현은 RDBMS 나 Zookeeper 를 사용할 수 있다.

## Rest Interface

See [Databus V2 Protocol](https://github.com/linkedin/databus/wiki/Databus%202.0%20Protocol)

## JMX 

JMX 는 Jetty container 기본으로 제공한다. 

relay 모니터링을 위해 다영한 MBeans 를 노출한다.

JavaDoc 을 보라

* ContainerStatsMBean
* DbusEventsTotalStatsMBean
* DbusEventsStatisticsCollectorMBean

## Schema Registry

schema registry 는 Databus 에 알려진 모든 데이터베이스 테이블의 스키마를 가진 패키지이다. 

## Creating Avro recods for the table schema


> 
- mysql 용 avro schema generator 관련 <br/>
  아래는 기본적으로 oracle 을 위한 것이고, mysql 을 위한 avro schema 생성 툴은 보이지 않는다. <br/>
  황당한건, aesop 의 avro schema generator 를 이용하면 된다는 글을 확인함.. <br/>
  Databus 넌 사용하면 안되는거니???<br/>
- [https://github.com/linkedin/databus/issues/46](https://github.com/linkedin/databus/issues/46)<br/>
- [https://github.com/Flipkart/aesop/wiki/AVRO-Schema-Generator](https://github.com/Flipkart/aesop/wiki/AVRO-Schema-Generator)<br/>

`gradle assemble` 을 수행한 후, 압축 해제되고 `build/databus2-cmdline-tools-pkg/distributions/` 에 tarball 생성된다.

```bash
$ ./bin/dbus2-avro-schema-gen.sh -namespace com.linkedin.events.example.person -recordName Person \
    -viewName "sy\$person" -avroOutDir /path/to/work/trunk/schemas_registry -avroOutVersion 1 \
    -javaOutDir /path/to/work/trunk/databus2-example-events/databus2-example-person/src/main/java \
    -userName person -password person
Processed command line arguments:
recordName=Person
avroOutVersion=1
viewName=sy$person
javaOutDir=/path/to/work/trunk/databus2-example-events/databus2-example-person/src/main/java
avroOutDir=/path/to/work/trunk/schemas_registry
userName=person
password=person
namespace=com.linkedin.events.example.person
Generating schema for sy$person
Processing column sy$person.TXN:NUMBER
Processing column sy$person.KEY:NUMBER
Processing column sy$person.FIRST_NAME:VARCHAR2
Processing column sy$person.LAST_NAME:VARCHAR2
Processing column sy$person.BIRTH_DATE:DATE
Processing column sy$person.DELETED:VARCHAR2
Generated Schema:
{
“name” : “Person_V1”,
“doc” : “Auto-generated Avro schema for sy$person. Generated at Dec 04, 2012 05:07:05 PM PST”,
“type” : “record”,
“meta” : “dbFieldName=sy$person;”,
“namespace” : “com.linkedin.events.example.person”,
“fields” : [ {

"name" : "txn",
"type" : [ "long", "null" ],  
"meta" : "dbFieldName=TXN;dbFieldPosition=0;"  

}, {

"name" : "key",
"type" : [ "long", "null" ],  
"meta" : "dbFieldName=KEY;dbFieldPosition=1;"  

}, {

"name" : "firstName",
"type" : [ "string", "null" ],  
"meta" : "dbFieldName=FIRST_NAME;dbFieldPosition=2;"  

}, {

"name" : "lastName",
"type" : [ "string", "null" ],  
"meta" : "dbFieldName=LAST_NAME;dbFieldPosition=3;"  

}, {

"name" : "birthDate",
"type" : [ "long", "null" ],  
"meta" : "dbFieldName=BIRTH_DATE;dbFieldPosition=4;"  

}, {

"name" : "deleted",
"type" : [ "string", "null" ],  
"meta" : "dbFieldName=DELETED;dbFieldPosition=5;"  

} ]
}
Avro schema will be saved in the file: /path/to/work/trunk/schemas_registry/com.linkedin.events.example.person.Person.1.avsc
Generating Java files in the directory: /path/to/work/trunk/databus2-example-events/databus2-example-person/src/main/java
Done.
```

(Note that /path/to/work will be whatever path prefix you used in the -avroOutDir and -javaOutDir options.)