---
author: springloops
comments: true
date: 2016-06-28
layout: post
title: 'Linkedin Databus Test1 - Databus 로컬 환경 구성'
categories:
- OPEN SOURCE/Linedin Databus
tags:
- LinkedIn
- Databus
- tools
- opensource
- CDC
---

LinkedIn Databus 테스트 관련 정리입니다. 3개 글로 되어 있습니다.

>
- [LinkedIn Databus Test1: Databus 로컬 환경 구성.](#)
- [LinkedIn Databus Test2: Databus Example - Relay.](/archivers/linkedin-databus-test2)
- [LinkedIn Databus Test3: Databus Example - Client.](/archivers/linkedin-databus-test3)

# Databus Build
Dababus 는 바이너리 실행 파일을 제공하지 않는다.
직접 빌드가 필요한데, 어렵지 않고 간단하게 진행가능하다.

## ojdbc6 다운로드.

[여기서 다운로드 받기](http://www.oracle.com/technetwork/database/enterprise-edition/jdbc-112010-090769.html)

다운로드 받은 ojdbc6.jar 파일을 `ojdbc6-11.2.0.2.0.jar` 로 변경한다.

```bash

mv ojdbc6.jar ojdbc6-11.2.0.2.0.jar

```

해당 ojdbc 파일을 `sandbox-repo/com/oracle/ojdbc6/11.2.0.2.0/` 아래로 이동한다.

```bash

mv ojdbc6-11.2.0.2.0.jar ${databus-local-repo}/sandbox-repo/com/oracle/ojdbc6/11.2.0.2.0/

```

## Databus Source 빌드

```bash

databus-local-repo$ gradle -Dopen_source=true assemble

```

# Mysql Setup.
Databus 는 Mysql Binlog 을 읽기 때문에, Mysql 에서 binlog를 남기기 위한 설정 필요하다.

```bash
sudo vi /etc/my.cnf

[mysqld]
server-id           = 1
log_bin             = /usr/local/mysql/data/mysql-bin.log
expire_logs_days    = 10
max_binlog_size     = 100M
binlog_checksum     = NONE

:wq
```

`binlog_checksum` 을 NONE 으로 설정하지 않으면, Databus relay 에서 파싱 오류를 발생하고, 정상 동작하지 않는다.

```bash
2016-06-28 13:39:38,569 +82255 [binlog-parser-1] (ERROR) {AbstractBinlogParser} failed to parse binlog
org.apache.commons.lang.exception.NestableRuntimeException: ErrorPacket[packetMarker=255,errorCode=1236,slash=#,sqlState=HY000,errorMessage=Slave can not handle replication events with the checksum that master is configured to log; the first event 'mysql-bin.000001' at 4, the last event read from '/usr/local/mysql/data/mysql-bin.000001' at 123, the last byte read from '/usr/local/mysql/data/mysql-bin.000001' at 123.]
	at com.google.code.or.binlog.impl.ReplicationBasedBinlogParser.doParse(ReplicationBasedBinlogParser.java:101)
	at com.google.code.or.binlog.impl.AbstractBinlogParser$Task.run(AbstractBinlogParser.java:244)
	at java.lang.Thread.run(Thread.java:745)
```
