---
author: springloops
comments: true
date: 2012-10-24 09:07:31+00:00
layout: post
link: https://springloops.wordpress.com/2012/10/24/solr-solr-replication-ver-36/
published: false
slug: solr-solr-replication-ver-36
title: '[Solr] solr replication ver 3.6'
wordpress_id: 68
categories:
- Open Source/Apache Lucene/Solr
---

Solr Replication

  


서버의 부하를 줄여주기 위해 Solr는 Replication 을 제공합니다.

  


간단하게 만들어 놓은 Solr replication 기능을 사용해보도록 하겠습니다.

  


우선 Master 서버와 Slave 서버를 정의 하기 위해 solrconf.xml 파일을 수정해야 합니다.

  


Master 의 solrconfig.xml











<requestHandler name="/replication" class="solr.ReplicationHandler" >




<lst name="master">

<!-- startup, commit, optimize 가 발생하면 replication을 시작합니다 -->




<str name="replicateAfter">startup</str>




<str name="replicateAfter">commit</str>

<str name="replicateAfter">optimize</str>




  


<!-- 인덱스를 백업하겠다는 의미로... 필수는 아님. -->




<!-- <str name="backupAfter">optimize</str> -->




  





<!-- 인덱스파일 이외에 추가로 복제해야할 파일을 정의 합니다. 구분자는 , 구요,, 이번엔 인덱스만 복제하겠습니다.-->




<!-- <str name="confFiles">schema.xml,stopwords.txt,elevate.xml</str> -->




<!--The default value of reservation is 10 secs.See the documentation below . Normally , you should not need to specify this -->




<str name="commitReserveDuration">00:00:10</str>




</lst>




<!-- keep only 1 backup. Using this parameter precludes using the "numberToKeep" request parameter. (Solr3.6 / Solr4.0)-->




<!-- (For this to work in conjunction with "backupAfter" with Solr 3.6.0, see bug fix https://issues.apache.org/jira/browse/SOLR-3361 )-->




<str name="maxNumberOfBackups">1</str>




</requestHandler>  


  

