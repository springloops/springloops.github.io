---
author: springloops
comments: true
date: 2014-06-09 00:44:14+00:00
layout: post
link: https://springloops.wordpress.com/2014/06/09/predictionio-concepts/
slug: predictionio-concepts
title: PredictionIO Concepts
wordpress_id: 102
categories:
- Open Source/PredictionIO
tags:
- machine learning
- predictionIO
---

In-house recommendation system 개발시 참고할 목적으로 PredictionIO 문서를 읽어봤습니다.

읽으면서 잊지 않으려고 멍멍이 발 번역, 정리한 내용입니다.

개떡같이 썼어도 찰떡같이 이해하는 센스!

# Concepts

## BASICS OF PREDICTIONIO

![](http://docs.prediction.io/current/_images/concepts-app-engine-algo.png)

#### PredictionIO Server의 역할

  * Data 수집
  * 예측결과 REST API 제공

#### PredictionIO Server의 구성

  * **App**  

    * Server 안에 **App은 database 의 DB 혹은 Collection이다**. (ex: orcle -> table space, mysql -> DB, mongoDB -> DB.....)
    * 관계 데이터(Relevant Data), 사용자 행동등의 **데이터는 App에 수집**된다.
    * App은 하나 또는 여러개의** 예측 엔진(prediction engine)을 포함**한다.
    * App Data는 엔진들 사이에서 **공유**된다.  
  

  * **Engine**  

    * 하나의 **엔진은 반드시 prediction type (or engine type), 에 속해야** 한다. (type = Item Recommendation Item Similarity 내장 타입)
    * 각 **엔진은 데이터를 처리**하고, 각 **엔진별 독립적으로 예측 모델(predictive model)을 구성**한다.
    * 따라서, **모든 엔진은 자신의 예측 결과 셋을 제공**한다.  

      * 예를 들어, App 안에 두개의 engine을 만들고, 하나의 엔진은 사용자에게 뉴스를 추천, 다른 하나는 사용자에게 친구를 제안하는 형태..
    * 알고리즘은 반드시 각 엔진에 배포되어야 한다.  
  

  * **Algorithm**  

    * 내장 알고리즘의 수는 엔진의 각 타입에 사용할 수 있다.
    * 알고리즘, 파라미터의 설정은 예측모델(predictive model)을 어떻게 구성할지 결정한다.
    * 예측 정확도와 성능은 상황에 맞는 적당한 알고리즘과 파라미터 설정을 통해 향상시킬 수 있다.
    * 알고리즘 평가 툴 제공...

## Data Collection

#### PredictionIO의 데이터 구조

PredictionIO App은 주로 3가지 타입의 데이터를 수집한다.

  * **User Data**  

    * required = UserID (String)
    * 각 User 레코드는 어플리케이션에서 고유한 User 또는 Customer 다.
    * 필수 속성은 UserId 이고, 일반적으로 database의 user id 와 일치한다.
    * 또한, 나이, 성별, 위치, 소속 등의 여분의 데이터를 제공할 수 있다.  

  * **Item Data**  

    * required = ItemID (String), Item type(String Array)
    * Item 레코드는 Object 다.
    * Object는 book, deal, music 등 어떤 컨텐츠이던간에 무엇이든 될 수 있다.
    * Item 레코드는 2개의 필수 속성이 있다. ItemID, Item type
    * 이외의 추가 데이터를 제공할 수 있다.
    * UserID와 유사하게 ItemID는 일반적으로 database의 deal id 와 일치 한다.
    * Item type은 String이고, Type은 item의 항목을 구분합니다.   
  

  * **Behavioral Data**  

    * User-to-Item, User-to-User의 행위는 behavioral data로 수집된다.
    * 이것은 예측모델(Predictive model)을 구성하기 위해 사용된다.
    * behavior 레코드는 다음과 같이 정의된다. => User _A_ **likes** Item _X, **like **_는 User-to-Item 의 action type이다.  

      * **PredictionIO의 내장 Action Type**  

        * **like : 사용자가 좋아함**
        * **dislike : 사용자가 싫어함**
        * **rate : 사용자의 평가. 1~5 사이의 단계 점수 : 1 낮음, 5 높음, 3 중립. 커스터마이징 가능**
        * **view : 유저가 본 Item, 분명히 표현하지 않은 선호가 될수 있다. **
        * **conversion : 사용자의 분명히 표현한 강한 선호. ex) 구매, 다운로드 등..****  
**
    * **Action type 역시 커스터마이징 가능하다.**

## Predictive Modeling

  * 예측 모델링은 모델을 구축하거나 훈련시키는 과정
  * 모델의 정확도와 성능은 선택한 알고리즘과 해당 알고리즘의 파라미터 설정에 의해 결정
  * PredictionIO에서 모든 엔진은 예측모델을 독립적으로 관리  

    * 다른 말로, 각 엔진에 하나의 알고리즘이 배포되어진다.
  * 엔진 타입(**engine type**)의 구현에 따라, 시스템은 **수집된 데이터를 배치 또는 실시간으로 훈련** 시킬수 있다.

#### Algorithm

때론 learning Algorithm이라고 불린다.

데이터로부터 어떻게 예측 학습 방법을 결정한다.

엔진의 각 유형은 사용할 수 있는 내장 알고리즘을 제공한다.

새로운 엔진을 생성할때, 알고리즘은 기본 파라미터와 함께 배포된다.

사용자에 필요에 따라, 다른 하나를 배포할 수 있다.

#### Algorithm Parameter

일부 알고리즘은 파라미터 값을 명시해야 한다.

파라미터값은 알고리즘의 학습 방법을 조정한다.

#### Choose an Algorithm

알고리즘은 가정 및 이론에 매우 의존한다.

상황에 딱 맞는 하나의 해결책은 없다.

특정 상황에 대한 예측 정확도를 개선하기 위해, 알고리즘과 파라미터의 다양한 조합을 평가 해야한다.

[정확도 최적화 방법은 여기서 배우자.](http://docs.prediction.io/current/concepts/optimization.html)

  


## PREDICTION SERVING

예측 모델(predictive model)과 함께, 엔진의 예측 결과를 REST API를 통해 조회할 수있다.

엔진의 유형에 의존하여, 배치 처리 결과나 실시간 결과와 다양한 필터링 그리고 랭킹 옵션을 통해 검색할 수 있다.

자세한건 [Endpoints API](http://docs.prediction.io/current/apis/index.html)를 참고 하라...

  


## SYSTEM DESIGN OF PREDICTIONIO

[이건 그림이니까... 직접 가서 보자.](http://docs.prediction.io/current/concepts/systemdesign.html)

  


  

