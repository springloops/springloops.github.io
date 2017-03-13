---
author: springloops
comments: true
date: 2014-11-05 07:50:32+00:00
layout: post
link: https://springloops.wordpress.com/2014/11/05/tomat7-configuration/
slug: tomat7-configuration
title: tomat7 - configuration
wordpress_id: 105
categories:
- Open Source/Apache Tomcat
tags:
- tomcat
- tomcat configuration
- tomcat7
---

**설정 파일들**  





  * catalina.policy



    * 보안 정책 권한 설정 파일


    * 톰캣 실행시 -security 옵션을 사용하면, 보안 정책 적용



  * catalina.property



    * 서버를 시작할 때, 검색하는 서버, 공유 로더, jar 등의 공유 정의를 설정



  * server.xml



    * 톰캣의 IP 주소, 포트, 가상 호스트, 컨텍스트 경로 등 정의



  * tomcat-users.xml



    * 역할에 기반한 인증, 승인 정의


    * 데이터베이스의 사용자/암호/역할을 이용한 인증과 컨테이너로 관리되는 보안 구현에 사용



  * logging.properties



    * 로깅 설정



  * web.xml



    * 모든 웹 어플리케이션의 기본 값 정의



  * context.xml



    * 어플리케이션을 실행할 때 이 파일의 내용을 로드


    * 세션 저장, 코멧 (comet) 연결 추적 등





  

**DataSource 설정**  





  * JDBC : Java DataBase Connectivity



    * 자바 기반 데이터베이스 접근 기술


    * 클라이언트가 데이터베이스 서버에 접근할 수 있도록 API 제공


    * 관계형 데이터베이스에 맞게 설계


    * 질의 / 갱신 기능 제공



  * JNDI : Java Naming and Directory Interface



    * 네이밍, 디렉토리 기능을 제공하는 자바 플랫폼 API



  * DataSource



    * JDBC API를 이용해 관계형 데이터베이스에 접근할 때 사용하는 자바 객체


    * JNDI 와 통합되고, 데이터소스 객체를 JNDI 네이밍 서비스에 등록해서 사용


    * 데이터소스는 애플리케이션 자신만 접근 가능





  





  * ******DataSource 설정 방법**


  * DataSource 에 접근하기 위한 설정 방법이 다양하게 있지만, 그중 한가지만 작성하면 다음과 같다.



    1. server.xml  에 전역 데이터소스 설정 ( 작성 법은 인터넷 뒤지면 많이 나오니까… )


    2. 각 웹 어플리케이션의 설정 파일  ( web.xml ) 에 <resource-ref> 등록  ( 작성 법은 인터넷 뒤지면 많이 나오니까… )


    3. 데이터베이스별 제공되는 jdbc connector.jar 를 어플리케이션이 참조 할 수 있는 위치에 복사


    4. jndi 를 통해 접속할 수 있는 코드 작성 ( 작성 법은 인터넷 뒤지면 많이 나오니까… )





  

**톰캣 관리자 ( Tomcat Manager ) **  





  * 기능



    * 자세한건 문서 참고, 설정 방법이 너무 많다;;;


    * 책에 있는 방법을 별로 선호하지 않는다.. 난;;


    * 원격으로 새 애플리케이션 배포


    * idle 세션 청소


    * 컨테이너 재시작 없이 애플리케이션 배포 철회


    * 메모리 누수 분석


    * JVM 상태


    * 서버 상태





  

