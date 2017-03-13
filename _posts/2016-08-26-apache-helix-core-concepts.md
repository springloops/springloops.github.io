---
author: springloops
comments: true
date: 2016-08-26
layout: post
title: 'Apache Helix Core Concepts'
categories:
- OPEN SOURCE/Apache Helix
tags:
- apache Helix
- opensource
---

Apache Helix 개념 정리

원문 : [http://helix.apache.org/Concepts.html](http://helix.apache.org/Concepts.html)

# Apache Helix

**Zookeeper 를 사용한 복제 및 파티션된 리소스 분산 클러스터 관리 프레임워크**

## Concepts

Helix 다음과 같이 주어진 특성과 관련된 작업(task)에 대한 발상에 기반한다. 

* Location, 이 노드 N1 이용가능하다.
* State, 실행 정지등.

Helix 용어에서 task 는 **resource** 라고 한다.

### Ideal State

**IdealState** 는 하나의 위치(location)과 상태(state)로 작업(task)를 매핑할 수 있다. Helix 에서 표준 표현 방법은 다음과 같다.

```
"TASK_NAME" : {
	"LOCATION" : "STATE"
}
```

리소스(resource) "myTask" 를 node "N1" 에서 실행하기 윈한 간단한 경우를 생각해보자. 아래와 같이 IdealState 로 표현될 수 있다.

```
{
	"id" : "MyTask",
	"mapFields" : {
		"myTask" : {
			"N1" : "ONLINE"
		}
	}
}
```

### Partition

만일 하나의 상자에 비해 너무 큰 작업(task)을 얻을 경우, 하위 작업 (subtasks)으로  분할 할 수 있다. 각 하위 작업 (subtask)는 Helix 안에서 **partition** 으로 불린다. task 를 3개의 subtasks/partitions 으로 나누고 싶다면, IdealState 아래 보이는 것처럼 바꿀수 있다.
"myTask_0", "myTask_1", "myTask_2" 는 MyTask 의 partitions을 나타내는 논리적인 이름이다. 각 작업(task)는 N1, N2 그리고 N3에서 각자 실행된다.

```
{
	"id" : "MyTask",
	"simpleFields" : {
		"NUM_PARTITIONS" : "3"
	},
	"mapFields" : {
		"myTask_0" : {
			"N1" : "ONLINE"
		},
		"myTask_1" : {
			"N2" : "ONLINE"
		},
		"myTask_2" : {
			"N3" : "ONLINE"
		}
	}
}
```

### Replica

Partitioning 은 하나의 data/task 를 여러 하위부분(subparts)에 분할 할 수 있다. 하지만 각 partition의 속도 증가 요구애 대해 말해보자. 기본 해결 방법은 각 partition에 대한 여러 복사본을 갖는 것이다. Helix 는 partition을 복사본을 **replica** 라고 부른다. replica 를 추가하면 실패시 시스템이 가용성을 향상 시킬수 있다. 검색 시스템에서 자주 사용되는 이 방법을 볼수 있다. 인덱스는 shard 안으로 분리되고, 각 shard 는 여러 복사본을 갖는다.

각 task 에 대해 replica 를 추가하길 원한다면, IdealState는 간단하게 보이는 것 처럼 바꿀수 있다.

```
{
	"id" : "MyTask",
	"simpleFields" : {
		"NUM_PARTITIONS" : "3",
		"REPLICAS" : "2"
	},
	"mapFields" : {
		"myIndex_0" : {
			"N1" : "ONLINE",
			"N2" : "ONLINE",
		},
		"myIndex_1" : {
			"N2" : "ONLINE",
			"N3" : "ONLINE"
		},
		"myIndex_2" : {
			"N3" : "ONLINE",
			"N1" : "ONLINE"
		}
	}

}
```

### State

작업(task)이 데이터베이스를 나타내는 조금 더 복잡한 시나리오를 보자. 일반적인 read-only 인 인덱스와 같지 않게, database 는 read, write 를 지원한다. replicas 사이에서 일관성 있는 데이터를 유지하는 것은 분산 데이터 저장에 매우 중요한다. 일반적으로 적용되는 기술은 하나의 replica 에 MASTER 를 나머지는 replicas는 SLAVE 로 할당하는 것이다. 모두 MASTER 에서 쓰고 SLAVE로 복제된다.

Helix 는 각 replica에 다른 상태(**states**) 를 할당 할 수 있다. 두개의 MySql 인스턴스 N1, N2 를 가지고 있고, 하나를 MASTER로, 다른 하나를 SLAVE로 제공한다고 해보자. IdealState 는 다음과 같이 바꿀 수 있다.

```
{
	"id" : "myDB",
	"simpleFields" : {
		"NUM_PARTITIONS" : "1",
		"REPLICAS" : "2"
	},
	"mapFields" : {
		"myDB" : {
			"N1" : "MASTER",
			"N2" : "SLAVE"
		}
	}
}
```

### State Machine and Transitions

IdealState 하나에 정확하게 클러스터의 원하는 상태를 지정할 수 있다. 주어진 IdealState, Helix는 클러스터가 IdealState에 도달하도록 보장하는 책임을 갖는다. Helix **Controller**는 IdealState를 판독하고 그것이 IdealState와 일치 할 때까지 하나의 상태에서 다른 상태로 변환하기 위한 적절한 조치를 취할 각 참가자(Participant)에게 명령한다. Helix에서는 이 행동을 **transitions** 이라고 한다.

다음 논리적 질문은 : 어떻게 controller는 이상적인 상태(IdealState)에 도달하는 데 필요한 전환을 계산합니까?
이것은 유한 상태 기계(**[Finite State Machine](http://showmiso.tistory.com/156)**) 개념을 가져온다. 

`어려운 말인듯 싶으나, 정의된 상태(State)와 전이(Transistion)가 있고, 이벤트 발생시 현재 상태에 맞는 전이 과정을 수행한다 정도로 생각하면 될듯`

유한 상태 기계의 사용 목적
* 가능한 상태들을 명확히 규정할 수 있다.
* 상태 중복을 피할 수 있다.
* 전이들을 명확하게 규정할 수 있다.
* 따라서 기계의 동작을 분명하게 규정할 수 있다.
* 프로그래밍에서 FSM에 기반한 객체를 만든다면, 안정적인 작동을 보장할 수 있다.

Helix 는 응용 프로그램이 유한 상태 머신(Finite state machine)에 플러그인 되도록 허용한다.
state machine 은 다음과 같이 구성된다.

* **State(상태)**: replica 의 역할 설명
* **Transition(전이)**: 엑션에 따라 replica 상태에서 다른 하나의 상태로 이동 할 수있다, 따라서  역할이 바뀐다.

여기 MasterSlave 상태 머신의 예가 있다.

```
          OFFLINE  | SLAVE  |  MASTER
         _____________________________
        |          |        |         |
OFFLINE |   N/A    | SLAVE  | SLAVE   |
        |__________|________|_________|
        |          |        |         |
SLAVE   |  OFFLINE |   N/A  | MASTER  |
        |__________|________|_________|
        |          |        |         |
MASTER  | SLAVE    | SLAVE  |   N/A   |
        |__________|________|_________|
```

Helix 의 각 resource 는 하나의 상태 머신과 연관될 수 있다. 이것은 같은 클러스터안에서 인덱스 또는 다른 데이터베이스 같은 하나의 Resource를 갖을 수 있다는 것을 의미한다. 다음 보이는 것 처럼, 상태 머신과 각 리소스를 연결할 수 있다;

```
{
  "id" : "myDB",
  "simpleFields" : {
    "NUM_PARTITIONS" : "1",
    "REPLICAS" : "2",
    "STATE_MODEL_DEF_REF" : "MasterSlave",
  },
  "mapFields" : {
    "myDB" : {
      "N1" : "MASTER",
      "N2" : "SLAVE",
    }
  }
}

```

### Current State

resource의 **CurrentState**는 단순히 참여하는 노드, 참가자(**Participant**)의 실제 상태를 나타낸다. 아래 예 에서 :

* INSTANCE_NAME: 프로세스를 나타내는 고유한 이름.
* SESSION_ID: 프로세스가 클러스터에 참여 할때마다 자동으로 할당되는 ID

```
{
  "id":"MyResource"
  ,"simpleFields":{
    ,"SESSION_ID":"13d0e34675e0002"
    ,"INSTANCE_NAME":"node1"
    ,"STATE_MODEL_DEF":"MasterSlave"
  }
  ,"mapFields":{
    "MyResource_0":{
      "CURRENT_STATE":"SLAVE"
    }
    ,"MyResource_1":{
      "CURRENT_STATE":"MASTER"
    }
    ,"MyResource_2":{
      "CURRENT_STATE":"MASTER"
    }
  }
}
```

클러스터의 각 노드는 자신의 CurrentState 를 갖는다.

### External View

참가자(Participant)들과 통신하기 위해서, 외부 Client는 각 참자자 (Participant)들의 상태를 알 필요가 있다. external client는 **spectators**라고 불린다. spectator의 간단한 수명을 확인하기 위해, Helix는 모든 노드의 current state 의 집계 뷰(view) 인 ExternalView 를 제공한다.
ExternalView 는 IdealState와 유사한 형식을 갖는다.

```
{
  "id":"MyResource",
  "mapFields":{
    "MyResource_0":{
      "N1":"SLAVE",
      "N2":"MASTER",
      "N3":"OFFLINE"
    },
    "MyResource_1":{
      "N1":"MASTER",
      "N2":"SLAVE",
      "N3":"ERROR"
    },
    "MyResource_2":{
      "N1":"MASTER",
      "N2":"SLAVE",
      "N3":"SLAVE"
    }
  }
}

```

### Rebalancer

Helix의 핵심 구성 요소는 모든 클러스터 이벤트에 Rebalancer 알고리즘을 실행하는 컨트롤러다. 클러스터 이벤트는 다음 중 하나가 될수 있다.

* 노드들의 시작과 중지
* 노드들의 soft/hard 실패에 대한 감지
* 사로운 노드들의 추가/삭제
* Ideal state 변화

이러한 구성 변경과 같은 몇가지 더 많은 예제가 있다. 중요한건 : rebalancer를 트리거 하는 방법에는 여러가지가 있다.

Rebalancer 프로그램이 실행되면 단순히 다음을 수행한다.

* ideal state 와 current state를 비교한다.
* 전이(Transition)이 ideal state가 도달하는데 필요한 계산을 한다.
* 각 참가자(Participant)로 전환(Transistion) 발행

위 단계는 시스템의 모든 변화를 발생한다. 한번 current state 가 IdealState와 일치 하면, 시스템은 'IdealState = CurrentSate = ExternalView' 의미로 안정적인 것으로 간주 된다.

### Dynamic IdealState

Helix를 강력하게 만드는 것들 중 하나는 IdealState를 동적으로 변경 할 수 있다는 점이다. 이것은 하나의 노드 실패와 같은 클러스터 이벤트를 수신하고, 동적으로 이상적인 상태를 변경할 수 있음을 의미한다. Helix는 시스템의 각각의 전이(Transition)를 트리거처리 한다.

Helix는 ideal state 를 조정하는 다양한 보장을 한다. 언제든 클러스터 이벤트는 발생하고, 세개 모드 중 하나로 Helix는 작동할 수 있다.

* FULL_AUTO : Helix는 자동 제약에 기초하여 각각의 replica 의 location 과 state를 결정한다.
* SEMI_AUTO : Helix의 각 replica 에 살수 있는 location는 "preference list"에서 가져오고, 자동 제약에 기초하여 state를 결정한다.
* CUSTOMIZED: Helix will take in a map of location to state and fire transitions to get the external view to match


[http://helix.apache.org/Architecture.html](http://helix.apache.org/Architecture.html) 도 보자.

| common | Helix | 
|--------|-------|
| IdealState | Location : State 매핑 구조 |
| 작업   | Resource |
| 분할된 하위 작업 | Partition|
| 작업의 복사본 | Replica |

