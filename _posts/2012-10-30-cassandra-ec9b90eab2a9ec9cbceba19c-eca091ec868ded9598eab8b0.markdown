---
author: springloops
comments: true
date: 2012-10-30 04:54:07+00:00
layout: post
link: https://springloops.wordpress.com/2012/10/30/cassandra-%ec%9b%90%ea%b2%a9%ec%9c%bc%eb%a1%9c-%ec%a0%91%ec%86%8d%ed%95%98%ea%b8%b0/
slug: cassandra-%ec%9b%90%ea%b2%a9%ec%9c%bc%eb%a1%9c-%ec%a0%91%ec%86%8d%ed%95%98%ea%b8%b0
title: cassandra 원격으로 접속하기~
wordpress_id: 70
categories:
- Open Source/Cassandra
tags:
- cassandra remote client
- 카산드라 원격접속
---

뭐... 카산드라를 원격지의 서버에 설치하고 로컬에서 원격지로 접속할 경우

  


기본 설정으로 실행시키면 접속이 되지 않습니다.

  


cassandra.yaml 의  rpc-address 의 값을 서버의 ip또는 0.0.0.0 으로 세팅해야합니다.

  


나중에 cassandra 설정파일에 대한 것도 정리를 하겠지만...

  


등록해놓은 ip로 listen 하겠다는 의미입니다..

  


  


  


  


  

