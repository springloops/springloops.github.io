---
author: springloops
comments: true
date: 2016-10-11
layout: post
title: '다시 보는 Java : FileChannel transferTo()'
categories:
- Java/FileChannel transferTo
tags:
- java
- 발 번역
- look again
- FileChannel transferTo
- transferTo
- NIO
---

2008년 2월에 포스팅된 글인 IBM developerWorks의 Efficient data transfer through zero copy

Sathish Palaniappan with Pramod Nagaraja 글을 읽기 위해 발 번역 (구글 번역이와 직역 의역 막 섞음) 했습니다. 

옛날에 developerWorks 한글 문서 정말 많이 읽었는데... 많이 늦었습니다.

사실 번역이라기 보단, 영어 독해 능력이 떨어져 원문을 차근차근 읽기 위해서 작성한 글 입니다.

언제나 그렇듯, 원문을 참고하세요. [Efficient data transfer through zero copy](http://www.ibm.com/developerworks/library/j-zerocopy/)

>
- 다시 보는 Java [Introduction](/archivers/look-again-java-1-intro)
- 다시 보는 Java [Socket Direct Protocol](/archivers/look-again-java-2-sockets-direct-protocol)
- 다시 보는 Java [NIO Channel](/archivers/look-again-java-3-NIO-Channel)
- 다시 보는 Java [FileChannel transferTo()](/archivers/look-again-java-4-FileChannel-transferTo())








>
- 다시 보는 Java [Introduction](/archivers/look-again-java-1-intro)
- 다시 보는 Java [Socket Direct Protocol](/archivers/look-again-java-2-sockets-direct-protocol)
- 다시 보는 Java [NIO Channel](/archivers/look-again-java-3-NIO-Channel)
- 다시 보는 Java [FileChannel transferTo()](/archivers/look-again-java-4-FileChannel-transferTo())
