---
author: springloops
comments: true
date: 2009-07-19 04:12:00+00:00
layout: post
link: https://springloops.wordpress.com/2009/07/19/open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-1/
slug: open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-1
title: 'Open FrameWork – iBATIS : 아이바티스 분석 (1)'
wordpress_id: 9
categories:
- Open Source
tags:
- 분석
- framework
- iBATIS
- 아이바티스
- Java
- Open Source
---

  




**iBATIS Framework 분석 (1)  

Analysis of iBATIS Open Framework. (1)**




** **







먼저 말한대로 iBATIS에 대해서 포스팅을 해봅니다  

I said that will write about analysis of iBATIS.




전체적인 구성에 대해서 생각해본적은 없지만 크게 다음과 같이 진행 되지 않을까 합니다.  

I didn't think about this post. however I will try to next step :







1. iBATIS는 무엇인가?  

What is iBATIS?




2. iBATIS의 사용 방법  

How to use iBATIS




3. iBATIS의 뼈대를 이루는 소스의 흐름 분석  

analysis to iBATIS code







추신  

P.S.  

저는 영어보다 프로그램을 더 잘합니다.  

I can do better Program than English.  

사실 영어는 잘 못합니다.  

Actually, I'm not good at English.  

이 글에 대한 기술적인 오류를 포함하여, 영작의 잘못된 점도 비평해준다면 고맙겠습니다.  

Please write reply if you find about wrong something.  

And welcome any reply too.





[#M_더보기|접기|1. iBATIS는 무엇인가?  

1. What is iBATIS?  

  
  

  
이번 포스트는 iBATIS에 대한 사용법도 아니고, 소스 분석 부분도 아니기에 길게 쓰고 싶지 않네요.  

I will write short post. Because this chapter is not manual and not analysis code.  

  
여기서는 iBATIS가 무엇인지, 어디서 존재 하는지, 왜 사용 해야 하는지에 대해서 정말 간단하게 알아 봅시다.  

This chapter is summary about what, where, why iBATIS.  

  
한 문장으로 답해 보겠다  

Answer the question.  

  
  

  
iBATIS는 퍼시스턴스 영역에 존재 하는 SQL 매핑 프레임워크이다.  

iBATIS is SQL mapping framework in Persistence area.  

  
  

  
그럼 퍼시스턴스 영역에 대하여 간단하게 알아 보자.  

So, What is Persistence Area?  

  
아래 다이어그램을 보자.  

Look at next diagram,  

MVC 패턴을 아는 개발자라면 다음 그림을 보면 쉽게 이해가 될것이다.  

If you know MVC pattern, you can understand easy.  

  
  

  
![](http://localhost/wordpress/wp-content/uploads/1/cfile9.uf.170B47284AD55D2B62C71C.jpg)  

퍼시스턴스는 일반적으로 사용자가 개발 또는 다른 방법으로 애플리케이션에 입력하고  

  
애플리케이션이 종료 후에도 남아 있는 데이터를 뜻하고, 데이터베이에 접근하여 값을 넣고 빼는 역할을 하는 영역이다.  

  
  

  
물론 자체 개발을 할수도 있지만, iBATIS 또는 SQL 매핑 프레임웍(ORM) 을 사용 한다면 개발자를  

  
위해 객체와 데이터베이스 사이의 매핑 관련 잡다한 작업을 숨겨준다.  

  
  

  
직접 jdbc 를 이용 하여 데이터를 넣고 빼고 하기 위해 connection을 열고 statement를 연결하고  

  
결과를 resultset으로 받고, 다시 이 모든 자원을 close 하는 작업을 없애 준다는 말이다.  

이부분에 대해서는 곧 사용법을 공부하며 알수 있을것이다.  

  
  

  
끝내기 전에 각 Layer들이 화살표로 연결 되어 있는데, 각 Layer 전달 시 명시 되어 있는 패턴에 대해서 간략하게 알아 보자.  

물론 iBATIS는 퍼시스턴스 영역이므로 DAO에 대해서 조금이나마 중점을 두고 알아보자.  

  
  

  
Services Locator  

  
비즈니스 컴포넌트와 서비스의 위치를 통일된 방법으로 찾아 주는 패턴.  

예를 들어 EJB 에서는 Session의 정보를 가져오기 위해 EJB HOME에서 HOME 생성 메소드를 호출 하거나, JNDI 를 이용하여   

  
관련 객체 바인딩을 하는 방식등이 이에 해당 될수 있겠다.  

  
사실 EJB의 경우 손안댄지 2년이 되어 가기 때문에 옳바른 예제가 될지 모르겠습니다.  

  
또한 언제가 될지 모르지만 GoF 디자인 패턴, J2EE 패턴에 대해서 포스팅을 하려고 마음 먹고 있는데,   

  
그때 패턴에 대해서는 더욱 자세하게 포스팅 해보겠습니다.  

  
  

  
DAO (Data Access Object) 패턴  

  
비즈니스 영역과 퍼시스턴스 로직이 서로 섞여있게 되면, 비즈니스 로직과 퍼시스턴스 구현  

사이에 직접적인 종속 관계가 성립합니다(High Coupling)  

컴포넌트 안에서 이런 코드 종속성은 소스 타입 변경시(유지 보수시) 어렵게 합니다.  

  
DAO는 데이터 소스 세부 구현을 클라이언트가 알 필요가 없도록 내부 구현을 추상화 시켜  

  
DAO 자체 구현이 변경 되어도 클라이언트는 수정 작없 없이 기존의 인터페이스를 사용 하면 접근  

가능 하게 합니다.  

  
  

  
iBATIS 는 내부적으로 JDBC 의 작업을 구현 하고 인터페이스를 제공 합니다.  

클라이언트는 iBATIS를 사용 하여 기존의 반복적이고 많은 양의 JDBC 코드를   

  
작성하지 않아도 된다고 합니다.(역시 공부하면서 확인해 보자구요..)  

  
DAO를 이용한 iBATIS를 이용하여 잘 분리 하여 구현할 경우 추후, 다른 매핑 프레임웍으로  

변경이 되어도 큰 작없없이 사용이 가능할 것입니다.  

  
  

  
이상으로 iBATIS가 무엇이고, 어디에 존재하고, 왜 사용하는지에 대해 알아 봤는데..  

다시 읽어 보니 마찬가지로 구멍이 너무 많네요.  

  
  

  
  

  
또한 영문도 쓰다가 도저히 감당이 안되네요.. 문법도 맞는지 틀리는지..  

모르는건 부끄러운게 아니다 라고 하지만.. 모른채 남겨 두면 안되겠죠..  

  
더 노력해 보겠습니다.  

다음 번 글은 조금더 잘 써보겠습니다.  

머 이건 실제 생활에서의 연습일 뿐이니까요..  

  
_M#]
    
    <img src="http://somethingbook.wordpress.com/wp-includes/js/tinymce/plugins/wordpress/img/trans.gif" alt="" class="mceWPmore mceItemNoResize" title="More...">
    
    
     
    
    =======================================
    <address>프로그래머 오성민
    opsbina@gmail.com
    010-6358-2651</address>
    
