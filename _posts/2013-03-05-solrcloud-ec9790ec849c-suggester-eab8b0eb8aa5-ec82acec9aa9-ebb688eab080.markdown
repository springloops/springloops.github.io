---
author: springloops
comments: true
date: 2013-03-05 09:09:49+00:00
layout: post
link: https://springloops.wordpress.com/2013/03/05/solrcloud-%ec%97%90%ec%84%9c-suggester-%ea%b8%b0%eb%8a%a5-%ec%82%ac%ec%9a%a9-%eb%b6%88%ea%b0%80/
slug: solrcloud-%ec%97%90%ec%84%9c-suggester-%ea%b8%b0%eb%8a%a5-%ec%82%ac%ec%9a%a9-%eb%b6%88%ea%b0%80
title: solrcloud 에서 suggester 기능 사용 불가.
wordpress_id: 86
categories:
- Open Source/Apache Lucene/Solr
tags:
- autocomplete
- solr
- solr 4.0
- solrcloud
- suggeest
- suggester
---

solrcloud 에서 suggester 가 동작을 하지 않더라구요.

  


돌려보려고 이런 저런 설정을 해보았으나 동작하지 않았구요.

  


이상하게도 solr 한대로 구성하면 잘되는게 solrcloud로 구성을 하면 동작을 하지 않더군요.

  


하루종일 삽질을 하다가 LucidWork에서 설명해 놓은 글을 보았습니다.

  


인덱스가 분산되어 있는 경우 suggester가 동작하지 않는다는군요...

  


이상한건,

  


suggester는 SpellCheckerComponent 를 기반으로 동작하는 것으로 알고 있는데,

  


spellChecker는 동작을 한다는 것입니다;;;;

  


으헐헐;;

  


  


<table style="font-size:16px;line-height:21.328125px;color:rgb(51,51,51);clear:left;background-image:none;background-color:rgb(248,211,209);border:1px solid rgb(223,152,152);padding:10px;width:1232px;border-top-left-radius:5px;border-top-right-radius:5px;border-bottom-right-radius:5px;border-bottom-left-radius:5px;font-family:'Open Sans', Tahoma, Arial, Helvetica, sans-serif;" class="warningMacro" ><tbody ><tr style="font-size:12pt;line-height:16pt;background-image:none;" >
<td style="font-size:12pt;line-height:16pt;background-image:none;padding:0;border:none;" valign="top" >![](http://docs.lucidworks.com/images/icons/emoticons/forbidden.gif)
</td>
<td style="font-size:12pt;background-image:none;padding:0;border:none;" >The following LucidWorks features may encounter significant problems when working in SolrCloud mode:

  * Click Scoring cannot be used in SolrCloud mode at this time.
  * Auto-complete-related suggestions should be pulled from a single index node if auto-complete is enabled; distributed auto-complete indexing is not available.
  * De-duplication does not work in SolrCloud due to a bug in Solr ([SOLR-3473](https://issues.apache.org/jira/browse/SOLR-3473)).
  * SSL does not work with SolrCloud due to a bug in Solr ([SOLR-3854](https://issues.apache.org/jira/browse/SOLR-3854))
</td></tr></tbody></table>

  


LucidWork : [http://docs.lucidworks.com/display/lweug/Using+SolrCloud+in+LucidWorks](http://docs.lucidworks.com/display/lweug/Using+SolrCloud+in+LucidWorks)
