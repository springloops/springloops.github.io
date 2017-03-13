---
author: springloops
comments: true
date: 2016-06-28
layout: post
title: 'Linkedin Databus Test2 - Databus Example - Relay'
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
- [LinkedIn Databus Test1: Databus 로컬 환경 구성.](/archivers/linkedin-databus-test1)
- [LinkedIn Databus Test2: Databus Example - Relay.](#)
- [LinkedIn Databus Test3: Databus Example - Client.](/archivers/linkedin-databus-test3)

Databus 에서 제공하는 example 을 실행해 보자.

# Databus Relay 실행


정상적으로 빌드가 되었으면 `databus-master/build/databus2-example-relay-pkg/distributions` 디렉토리 안에

`databus2-example-relay-pkg-2.0.0.tar.gz` 파일이 생성되었을 것이다.

압축을 풀어 두자. 

`$ tar -zxvf databus2-example-relay-pkg.tar.gz`


## 테스트 데이터 생성

기본적을 제공되는 데이터 생성 스크립트 `./bin/create_person.sh` 를 실행하면 된다. 
(mysql 포트변경 및 root 패스워드 변경이 필요할 것이다.)

안되면, copy & paste 로 직접 쿼리 하자.

```bash

cd bin && ./create_person.sh

#!/bin/bash
script_dir=`dirname $0`
#Setup this to point appropriately to MySQL instance
MYSQL='/usr/local/bin/mysql --protocol=tcp --port=3066' 
SQL_DIR=../sql

$MYSQL -uroot -e 'CREATE DATABASE IF NOT EXISTS or_test;';
$MYSQL -uroot -e "CREATE USER 'or_test'@'localhost' IDENTIFIED BY 'or_test';";
$MYSQL -uroot -e "GRANT ALL ON or_test.* TO 'or_test'@'localhost';"
$MYSQL -uroot -e "GRANT ALL ON *.* TO 'or_test'@'localhost';"
$MYSQL -uroot -e "GRANT ALL ON *.* TO 'or_test'@'127.0.0.1';"

${MYSQL} -uor_test -por_test -Dor_test < ${SQL_DIR}/create_person.sql
${MYSQL} -uor_test -por_test -Dor_test < ${SQL_DIR}/insert_person_test_data_1.sql
${MYSQL} -uroot -e 'RESET MASTER;'

```

9건의 데이터가 생성되었을 것이다.

## sources 파일 설정

마찬가지로 기본 제공되는 소스 설정 파일 `conf/sources-or-person.json` 에 자신의 포트와, mysql server-id 가 맞도록 수정해아한다.

```bash

{
    "name" : "person",
    "id"  : 1,
    "uri" : "mysql://or_test%2For_test@localhost:3306/3306/mysql-bin",
    "slowSourceQueryThreshold" : 2000,
    "sources" :
    [
        {
        "id" : 40,
        "name" : "com.linkedin.events.example.or_test.Person",
        "uri": "or_test.person",
        "partitionFunction" : "constant:1"
         }
    ]
}

```

## Relay 실행

```bash

$ ./bin/start-example-relay.sh or_person -Y ./conf/sources-or-person.json

```

정상적으로 실행되었으면, pid 가 있을 것이다.

`databus-master/build/databus2-example-relay-pkg/distributions/logs` 아래 로그를 확인하면 더 자세한 내용을 볼 수 있다.

## Relay 테스트

### 소스확인

```bash

$ curl -s http://localhost:11115/sources
[{"name":"com.linkedin.events.example.or_test.Person","id":40}]

```

### 데이터 변경

```sql

update person set first_name='John' where id=1;

```

### 변경 데이터 건수 확인

```bash

$ curl -s http://localhost:11115/containerStats/inbound/events/total?pretty | grep numDataEvents

"numDataEvents" : 1,

```

## TroubleShooting

Databus Example 을 돌리기 위한 Relay 와 Client 의 source name 이 맞지 않는다.
(정확히는 oracle 용 Client 만 제공되는 것 같아 보인다.)

그러한 이유로, 

다음과 같은 에러를 발생시킨다.

`DatabusException: Mismatch in db schema vs avro schema`


정상적인 실행을 위해 Client 를 수정할 경우 ReBuild 가 필요하므로 (소스에 하드코딩되어 있음),

Relay 설정을 변경하여 동작하도록 했다.

1. ./conf/sources-or-person.json 파일안의 name 값 수정

"name" : "com.linkedin.events.example.or_test.Person", 값을 
"name" : "com.linkedin.events.example.person.Person" 으로 변경한다.

client 프로그램에 저 이름이 하드코딩되어 있다.

```bash

{
    "name" : "person",
    "id"  : 1,
    "uri" : "mysql://or_test%2For_test@localhost:3306/3306/mysql-bin",
    "slowSourceQueryThreshold" : 2000,
    "sources" :
    [
        {
        "id" : 40,
        "name" : "com.linkedin.events.example.person.Person",
        "uri": "or_test.person",
        "partitionFunction" : "constant:1"
         }
    ]
}

```

2. ./schemas_registry 안의 값 수정

기본적으로 다음과 같은 파일 두건이 저장되어 있다.

- com.linkedin.events.example.person.Person.1.avsc
- com.linkedin.events.example.or_test.Person.1.avsc

`com.linkedin.events.example.person.Person.1.avsc` 파일을 삭제한다.

`com.linkedin.events.example.or_test.Person.1.avsc` 파일안의 namespace 값을 `com.linkedin.events.example.person.Person` 로 수정한다.

`com.linkedin.events.example.or_test.Person.1.avsc` 파일의 이름을 `com.linkedin.events.example.person.Person.1.avsc` 로 변경한다.

`index.schemas_registry` 파일을 열고, `com.linkedin.events.example.or_test.Person.1.avsc` 내용을 삭제한다.

그리고 실행하면 정상적으로, relay 를 구성한다.



## Relay RestAPI 

지금까지 발견한 RestAPI 정리함.

* curl http://localhost:11115/containerStats/inbound/events/total?pretty
* curl http://localhost:11115/sources
* curl http://localhost:11115/relayStats/outbound/http/clients 

코드를 보면 뭐.. 더 있겠지...


