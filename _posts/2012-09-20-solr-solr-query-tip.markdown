---
author: springloops
comments: true
date: 2012-09-20 05:32:17+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/20/solr-solr-query-tip/
slug: solr-solr-query-tip
title: '[solr] solr query tip'
wordpress_id: 46
categories:
- Open Source/Apache Lucene/Solr
tags:
- fq
- function query
- sola admin
- solr
- solr tip
- sort
- TIP
---

solr 를 이용해서 검색 할 때 sorting 이라던지, solr가 제공해주는 function query 를 사용할 경우 잠깐 삽질한 내용을 기록함.

  


뭐 진짜 간단한 팁임.

  


쿼리 스트링을 직접 쳤다면 발생하지 않았겠지만, solr의 admin 기능을 이용해서 테스트 할때,

추가 기능을 입력하는 구분자 &로 쿼리 영역에 작성했었다

하지만 admin 페이지의 쿼리는 모든 문자를 검색어로 받아들이는 것이다!!!

  


디버깅 모드를 켜놓고 작업을 해보면 확인가능함.

  


따라서 해당 추가 쿼리는 주소창에 직접 작성해야 가능했다.

  


  


예로 sorting을 할경우

  


q=title:apple&sort=created_at+desc

  


sort 부분을 admin 페이지에서 입력하면 검색어로 인식해버린다..

sort 부분을 검색 결과 페이지의 주소창에 추가해서 재검색 하면 결과가 나온다.

  


  


  


왜 안되나 해서 찾아본 문서만 잔득 쌓였다.

덕분에 많은 기능을 알게 됐구먼..ㅎㅎ
