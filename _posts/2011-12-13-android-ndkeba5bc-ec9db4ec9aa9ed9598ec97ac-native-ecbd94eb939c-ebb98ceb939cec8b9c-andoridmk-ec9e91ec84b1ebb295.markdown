---
author: springloops
comments: true
date: 2011-12-13 07:01:24+00:00
layout: post
link: https://springloops.wordpress.com/2011/12/13/android-ndk%eb%a5%bc-%ec%9d%b4%ec%9a%a9%ed%95%98%ec%97%ac-native-%ec%bd%94%eb%93%9c-%eb%b9%8c%eb%93%9c%ec%8b%9c-andoridmk-%ec%9e%91%ec%84%b1%eb%b2%95/
slug: android-ndk%eb%a5%bc-%ec%9d%b4%ec%9a%a9%ed%95%98%ec%97%ac-native-%ec%bd%94%eb%93%9c-%eb%b9%8c%eb%93%9c%ec%8b%9c-andoridmk-%ec%9e%91%ec%84%b1%eb%b2%95
title: Android NDK를 이용하여 Native 코드 빌드시 Andorid.mk  작성법?
wordpress_id: 39
categories:
- Tip
tags:
- android.mk
- JNI
---


# 이 Android.mk에 언급된 여러 지역 파일들의 기준 경로 $(call my-dir)는 현재 폴더(Android.mk가 존재하는 폴더)를 뜻함.  

**
LOCAL_PATH := $(call my-dir)**





  







**
include $(CLEAR_VARS)**





  

# 이 변수의 값은 make 명령의 주된 빌드 대상이름으로 쓰임, 출력 파일(공유 라이브러리 파일)의 이름에도 포함된다.





**
LOCAL_MODULE  := native_filter**  

  

# 컴파일 플레그들로, 현재 라이브러리에 필요한 것들만 지정한다.  

**
LOCAL_CFLAGS := -Wall -02 -fpic **  

  

# 컴파일할 C 소스 파일들을 나열





**
LOCAL_SRC_FILES := ImageFilter.c **





**				 Spline.c**





  







# 추가 라이브러리를 사용할경우 아래 작성 필요!





# 이 라이브러리에 링크할 외부 라이브러리들을 지정한다.   

#where : NDK/platforms/<level>/arch-arm/usr/include





#math.h : m





#log.h : log





#jnigraphics : bitmap.h  







**
LOCAL_LDLIBS	:= -llog**





  







**
include $(BUILD_SHARED_LIBRARY)**  

  

  

<NDK_rx> /docs의 문서를 추가 참고~
