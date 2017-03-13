---
author: springloops
comments: true
date: 2009-10-14 04:17:01+00:00
layout: post
link: https://springloops.wordpress.com/2009/10/14/open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-3/
slug: open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-3
title: 'Open FrameWork - iBATIS : 아이바티스 분석 (3)'
wordpress_id: 11
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

  




이상한 말을 하느라 ibatis 내용은 실제 하나도 안했는데... 벌써 제목만 3번이네요.ㅎㅎㅎ  

  
이제 진짜 한번 따라가 봅시다~








[#M_내용보기|접기|


**1. Resources 생성!  

  
**static 영역 부터 흐름을 따라가 봅시다.  

  
Reader reader = Resources.getResourceAsReader("com/mydomain/data/SqlMapConfig.xml");  

  
정확하게 기억이 나진 않지만 ㅡㅡ;;;  

  
스트럿츠 역시 프레임워크에서 필요한 리소스를 읽을때 위와 같이 자체적으로 파일을 읽는 객체를 만들어   

  
사용하고 있었던거 같아요.  

  
그럼 java io 에 있는 File 클래스를 사용해서 읽는 것과 어떤 차이점이 있는지 알아볼까요?ㅎㅎㅎ  

  
자 ibatis 의 Resources 클래스를 까발라 봅시다.![](http://somethingbook.wordpress.com/wp-includes/js/tinymce/plugins/wordpress/img/trans.gif)  

  
Resources 클래스는 ibatis 프레임웍안에 com.ibatis.common.resources 밑에 존재 하네요.  

  
  

API 를 보면 다음과 같은 메소드가 있습니다.  

  
많아 보이지만 하는일은 참 단순하니 릴렉스 하고 함 봅시다.  

  
public static java.lang.ClassLoader getDefaultClassLoader()  

public static void setDefaultClassLoader(java.lang.ClassLoader defaultClassLoader)  

  
public static java.net.URL getResourceURL(java.lang.String resource)  

public static java.net.URL getResourceURL(java.lang.ClassLoader loader, java.lang.String resource)  

  
public static java.io.InputStream getResourceAsStream(java.lang.String resource)  

public static java.io.InputStream getResourceAsStream(java.lang.ClassLoader loader, java.lang.String resource)  

  
public static java.util.Properties getResourceAsProperties(java.lang.String resource)  

public static java.util.Properties getResourceAsProperties(java.lang.ClassLoader loader, java.lang.String resource)  

  
public static java.io.Reader getResourceAsReader(java.lang.String resource)  

public static java.io.Reader getResourceAsReader(java.lang.ClassLoader loader, java.lang.String resource)  

  
public static java.io.File getResourceAsFile(java.lang.String resource)  

public static java.io.File getResourceAsFile(java.lang.ClassLoader loader, java.lang.String resource)  

  
public static java.io.InputStream getUrlAsStream(java.lang.String urlString)  

public static java.io.Reader getUrlAsReader(java.lang.String urlString)  

public static java.util.Properties getUrlAsProperties(java.lang.String urlString)  

  
public static java.lang.Class classForName(java.lang.String className)  

  
public static java.lang.Object instantiate(java.lang.String className)  

public static java.lang.Object instantiate(java.lang.Class clazz)  

  
public static java.nio.charset.Charset getCharset()  

public static void setCharset(java.nio.charset.Charset charset)  

  
공통된 것 끼리 묶어서 나열 해봤습니다.  

  
주된 일은 인자로 받는 정보를 다음과 같은 객체에 담아 넘겨 주고 있습니다.  

  
URL, InputStream, Properties, Reader, File, Class, Object  

  
자신이 가지고 있는 파일의 형식이나 처리 하기 편한 객체형을 선택해서 리소스를 불러 올수있게 되어 있습니다.  

  
이런건 다른 프로젝트에서도 사용할수 있을것 같습니다.  

  
샘플 코드에서는




Reader reader = Resources.getResourceAsReader("com/mydomain/data/SqlMapConfig.xml");  

sqlMapper = SqlMapClientBuilder.buildSqlMapClient(reader);  

reader.close();  

  
위와 같이 리더로 받아서 sqlMapper 를 생성하고 있습니다.  

  
SqlMapClientBuilder 클래스 역시 다양한 형태의 리소스를 받기 위해 다양한 인자를 받는 buildSqlMapClient 팩토리 메소드를 오버로드를 이용하여 보유 하고 있겠네요.  

  
buildSqlMapClient 메소드는 넘겨 받은 리소스 객체를 내부적으로 파싱하고 SqlMapConfig 객체에 담아 리턴 하겠지요.  

  
어디 진짜 그런지 Resources.getResourceAsReader 메소드를 한번 읽어 보고, SqlMapClientBuilder.buildSqlMapClient() 메소드를 읽어 보죠.  

  
Resources.getResourceAsReader 메소드는 다음과 같이 InputStreamReader로   

  
리소스(SqlMapConfig.xml) 파일을 읽어서 Reader 클래스로 리턴 하고 있습니다.  

  
(역시 내부적으로 다양한 형식을 처리 할수 있는 오버로드 된 메소드들이 있네요.)  

  
public static Reader getResourceAsReader(String resource) throws IOException {  

Reader reader;  

if (charset == null) {  

reader = new InputStreamReader(getResourceAsStream(resource));  

} else {  

reader = new InputStreamReader(getResourceAsStream(resource), charset);  

}  

  

return reader;  

}  

  
  

소스를 보다 보니 알아두면 나중에 삽질을 조금 줄일수 있을것 같아 한마디 더 적어 봅니다.  

  
charset 이 null 이 아닐 경우 charset 멤버 변수에 담겨 있는 케릭터셋으로 Reader 를 생성합니다.  

  
charset 의 설정은 위 메소드가 나열 되어 있듯 setCharset(java.nio.charset.Charset charset) 로 설정합니다.  

  

String 타입이 아닌 nio 의 Charset 객체를 받고 있습니다.  

  
또한 charset 의 값이 null 일경우는 시스템 기본 케릭터 셋을 따라 갑니다.  

  

그럴 경우 unicode 같은 케릭터 셋은 제대로 작동안 할수 있다고 주석으로 나와 있네요.  

  
왠만하면 케릭터 셋을 정의 해주는 것이 좋겠습니다.  

  
  



_M#]  

  
  

  
======================= 


프로그래머 오성민  

Phone: 010-6358-2651  

E-mail: opsbina@gmail.com  

nateon: [if-iamyou@nate.com](mailto:if-iamyou@nate.com)
