---
author: springloops
comments: true
date: 2012-09-20 05:55:02+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/20/solr-solr-%ed%8a%b9%ec%a0%95-%eb%ac%b8%ec%84%9c-%ec%83%81%eb%8b%a8%ec%97%90-%eb%85%b8%ec%b6%9c-%ec%8b%9c%ed%82%a4%ea%b8%b0-qeuryelevationcomponent/
slug: solr-solr-%ed%8a%b9%ec%a0%95-%eb%ac%b8%ec%84%9c-%ec%83%81%eb%8b%a8%ec%97%90-%eb%85%b8%ec%b6%9c-%ec%8b%9c%ed%82%a4%ea%b8%b0-qeuryelevationcomponent
title: '[solr] solr 특정 문서 상단에 노출 시키기 - QeuryElevationComponent'
wordpress_id: 48
categories:
- Open Source/Apache Lucene/Solr
tags:
- Elevation
- 특정 문서
- 올리기
- QueryElevationComponent
- solr
- solr wiki
- sort
- Wiki
---

**QueryElevationComponent**

  


  


solr 정렬관련해서 찾다가 알게된 기능입니다.

사용해보진 않았지만, 문서만 읽어 보고 올려봅니다.

  
  


  


기본적인 개념은 이렇습니다.

  


solrconfig.xml 에 QeuryElevationComponent 와 elevate RequestHandler를 등록합니다.
    
    <span style="color:rgb(0,0,0);">  <searchComponent name="elevator" class="org.apache.solr.handler.component.QueryElevationComponent" >
        <str name="queryFieldType">string</str>
        <str name="config-file">elevate.xml</str>
      </searchComponent>
    
      <requestHandler name="/elevate" class="solr.SearchHandler">
        <lst name="defaults">
          <str name="echoParams">explicit</str>
        </lst>
        <arr name="last-components">
          <str>elevator</str>
        </arr>
      </requestHandler></span>

  


  


elevate 설정 파일을 만듭니다. (ex elevate.xml)
    
    <span style="color:rgb(0,0,0);"><elevate>
    
     <query text="AAA">
      <doc id="A" />
      <doc id="B" />
     </query>
    
     <query text="ipod">
      <doc id="A" />
    
      <!-- you can optionally exclude documents from a query result -->
      <doc id="B" exclude="true" />
     </query>
    
    </elevate></span>

  


  


elevate RequestHandler로 호출을 할때

  


elevate.xml에 등록된 키워드의 검색일 경우 (ex AAA or ipod)

  


<doc id="A"> 라고 지정되어 있으니, id field에 A가 포함된 녀석이

  


검색의 상단으로 출력된다는 것이다.

  


  


  


field의 등록된 값에 따라 RDB의 GROUP BY 처럼 쓸수도 있을것 같고....

  


단순히 해당 키워드만 위로 올려주고 싶을때

  


(예: 독도 검색시 대한민국 관련 문서를 올려줄 경우..^^)

  


쓸 수 있겠습니다.

  


  


* 먼저 올린 [data import component](http://somethingbook.tistory.com/entry/solr-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%EC%9D%98-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%A5%BC-indexing-%ED%95%98%EA%B8%B0) 처럼 설정 파일을 (ex : elevate.xml) re-load 해주는 기능이 있다면

  


검색 서비스를 운영하는 관리자 측에서 이벤트 성 검색을 처리 할 수도 있을것 같은데, 아직은 없어 보이네요.

  


아무튼 Solr 는 참 멋진 패키지인듯 합니다.

  


  


참고

[http://wiki.apache.org/solr/QueryElevationComponent](http://wiki.apache.org/solr/QueryElevationComponent)
