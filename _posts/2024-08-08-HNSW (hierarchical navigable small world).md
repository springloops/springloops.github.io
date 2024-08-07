---
author: springloops
comments: true
date: 20240-08-08
layout: post
title: HNSW (hierarchical navigable small world)
categories:
  - Data Mining, DL
  - ANN
  - Vector_Search
tags:
  - ANN
  - vector
  - Similarity
  - GRAPH
  - HNSW
---

**참고문서)**
- [Understanding Hierarchical Navigable Small Worlds (HNSW)](https://www.datastax.com/ko/guides/hierarchical-navigable-small-worlds)
- [스몰 월드 효과(Small World Effect; 좁은 세상 효과)](https://ltvw.tistory.com/entry/%EC%8A%A4%EB%AA%B0-%EC%9B%94%EB%93%9C-%ED%9A%A8%EA%B3%BCSmall-World-Effect-%EC%A2%81%EC%9D%80-%EC%84%B8%EC%83%81-%ED%9A%A8%EA%B3%BC)
- [Similarity Search, Part 4: Hierarchical Navigable Small World (HNSW)](https://towardsdatascience.com/similarity-search-part-4-hierarchical-navigable-small-world-hnsw-2aad4fe87d37)
- [Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs](https://arxiv.org/abs/1603.09320)



## HNSW 이해 해보기


[Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs](https://arxiv.org/abs/1603.09320)



HNSW는 NSW와 Skip List 알고리즘의 장점을 결합한 알고리즘으로, 두 알고리즘에 대해 먼저 이해할 필요가 있다.

---
- [[#1. Navigable Small Words (NSW)|1. Navigable Small Words (NSW)]]
	- [[#1. Navigable Small Words (NSW)#가장 가까운 노드를 검색할 때|가장 가까운 노드를 검색할 때]]
	- [[#1. Navigable Small Words (NSW)#그래프를 구성할 때|그래프를 구성할 때]]
	- [[#1. Navigable Small Words (NSW)#NSW의 단점|NSW의 단점]]
- [[#2. Skip Lists|2. Skip Lists]]
	- [[#2. Skip Lists#노드를 탐색할 때|노드를 탐색할 때]]
		- [[#노드를 탐색할 때#예를들어 9번 노드를 찾는다면,|예를들어 9번 노드를 찾는다면,]]
	- [[#2. Skip Lists#노드를 삽입할 때|노드를 삽입할 때]]
		- [[#노드를 삽입할 때#2번째 레이어에 7번 노드를 추가한다고 가정하면|2번째 레이어에 7번 노드를 추가한다고 가정하면]]
- [[#3. HNSW 이해 해보기|3. HNSW 이해 해보기]]
	- [[#3. HNSW 이해 해보기#그래프를 구성할 때|그래프를 구성할 때]]
		- [[#그래프를 구성할 때#동작 방식|동작 방식]]
	- [[#3. HNSW 이해 해보기#노드를 검색할 때|노드를 검색할 때]]
---

### 1. Navigable Small Words (NSW)

NSW: 조건과 가장 근사한 노드를 찾는 그래프 기반 알고리즘

탐색가능한 작은 세계 (NSW)는 모든 노드는 다른 노드로부터 적은 홉 내에 도달 할 수 있는 가정에서 이뤄진다.

*케빈 베이컨의 6단계 법칙 - 여섯다리만 건너면 세상 사람들은 모두 아는 사이다.*
![](https://blog.kakaocdn.net/dn/bsUaGH/btruoKVygQ1/V3Oer9PNsGqoxDxzi16bKk/img.jpg)


NSW는 Graph기반 알고리즘으로 각 데이터를 노드로 표현하고 노드간 관계를 연결한다.

검색 시점에 데이터가 조건(쿼리)으로 들어오면, 특정 노드로부터 시작해 이웃 노드들중 쿼리와 가장 가까운 곳으로 이동하고, 더 이상 가까운 노드가 없을 때까지 반복해서 탐색한 결과 가장 가까운 노드를 결과로 리턴한다.

#### 가장 가까운 노드를 검색할 때 

검색 시점에 데이터(쿼리)가 주어지면, 가장 유사한 노드를 찾기위해 탐색을 시작한다. 이 과정은 다음과 같이 진행된다: 
먼저, 선택된 출발 노드에서 시작하여 이웃 노드들 중에서 쿼리와 가장 가까운 노드를 찾아 이동한다. 이와 같은 방식으로, 더 이상 가까운 노드가 없을 때까지 반복적으로 탐색을 수행한다. 마지막으로, 가장 가까운 노드를 결과로 반환하게 된다.

![이미지1](https://cdn.sanity.io/images/bbnkhnhl/production/074897a472c8520eba30b2828bd04ded70d4ea33-512x317.png?w=1080&q=75&fit=clip&auto=format)

**예시)**
위 그림과 같이 그래프가 구성되어 있을 때, 
1. "A" 노드에 가장 가까운 노드를 찾기위해 "0" 노드에서 부터 탐색을 시작한다. 
2. "0" 노드에 연결된 3개의 노드에 대해서 "A"와 가장 가까운 노드를 찾고, 그 결과 가장 가까운 노드는 "1" 노드다.
3. 반복해서 "1" 노드와 연결된 3개의 노드에 대해서 "A"와 가장 가까운 노드를 찾고, 그 결과 가장 가까운 노드는 "2" 노드다.
4. 반목해서 "2" 노드와 연결된 노드 5개의 노드에 대해서 "A"와 가장 가까운 노드를 찾고, 그 결과 가장 가까운 노드는 "3" 노드다.
5. 마지막으로 "3" 노드와 연결된 노드 중 "A"와 가장 가까운 노드는 자신으로 "3" 노드가 최종적으로 가장 가까운 노드로 판단한다.


#### 그래프를 구성할 때

>[!옵션]
>- M: 연결(edge) 수
>	- 많을 수록 연산이 많아지고, 메모리가 많이 소비되지만, recall을 높을 수 있다.
>	- 

위 예시에서 쿼리 노드와 가장 유사한 노드를 탐색할 때, 쿼리와의 유사 비교 대상은 노드와 연결된 이웃 노드들이였다. 이때 연결된 노드가 많을수록 연산이 시간이 길어질 수 있음을 생각할 수 있다.

**NSW는 그래프를 구성할 때 M 옵션으로 노드당 연결해야할 최대 연결 수를 정의한다.**

M값이 클수록 노드 간의 연결이 더 많아져 밀도 높은 그래프가 되지만, 연결 수 만큼 더 많은 메모리를 사용하고 신규 노드 삽입시 대상, 검색 시 쿼리와 가까운 노드를 찾기 위한 처리 속도가 느려진다.

그래프에 신규 노드가 삽입되면 가장 가까운 M개의 노드를 검색하고, 검색된 노드들 간의 양방향 연결을 생성한다. 이때 노드간에 너무 많은 연결이 생성되면 성능에 영향을 줄 수 있기 때문에 Mmax 매개변수를 두어 노드가 가질 수 있는 최대 연결 수를 결정한다.


#### NSW의 단점
그래프의 전체 구조를 탐색하지 않고, 항상 가장 가까운 노드에 해당하는 가장 짧은 경로를 취해 탐색하기 때문에 지역 최적화(local optimum)에 빠질 수 있다.

![Nearest Node Search Value A Alternate Node 0](https://cdn.sanity.io/images/bbnkhnhl/production/e3833bc4ba83fd5871cd3d8a67529772e618c2a1-512x323.png?w=1080&q=75&fit=clip&auto=format)

위와 같은 그래프에 "A"노드를 신규 삽입하고, 탐색 시작점을 노드 "0"으로 가정하면, 
노드 "0"와 연결된 2개의 노드 중 노드 "A"와 가장 가까운 노드는 "0"이되고, 
NSW는 가장 가까운 노드만 탐색하기 때문에 이 상태에서 탐색을 멈추고 노드 "0"과 "A"를 연결하게 된다.

### 2. Skip Lists
Skip Lists 기본적으로 정렬된 linked list 형태이고, 정렬된 linked list를 다층 레이어로 구성해 빠르게 삽입 삭제 쿼리 작업을 가능하게 해주는 구조다.

![10 Node Four-Layer Skip List](https://cdn.sanity.io/images/bbnkhnhl/production/674937ac50ef6e38a89151b9442b4b1ca7567e23-470x110.png?w=1080&q=75&fit=clip&auto=format)

Skip Lists의 맨 아래 레이어에서 모든 노드는 순서대로 다음 노드에 연결된다. 레이어가 추가 될 수록 (위로 쌓음) 노드들의 연결은 건너뛰어(skip)진다.

Skip Lists에서 노드의 생성 여부는 확률로 결정되고, 보통 상위 계층으로 갈 수록 노드가 나타날 확률이 낮아지기 때문에, 상위로 갈 수록 노드 수가 줄어든다.

#### 노드를 탐색할 때
노드를 탐색할 때 
1) 최상위 레이어의 헤드에서 시작하고, 
2) 찾고자 하는 노드보다 크거나 같은 다음 요소를 찾을 때까지 연결된 다음 노드로 진행한다.
3) 다음 요소를 찾지 못할 경우 다음 레이어로 내려가고, 2) 번 작업을 반복한다.

##### 예를들어 9번 노드를 찾는다면,

![[Skip_list.svg (1).png]]

1. 최상위 4번째 레이어에서 탐색 시작, 발견된 노드가 없기 때문에 3번째 레이어로 이동
2. 3번째 레이어 1번 노드에서 4번노드로 이동, 4번 노드는 9보다 작기 때문에 다음 6번 노드로 이동
3. 3번째 레이어 6번 노드에서 다음으로 발견된 노드가 없기 때문에 2번째 레이어로 이동
4. 2번째 레이어 6번 노드에서 다음 9번 노드로 이동 
5. 완료

일반 linked list일 경우 1부터 9까지 모든 노드를 탐색해야함.

#### 노드를 삽입할 때
삽입은 노드가 포함될 레이어를 확률적으로 결정하고, 신규 노드보다 큰 노드를 찾고 연결을 유지하기 위해 보다 큰 노드 이전 노드를 기억해둔다.
신규 노드보다 큰 노드를 찾으면, 신규 노드 보다 큰 노드와 그 이전 노드의 사이에 신규 노드를 연결한다.
그런 다음 신규 노드가 최하위 레이어에 나타날 때까지 다음 레이어로 반복 진행한다.
삭제역시 비슷하게 동작함.


![[Skip_list.svg.png]]

##### 2번째 레이어에 7번 노드를 추가한다고 가정하면
- 2번째 레이어에서 7보다 큰 노드를 찾는다.
- 노드 8이 발견되고, 이전 노드 6을 기억한다
- 6과8번 노드 사이 7번 노드를 삽입 앞뒤(6,8)노드와 연결 한다.
- 1번째 레이어로 내려가 반복한다.

### 3. HNSW 이해 해보기

HNSW는 다층 계층형 구조로, NSW와 Skip Lists를 결합해 불필요한 연산을 빠르게 건너뛰고 관련 검색 영역에 집중하게 한다.

HNSW의 각 레이어(계층)는 데이터 요소들의 부분 집합을 포함하는 NSW로 표현되고, 
Skip Lists와 같이 레이어들로 구성되고 가장 상위 레이어는 노드가 가장 희박하고, 가장 하위 레이어는 모든 요소를 포함하고 있다.

HNSW는 전체 데이터셋을 저장하는 베이스 레이어에서 시작되고, 계층을 올라갈 수록 후속 상위 계층은 그 아래 계층의 단순화된 개요 역할(overview)을 한다.
상위 계층은 더 적은 수의 노드를 포함하고 그래프에서 더 긴 점프를 가능하게 한다. (skip list에서 6-9로 한번에 이동하는 효과)

Skip Lists에서 처럼 노드가 어느 레이어에 포함될지 여부는 확률적으로 결정되고, 계층이 올라갈 수록 그 확률은 기하급수적으로 감소한다. 

#### 그래프를 구성할 때

![[스크린샷 2024-08-06 오후 1.07.35.png|350]]
[노드 삽입 동작로직 - Algorithm 1](https://arxiv.org/abs/1603.09320)

> [!매개변수]
> - M: 노드당 최소 연결 수
> - Mmax: 노드당 최대 연결 수
>- Mmax0: 최하위 레이어의 노드당 최대 연결 수
>- efConstruction: 탐색시 사용되는 동적 후보 목록의 크기 
>	- 동적인 이유는 신규 노드가 할당 받은 레이어까지는 후보 수가 1이고, 할당 받은 레이어 이후 부터는 정의한 하이퍼파라라미터로 변경되기 때문인듯..
>- mL: 노드의 레이어 결정 매개변수

##### 동작 방식
1. HNSW 생성시 각 노드는 자신의 레이어 레벨을 확률적으로 결정한다. -> l
	- ![[스크린샷 2024-08-06 오후 1.14.18.png|300]]
	- <span style="color:rgb(167, 165, 165)">mL은 설정할 수 있는 파라미터로, 논문에서는 edge의 개수를 M이라고 했을 때 1/ln(M)을 최적 값으로 제안하고 있다. 레이어 레벨의 계산식에 로그가 씌워져 있어</span> <span style="color:rgb(222, 59, 208)">상위 레벨로 올라갈 수록 노드의 개수는 기하급수적으로 감소하고, 대부분의 노드 레벨은 0이다.</span>
노드가 레벨을 할당받으면 삽입은 두 단계로 이루어진다.
최상위에서 할당된 레이어까지는 최근접 이웃 1개를 찾고, 할당된 레이어에 도달하면 정의한 efConstruction 수 만큼 최근접 이웃을 찾는다.

2. 최상위 레이어로부터 시작해 greedy하게 가장 가까운 정점을 1개를 찾는다. 찾은 정점은 다음 레이어에서 진입점으로 사용된다. 반복적으로 레이어를 찾으면서 할당 받은 레이어(l)에 도착하면 다음 단계를 진행한다.
3. 신규 노드를 현재 레이어에 할당하고, 이후 설정한 efConstruction 수 만큼 가장 가까운 이웃을 찾고, M 보다 크고 Mmax 보다 적은 수의 edge를 생성한다. 반복적으로 하위 레이어에서도 처리하고 마지막 에이어0에 까지 삽입이 완료되면 알고리즘은 종료한다.

만약 할당 받은 l이 기존 최상위 레이어 L 보다 높을 경우, 레이어를 추가하고 진입점을 추가하지만, 이럴 확률은 극히 적다고...

![https://towardsdatascience.com/similarity-search-part-4-hierarchical-navigable-small-world-hnsw-2aad4fe87d37](https://miro.medium.com/v2/resize:fit:1400/1*jEGA6ZXYR0qwgrtIEJ9vWg.png)
[https://towardsdatascience.com/similarity-search-part-4-hierarchical-navigable-small-world-hnsw-2aad4fe87d37](https://miro.medium.com/v2/resize:fit:1400/1*jEGA6ZXYR0qwgrtIEJ9vWg.png)

#### 노드를 검색할 때

HNSW의 검색은 가장 긴 edge를 가진 층인 최상위 계층에서 시작한다. NSW와 마찬가지로 각 계층에서 greedy하게 local minimum에 도달할 때까지 노드를 순회한다. local minimum에 도달하면 하위 계층으로 내려가고 이 과정을 반복한다. 모든 계층에서 노드당 최대 edge 개수를 제한하면(efSearch 하이퍼파라미터) HNSW의 검색을 로그 스케일로 수행할 수 있다.

![](https://miro.medium.com/v2/resize:fit:1400/1*ziU6_KIDqfmaDXKA1cMa8w.png)[https://towardsdatascience.com/similarity-search-part-4-hierarchical-navigable-small-world-hnsw-2aad4fe87d37](https://towardsdatascience.com/similarity-search-part-4-hierarchical-navigable-small-world-hnsw-2aad4fe87d37)

각 계층에서 가장 가까운 이웃을 하나만 찾는 대신 쿼리 벡터에 가장 가까운 efSearch를 찾고 이러한 각 이웃을 다음 계층의 진입점으로 사용한다. (efConstruction과 유사)
