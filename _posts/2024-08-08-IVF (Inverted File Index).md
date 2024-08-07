---
author: springloops
comments: true
date: 20240-08-08
layout: post
title: IVF (Inverted File Index)
categories:
  - Data Mining, DL
  - ANN
  - Vector_Search
tags:
  - ANN
  - vector
  - Similarity
  - IVF
---
**참고문서)**

- [검색증강생성(RAG) - 벡터 인덱스 기초 및 IVF](https://jerry-ai.com/28)
- [Understand Inverted File Flat Vector Indexes](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/understand-inverted-file-flat-vector-indexes.html)
- [Faiss: The Missing Manual - Inverted File Index](https://www.pinecone.io/learn/series/faiss/vector-indexes/#Inverted-File-Index)

  

  

## Inverted File Index란

  

클러스터링을 통한 검색 범위를 축소하는 방식으로 구성된다.  
사용하기 쉽고 검색 품질이 높으며 속도가 합리적이라 인기있는 인덱스라고 한다.

디리클레 테셀레이션(훨씬 더 멋진 이름)이라고도 불리는 보로노이 다이어그램(Voronoi diagram)의 개념을 기반으로 작동한다.

보로노이 다이어그램을 이해하기 위해 고차원 벡터를 2D 공간에 배치하는 것을 상상해 보면,  
2D 공간에 몇 개의 점을 추가로 배치하면 '클러스터'(이 경우 보로노이 셀) 중심이 된다. 그런 다음 각 중심으로부터 동일한 반지름을 확장하고, 어느 시점에서 각 셀 원의 둘레가 서로 충돌하여 셀 가장자리가 만들어진다.

  

옵션

- nlist: 생성할 클러스터 수
    - 클러스터수가 작으면, 해당 클러스터에 많은 노드들이 추가된다.
- nprobe: 검색시 사용할 클러스터 수

  

  

| ![[스크린샷 2024-08-06 오후 3.17.13 (2).png]] | ![[스크린샷 2024-08-06 오후 3.17.18 (2).png]] |     |
| --------------------------------------- | --------------------------------------- | --- |

  

### 역 인덱스

역 인덱스(inverted index, 또는 역색인)는 **키워드를 통해 데이터를 찾아내는 방식**이다. 보통 책의 가장 뒷부분에서 볼 수 있는, '찾아보기' 파트가 역 인덱스의 예로 볼 수 있다.

역 인덱스 방식에서는 데이터가 증가하더라도 검색해야 할 행의 수(키워드)가 증가하는 것이 아니라, 역 인덱스가 가리키는 ID의 배열 값(페이지 번호)만 추가되기 때문에 큰 속도 저하 없이 빠른 검색이 가능하다.   
![](https://blog.kakaocdn.net/dn/zWysk/btsEGRCN6JP/dsxcptcqntKntKrdPKChak/img.png)

  

### k-means 클러스터링

IVF의 핵심은 데이터를 사전에 유사한 것들끼리 모아 몇 개의 군집으로 묶어놓는 것이다.

즉, 클러스터링 작업이라 할 수 있는데  k-means(k-평균) 클러스터링은 IVF에서 가장 보편적으로 사용되는 클러스터링 알고리즘이다.

k-means 클러스터링은, 별도의 레이블이 없는 데이터에 적용할 수 있는 비지도(unsupervised) 클러스터링 알고리즘 중 하나이다. 

k-means 클러스터링의 과정은 다음과 같다:

1. 먼저 k개 만큼의 클러스터에 대한 중심점(centroid)을 임의로 설정한다.
2. 중심점과 다른 데이터에 대한 거리 계산을 반복하면서, 중심점이 해당 군집을 더 잘 표현할 수 있도록(=평균에 가깝도록) 중심점 좌표를 업데이트한다.
3. 이 작업을 중심점의 변화가 미비해지는 어떤 기준에 이를 때까지 충분히 반복한다.
4. 최종적으로, 데이터들은 가장 가까운 중심점에 해당하는 클러스터로 할당되면서 k개의 클러스터가 형성된다. 

  

![](https://blog.kakaocdn.net/dn/bNnZni/btsEEYJOLSD/5RYJ3Syr3Vd9UAMhxHvx6k/img.png)

  

### 탐색

만약 100개의 데이터를 10개의 클러스터로 나누었다면, 쿼리가 입력되었을 때 100개 데이터에 모두 유사도 계산을 수행하지 않고 10개 클러스터의 대푯값(k-means 클러스터링에서는 중심점)들에 대해서만 유사도 계산을 수행한다.

이후, 입력 쿼리가 속하는 클러스터의 데이터에 대해서만 세부 검색을 다시 수행할 수도 있다. 만약 데이터가 균일하게 10개씩 클러스터링되었다면, 20회 이하의 계산으로 검색을 마칠 수 있다.

물론, 대푯값이 해당 클러스터를 100% 대표할 수 없기에 완벽한 정확도의 검색은 어렵지만, 이렇게 벡터 공간을 사전에 세분화해놓으면 데이터 양이 많아질 때 검색 효율을 크게 높일 수 있다.

  

## 단점

- K means 처럼 K 값이 설정되면 증분이 어렵다.
- 가장 자리 문제(edge problem)가 발생한다.  
    ![](https://www.pinecone.io/_next/image/?url=https%3A%2F%2Fcdn.sanity.io%2Fimages%2Fvr8gru94%2Fproduction%2F5a44e6ded9916f127a76d45708baa20e02802574-700x437.png&w=1920&q=75)
    - 검색시 쿼리 벡터Q는 셀 중 하나에 위치해야 하며, 이 시점에서 검색 범위를 해당 단일 셀로 제한합니다. (빨간 클러스터)
    - 하지만 쿼리 벡터가 셀의 가장자리 근처에 위치할 경우 가장 가까운 다른 데이터 포인트가 다른 클러스터에 있을 수 있다. (초록색 클러스터)
    - 해결 방법은 nprobe 값을 늘려 검색할 클러스터 수를 늘리는 방법이 있다