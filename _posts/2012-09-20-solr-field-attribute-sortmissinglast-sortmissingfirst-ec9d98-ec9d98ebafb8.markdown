---
author: springloops
comments: true
date: 2012-09-20 02:27:46+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/20/solr-field-attribute-sortmissinglast-sortmissingfirst-%ec%9d%98-%ec%9d%98%eb%af%b8/
slug: solr-field-attribute-sortmissinglast-sortmissingfirst-%ec%9d%98-%ec%9d%98%eb%af%b8
title: '[Solr] Field Attribute : sortMissingLast, sortMissingFirst 의 의미'
wordpress_id: 44
categories:
- Open Source/Apache Lucene/Solr
tags:
- solr
- sort
- sortMissingFirst
- sortMissingLast
---

Solr 에서 Document Field를 정의 할때 속성으로 sortMissingLast, sortMissingFirst 가 존재 합니다.

  


sortMissingLast = true 일 경우

  


sortMissingLast 속성이 정의된 컬럼의 데이터가 없을 경우 해당 Document를 Document 목록의 최 하단에 출력합니다.

  


sortMissingFirst = true 일 경우

  


sortMissingFirst 속성이 정의된 컬럼의 데이터가 없을 경우 해당 Document를 Document 목록의 최 상단에 출력합니다.

  


  


참고

[http://lucidworks.lucidimagination.com/display/solr/Solr+Field+Types](http://lucidworks.lucidimagination.com/display/solr/Solr+Field+Types)

[http://lucene.472066.n3.nabble.com/about-sortMissingLast-and-sortMissingFirst-td473881.html](http://lucene.472066.n3.nabble.com/about-sortMissingLast-and-sortMissingFirst-td473881.html)
