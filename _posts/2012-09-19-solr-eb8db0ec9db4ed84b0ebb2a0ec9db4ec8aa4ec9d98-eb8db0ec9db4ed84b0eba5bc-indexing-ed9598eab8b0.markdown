---
author: springloops
comments: true
date: 2012-09-19 07:10:45+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/19/solr-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4%ec%9d%98-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%a5%bc-indexing-%ed%95%98%ea%b8%b0/
slug: solr-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4%ec%9d%98-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%a5%bc-indexing-%ed%95%98%ea%b8%b0
title: '[solr] 데이터베이스의 데이터를 indexing 하기'
wordpress_id: 43
categories:
- Open Source/Apache Lucene/Solr
tags:
- data import
- Database
- db indexing
- Import
- index
- indexing
- lucene
- solr
---

**Solr (현재 3.6버전) 가 Tomcat(현재 6 버전)에 설치되어 있다는 전제로 진행합니다.**

  


Solr 를 Tomcat에 설치 하고, 인덱싱을 해야하는데,

인덱싱을 하기 위해 데이터를 XML 형태로 Parsing 해야합니다.

  


RDB에 이미 데이터가 존재 하는경우 데이터를 XML 로 parsing하지 않고

DB에서 바로 인덱싱 하기 위한 방법을 정리해봅니다.

  


  


1. solr 를 다운로드  받고 압축을 푼 디렉토리를 확인하면 apache-solr-3.6.1/dist/ 디렉토리가 있습니다.

  


예) apache-solr-3.6.1/dist

  


해당 디렉토리 안에 libs 안의 관련 jar 파일들을 복사해서 Solr 컨텍스트의 WEB-INF/lib 안에 넣습니다.

apache-solr-dataimporthandler-3.6.1.jar,

apache-solr-dataimporthandler-extras-3.6.1.jar

  


2. 다음으로 solrconfig.xml 을 수정해야합니다.

아래 나오는 테그를 입력합니다.

  


<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">

<lst name="defaults">

<str name="config">db-data-config.xml</str> <!-- db 정보가 담긴 xml 의 위치 -->

</lst>

</requestHandler>

  


solrconfig.xml 샘플을 복사해서 사용할 경우

  


파일 안의 아래부분을 주석 처리 합니다.

  


<lib dir="../../dist/" regex="apache-solr-dataimporthandler-d.*.jar" />

<lib dir="../../contrib/dataimporthandler/lib/" regex=".*.jar" />

  


위 설정은 해당 jar 의 위치를 설정 하는 것인데, jar 파일을 컨텍스트로 복사 했기때문에 필요 없습니다.

또는 저 위치를 다른곳에 두었다면 경로를 정의 해주면 됩니다.

  


3. 이제 solrconfig.xml 에 정의한 db 정보에 대한 파일을 만들어야 합니다. (db-data-config.xml)

  


<?xml version="1.0" encoding="UTF-8" ?>

<dataConfig>

<dataSource driver="com.mysql.jdbc.Driver" url="jdbc:mysql://211.172.241.88:3306/foword?zeroDateTimeBehavior=convertToNull&amp;characterEncoding=UTF-8&amp;characterSetResults=UTF-8&amp;useEncoding=true&amp;emulateLocators=true" user="foword" password="20090810ab" />

<document>

<entity name="document" query="SELECT id, data_url, title, description, original_content, created_at FROM test limit 1000">

<field column="id" name="id" />

<field column="data_url" name="data_url" />

<field column="title" name="title" />

<field column="description" name="desc" />

<field column="original_content" name="content" />

<field column="created_at" name="created_at" />

  


<!-- RDB의 subquery 처럼 정의한 entity name 의 field column으로 추가 쿼리 한뒤 index 에 추가로 field 를 만들수 있습니다. -->

<entity name="document_sub" query="SELECT document_name FROM test_sub WHERE doc_id=${document.id}">

<field column="document_name" name="doc_name" />

</entity>

  


</entity>

</document>

</dataConfig>

  


위와 같이 정리 합니다.

  


4. 마지막으로 solr 의 index를 정의 하는 schema.xml 을 수정해야합니다.

schema.xml 파일을 열고 db-data-config.xml 에서 정의한대로 <fields> 영역에 각각 <field> 엘리먼트로 채워 줍니다.

  


<field name="id" type="string" indexed="true" stored="true" required="true"/>

<field name="data_url" type="string" indexed="true" stored="true" required="true"/>

<field name="title" type="text_general" indexed="true" stored="true"/>

<field name="desc" type="text_general" indexed="true" stored="true"/>

<field name="content" type="text_general" indexed="true" stored="true"/>

<field name="created_at" type="date" indexed="true" stored="true"/>

  


  


물론... jdbc  driver 는 tomcat/{context}/WEB-INF/lib 에 넣어야 겠죠...

  


  


아래는 인덱스를 발동시키는 사용법입니다.

  


http://localhost[:port]/[context/]dataimport?command=full-import

  


command 에 대해서..

  


full-import : 기존의 인덱싱을 삭제 하고 전체 데이터를 indexing 합니다

delta-import : 기존의 인덱싱을 두고 추가로 indexing  합니다. 전체 인덱스는 증가하겠죠..

reload-config : 설정 파일을 리로딩합니다.

abort : indexing 작업을 중단합니다.

  


  


참고 :

  


[http://wiki.apache.org/solr/DataImportHandler](http://wiki.apache.org/solr/DataImportHandler)

Apache Solr 3.1 Cookbook - How to properly configure Data Import Handler with JDBC

  

