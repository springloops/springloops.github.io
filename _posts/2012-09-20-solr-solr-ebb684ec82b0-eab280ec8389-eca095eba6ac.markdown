---
author: springloops
comments: true
date: 2012-09-20 08:48:29+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/20/solr-solr-%eb%b6%84%ec%82%b0-%ea%b2%80%ec%83%89-%ec%a0%95%eb%a6%ac/
slug: solr-solr-%eb%b6%84%ec%82%b0-%ea%b2%80%ec%83%89-%ec%a0%95%eb%a6%ac
title: '[solr] solr 분산 검색 정리 1.Solr DistributedSearch'
wordpress_id: 50
categories:
- Open Source/Apache Lucene/Solr
tags:
- cloud
- distributed search
- multiindex
- shard
- solr
- solr distributed
- solr shard
- solrcolud
---

**Solr DistributedSearch 정리.. (2012.09.20 Solr Ver: 3.6)**

  


solr single instance에 요청이 집중 될 경우 성능적인 이슈가 있을텐데

  


이를 부하를 분산 시켜 주기 위한 Solr 자체적으로 제공하는 기능을 몇가지 정리 해봤습니다.

  


**1. Distributed Search**

**2. Replication index : http://wiki.apache.org/solr/SolrReplication**

**3. solrcloud (ver4.0) : http://wiki.apache.org/solr/SolrCloud**

**4. clustering Component : ****http://wiki.apache.org/solr/ClusteringComponent**

** (4 번은 검색 결과를 클러스터링 합니다.... 조금 다른 의미죠..) **

**5. solr + katta + hadoop**

  


  


실제 작업해본것도 있고 그렇지 않은것도 있습니다만,

  


작업 해보는 대로 정리해서 수정하겠습니다.

  


  


1번 방법부터 설명하겠습니다.

  


**1. Distributed Search**

참고 : [http://wiki.apache.org/solr/DistributedSearch](http://wiki.apache.org/solr/DistributedSearch)

  


말그대로 분산 검색입니다.

Solr 검색 인스턴스를 물리적인 서버 각각 설치하는 방법입니다.

  


server1 , server2 에 각각의 solr를 설치 하고 각각 Indexing을 합니다.

정말 단순합니다.

저같은 경우는 server1에 solr를 설치 하고, 해당 서버 관련 파일들(tomcat, solr)을 server2로 복사했습니다.

  


참조 : [http://somethingbook.tistory.com/entry/solr-데이터베이스의-데이터를-indexing-하기](http://somethingbook.tistory.com/entry/solr-데이터베이스의-데이터를-indexing-하기)

인덱싱의 경우 data import 방식으로 밀어 넣었습니다.

data shading 을 위해 각 서버에 배포된 db-data-config.xml 파일의 sql query를 수정하였습니다.

  


ex) server 1 : where id =<1 and id >=10000

server 2 : where id =<10001 and id >=20000

  


실제 solr wiki 에선 **shard를 위한 로직은 서비스 사용자의 몫**이라 말하고 있습니다.

해당 서비스에 맞게 분산 인덱싱을 위한 로직을 정의 하면 되겠네요.

  


그리고 각각의  서버로 ?command=full-import를 실행하는 것입니다.

  


각각 따로 배포된 인덱스를 어떻게 통합적으로 검색을 하느냐..

바로 아래와 같습니다.

  


ex) http://192.168.0.240:8080/solr/select?shards=192.168.0.240:8080/solr,192.168.0.241:8080/solr&q=title:apple

  


shareds 파라미터로 각 서버의 컨텍스트 까지의 URL을 입력합니다.

각 서버별 구분자는 " , " 입니다.

  


그 뒤로는 single instance 와 같은 방식으로 진행하시면 됩니다.

  


검색 요청을 날려보면 두 서버에 로그가 쌓이는 것을 확인할수 있네요~

  


이 구성은 몇가지 단점이 있습니다.

  


solr wiki에도 정의되어 있듯이 **주의 할점**과 몇 가지 **기능에 제약**이 있습니다.

  


기능적 제약에 대해선 문서를 참조 하시고....

  


참조 : [http://wiki.apache.org/solr/DistributedSearch#Distributed_Deadlock](http://wiki.apache.org/solr/DistributedSearch#Distributed_Deadlock)

  


주의 할점 한가지에 대해서 설명하겠습니다.

  


**Distributed Search의 경우 각 서브 쿼리를 수행후 최상위 쿼리를 수행한다고 합니다.**

**이 말인 즉 shard로 지정된 서버에서 쿼리를 수행 후 최상위 서버 (도메인) 으로 등록된 서버로 **

**가져와 종합 하여 클라이언트에게 응답하는 방식인듯 합니다.**

  


_때문에 최상위 서버는 하위 shard로 지정된 서버의 요청도 받게 되기 때문에 해당 최상위 서버의 HTTP Connection Thread 갯수를 충분히 늘려줘야 한다고 하네요._

  


부족할 경우 병목이 발생한다고 합니다.

  


구성을 Top server 1, Sub server1 로 구성 하고

  


요청에 대한 로그를 확인해 보면 최상위 서버에 요청이 3번, 서브 서버에 한번이 남습니다.

  


추측하건데

  


**client 로 부터 Top server는 요청을 받으면 loop back 으로 자신의 index 검색을 요청하고, **

**Sub server 로 요청을 합니다.**

**Sub server에서 요청에 대한 검색을 한뒤 검색 결과를 request를 통해 Top server로 냅니다.**

**Top server 는 자신의 검색결과와 Sub server의 검색 결과를 종합하여 응답하는 형식인듯 합니다.**

**  
**

**그래서 Top server 는 부하를 shard 수 만큼 받게 되는 것 같습니다.**

  


(실제로 요청을 하고 로그를 확인 해보면 최상위 서버에는 요청이 shard 서버보다 더 많이 남는 것을 확인 할 수 있습니다.

확인해 보세요~^^;;)

  


  


추가적으로 물리적으로 한대의 서버에 다수의 solr 인스턴스를 띄우는 방법에 대해서 잘 정리된 blog를 첨부 합니다.

첨부 : [http://dol9.tistory.com/172](http://dol9.tistory.com/172)

  


하지만 이 방법은 물리적으로 서버의 부하를 줄여주는 방식이 아니기 때문에 얼마나 성능상 효과를 볼수 있을지 의문이네요

  


개인적으로는 비추인 구성인듯 합니다.

  


마지막으로 한대의 서버에 여러 인스턴스를 띄우는 경우 하나의 관리자 페이지에서 core를 관리 할수 있지만

  


물리적으로 여러대로 구성된 **Distributed Search의 경우 각 서버별 관리자 페이지에서 core를 관리 해야 하기때문에 ****관리상 불편함이 존재 하겠습니다.**
