---
author: springloops
comments: true
date: 2012-09-24 04:59:28+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/24/katta-katta-%ec%a0%95%ec%9d%98/
slug: katta-katta-%ec%a0%95%ec%9d%98
title: '[katta] katta 정의'
wordpress_id: 53
categories:
- Open Source/katta
tags:
- Distributed
- distributed search
- hadoop
- hadoop integration solr
- hadoop+solr
- integration
- katta
- lucene
- solr
---

Solr (lucene) 과 hadoop 간의 통합에 대한 방법을 생각하다가 찾은 katta 라는 솔루션을 알게 됬는데

  


작업 진행 전에 katta에 대한 정리를 먼저 해봅니다.

  


**[katta site](http://katta.sourceforge.net/documentation) [How Katta works](http://katta.sourceforge.net/documentation/how-katta-works) 의 내용을 필요에 의해 발 번역한것입니다. **

**  
**

**꼭 ! 원문 문서를 읽어 보세요. **

**  
**

**번역이 허접합니다.**

**  
**

**  
**

**How Katta works**

**  
**

**Intro**

**  
**

**카타는 분산환경에서 돌아가는 응용프로그램입니다.**

Hadoop MapReduce, Hadoop DFS, HBase 등과 같은...

**  
**

**  
**

**Overview & Content Server**

**  
**

분산 환경에서 Master 서버는 shard Index를 가지고 있는 Node 서버를 관리해야 하고

Node 서버는 각각 자기고 있는 shard Index를 제공합니다.

당연히 Node 는  content server에 따라 다른 데이터 shard index를 제공할텐데,

현재 katta가 제공하는 content server 는 Lucene Index 와 Hadoop mapFile 입니다.

추가로 katta를 사용하는 사용자가 추가로 구현하여 사용할 수 있습니다.

  


  


**Data Structure**

  


katta 의 index는 하위 폴더(**subfolders**)의 집합이 있는 폴더입니다.

그 하위 폴더를 **index shards** 라고 부릅니다.

Content  Server에서 말한대로 하위 폴더는   
**Lucene Index, Hadoop mapFile, 사용자가 구현한 포멧**의 데이터를 가지고 있을 수 있습니다.

  


Lucene을 써보셨다면 아시겠지만 Lucene Index는 Index Writer에 의해 생성이 됩니다.

  


**_Lucene(Solr)와 katta를 사용할 경우, _**

_**katta index는 ****lucene의 Index Wirter가 만들어낸 **_

_**lucene 인덱스를 ****단순히 ****katta 하위 폴더로 복사한 것입니다.**_

**  
**

따라서 katta index는 하나의 서버나 사용자의 목적에 맞도록

Hadoop map reduce 를 사용해서 만들수 있습니다.

그런 툴을 제공한다고 합니다.

-- 이부분은 Hadoop을 아직 잘 모르는 상태라, 여기까지만...

  


  


Master - Node Communication

  


Master - 각 노드의 실패에 대하여 master는 가능한 빨리 알아야 하기 때문에

분산 시스템에서 node 간의 통신은 중요합니다.

일반적으로 마스터와 노드간에 hartbeat 메시지를 이용해서 통신합니다만,

katta는 다른 방식으로 구현합니다.

  


**katta는 분산 구성 및 Zookeeper, 야후! Research project를 이용하여 마스터-노드간 통신을 합니다.**

(Zookeeper를 또 봐야 겠네요.. 어짜피 한번은 맞닥드릴 놈이긴 했지만.. 시간이 문제입니다...)

  


Zookeeper는 실제 파일 시스템이 아닌 가상 분산파일 시스템을 읽고 쓸수 있습니다.

  


각 Node는 폴더 안에 임시파일 "nodes/live"를 작성하고, Master는 폴더의 변화를 구독합니다.

노드의 실패가 발생 할 경우 Zookeeper는 임시 파일을 삭제하고 마스터에게 통보 합니다.

  


비슷한 방식으로 Master의 실패를 처리합니다.

  


활성 상태를 "/master"에 기록하고 secondary master는 이 파일의 알림을 구독합니다.

  


비슷하게 MasterOperation : indexDeploy-/IndexUndeployOperation

NodeOperaions : ShardDeploy-/ShardUndeployOperation

  


이 있다고 합니다 (거지같은 번역/이해입니다...;;;; )

  


이런 명령은 zookeeper의 blockqueue에 저장 됩니다.

각 master/node는 blockqueue를 접근할 수 있고, 순차적으로 blockqueue에 작업을 처리 합니다.

  


예)

1) 배포클라이언트는 IndexDeployOperation을 구성하고 마스터 큐로 등록합니다.

2) 마스터는 명령을 잡아 shard-assignment 를 계획하고, nodes 에게 ShardDeployOperations 추가 합니다.

3) node는 해당 명령을 잡아 shard가 포함된 곳에 배포 합니다. ( 루씬 인덱스를 hdfs 에 등록)

성공적으로 배포되면 zookeeper "/shard-to-nodes" 폴더 아래 배포 됩니다.

4) 모든 노드의 작업이 완료되면 마스터는 공지하고, 일부 메타 데이터와 인덱스를 zookeeper에 게시 합니다.

5) 검색 클라이언트는 "/indices" 폴더를 감시하고 새로운 인덱스가 발생시 자동으로 공지합니다.

  


  


**Client Node Communication**

  


클라이언트의 검색 요청은 노드들과 통신합니다. ( Hadoop RPC 사용)

  


  


**Loading Shards to Nodes**

  


검색의 성능이 중요 하기 때문에, Katta는 첫째로 shards를 로컬 하드디스크로 복사합니다.

hadoop  파일시스템의 모든 uri 는 소스로 사용할 수 있습니다.

NAS 처럼...

  


쩝쩝쩝...

아 허접하다;;;;

  


  


  


  


  


  


  


  


**  
**

**  
**

**  
**

**  
**

**  
**

**  
**
