---
author: springloops
comments: true
date: 2009-10-12 04:14:00+00:00
layout: post
link: https://springloops.wordpress.com/2009/10/12/open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-2/
slug: open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-2
title: 'Open FrameWork – iBATIS : 아이바티스 분석 (2)'
wordpress_id: 10
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

  




자 오랜만입니다! ㅎㅎㅎ  

  
한동안 귀차니즘에 빠지면서 글을 올리지 못했습니다.  

  
아주 조금씩 iBatis 를 사용해보았고요..  

  
이제 iBatis를 분석해보려고 합니다. 마찬가지로 함께 나눠 봅시다. ;-)  

**  

  



[#M_내용 보기|접기|  

SqlMapConfig.xml 및 SqlMap.xml 파일의 설정 부분은 다음에 포스팅을 하거나, 다른 문서를 참조 해주세요.  

**  

iBatis 를 다운 받아 보면 그안에 샘플 파일이 있습니다.  

****  

그 샘플 예제를 바탕으로 흐름에 대해서 분석하도록 하겠습니다.  

**  

샘플 코드로 ibatis 의 흐름을 따라가 보면서 내부적으로 일어나는 행위를 훔쳐 볼건데,  

  
jar 파일을 클래스 패스에 걸어 놓고 실행 시키면 아무래도 추적하는데 쉽지 않지요.  

  
해서 ibatis-src 파일을 이클립스의 java project로 import 해서 디버깅 모드로 돌려 보겠습니다.  

  
ibatis-src 파일을 import 하면 이클립스에서 여러 클래스 파일이 없다고 오류를 보여 줍니다.  

  
관련 lib 가 필요 하단 얘기 죠.  

  
제가 찾아보니 다음과 같은 추가 lib 가 필요 했습니다. 


**- oscache-2.4.1.jar**




잦은 반복되는 작업을 수행할때 서비스 시간을 줄이기 위해서 캐싱 기능을 사용하기 위해 Java Cahce 를 사용한다고 합니다.




그것과 같이 캐싱 기능을 제공 하는 lib 라고 합니다.




캐싱 할수 있는 것은 JSP 페이지 캐싱 (Filter 이용), 객체 캐싱, JMX 메시지 캐싱 이 있다고 합니다.




**[출처]** [http://sokum.tistory.com/28](http://sokum.tistory.com/28) | **작성자** 소금마왕




****




**- log4j-1.2.15.jar**




로깅 작업을 해주는 API




****




**- jta-1_1.jar**




Java Transaction Api 로 트랜젝션을 관리 해주는 api 인듯 합니다.




사용자 트랜젝션, 지역 전역 트랜젝션을 제공하는 api 로 알고 있습니다.




****




**- commons-logging-1.0.4.jar**




Commons Loggin API는 자카르타 Commons에 포함되어 있는 프로젝트들이 로거로 사용하는 API이다. **Commons Logging API는 자체적으로 로깅 기능을 구현하고 있지는 않으며**, 로깅 요청을 Log4J나 자바1.4로깅 API와 같이 이미 존재하는 로깅 API에 전달하는 다리 역할을 한다.




**[출처]** [Commons Logging과 Log4J](http://blog.naver.com/kotaeho0512/50033476064)|**작성자** [레인보우](http://blog.naver.com/kotaeho0512)




****




**- commons-dbcp-1.2.2.jar**




Database Pool 을 제공하는 api .







**- cglib-2.2.jar**




CGLIB는 코드 생성 라이브러리로서(Code Generator Library) 런타임에 동적으로 자바 클래스의 프록시를 생성해주는 기능을 제공한다. CGLIB를 사용하면 매우 쉽게 프록시 객체를 생성할 수 있으며, 성능 또한 우수하다. 더불어, 인터페이스가 아닌 클래스에 대해서 동적 프록시를 생성할 수 있기 때문에 다양한 프로젝트에서 널리 사용되고 있다.




**[출처]** [http://javacan.tistory.com/104](http://javacan.tistory.com/104) | **작성자** 자바캔







후후.. 잘모르고 있던 그리고 잼있어 보이는 라이브러리를 알게 되었네요.  

  
ibatis 분석이 얼추 끝나면 JTA, cglib, oscache 셋 중에 하나에 대해서 포스팅 해보겠습니다.  

  
언제가 될까.. 이제 ibatis 시작 하는뎁.....  

  
여튼 위의 라이브러리를 path에 등록 시켜 주면 빨간불이 사라 집니다.  

  
그럼 소스를 이클립스 프로젝트에 생성 해 봅시다.  

  
미리 말한대로 아래의 소스를 사용 하겠습니다.  

**  

[ibatis 샘플 예제 코드]**




package com.mydomain.data;




import com.ibatis.sqlmap.client.SqlMapClient;  

import com.ibatis.sqlmap.client.SqlMapClientBuilder;  

import com.ibatis.common.resources.Resources;  

import com.mydomain.domain.Account;




import java.io.Reader;  

import java.io.IOException;  

import java.util.List;  

import java.sql.SQLException;




public class SimpleExample {




private static SqlMapClient sqlMapper;




static {  

try {  

Reader reader = Resources.getResourceAsReader("com/mydomain/data/SqlMapConfig.xml");  

sqlMapper = SqlMapClientBuilder.buildSqlMapClient(reader);  

reader.close();  

} catch (IOException e) {  

throw new RuntimeException("Something bad happened while building the SqlMapClient instance." + e, e);  

}  

}




public static List selectAllAccounts () throws SQLException {  

return sqlMapper.queryForList("selectAllAccounts");  

}




public static Account selectAccountById (int id) throws SQLException {  

return (Account) sqlMapper.queryForObject("selectAccountById", id);  

}




public static void insertAccount (Account account) throws SQLException {  

sqlMapper.insert("insertAccount", account);  

}




public static void updateAccount (Account account) throws SQLException {  

sqlMapper.update("updateAccount", account);  

}




public static void deleteAccount (int id) throws SQLException {  

sqlMapper.delete("deleteAccount", id);  

}




}




Account.java 도 심어 놓고, sqlmapConfig.xml, Account.xml 도 넣습니다.  

  

Account.xml 과 맞도록 DB 도 구축해야 겠지요..  

  

환경에 대해서는 자세히 말씀 안드리겠습니다.  

  

........




자! 환경이 구성되었다 치고 디버깅 모드로 슬슬 읽어 봅시다~

_M#]**


  

  
=======================




프로그래머 오성민  

Phone: 010-6358-2651  

E-mail: opsbina@gmail.com  

nateon: [if-iamyou@nate.com](mailto:if-iamyou@nate.com)
