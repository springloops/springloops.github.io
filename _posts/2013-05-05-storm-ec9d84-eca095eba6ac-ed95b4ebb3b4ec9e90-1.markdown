---
author: springloops
comments: true
date: 2013-05-05 10:05:59+00:00
layout: post
link: https://springloops.wordpress.com/2013/05/05/storm-%ec%9d%84-%ec%a0%95%eb%a6%ac-%ed%95%b4%eb%b3%b4%ec%9e%90-1/
slug: storm-%ec%9d%84-%ec%a0%95%eb%a6%ac-%ed%95%b4%eb%b3%b4%ec%9e%90-1
title: Storm 을 정리해보자 - 1
wordpress_id: 88
categories:
- Open Source/Apache Storm
tags:
- Storm
- storm 구성
- storm 정의
---

Strom 은

스트림 데이터를 처리하기 위한 분산, 신뢰성, 결함 허용 시스템 입니다.

  


작업은 각각의 간단한 프로세스 태스크 컴포넌트에게 위임이 됩니다.

  


  


1. SPOUT : 데이터 입력 컴포넌트

  


Storm 클러스터에서 스트림(연속된) 데이터를 입력받는 컴포넌트를 SPOUT 라고 부릅니다.

  


2. BOLT : Spout의 입력 스트림을 처리하는 컴포넌트

  


BOLT 는 저장소의 데이터를 유지하거나 또 다른 BOLT 로 전달합니다.

  


Storm 클러스터는 bolt의 체인으로 생각할 수 있고, Spout 이 전달한 데이터를 변환 처리 합니다.

  


TV 방송의 자막에서 특정한 단어의 수를 세고 저장하는 프로세스가 있다고 생각했을때,

  


연속 적으로 TV의 자막이 SPOUT 으로 들어 오면 SPOUT은 입력받은 텍스트 라인을 BOLT 로 전달 합니다.

BOLT 는 정해진 규칙에 따라 텍스트의 라인을 처리하고, 저장 처리를 하는 BOLT로 전달 하고 해당 BOLT는 DATABASE에 저장합니다.

언제든 결과가 필요하면, 단순히 저장된 DATABASE로 쿼리를 하면 됩니다.

  


TOPOLOGY : SPOUT 과 BOLT의 연결 구성을 topoogy라 부릅니다.

  


  


![](http://localhost/wordpress/wp-content/uploads/1/cfile28.uf.2766414351862FB80A90A8.png)

  


  


  


Storm의 일반적인 사용 사례

1. Processing stream

스트림 처리.

다른 스트림 처리 시스템과는 달리 중간에 Queue 큐가 필요 없습니다.

  


2. Continuous computation

끊임없는 연산

싸이트의 통계 데이터에 대한 실시간 업데이트, 조회 결과를 지속적으로 클라이언트에게 전달한다.

  


3. Distributed remote procedure call

분산 원격 처리 요청

CPU 작업을 쉽게 병렬화 한다.

  


참고 : [Getting started with Storm](http://shop.oreilly.com/product/0636920024835.do)

  


  


  


  

