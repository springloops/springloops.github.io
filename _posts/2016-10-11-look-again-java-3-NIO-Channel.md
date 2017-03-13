---
author: springloops
comments: true
date: 2016-10-11
layout: post
title: '다시 보는 Java : NIO Channel'
categories:
- Java/FileChannel transferTo
tags:
- java
- 발 번역
- look again
- NIO Channel
- Channel
- NIO
---

# Java NIO Channel

>
- 다시 보는 Java [Introduction](/archivers/look-again-java-1-intro)
- 다시 보는 Java [Socket Direct Protocol](/archivers/look-again-java-2-sockets-direct-protocol)
- 다시 보는 Java [NIO Channel](/archivers/look-again-java-3-NIO-Channel)
- 다시 보는 Java [FileChannel transferTo()](/archivers/look-again-java-4-FileChannel-transferTo())

## Channel in JavaDoc

Channel 이 뭔지 JavaDoc 을 읽음.

[Channel Interface in JavaDoc](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Channel.html)

I/O 작업을 위한 결합.

하드웨어 장치 같은 엔티티, 파일, 네트워크 소켓, 또는 프로그램 컴포넌트들에 커넥션을 여는 것을 나타내는 채널은 하나 이상의 별개의 I/O 연산을 수행할 수 있다.

채널은 open 또는 close 중 하나다. 채널은 생성시 open 되고, 한번 닫히면서 마감한다. 한번 채널이 닫히면, 그것을 바탕으로 실행되는 I/O 조작은 CloasedChannelException 이 발생한다. 채널이 열려있는지 여부를 isOpen 메소드를 호출하여 테스트 할 수 있다.

채널은, 일반적으로, 인터페이스와 Channel 인터페이스를 구현하고 확장한 클래스들의 규격에서 설명 된 바와 같이, 멀티쓰레드 액세스를 안전하게 사용하기 위한 것이다.


## Package java.nio.channels Descriptio

[Package java.nio.channels Description](http://docs.oracle.com/javase/7/docs/api/java/nio/channels/package-summary.html#package_description)

하드웨어 장치 같은 엔티티, 파일, 네트워크 소켓, 또는 프로그램 컴포넌트들에 커넥션을 여는 것을 나타내는 채널은 하나 이상의 별개의 I/O 연산을 수행할 수 있다.
예를들어 읽고 쓰기 같은. 
Channel 인터페이스에 명시한 것처럼, Channels는 open 나 closed 둘 다 (asynchronously closeable) 와 interruptible 하다.

Channel 인터페이스는 몇몇의 인터페이스로부터 확장된다.





>
- 다시 보는 Java [Introduction](/archivers/look-again-java-1-intro)
- 다시 보는 Java [Socket Direct Protocol](/archivers/look-again-java-2-sockets-direct-protocol)
- 다시 보는 Java [NIO Channel](/archivers/look-again-java-3-NIO-Channel)
- 다시 보는 Java [FileChannel transferTo()](/archivers/look-again-java-4-FileChannel-transferTo())
