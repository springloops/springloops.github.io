---
author: springloops
comments: true
date: 2014-11-05 07:51:50+00:00
layout: post
link: https://springloops.wordpress.com/2014/11/05/java-troubleshooting/
slug: java-troubleshooting
title: java Troubleshooting
wordpress_id: 108
categories:
- java
tags:
- Java
- java Troubleshooting
- troubleshooting
---



  * ******JVM 분석**



    * ******jmap -heap “PDI”**



      * 힙 사용


      * From space


      * To spage


      * Tenured generation


      * Perm gneration


      * Eden space 등의 정보를 제공






  





  * ******쓰레드 덤프 생성**



    * ******kill 이용**



      * kill -3 "java pid”


      * catalina.out 으로 출력된다.



    * ******jstack 을 이용**


<table cellpadding="0" width="747.0" style="width:747px;border-style:solid;border-width:1px;border-color:#808080;" cellspacing="0" >
<tbody >
<tr >

<td style="width:46px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * 옵션


</td>

<td style="width:689px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * 설명


</td>
</tr>
<tr >

<td style="width:46px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * -f


</td>

<td style="width:689px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * 자바 스택 생성을 강제함. 프로세스가 멈춰버린(hang) 상태일 때 주로 사용


</td>
</tr>
<tr >

<td style="width:46px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * -l


</td>

<td style="width:689px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * 상세한 출력 (잠금과 관련한 상세 정보 표시)


</td>
</tr>
<tr >

<td style="width:46px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * -m


</td>

<td style="width:689px;border-style:solid;border-width:1px;border-color:#808080;padding:2px;" valign="top" >

  * 혼합 모드 자바 스택 생성


</td>
</tr>
</tbody>
</table>



      * jstack -f PID > treaddump.txt


      * 64비트에서



        * jstack -J-d64 -m PID







    * ******스레드 해석 하기 참고 : **[_**http://helloworld.naver.com/helloworld/textyle/10963**_](http://helloworld.naver.com/helloworld/textyle/10963)


    *   

**자바 애플리케이션 성능 튜닝의 도(道) : **[_**http://helloworld.naver.com/helloworld/textyle/184615**_](http://helloworld.naver.com/helloworld/textyle/184615)






  





  * ******에러 해결책**



    * ******JVM 메모리 문제**



      * 메모리 고갈 예외



        * 예외 : java.lang.OutOfMemoryError: Java heap space



          * 원인



            * 메모리 자원을 많이 사용



          * 해결 방법



            * 최대 힙 사이즈를 증가시킨다.


            * JVM 은 전체 메모리의 70% 만 할당 할 수 있다. (30% 는 OS)


            * Jmap 을 통해 JVM 상태 확인 후 적절한 메모리 설정을 한다.



          * JVM Option



            * -Xms512m -Xmx1024m




        * 예외 : java.lang.OutOfMemoryError: PermGen space



          * 원인



            * PermGen 은 사용자 클래스 정보를 포함하는 메타데이터를 저장하는 영역


            * 코드가 큰 어플리케이션에서는 PermGen 을 모두 소비하면서 에러를 발생할 수 있다.



          * 해결 방법



            * PermGen 영역의 크기를 늘린다.



          * JVM Option



            * -XX:MaxPermSize=128m




        * 예외 : java.lang.StackOverflowError



          * 원인



            * 중첩된 메소드 호출이 많아서 스택이 오버플로우되면서 발생



          * 해결 방법



            * 중첩 코드를 줄이거나


            * 메소드 내 코드를 간략히 하거나


            * stack 사이즈를 늘린다.



          * JVM Option



            * -Xss=128k









  

