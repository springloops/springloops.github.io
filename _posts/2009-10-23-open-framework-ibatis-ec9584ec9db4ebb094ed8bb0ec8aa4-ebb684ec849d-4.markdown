---
author: springloops
comments: true
date: 2009-10-23 08:23:34+00:00
layout: post
link: https://springloops.wordpress.com/2009/10/23/open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-4/
published: false
slug: open-framework-ibatis-%ec%95%84%ec%9d%b4%eb%b0%94%ed%8b%b0%ec%8a%a4-%eb%b6%84%ec%84%9d-4
title: 'Open FrameWork - iBATIS : 아이바티스 분석 (4)'
wordpress_id: 14
categories:
- Open Source
---

  

요즘 개인적으로 아주아주아주아주 안좋은 일이 생겨서...  

  
정신이 없었습니다.  

  
모 지금도 마찬가지이지만, 하던거 계속 하려고 합니다.  

  
생각보다 분량만 많아 지는 느낌이 듭니다만 더 잘 써볼수 있도록 하겠습니다.  

  
[2009/10/14 - [분류 전체보기] - Open FrameWork - iBATIS : 아이바티스 분석 (3)](http://www.somethingbook.net/entry/Open-FrameWork-iBATIS-아이바티스-분석-3)  

[2009/10/12 - [분류 전체보기] - Open FrameWork – iBATIS : 아이바티스 분석 (2)](http://www.somethingbook.net/entry/Open-FrameWork-–-iBATIS-아이바티스-분석-2)  

[2009/07/19 - [분류 전체보기] - Open FrameWork – iBATIS : 아이바티스 분석 (1)](http://www.somethingbook.net/entry/Open-FrameWork-–-iBATIS-아이바티스-분석-1)  

  
지난번에 Resource를** **알아보았고, 이어서 SqlMapClient의 생성과 내부 역할에 대해서 알아보겠습니다.  

  
  

  

[#M_더보기|접기|**SqlMapClientBuilder  

  
**SqlMapClient를 생성하는 과정을 알아보기 위해 먼저 SqlMapClientBuilder 에 대해서 알아 보겠습니다  

  
SqlMapClientBuilder 가 SqlMapClient 를 생성해 주는 펙토리 패턴을 사용한 클래스 이기 때문입니다.  

  
SqlMapClientBuilder 클래스를 열어 보면 알수 있는데 단순히 SqlMapConfigParser 클래스의 래퍼로써   

  
SqlMapConfigParser 클래스의 parse() 메소드에 읽어 드린 Resource (SqlMapConfig.xml) 를 파싱하도록 위임 하고 있습니다.  

  
실제 SqlMapConfigParser 클래스의 parse() 메소드가 SqlMapConfig.xml를 파싱하여 SqlMapClient 를 리턴 합니다.  

  
아마도! 맵 형태의 SqlMapClient 객체를 리턴하지 않을까 합니다.  

(SqlMapClientBuilder 의 buildSqlMapClient () 메소드 역시 다양한 형태의 Resource 를 받기 위해 오버로드 되어 있습니다.)  

  
예제 코드로 이 부분입니다.  

sqlMapper = SqlMapClientBuilder.buildSqlMapClient(reader);  

  
이제 두번째 줄이군요. 이글을 이어서 쓰는게... 얼마만인지...  

  
여튼 확인해보지요.  

  
  

**SqlMapConfigParser  

  
  

**  

  
_M#]  

  
  

======================= 


프로그래머 오성민  

Phone: 010-6358-2651  

E-mail: opsbina@gmail.com  

nateon: [if-iamyou@nate.com](mailto:if-iamyou@nate.com)
