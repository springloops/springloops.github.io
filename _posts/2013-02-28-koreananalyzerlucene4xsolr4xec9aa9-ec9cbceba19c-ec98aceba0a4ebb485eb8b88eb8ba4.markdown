---
author: springloops
comments: true
date: 2013-02-28 03:03:38+00:00
layout: post
link: https://springloops.wordpress.com/2013/02/28/koreananalyzerlucene4xsolr4x%ec%9a%a9-%ec%9c%bc%eb%a1%9c-%ec%98%ac%eb%a0%a4%eb%b4%85%eb%8b%88%eb%8b%a4/
slug: koreananalyzerlucene4xsolr4x%ec%9a%a9-%ec%9c%bc%eb%a1%9c-%ec%98%ac%eb%a0%a4%eb%b4%85%eb%8b%88%eb%8b%a4
title: KoreanAnalyzer-Lucene4x/Solr4x용 으로 올려봅니다.
wordpress_id: 85
categories:
- Open Source/Apache Lucene/Solr
tags:
- '4.0'
- koreanAnalyzer
- koreanAnalyzer 4.0
- lucene
- lucene 4.0
- lucene 4.0 한글 analyzer
- lucene 4.1
- solr
- solr 4.0
- solr 4.1
- solr 한글 analyzer
---

  


이번에 Solrcloud를 구성할 일이 있었어요.

최신 koreanAnalyzer가 solr4 이상에서 작동하지 않아

KoreanAnalyzer도 solr4에서 동작하도록 바꿔보았습니다.

  


우선 제가 형태소 분석에 대한 지식이 부족하여 실제 분석 부분은 KoreanAnalyzer 3x 그대로 사용하도록 했고,

인터페이스 부분만 바꿔 주었습니다.

  


cvs에서 최신 kr.analyzer.3x 소스를 이용해서 변환했습니다.

  


주된 변경 작업은 다음과 같습니다.

  


- Filter, Tokenizer 의 factory들이 4.x 부터 solr 에서 lucene 으로 이동되었습니다. 해당 사항 적용했습니다.

  


- AttributeWrapper 클래스 삭제. AttributeWrapper 가 멤버변수로 TermAttribute 클래스를 가지고 있습니다만,

4.0에서 TermAttribute가 포함되어 있지 않습니다.

KoreanAnalyzer 내부적으로 AttributeWrapper 클래스를 사용하는 곳이 없고, AttributeWrapper 클래스가 단순 bean인것을

확인하고 삭제 하였습니다.

  
- KoreanFilter 클래스 setTermBufferByQueue() 메소드154 line

offsetAtt.setOffset(tokStart+pos, tokStart + pos + term.length());

구문을 주석처리 하였습니다.

KoreanFilter 클래스를 디버깅 해보니 한문의 경우 한글로 변환하여 curTermBuffer에 추가 하고,

현재 텀의 인덱스를 구하는데 원문 텀에 없어 -1이 리턴되고 setOffset() 메소드의 인자로 -1이 들어갑니다.

OffsetAttribute 의 setOffset 메소드가 -1을 인자로 받으면 Exception을 발생하도록 api 가 변경되었고,

api에선 setOffset 메소드는 최초 한번 설정되고 다시 변경되지않기를 바라고 있더라구요.

  


- KoreanAnalyzer 클래스의 replaceInvalidAcronym 변수에 의한 분기로직을 주석처리하였습니다.

replaceInvalidAcronym 기능은 lucene_24버전 부터 제공하고, Lucene_41 API 가 지원하는 버전은 Lucene_30 부터라,

Lucene_41을 쓴다면 해당 로직은 의미가 없어지기때문입니다.

  


- 추후 버전 변경시 로직의 분기 인터페이스 수정 없이 적용할 수 있도록 각 KoreanFilter, KoreanTokenizer 생성자에 version을 인자로 추가 하였습니다.

  


- kr.morph 프로젝트의 빌드패스에서 lucene3x 부분을 제거 했습니다.

solr4x 대에 넣을경우, lucene api 버전이 달라 Exception이 발생하더라구요.

  


KoreanAnalyzer cvs에 올려있는 test 페키지의 junit 코드로 돌려본 결과 모두 성공하였고,

  


KoreanAnalyzerTest 의 토큰 결과 값도 3.x 버전과 동일함을 확인하였기에 올려봅니다.

  


마지막으로 Factory의 페키지가 변경되었기에, solr의  schema.xml 에서도 경로 명을 아래와 같이 변경해주셔야 합니다.

  


<fieldType name="text_ko" class="solr.TextField" positionIncrementGap="100">

<analyzer type="index">

<tokenizer class="org.apache.lucene.analysis.kr.KoreanTokenizerFactory"/>

<filter class="org.apache.lucene.analysis.kr.KoreanFilterFactory"/>

.....

</analyzer>

<analyzer type="query">

<tokenizer class="org.apache.lucene.analysis.kr.KoreanTokenizerFactory"/>

<filter class="org.apache.lucene.analysis.kr.KoreanFilterFactory"/>

.....

</analyzer>

</fieldType>

  


[](http://localhost/wordpress/wp-content/uploads/1/cfile7.uf.154F1335512EC9041CA664.jar)cfile7.uf.154F1335512EC9041CA664.jar

[](http://localhost/wordpress/wp-content/uploads/1/cfile29.uf.1231EC35512EC904308D0A.jar)cfile29.uf.1231EC35512EC904308D0A.jar

[](http://localhost/wordpress/wp-content/uploads/1/cfile22.uf.19414235512EC9051E2A35.jar)cfile22.uf.19414235512EC9051E2A35.jar

  


  

