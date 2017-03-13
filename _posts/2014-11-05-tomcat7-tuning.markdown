---
author: springloops
comments: true
date: 2014-11-05 07:49:51+00:00
layout: post
link: https://springloops.wordpress.com/2014/11/05/tomcat7-tuning/
slug: tomcat7-tuning
title: tomcat-7 tuning
wordpress_id: 104
categories:
- Open Source/Apache Tomcat
tags:
- tomcat
- tomcat tuning
- tomcat7
- tuning
---

**톰캣 컴포넌트 튜닝**  





  * 스레드 튜닝


  * 포트 커스터마이즈


  * JVM 튜닝 등




  

**톰캣 커넥터의 종류**  

  

- 커넥터?   

요청을 수락하고 응답을 리턴하는 교차점  





  * HTTP 커넥터



    * HTTP 프로토콜을 기반으로 하는 HTTP 커넥터


    * HTTP/1.1 프로토콜만 지원


    * 독립형 웹 서버로 동작할 수 있다. ( iis / apache 역할도 한다는 얘기임 )


    * JSP/Servlet 호스트 기능도 제공


    * [_http://tomcat.apache.org/tomcat-7.0-doc/config/http.html_](http://tomcat.apache.org/tomcat-7.0-doc/config/http.html%0A)



  * AJP 커넥터



    * AJP ( 아파치 JServ 프로토콜 ) 와 AJP를 통한 웹 서버 통신을 기반으로 한다.


    * 자바 서블릿 컨테이너를 인터넷에 노출하고 싶지 않을때 AJP 커넥터를 사용


    * 다른 프론트엔드 서버를 사용한다는 얘기임.. ( iis / apache 등)


    * AJP 구현체로 mod_jk, mod_proxy 등이 있음.


    * [_http://tomcat.apache.org/tomcat-7.0-doc/config/ajp.html_](http://tomcat.apache.org/tomcat-7.0-doc/config/ajp.html)



  * APR (AJP/HTTP) 커넥터



    * APR : Apache Portable Runtime


    * 확장성, 성능, 다른 웹 서버와의 협력 작업 상황에서 활용


    * Open SSL, 공유 메모리, 유닉스 소켓 등 부가 기능을 제공


    * [_http://tomcat.apache.org/tomcat-7.0-doc/config/apr.html_](http://tomcat.apache.org/tomcat-7.0-doc/config/apr.html%0A)





  

  

**톰캣 7 스레드 최적화**  

  

- **스레드 풀**  

톰캣 웹 서버가 수락할 수 있는 연결 수 또는 서버가 수락할 수 있는 요청 수  

( 톰캣은 요청을 받으면 스레드를 생성한다, 이때 생성되는 스레드를 풀로 관리 한다는 얘기임 )  

  

**톰캣의 스레드 풀 종류**  





  * ******공유 스레드 풀 ( 공유된 실행자 )**



    1. 많은 커넥터가 공유하는 풀을 말함


    2. 예를 들어, 네개의 커넥터가 있을 경우, 모든 커넥터에서 같은 스레드 풀을 공유


    3. 커넥터 설명은 위에…


    4. ******설정방법**



      1. ******server.xml **에 **services **섹션에 정의


      2. executor 정의



        * <Executor name=“tomcatThreadPool (임의의 값)” namePrefix=“catalina-exec-“ maxThreads=“150” minSpareThread=“4” />



      3. connector 정의



        * <Connector executor=“tomcatThreadPool (정의한 executor 값)” port=“8080” protocal=“HTTP/1.1” connectionTimeout=“20000” redirectPort=“8443” />







  





  * ******전용 스레드 풀**



    * 공유 스레드 풀과 반대로 특정 connector 전용으로 할당하는 스레드 풀


    * ******설정방법**



      * ******server.xml** 에 **connector** 섹션에 정의



        * <Connector port=“8080” protocal=“HTTP/1.1” SSLEnabled=“true” **maxThreads=“150”** schema=“https” secure=“true” clientAuth=“false” sslProtocol =“TLS” />







  

**maxThreads **  

서버가 수락할 수 있는 최대 요청 수 정의  

환경에 따라 최적의 Thread 수를 설정해야한다.  





  * thread 수를 늘리고 cpu  사용량을 확인한다.


  * cpu 사용량이 높으면 thread 값을 낮추고, 보통이면 더 많은 thread 를 설정할 수 있다.




**masKeepAlive**  

동시에 대기할 수 있는 (종료되지 않고) TCP 연결 수 를 정의 한다.  

기본값은 1, 즉 비활성화  





  * maxKeepAlive = 1 이면,



    * 톰캣에서 SSL 터미네이션을 수행하지 않는다.


    * 부하 균형 기법을 사용한다.


    * 동시에 더 많은 사용자를 수용한다.



  * maxKeepAlive > 1 이면,



    * 톰캣에서 SSL 터미네이션을 수행한다.


    * 적은 동시 사용자를 수용한다.





**JVM 튜닝**  





  * 톰캣 프로세스 PID 확인



    * ps -ef | grep java



  * jmap을 통한 톰캣 인스턴스의 메모리 확인



    * 옵션

<table cellpadding="0" width="667.0" style="width:667px;border-style:solid;border-width:1px;border-color:#808080;" cellspacing="0" >
<tbody >
<tr >

<td style="width:108px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >


      * -dump


</td>

<td style="width:547px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * 자바 합을 hprof 바이너리 형식으로 덤프


</td>
</tr>
<tr >

<td style="width:108px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * -finalizer info


</td>

<td style="width:547px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * 소멸(finalization) 을 기다리는 오브젝트 정보 출력


</td>
</tr>
<tr >

<td style="width:108px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * -heap


</td>

<td style="width:547px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * 힙 요약 정보 출력


</td>
</tr>
<tr >

<td style="width:108px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * -histo


</td>

<td style="width:547px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * 힙 히스토그램 출력


</td>
</tr>
<tr >

<td style="width:108px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * -permstat


</td>

<td style="width:547px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

    * 자바 힙의 영구 세대 (permanent generation)를 클래스 로더 통계로 출력


</td>
</tr>
</tbody>
</table>

    * 


  * [_http://docs.oracle.com/javase/6/docs/technotes/tools/share/jmap.html_](http://docs.oracle.com/javase/6/docs/technotes/tools/share/jmap.html)


  * 문법



    * 32 bit



      * jmap -heap <process id>



    * 64 bit



      * jmap -J-d64 -heap <process id>



    * 결과 해석 방법은 인터넷으로…


    * 





  

  

