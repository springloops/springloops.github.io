---
author: springloops
comments: true
date: 2012-10-30 04:11:40+00:00
layout: post
link: https://springloops.wordpress.com/2012/10/30/cassandra-%ec%84%a4%ec%b9%98%ec%8b%9c-%eb%ac%b8%ec%a0%9c%ec%a0%90-%ed%95%b4%ea%b2%b0/
slug: cassandra-%ec%84%a4%ec%b9%98%ec%8b%9c-%eb%ac%b8%ec%a0%9c%ec%a0%90-%ed%95%b4%ea%b2%b0
title: cassandra 설치시 문제점 해결
wordpress_id: 96
categories:
- Open Source/Cassandra
tags:
- cassandra
- cassandra troubleshooting
- cassandra warning
- 카산드라
- 카산드라 문제
- 카산드라 설치 문제
- 카산드라 해결
- 카산드라 워닝
---

이번에 cassandra 를 설치하면서 발생한 Warning 에 대한 해결 방법을 정리해 봤습니다.

  


설치후 실행시 다음과 같은 warning 메시지가 났을 경우에 대해서...

  


**1) JNA not found. Native methods will be disabled.**

  


**원인** : cassandra의 실행 CLASSPATH 에 JNA 관련 lib가 존재하지 않아 발생하는 알람이다.

  


[JNA](https://github.com/twall/jna) (Java Native Access) 란 ?

  


java는 JVM 위에서 돌아가는 프로그램언어로 OS 레벨의 최적화될 수 없다.

  


이를 사용하기 위해 JNI (Java Native Interface)가 존재 하지만, JNI를 사용하기 위해선

  


c언어를 사용할 경우 .h, .c, 인터페이스 등 복잡한 구조를 생성해야 하는데,

  


이 복잡한 구조를 사용하지 않고 쉽게 가져갈수 있도록 하는 새로운 API 로 보면 되겠다.

  


JNA를 직접 사용해본건 아니라 여기까지만...

  


  


cassandra 에서는 java 로 만들어져 있고, 내부적으로 OS 단의 자원을 사용하기 위해 JNA를 이용해 구현 되어있다.

  


두가지 기능을 사용한다고 알려져 있는데

  


첫째로, [mlockall](http://www.freebsd.org/cgi/man.cgi?query=mlockall)

스와핑 영역의 데이터에 대해 lock을 걸어 자주 엑세스 되지않도록 하는 기능

  


두번째, 스냅샷 성능 향상을 위한 하드링크

  


라고 하는데... 자세한건 더 공부 해보고...ㅠㅠ

  


**해결 방법**

  


JNA를 설치 하여 cassandra/lib 아래 복사하거나 소프트 링크를 건다.

  


패키지 설치의 경우 : sudo apt-get install libjna-java 

직접 다운로드 설치의 경우 : [https://github.com/twall/jna](https://github.com/twall/jna)

jna.jar, platform.jar 를 다운 받아 이동 시킨다.

  


[http://journal.paul.querna.org/articles/2010/11/11/enabling-jna-in-cassandra/](http://journal.paul.querna.org/articles/2010/11/11/enabling-jna-in-cassandra/)

  


  


**2) Unable to lock JVM memory (ENOMEM). This can result in part of the JVM being swapped out, especially with mmapped I/O enabled. Increase RLIMIT_MEMLOCK or run Cassandra as root.**

  












**원인** : JNA 를 사용하도록 하고 root 가 아닌 사용자로 실행할 경우 메모리 락제한이 걸림

  


**해결 방법** : 메모리 락 제한을 풀어줌... (리눅스는 메모리도 파일..)

  


  


/etc/security/limits.conf 의 파일에 다음과 같이 세팅한다.
    
    * soft nofile 32768
    * hard nofile 32768
    root soft nofile 32768
    root hard nofile 32768
    * soft memlock unlimited
    * hard memlock unlimited
    root soft memlock unlimited
    root hard memlock unlimited
    * soft as unlimited
    * hard as unlimited
    root soft as unlimited
    root hard as unlimited
    

  


[http://www.datastax.com/docs/1.1/troubleshooting/index](http://www.datastax.com/docs/1.1/troubleshooting/index)



















  


  

