---
author: springloops
comments: true
date: 2012-09-26 05:43:29+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/26/solr-%ed%8e%8c-using-hadoop-to-create-solr-indexes/
slug: solr-%ed%8e%8c-using-hadoop-to-create-solr-indexes
title: '[Solr 펌] Using Hadoop to Create SOLR Indexes'
wordpress_id: 60
categories:
- Open Source/Apache Lucene/Solr
tags:
- hadoop
- indexing
- solr
- solr + hadoop
---

# Using Hadoop to Create SOLR Indexes

  


  


Hadoop과 Solr를 어떻게 연결 시켜야 하는지 몇일을 고민한거 같다.

  


Hadoop을 잘 모르는 상태여서 더 고민을 한거 같은데,

  


아래 블로그를 보고 좀 가닥이 잡힌듯 하다.

  


[http://www.likethecolor.com/2010/09/26/using-hadoop-to-create-solr-indexes/](http://www.likethecolor.com/2010/09/26/using-hadoop-to-create-solr-indexes/)

## Can SOLR access the Hadoop file system?

It turns out that in its current state SOLR must read/write to a local file system. 
