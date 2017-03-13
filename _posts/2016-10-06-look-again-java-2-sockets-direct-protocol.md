---
author: springloops
comments: true
date: 2016-10-06
layout: post
title: '다시 보는 Java : Socket-Direct-Protocol'
categories:
- Java/Sockets Direct Protocol
tags:
- java
- SDP
- Sockets Direct Protocol
- 발 번역
- look again
---

2013년 8월에 포스팅된 글인 infoq 의 Java 7 Sockets Direct Protocol - Write Once, Run Everywhere... and Run (Some Places) Blazingly 글을 읽기 위해 발 번역 (구글 번역이와 직역 의역 막 섞음) 했습니다. 

정말 많이 늦었습니다.

사실 번역이라기 보단, 영어 독해 능력이 떨어져 원문을 차근차근 읽기 위해서 작성한 글 입니다.

언제나 그렇듯, 원문을 참고하세요. [Java 7 Sockets Direct Protocol – Write Once, Run Everywhere …. and Run (Some Places) Blazingly
](https://www.infoq.com/articles/Java-7-Sockets-Direct-Protocol)


>
- 다시 보는 Java [Introduction](/archivers/look-again-java-1-intro)
- 다시 보는 Java [Socket Direct Protocol](/archivers/look-again-java-2-sockets-direct-protocol)
- 다시 보는 Java [NIO Channel](/archivers/look-again-java-3-NIO-Channel)
- 다시 보는 Java [FileChannel transferTo()](/archivers/look-again-java-4-FileChannel-transferTo())



# Java 7 Sockets Direct Protocol – Write Once, Run Everywhere ... and Run (Some Places) Blazingly
Posted by Ben Cotton on Aug 15, 2013 

이 문서에서는 새로운 [Java Sockets Direct Protocol](http://docs.oracle.com/javase/tutorial/sdp/sockets/) (SDP) 기술, 최근 Java 7 SDK에 도입된 매우 흥미로운 돌파구에 대해 조사합니다. SDP는 Java의 보편적 일반적인 기능을 사용하기 위한 커뮤니티에 초 고성능 컴퓨팅(Ultra High Performance Computing : UHPS)의 힘을 주고 매우 드문 사용 사례에 대한 장점에 힘을 준다: RDMA (Remote Direct Memory Access)는 운영체제를 거치지 않고 다른 컴퓨터의 메모리에 직접 액세스 할 수 있는 낮는 지연을 위한 어플리케이션에게 프로토콜을 제공한다. UHPC 커뮤니티는 대부분 가장 엄격한 비 손상(non-compromising) 낮은 대시 시간(low-latency)과 높은 처리량 (high throughput)에 대한 요구 사항을 갖기 떄문에, UHPC은 최고의 RDMA 기능을 사용할 능력이 필요한 것은 당연하다. Java 7 Sockets Direct Protocol 제공의 도입으로, UHPC 커뮤니티는 이제 Java 어플리케이션 코드로 네이티브 인피니밴드(infiniBand) RDMA 기능의 전체 전력에 직접 결합 할 수 있도록 작성 가능하게 하는 Java 플랫폼을 갖는다.

** 새로운 Java Sockets Direct Protocol 에 대하여 깊게 알아보기 전에, 간단하게 Java의 네트워킹과 소캣 API 역사를 검토하자. **

1995년에 Sun Microsystems는 세계에 Java를 소개하고, 즉시 'Java - Write Once, Run Everywhere' 라는 캐치프라이즈의 보편적인 인식으로 함께 나팔을 불기 시작했다. 우리 모두 알고 있듯이, 이 생각은 간단했다 : 지금부터 C++ 코드의 작성 대신 (그것은 build/deploy하는데 지독하게 어려웠고 'run everywhere' 이식성에 가까운게 아무것도 없었다.) Java라고 불리는 어떤 것을 가지고 어플리케이션을 작성할 수 있고, 가상머신(OS 실행 환경 아래에 있는 것이 아닌)에 build/deploy 할 수 있다. 이것은 Java virtual machine (JVM)에게 이식성의 단독 책임을 위힘하고, 자바 어플리케이션 프로그래머를 크게 해방사켰다. JVM 이 약속을 만들었다: 만약 하나의 Java VM(일부 특정 OS에 대한) 에서 일할 수 있는 Java 코드를 build/deploy 할 수 있다면, 플랫폼은 어떤 코드든 어떤 OS 에서도 정확하게 동작하는 것을 보장한다(호환하는 Java VM 을 사용할 수 있다면). 더 이상 조건에 따른 컴파일과 사전 처리 매크로는 필요 없다. (누구든 C++ 과 #ifdef 지옥을 기억하는가? JVM은 잔인한 부담을 갖는 개발자들을 안도하게 한다).

이것은 모든것에 극도로 유용하고 어플리케이션 프로그래밍 커뮤니티로부터 매우 잘 받아들여졌다. 우리가 잘 알고 있듯이, Java는 신속하고 강렬하게 유행되었다 - 컴퓨팅의 많은 역사에서 타의 추종을 불허하는 속도로, 많은 프로그래밍 언어 플랫폼에서 전세계적으로 받아들여졌다.

처음에, Sun은 Java VM을 정확히 3개의 운영체제에서 실행하는 Java VM을 제공했다.: (i) Solaris (ii) Linux (iii) Windows. Microsoft은 몇 년 더 일찍(1993) Windows와 WinSOCK 프로토콜 스택을 제공했고, Windows는 이제 TCP/IP 네트워킹을 할 수 있기 때문이다 (그리고 Microsoft로부터 API를 통해 완벽하게 지원받았다). (물론)다양한 *nix 시스템은 1970년대 이후 TCP/IP 를 사용하고 있었다. Microsoft의 WinSOCK 도입은 Java가 되기위한 절대적으로 필요한 무엇이 되었다. WinSOCK 없이 Windows VM은 java.net.* 과 java.io.* API들은 적용할 수 없었다. 그게 없으면, Java는 세계의 데스크톱에서 독점하고 실행되고 있는 OS에서 VM의 네트워크 기능을 구축할 수 없었다. '어쩌면 백만' 데스크톱에 도달하는 Java 대신, 전체 TCP/IP를 사용하는 Windows와 함께, Java는 이제 '어쩌면 십억' 데스크톱에 도달할 수 있다.

## 상황이 바꼈다.

물론, Java는 여전히 'write once, run everywhere' 이다. 이식성은 여전히 중점 우선 사항이다. 그러나, 이제 Java 7과 Sochets Direct Protocol은 Java VM과 더 많은 것을 할 수 있다. 이식성 만이 우선 사항이 아니다; 이제는 ultra-high performance의 수용 사례가 Java VM의 주요 우선 순위다. 이제 Java VM은 Sockets Direct Protocol과 함께 네트워킹및 InfiniBand의 기본적 전력에 직접 액세스 Sockets APIs 같은 것을 제공할 수 있다. InfiniBand는 Ethernet 보다 매우 빠르다. InfiniBand는 UHP 컴퓨팅 커뮤니티에 선택의 위한 물리적 네트워크 계층 제공자다.

우리는 InfiniBand와 Java 7 VM에서 InfiniBand의 기능을 어떻게 최대한 활용하는지에 대해 이야기 할 것이다.

참고로 한가지 흥미로운 것은 (특히 역사적인 관점에서); Java는 Sockets Direct Protocol을 두개의 운영체제에 적용하는 것에 대해 결정했고, Microsoft Windows 는 이 운영체제 중의 하나가 아니라는 것이다. Java 7 SDP를 제공하는 두개의 운영체제는 Solaris와 Linux다. Solaris SDP 지원은 Solaris 10 이후 버전부터 표준이였다. 당신이 물리적인 InfiniBand NIC를 가지고 있는한, Java 7 SDP는 상자 밖에서 작동한다. 리눅스를 위해, SDP 지원은 Open Fabrics Enterprise Distribution package 를 통해 제공한다. 확인하기 위해 만약 당신의 리눅스 버전이 OFED 디바이스 드라이버가 설정 되었고, 정말로 물리적 InfiniBand NIC 아답터를 가지고 있다면, 단순히 입력하면 된다.

```
egrep "^[ \t]+ib" /proc/net/dev
```

이 명령에서 어떤 출력을 얻었다면, 당신은 운영체제에서 Java 7 SDP 를 사용하기 위한 모든 설정이 되어 있다.

모든 java.net.* 과 java.io.* 어플리케이션 코드는 - 당연히 - 여전히 Microsoft Windows 에서 Java 7 VM 을 사용하여 실행될 것이다... 그러나 Sockets Direct Protocol 없이 실행될 것을 주의하는것이 중요하다(따라서 이는 물리계층 제공자 처럼 이터넷에서 실행될 것이다). 당신이 InfiniBand를 지원을 제공하는 Windows Server 버전을 사용하더라도 마찬가지일 것이다(WinSOCK Direct 를 통해). 다시 모든것은 여전히 Microshfot 에서 실행될 것이고, 그것은 최대한 빠르게 실행되지 않는다. 

## 상황이 정말 바꼈다.

Java의 API 와 운영체제의 네트워크 프로토콜 스택과 연결해서 얘기해보자. 첫째, 네트워크 계층의 표준 [Open Systems Interconnection (OSI) model](http://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=20269) 은 다음과 같다.

|#|Layer|Protocol|Java SDK core APIs|
|-|-----|--------|------------------|
|7.|Application Layer|HTTP,FTP,SSL,etc.|java.net.HttpUrlConnection, javax.servlet.HttpServlet|
|6.|Presentation Layer||# 자바에서는 어플리케이션/프리젠테이션 OSI 계층과 구별하지 않는다.|
|5.|Session Layer|NetBios, RCP|# OSI 세션 계층을 위한 Java SDK core 는 지원하지 않는다.|
|4.|Transport Layer|TCP, UDP|java.net.Socket, java.net.ServerSocket, java.net.Datagram|
|3.|Network Layer|IP|java.net.InetAddress|
|2.|Data Link Layer|PPP|# OSI 데이터링크 계층을 위한 Java SDK core 는 지원하지 않는다.|
|1.|Physical Layer|Ethernet, InfiniBand| 물리계층을 위한 Java SDK core 는 지원하지 않는다. 그러나...

Java 7 Sockets Direct Protocol (InfiniBand로 부터 java.net.* 와 java.io.* core APIs 는 VM 과 연결된다.) |

OSI 네트워크 계층 뷰를 위해, Java 7 Sockets Direct Protocol 능력은 Java 어플리케이션 코드를 가능한 "금속 가까이" 한다. - Java 코드로 물리적인 H/W 에 더 가깝게 접근할 수 있게 한다는 말인듯. 마치 기계어 처럼 - Java SDP는 어플리케이션 코드로 부터 VM을 통해, InfiniBand의 *direct(직접)* (SDP 안의 'D') *native(자연적인)*, *pysical(물리적)* 연결을 제공한다. Java 7 SDP 기능은 어플리케이션의 변경 없이 core java.net.* 과 java.io.* APIs를 사용한다. InfiniBand OS 디바이스 드라이버에 Java VM의 특정 조인 포인트와 라이브러리를를 구성하고 java.net.*, java.io.* - OS 리소스의 전송계층을 위한 Java API(OSI 4 계층) -  의 자바 코드를 사용하여 전통적인 네트워크 프로토콜 스텍을 우회할 수 있고, (즉 그것은 OSI 3계층과 2 계층을 건너뛴다.) *direct(직접)* InfiniBand 로 간다.(OSI 1계층). 예상되는 성능 영향과 보상은 매우 중요하다.

## Java 7 과 Sockets Direct Protocol로, Java는 지금 RDMA를 수행 (Remote Direct Memory Access)

RDMA는 Remote Dynamic Memory Access 다 -- 그것은 두 Java VM 프로세스 사이의 어플리케이션 버퍼를 네트워크 사이로 이동시키는 방법이다 (*nix 사용자 주소 공산에서 실행). RDMA는 운영체제를 우회하기 때문에 전통적인 네트워크 인터페이스와 다르다. 이 RDMA를 통해 전달하기 위해 Java SDP 를 허용한다: (i) 절대적으로 낮은 응답 (ii) 매우 높은 처리량 (iii) 매우 작은 CPU 사용.

Java가 공개하는 RDMA 결합점은, SDP는 암묵적으로 매우 매력적인 "Zero-copy" 기능을 제공할 수 있게 한다. "Zero-copy"는 CPU가 다른 하나의 메모리 영역으로 부터 데이터를 복사하는 작업을 수행하지 않도록 하는 컴퓨터 동작을 설명한다. 네트워크 프로토콜 스택의 Zero-copy 버전은 크게 특정 어플리케이션 프로그램의 성능을 향상 시키고 보다 효율적으로 시스템 리소스를 이용한다. 성능은 데이터 복사 처리 하는동안 머신의 다른 작업으로 이동할 수 있도록 CPU에게 허용함으로써 향상된다. 또한, zero-copy 명령은 time-consuming mode의 사용자 영역과 커널 영역의 전환(switch) 수를 감소시킨다. 시스템 리소스는 대규모 복사 명령을 수행하기 위한 세련되게 CPU를 사용하여 더 효과적으로 활용되고, 다른 간단한 시스템 구성 요소를 복사 할 경우 낭비가 될 수 있다. 그것은 우리가 여기서 이야기 하는 Zero-copy 기능은 java.nio.channels.FileChannel의 transferTo() API 에서 얻을 수 있는 Zero-copy 기능이 아니라는 것이 중요하다. 그것은 훨씬, 훨씬 더 확대되는 것이다. Java 7 SDP로, 당신은 직접 네이티브 InfiniBand Zero-copy 프로토콜 구현을 사용한다.

## 몇 가지 일반적인 Java 개발 뷰의 문맥과 함께 시각적으로 Sockets Direct Protocol 기능의 모습을 정확하게 묘사해보자

다음의 그림은 노드 1(java.net.Socket writer)과 노드 2(java.net.ServerSocket listner)가 Java 7 VM 구성 위로 배포되고, 두개의 JVMs은 InfiniBand 네트워크를 통해, OS 시스템 호출 또는 서비스의 실행 없이, 하나의 VM 으로 부터 다른 VM 으로 어플리케이션 데이터 버퍼를 교환할 수 있는 SDP를 지원하기 위한 부팅을 보여준다.

![](https://cdn.infoq.com/statics_s1_20161004-0306/resource/articles/Java-7-Sockets-Direct-Protocol/en/resources/InfiniBandFig1.jpg)

1. Java 7 application = Node 1 (JVM은 SDP 를 이용해 부팅됨)은 어플리케이션 데이터의 블럭을 네트워크에 걸쳐 java.net.ServerSocket *listener* 로 쓰기 위해 java.net.Socket API를 사용한다.

2. JVM은 SDP를 사용하여 부팅되었기 때문에 운영체제는 TCP/IP 스택은 완전히 무시된다. [참고 : TCP/IP 프로토콜에서 데이터 통신을 처리하는 방법](https://docs.oracle.com/cd/E38901_01/html/E38894/ipov-29.html) - 어플리케이션 데이터는 InfiniBand의 RDMA 기능을 통해 직접 쓰여진다 (네트워크 인터페이스 카드의 물리적 제공이 되기 위한 InfiniBand가 필요하다).

3. Java 7 application = Node 2 (JVM은 또한 SDP를 이용해 부팅됨) java.net.Socket writer로 부터 네트워크를 통해 RMDA를 경유하여 도착한 어플리케이션 데이터의 블럭을 *listen* 하기 위해 java.net.ServerSocket API 를 사용한다. (네트워크 인터페이스 카드의 물리적 제공이 되기 위한 InfiniBand가 필요하다).

4. 데이터는 Java 7 VM 어플리케이션 버퍼로 *직접 (directly)* 전송된다! OS 시스템 또는 서비스 호출 실행 없이!! - Node 1, Node 2 의 OS 모두 아니다 - 그것이 Java 7 Socket Direct Protocol 의 힘이다.

## Java 6 과 SDP로 실행된 Java 7에서 실행되는 동일한 응용 프로그램 사이의 논리적 성능 차이는 무엇인가?

두개의 다른 시나리오에서 다음 "deep-dive" Node 2의 그림(이전 페이지의 그림으로 부터)을 보자:

1. SDP와 Java 7을 사용하여 구성된(아래 왼쪽에 보여지는)것은, OSI 네트워크 레이어 프로토콜 스택을 여행하고 Java 어플리케이션 안으로 들어온 Node 1이 전송한 데이터는 Node 2는 어떻게 수신하는가? 얼마나 많은 단계를 작업하는가? 그것은 오직 하나의 단계다! (아래를 보라 - 이것은 UHPC Java apps을 위한 대단히 새로것이다; 이제 UHPC 커뮤니티는 수행하기 위해 필요한 무엇이든 Java 7 을 사용할 수 있다).

2. Java 6를 사용하여(SDP가 아닌 - 오른쪽 아래에 보여지는)것은, OSI 네트워크 레이어 프로토콜 스택을 여행하고 Java 어플리케이션 안으로 들어온 Node 1이 전송한 데이터는 Node 2는 어떻게 수신하는가? 얼마나 많은 단계를 작업하는가? 그것은 다섯 단계를 갖는다(아래를 보라 - 이것은 익숙한 TCP/IP 프로토콜 스택이다 - SDP 가 아니다. 그것은 대부분 작업하지만, UHPC 커뮤니티를 위한 작업은 아니다. UHPC 커뮤니티는 필요한 것을 완료하기 위해 Java 6를 사용할 수 없다.).

![](https://cdn.infoq.com/statics_s1_20161004-0306/resource/articles/Java-7-Sockets-Direct-Protocol/en/resources/Fig2small.jpg)

## Sockets Direct Protocol 을 사용한 Java 7 VM은 어떻게 관리하고 설정하나?

이건 그냥 InfiniBand 사용할 때 원문 봐도 무관할 듯;; 

The following is distilled from the configuration section of the [Oracle tutorial page that introduces Java 7 SDP](http://docs.oracle.com/javase/tutorial/sdp/sockets/index.html)

An SDP configuration file is a text file that the Java VM reads from the local file system at boot time. The SDP configuration file has exactly 2 different types of entries. Each type of entry is expressed exactly once per line:

1. An SDP configuration comment line
2. An SDP configuration rule line

A comment is indicated by the hash character (#) at the beginning of the line, and everything following the hash character will be ignored.

For configuration rule lines, there are exactly two types of rules:

1. BIND rules
2. CONNECT rules

A "bind" rule indicates that the SDP protocol transport should be used whenever a TCP socket binds to an address and port that match the rule. A "connect" rule indicates that the SDP protocol transport should be used when an unbound TCP socket attempts to connect to an address and port that match the rule.

Through the rules specified in the SDP configuration file, the Java VM can know exactly when to replace the normal TCP/IP protocol stack with InfiniBand’s VERBS/RDMA protocol stack.

The first keyword indicates whether the rule is a bind or a connect rule. The next token specifies either a host name or a literal IP address. When you specify a literal IP address, you can also specify a prefix, which indicates an IP address range. The third and final token is a port number or a range of port numbers.

Consider the following notation in this sample configuration file:

\# Use SDP when binding to 192.0.2.1

bind 192.0.2.1 *

\# Use SDP when connecting to all application services on 192.0.2.*

connect 192.0.2.0/24 1024-*

The first rule in the sample file specifies that SDP be used for any port (*) on the local IP address 192.0.2.1. You would add a bind rule for each local address assigned to an InfiniBand adaptor. (An InfiniBand adaptor is the equivalent of a network interface card (NIC) for InfiniBand.) If you had several IB adaptors, you would use a bind rule for each address that is assigned to those adaptors.

The second rule in the sample file specifies that whenever connecting to 192.0.2.* and the target port is 1024 or greater, SDP is used. The prefix on the IP address /24 indicates that the first 24 bits of the 32-bit IP address should match the specified address. Each portion of the IP address uses 8 bits, so 24 bits indicates that the IP address should match 192.0.2and the final byte can be any value. The -* notation on the port token specifies "and above." A range of ports, such as 1024—2056, would also be valid and would include the end points of the specified range.

## Sockets Direct Protocol 을 사용하여 Java 7 VM 부팅은 어떻게 하나?
 
 이것도 그냥 사용할 떄 원문 보자;;;

 ```
 &> java \
-Dcom.sun.sdp.conf=sdp.conf \
-Djava.net.preferIPv4Stack=true \
Application.class
 ```

Note the use of the IPv4Stack as the network format to be booted. Even though both Java 7 and InfiniBand use the more modern IPv6 network format, the mapping between the 2 on both Solaris and Linux is not supported. Thus, always use the cornerstone familiar (and decades reliable) IPv4 network format when booting a Java 7 VM that will use SDP.

## SDP 지원된 Java 7 VM 응용프로그램을 실행할때 얼마나 많은 성능향상을 예상하는가?

대충..

매우 긍정적인 질문이고, Java 7 SDP를 사용하여 무엇을 얻을수 있는가? 이 질문의 답은 이 문서에서 확실히 결정할 수 없다고;;; 성능은 많은 요인에 따라 다르게 얻을수 있다고;;
다음에 대해 항상 참이라고 알고있다고;;

InfiniBand 는 Ethernet 보다 상당히 빠르고, <br/>
Java 7 SDP 는 RDMA를 사용하고 최고의 Zero-copy 구현체이고, <br/>
데이터는 100% OS의 TCP/IP 스택을 무시하고 전송하고, <br/>
커널 주소영역과 사용자 주소영역안의 어플리케이션 코드 버퍼 사이에 데이터가 전송될때 모든 문맥 전환은 온다.<br/>

이 모든 것은 100% Java SDK API 로 투명하게 제공되고,<br/>
Java 어플리케이션에서 java.net.* 또는 java.io.* 를 사용한 코드에 대해 한줄의 수정할 필요가 없다.<br/>

많은것이 변경되었지만, <br/>
Java 슬로건대로 한번 쓰고 어디서든 실행하자! <br/>
뭐.. 자바 만세?!


## 내 결론.

InfiniBand 좋은거구나..<br/>
Java 7에 SDP 라는게 있었구나..<br/>
InfiniBand를 사용할 날이 올까??<br/>
transferTo() 나 볼걸? <br/>
여튼 새로운걸 알게 됨. 만세!<br/>
Java 만세!<br/>

>
- 다시 보는 Java [Introduction](/archivers/look-again-java-1-intro)
- 다시 보는 Java [Socket Direct Protocol](/archivers/look-again-java-2-sockets-direct-protocol)
- 다시 보는 Java [NIO Channel](/archivers/look-again-java-3-NIO-Channel)
- 다시 보는 Java [FileChannel transferTo()](/archivers/look-again-java-4-FileChannel-transferTo())

