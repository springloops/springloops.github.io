---
author: springloops
comments: true
date: 2014-11-05 07:51:07+00:00
layout: post
link: https://springloops.wordpress.com/2014/11/05/monitoring/
slug: monitoring
title: java monitoring
wordpress_id: 107
categories:
- java
tags:
- Java
- java monitoring
- Monitoring
---

**JConsole**  

  

JConsole 은 자바 모니터링과 관리 콘솔이다.  

그래픽적 도구로 어플리케이션 서버 성능 세부 정보를 제공한다.  

  





  * ******기능**



    * 로우 메모리 ( 메모리 부족이 발생할 수 있는 상황 ) 검출


    * GC, 클래스 로딩 상세 정보 출력 활성화/비활성화


    * 데드락 검출


    * 어플리케이션의 모든 로거의 로거 레벨 제어


    * OS 자원 접근 - 썬의 플랫폼 확장


    * 어플리케이션의 MBeans 관리





  





  * ******원격 JMX 활성화**



    * JConsole을 통해 모니터링하기 위해 JMX ( Java Management Extension) 를 활성화 해야한다.


    * JVM 옵션

<table cellpadding="0" width="667.0" style="width:667px;border-style:solid;border-width:1px;border-color:#808080;" cellspacing="0" >
<tbody >
<tr >

<td style="width:661px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >


      * -Dcom.sun.management.jmxremote 


      * -Dcom.sun.management.jmxremote.port=9999 (사용할 포트) 


      * -Dcom.sun.management.jmxremote.ssl=false   

-Dcom.sun.management.jmxremote.authenticate=false  




      *   

**의미**  

-Dcom.sun.management.jmxremote : 호스트가 로컬 시스템에 접근할 수 있도록 허용한다.  

-Dcom.sun.management.jmxremote.port=9999 : RMI (Remote Method Invocation) 가 연결될 포트 정의  

-Dcom.sun.management.jmxremote.ssl=false  : 통신에 사용하는 프로토콜을 정의한다. 기본값은 false 로 HTTP 프로토콜을 사용한다.  

-Dcom.sun.management.jmxremote.authenticate=false : 연결 인증 방식을 정의한다  

  




</td>
</tr>
</tbody>
</table>

    * 


  * JConsole 연결



    * java 가 설치된 환경에서


    * ?> jconsole


    * 접속할 원격 서버의 IP, PORT 입력


    * 탭별 기능



      * 메모리 개요



        * JVM 이 차지하는 메모리 공간을 포함하는 메모리 사용 현황을 그래픽적으로 표현


        * 분석 요구사항에 맞게 커스터마이즈된 메모리 차트 제공


        * GC 수행 기능 제공



      * 스레드 개요



        * 어플리케이션이 사용하는 THREAD 정보 제공


        * 스레드와 관련된 정보를 그래픽으로 제공


        * 개발 스레드의 상태와 분석 제공


        * 데드락 검출



      * MBeans



        * MBean (managed bean) 관련 정보를 제공


        * MBean 을 통해 이벤트 알림을 만들수 있다.


        * 동적으로 자원 할당


        * mbean은 나도 잘 모르니까…


        * [_http://docs.oracle.com/javase/tutorial/jmx/mbeans/standard.html_](http://docs.oracle.com/javase/tutorial/jmx/mbeans/standard.html)


        * Mbean 의 종류



          * 표준 mbean



            * mbean 인터페이스와 클래스를 조합한 형태를 말한다.


            * 인터페이스는 속성과 동작 전체 목록을 정의


            * 원격 인터페이스의 통신 기능 제공



          * 동적 mbean


          * 오픈 mbean


          * 모델 mbean







  

