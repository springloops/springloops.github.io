---
author: springloops
comments: true
date: 2012-09-27 15:33:51+00:00
layout: post
link: https://springloops.wordpress.com/2012/09/27/katta%ec%9d%98-%ec%9d%b8%eb%8d%b1%ec%8b%b1-%ed%8c%8c%ec%9d%bc%ec%9d%98-%ec%9c%84%ec%b9%98%ec%97%90-%eb%8c%80%ed%95%98%ec%97%ac/
slug: katta%ec%9d%98-%ec%9d%b8%eb%8d%b1%ec%8b%b1-%ed%8c%8c%ec%9d%bc%ec%9d%98-%ec%9c%84%ec%b9%98%ec%97%90-%eb%8c%80%ed%95%98%ec%97%ac
title: Katta의 인덱싱 파일의 위치에 대하여.
wordpress_id: 61
categories:
- Open Source/katta
tags:
- 루씬
- hadoop
- index
- indexing
- 쏠라
- 하둡
- 카타
- katta
- katta hadoop
- lucene
- lucene + hadoop
- solr
- solr + hadoop
---

이번주 동안 Solr 와 Hadoop 간 연동에 대해서 고민중에 있었다.

  


처음 내가 그린 그림은

  


Solr (Lucene)의 인덱스 파일을 HDFS안에 저장하고

  


Solr에서 Direct로 HDFS에 접속해서 인덱스 파일에 접근하는 방식이였다.

  


하지만 많은 검색 결과, Solr와 Hadoop의 결합은..

  


1) Hadoop의 HDFS에 데이터를 저장하고,

2) MapReduce로 index 파일을 생성,

3) Solr 서버의 local로 복사 한다.

  


위와 결론을 얻었고

  


이것은

모든 분산 Solr 서버는 각각 필요한 index 파일을 Local System에

가지고 있어야 한다는 것

이다.  
(replication and shard 모두)

  


지원하지 않는 이유는 아래와 같다.

  


[http://www.jaso.co.kr/119](http://www.jaso.co.kr/119)

1. lucene를 이용하여 index 구성시 hadoop 파일시스템에서 바로 index 파일을 생성하지 못한다. 이유는 hadoop 파일 시스템에서는 RandowmAccessFile 기능/lock 기능 등과 같은 기능이 제공되지 않기 때문이다.  
nutch의 경우 이런 제약으로 인해 local filesystem에서 index 파일을 만든 후 최종 파일을 hadoop 파일시스템으로 upload 한다.

  


[http://www.jaso.co.kr/125](http://www.jaso.co.kr/125)

Hadoop 메일링 리스트에 다음과 같은 내용이 올라왔는데 더그커팅의 답변  
Lucene index 파일을 Hadoop에 저장하면 문제가 있는가?  
Creating Lucene indexes directly in DFS would be pretty slow. Nutch  
creates them locally, then copies them to DFS to avoid this.

  


[[Solr 펌](http://somethingbook.tistory.com/entry/Solr-%ED%8E%8C-Using-Hadoop-to-Create-SOLR-Indexes)[] Using Hadoop to Create SOLR Indexes](http://somethingbook.tistory.com/entry/Solr-%ED%8E%8C-Using-Hadoop-to-Create-SOLR-Indexes)

  


위와 같은 이유로 나는 고민에 빠졌다.

  


Lucene + Hadoop을 결합한 솔루션으로 Katta라는 놈이 있는데,

  


이놈은 Lucene의 Index 파일을 Hadoop에 저장하고 있다는 것이다.

  


그럼 Katta는 어떻게 구현이 되있는 것인가?

  


혹시 HDFS와 File 을 Adapter 패턴으로 엮은 것인가? 하는 궁굼증 때문이였다.

[[Solr 펌](http://somethingbook.tistory.com/entry/Solr-%ED%8E%8C-Apache-Hadoop-as-top-batch-processing-framework-Solr-indexing)[] Apache Hadoop as top batch processing framework / Solr indexing](http://somethingbook.tistory.com/entry/Solr-%ED%8E%8C-Apache-Hadoop-as-top-batch-processing-framework-Solr-indexing)

  


  


해서 Katta 소스를 열어 보았다.

  


**그 결과 Katta 역시 내부적으로 HDFS 에 있는 index 파일을 Local System으로 복사해오는 **

**작업이 있었고, ****결과적으로 Local System의 index 파일을 참조 하는것이였다.**

  


소스코드 (net.sf.katta.node.ShardManager)





















/*

* Loads a shard from the given URI. The uri is handled bye the hadoop file

* system. So all hadoop support file systems can be used, like local hdfs s3

* etc. In case the shard is compressed we also unzip the content. If the

* system property katta.spool.zip.shards is true, the zip file is staged to

* the local disk before being unzipped.

*/

private void installShard(String shardName, String shardPath, File localShardFolder) throws KattaException {

LOG.info("install shard '" + shardName + "' from " + shardPath);

// TODO sg: to fix HADOOP-4422 we try to download the shard 5 times

int maxTries = 5;

for (int i = 0; i < maxTries; i++) {

URI uri;

try {

uri = new URI(shardPath);

FileSystem fileSystem = FileSystem.get(uri, new Configuration());

if (_throttleSemaphore != null) {

fileSystem = new ThrottledFileSystem(fileSystem, _throttleSemaphore);

}

final Path path = new Path(shardPath);

boolean isZip = fileSystem.isFile(path) && shardPath.endsWith(".zip");

  


File shardTmpFolder = new File(localShardFolder.getAbsolutePath() + "_tmp");

try {

FileUtil.deleteFolder(localShardFolder);

FileUtil.deleteFolder(shardTmpFolder);

  


if (isZip) {

FileUtil.unzip(path, shardTmpFolder, fileSystem, System.getProperty("katta.spool.zip.shards", "false")

.equalsIgnoreCase("true"));

} else {

fileSystem.copyToLocalFile(path, new Path(shardTmpFolder.getAbsolutePath()));

}

shardTmpFolder.renameTo(localShardFolder);

} finally {

// Ensure that the tmp folder is deleted on an error

FileUtil.deleteFolder(shardTmpFolder);

}

// Looks like we were successful.

if (i > 0) {

LOG.error("Loaded shard:" + shardPath);

}

return;

} catch (final URISyntaxException e) {

throw new KattaException("Can not parse uri for path: " + shardPath, e);

} catch (final Exception e) {

LOG.error(String.format("Error loading shard: %s (try %d of %d)", shardPath, i, maxTries), e);

if (i >= maxTries - 1) {

throw new KattaException("Can not load shard: " + shardPath, e);

}

}

}

}























































  


위 소스코드에 나온 ShardManager class는

  


Katta 프로그램이 실행 될때 argument로 "**startNode" 명령이 들어 왔을 때**

**  
**

실제 검색 서비스를 제공하는 Shard 프로세스를 띄우는 명령이다.

  


위 코드에서 보면 이 글의 결론이며, 내가 그린 그림을 결정지을 두줄의 코드가 보인다.

  


FileUtil.unzip() 과 fileSystem.copyToLocaFile()이다.

  


FileSystem.copyToLocalFile()  제목 그대로 HDFS의 파일을 LocalSystem으로 복사 하는 코드다.

  


FileUtil.unzip() 역시 내부적으로 FileSystem.copyToLocalFile()을 호출 하고 있다. (net.sf.katta.util.FileUtil)

  


결론은 다음과 같다.

  


그렇다.

  


**Index 파일은 Local에 저장되야 한다.**

  












  


마지막으로 Solr Katta SolrCloud에 대한 잘 정리된 slideshare 문서 하나 펌질하며 마친다. (날 낚은..)

  


[http://www.slideshare.net/chaoyu0513/hic2011-using-hadoop-lucenesolrforlargescalesearch-by-systex](http://www.slideshare.net/chaoyu0513/hic2011-using-hadoop-lucenesolrforlargescalesearch-by-systex)

  


**이제 홀가분한 마음으로 다음을 진행하자.**

**  
**

**  
**
