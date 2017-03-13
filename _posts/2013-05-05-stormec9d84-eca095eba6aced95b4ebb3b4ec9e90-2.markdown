---
author: springloops
comments: true
date: 2013-05-05 10:28:54+00:00
layout: post
link: https://springloops.wordpress.com/2013/05/05/storm%ec%9d%84-%ec%a0%95%eb%a6%ac%ed%95%b4%eb%b3%b4%ec%9e%90-2/
slug: storm%ec%9d%84-%ec%a0%95%eb%a6%ac%ed%95%b4%eb%b3%b4%ec%9e%90-2
title: Storm 을 정리해보자 - 2
wordpress_id: 89
categories:
- Open Source/Apache Storm
tags:
- 마스터 노드
- 워커 노드
- master
- Nimbus
- Storm
- storm 마스터 노드
- storm 구성
- storm 워커 노드
- storm 정의
- storm master node
- Storm master woker
- storm wokrer node
- supervisor
- WORKER
---

Storm 클러스터는 마스터(Master) 노드와 워커(Worker) 노드로 구성된다.

  


1. Master node : Nimbus

Master 노드는 데몬 프로세스로 구동 되며 Nimbus 라 불린다.

Nimbus 는 주위 클러스터에 코드를 배포하고, Worker 노드로 작업을 할당하고, Worker 노드의 실패를 모니터링 한다.

  


2. Worker Node : Supervisor

worker 노드 역시 데몬 프로세스로 실행 되며 Supervisor 라 불리고, 토폴로지 (Topology) 의 일부를 실행한다.

  


Storm 안의 topology는 다른 머신의 많은 워커 노드에서 실행됩니다.

  


Storm은 모든 클러스터의 상태를 Zookeeper 또는 로컬 디스크에 보관 하여,

상태를 갖고 있지 않기 때문에 (stateless) 시스템에 영향없이 실패 하거나 재시작 할 수 있습니다.

  


  


![](http://localhost/wordpress/wp-content/uploads/1/cfile3.uf.220AD4475186385701CDAF.jpg)

  


  


참고 : [Getting started with Storm](http://shop.oreilly.com/product/0636920024835.do)

  

