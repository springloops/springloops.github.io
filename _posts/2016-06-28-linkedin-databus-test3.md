---
author: springloops
comments: true
date: 2016-06-28
layout: post
title: 'Linkedin Databus Test3 - Databus Example - Client'
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
- [LinkedIn Databus Test2: Databus Example - Relay.](/archivers/linkedin-databus-test2)
- [LinkedIn Databus Test3: Databus Example - Client.](#)

Databus 에서 제공하는 example 을 실행해 보자.

# Databus Client 실행


마찬가지로 정상적으로 빌드가 되었으면 `databus-master/build/databus2-example-client-pkg/distributions` 디렉토리 안에

`databus2-example-client-pkg-2.0.0.tar.gz` 파일이 생성되었을 것이다.

압축을 풀어 두자. 

`$ tar -zxvf databus2-example-client-pkg.tar.gz`

## Databus Client 실행

Databus2 example client 프로그램은 변경된 데이터를 log4j를 통해 출력하는 프로그램이다.

다음과 같이 실행하면된다.

```bash

$ ./bin/start-example-client.sh person

$ tail -f ./logs/client.log

2016-06-29 15:44:58,066 +271 [callback-1] (INFO) {PersonConsumer} firstName: John, lastName: varadaran, birthDate: 315500400000, deleted: false
2016-06-29 15:44:58,069 +274 [callback-1] (INFO) {PersonConsumer} firstName: John2, lastName: varadaran, birthDate: 315500400000, deleted: false
2016-06-29 15:44:58,072 +277 [callback-1] (INFO) {PersonConsumer} firstName: John1, lastName: varadaran, birthDate: 315500400000, deleted: false
2016-06-29 15:44:58,074 +279 [callback-1] (INFO) {PersonConsumer} firstName: John2, lastName: varadaran, birthDate: 315500400000, deleted: false
2016-06-29 15:44:58,076 +281 [callback-1] (INFO) {PersonConsumer} firstName: John22, lastName: varadaran, birthDate: 315500400000, deleted: false
2016-06-29 15:44:58,079 +284 [callback-1] (INFO) {PersonConsumer} firstName: John, lastName: varadaran, birthDate: 315500400000, deleted: false
2016-06-29 15:44:58,081 +286 [callback-1] (INFO) {PersonConsumer} firstName: John1, lastName: varadaran, birthDate: 315500400000, deleted: false

```

Databus 에서 제공하는 Example 이 정확하게 동작하지 않아 삽질이 필요 했지만, 덕분에 많은 문서를 다 보지 않고도 약간의 문제점을 발견할 수 있었던것 같은 느낌적인 느낌이 있었다.

1. Table 별 Avro Schema 설정이 필요하고, 
2. Source 테이블이 변경될 경우, 기존 파이프라인은 어떻게 대응해야하는지,
3. 정적인 설정이 제법 있어보임에 관리 측면에서 고민이 필요할 것 같다.

무엇보다, 사용자 커뮤니티가 많지 않다는 점, 최종 커밋이 오래된 점이 날 불안하게 한다..

뭐 이제 Databus 문서와 코드를 정독 해보는 수 밖에......
