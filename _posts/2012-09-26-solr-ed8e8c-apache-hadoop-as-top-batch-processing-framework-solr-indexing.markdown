---
author: springloops
comments: true
date: 2012-09-26 01:46:52+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/26/solr-%ed%8e%8c-apache-hadoop-as-top-batch-processing-framework-solr-indexing/
slug: solr-%ed%8e%8c-apache-hadoop-as-top-batch-processing-framework-solr-indexing
title: '[Solr 펌] Apache Hadoop as top batch processing framework / Solr indexing'
wordpress_id: 58
categories:
- Open Source/Apache Lucene/Solr
tags:
- hadoop
- indexing
- integration
- solr
- solr + hadoop
- solr indexing
---

Hadoop과 Solr를 어떻게 연결 시켜야 하는지 몇일을 고민한거 같다.

  


Hadoop을 잘 모르는 상태여서 더 고민을 한거 같은데,

  


아래 블로그를 보고 좀 가닥이 잡힌듯 하다.

  


[http://itsitspace.blogspot.fr/2012/07/apache-hadoop-as-top-batch-processing.html](http://itsitspace.blogspot.fr/2012/07/apache-hadoop-as-top-batch-processing.html)

"Hadoop job can be used to upload data to Solr using SolrJ Java driver. Apache Solr search engine stores indexes on separate servers. A separate front end application can provide search interface directly from Apache Solr using SolrJ java driver to interface to Apache Solr. So on one side Hadoop uploads data (or documents) to Apache Solr, and on other side Java front end can provide search interface to Apache Solr."

  


[http://www.likethecolor.com/2010/09/26/using-hadoop-to-create-solr-indexes/](http://www.likethecolor.com/2010/09/26/using-hadoop-to-create-solr-indexes/)

## Can SOLR access the Hadoop file system?

It turns out that in its current state SOLR must read/write to a local file system.

  


어찌보면 당연한 것인데...

  


그럼 Katta는 머지????? 소스를 열어 봐야 겠다..

  


암튼..

  


요즘 정신없이 잼난다.
