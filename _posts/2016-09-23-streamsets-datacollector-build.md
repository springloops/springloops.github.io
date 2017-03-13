---
author: springloops
comments: true
date: 2016-09-23
layout: post
title: 'Streamsets DataCollector Source Build'
categories:
- OPEN SOURCE/Streamsets DataCollector
tags:
- Streamsets
- opensource
- DataCollector
- Streamsets DataCollector
- build
---

## MAVEN 을 이용한 Streamsets DataCollector Build 방법.

1. DataCollector 와 맞는 DataCollector-api 를 빌드해야한다.

```
git clone https://github.com/streamsets/datacollector-api.git

mvn clean install -DskipTests

```

2. bower 와 Grunt 를 global 설치 해야한다.

```
npm install -g bower

npm install -g grunt-cli
```

~~3. root-proto 안에 있는 pom.xml 수정 (datacollector-2.1.0.0-SNAPSHOT 현재)~~


~~참조하시라 : [sdc-user google groups](https://groups.google.com/a/streamsets.com/forum/#!topic/sdc-user/RWQO3p0SyzA)~~

해당 이슈 해결됨. [https://issues.streamsets.com/browse/SDC-3794](https://issues.streamsets.com/browse/SDC-3794)

```
<testcontainers.version>1.1.6-SNAPSHOT</testcontainers.version>

	위 1.1.6-SNAPSHOT 을 다음과 같이 1.1.5 로 변경한다.

<testcontainers.version>1.1.5</testcontainers.version>
```

4. DataCollector install

```
git clone https://github.com/streamsets/datacollector.git

mvn install -DskipTests <-P profile1,profile2>

```

5. DataCollector package

```
mvn package -Pdist,ui -DskipTests <-P profile1,profile2>
```

6. Run

```
datacollector/dist/target/streamsets-datacollector-2.1.0.0-SNAPSHOT/streamsets-datacollector-2.1.0.0-SNAPSHOT/bin/streamsets dc
```

코드 수정 중, maven repogitory 문제들과, bower, grunt 관련 문제로 빌드가 되지 않아 고생한 것들 정리. 

