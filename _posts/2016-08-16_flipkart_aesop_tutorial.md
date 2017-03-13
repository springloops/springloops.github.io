---
author: springloops
comments: true
date: 2016-08-16
layout: post
title: 'Flipkart Aesop Tutorial'
categories:
- OPEN SOURCE/Flipkart Aesop
tags:
- flipkart Aesop
- opensource
- tutorial
---

# Flipkart Aesop 한번 실행해보자.

## memory based

### Relay

```bash

# clone
$> git clone https://github.com/Flipkart/aesop.git

# build
$> cd aesop/samples/sample-memory-relay; mvn clean install

# run
$> java -Dlog4j.configurationFile=file:<absolute path to log4j2.xml> -cp "./target/*:./target/lib/*" \
org.trpr.platform.runtime.impl.bootstrap.BootstrapLauncher \
./src/main/resources/external/bootstrap.xml

```


### Client

```bash

# build
$> cd aesop/samples/sample-client; mvn clean install

# run
$> java -Dlog4j.configurationFile=file:<absolute path to log4j2.xml> -cp "./target/*:./target/lib/*" \
org.trpr.platform.runtime.impl.bootstrap.BootstrapLauncher \
./src/main/resources/external/bootstrap.xml

```

## mysql based

### Mysql Setup.
Databus 와 마찬가지로 Mysql Binlog 을 읽기 때문에, Mysql 에서 binlog를 남기기 위한 설정 필요하다.

```bash
sudo vi /etc/my.cnf

[mysqld]
server-id           = 1
log_bin             = /data/logs/mysql/binlog/mysql_binary_log
expire_logs_days    = 30
max_binlog_size     = 100M
binlog_checksum     = NONE
```

### Relay

```bash

# clone
$> git clone https://github.com/Flipkart/aesop.git

# build
$> cd aesop/samples/sample-mysql-relay; mvn clean install

# run
$> java -Dlog4j.configurationFile=file:<absolute path to log4j2.xml> -cp "./target/*:./target/lib/*" \
org.trpr.platform.runtime.impl.bootstrap.BootstrapLauncher \
./src/main/resources/external/bootstrap.xml

```
