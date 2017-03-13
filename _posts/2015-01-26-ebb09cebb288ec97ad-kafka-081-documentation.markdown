---
author: springloops
comments: true
date: 2015-01-26 15:47:44+00:00
layout: post
link: https://springloops.wordpress.com/2015/01/26/%eb%b0%9c%eb%b2%88%ec%97%ad-kafka-081-documentation/
slug: '%eb%b0%9c%eb%b2%88%ec%97%ad-kafka-081-documentation'
title: '[발번역] Kafka 0.8.1 Documentation - 1.1 Introduction'
wordpress_id: 109
categories:
- Open Source/Apache Kafka
tags:
- Apache
- apache kafka
- DOC
- Kafka
- kafka document
- kafka introduction
- MQ
---

공부하면서 기록한것, 틀린 해석이 있을수 있음.

  


원문 : http://kafka.apache.org/documentation.html

  


  





Kafka 0.8.1 Documentation

  


1. Getting Started




### 1.1 Introduction




Kafka is a distributed, partitioned, replicated commit log service. It provides the functionality of a messaging system, but with a unique design.




카프카는 분산, 파티션, 복제 커밋 로그 서비스다. 그것은 메시징시스템의 기능을 제공하지만, 유니크한 디자인을 가지고 있다.




What does all that mean?




이게 뭔 말이냐?




First let's review some basic messaging terminology:




먼저 기본 메시지 용어 에 대해 설명하자




  * Kafka maintains feeds of messages in categories called _topics_. 카프카는 메시지 피드를 토픽이라고 불리는 카테고리 안에 유지한다.
  * We'll call processes that publish messages to a Kafka topic _producers_. 우리는 카프카 토픽에게 메시지를 배포하는 프로세스를 프로듀서라 부를거다.
  * We'll call processes that subscribe to topics and process the feed of published messages _consumers_.. 우리는 토픽을 구독하고 배포된 메시지의 피드를 처리 하는 것을 컨슈머 라고 부를거다.
  * Kafka is run as a cluster comprised of one or more servers each of which is called a _broker_. 카프카는 클러스터 하나 또는 더 많은 서버의 클러스터로 이루어져 실행하는데, 이를 브로커라고 부를거야.



So, at a high level, producers send messages over the network to the Kafka cluster which in turn serves them up to consumers like this:




그래서, 고수준으로 보면, 프로듀서는 네트워크를 통해 메시지를 카프카 클러스터로 보내고, 그것은 컨슈머에게 제공되. 그림처럼.

  





![](http://kafka.apache.org/images/producer_consumer.png)

  


  





Communication between the clients and the servers is done with a simple, high-performance, language agnostic [TCP protocol](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol).




클라이언트와 서버사이에 통신은 높은 성능과, 언어 상관없는 TCP 프로토콜을 통해 간단하게 수행된다.




We provide a Java client for Kafka, but clients are available in [many languages](https://cwiki.apache.org/confluence/display/KAFKA/Clients).




우리는 카프카를 위해 자바 클라이언트를 제공하지만, 클라이언트는 많은 언어에서 사용 가능해.




  





#### Topics and Logs




Let's first dive into the high-level abstraction Kafka provides—the topic.




카프카 제공하는 고수준 추상화에 뛰어 들자 - 토픽




  





A topic is a category or feed name to which messages are published. For each topic, the Kafka cluster maintains a partitioned log that looks like this:




토픽은 메시지가 게시되는 카테고리 또는 피드 이름이다. 각 토픽에 대해서, 카프카 클러스터는 파티션된 로그를 유지관리한다. 보인는 것 처럼 :




![](http://kafka.apache.org/images/log_anatomy.png)

  


  





Each partition is an ordered, immutable sequence of messages that is continually appended to—a commit log.




각 파티션은 순서가 있고, 커밋로그에 연속적으로 추가되는 메시지의 불변의 연속물 이다.




The messages in the partitions are each assigned a sequential id number called the _offset_ that uniquely identifies each message within the partition.




파티션 안의 메시지는 오프셋이라고 불리는 각각 순차적 아이디 숫자가 할당 되어 지고, 그것은 파티션에서 각 메시지의 고유한 식별자이다.




The Kafka cluster retains all published messages—whether or not they have been consumed—for a configurable period of time.




카프카 클러스는 모든 게시된 모든 메시지를 시간의 설정 기간 동안 유지한다 - 소비되었든 아니든 -




For example if the log retention is set to two days, then for the two days / after a message is published / it is available for consumption, after which it will be discarded to free up space.




예를 들어 로그 보존이 이틀로 설정 되었다면, 이틀 동안 메시지는 소비를 위해 게시되고, 이후 공간 확보를 위해 삭제될 것이다.




Kafka's performance is effectively constant with respect to data size so retaining lots of data is not a problem.




카프카의 성능은 유효한 상수임에 따라 데이터의 크기에 대하여 매우 많은 데이터를 유지하는데 문제가 없다.




In fact the only metadata retained on a per-consumer basis is the position of the consumer in the log, called the "offset".




사실 컨슈머 당 기본 유지하는 메타데이터는 오직 로그에서 컨슈머의 위치이고,  오프셋 이라고 부른다.




This offset is controlled by the consumer:




이 오프셋은 컨슈머에 의해 관리된다:




normally a consumer will advance its offset linearly as it reads messages, but in fact the position is controlled by the consumer and it can consume messages in any order it likes.




일반적으로 컨슈머는 메시지를 읽고 그 오프셋을 선형으로 전진시키지만, 사실 위치는 컨슈머에 의해 관리되고, 그것이 좋아하는 임의의 순서대로 메시지를 소비할 수 있다.




For example a consumer can reset to an older offset to reprocess.




예를들어 컨슈머는 오래된 오프셋을 재처리 하기 위해 재설정할 수 있다.







This combination of features means that Kafka consumers are very cheap—they can come and go without much impact on the cluster or on other consumers.










이 기능의 조합은 카프카 컨슈머는 매우 저렴한것을 의미한다. - 그들은 클러스터나 다른 소비자에게 영향 없이 오갈수 있다.










For example, you can use our command line tools to "tail" the contents of any topic without changing what is consumed by any existing consumers.







예를들어, 당신은 어떤 존재하는 컨슈머에의해 소비되어 지는 어떤 토픽에 변화 없이 우리의 명령어 라인 툴인 “tail” 을 사용할 수 있다. - 자신과 상관없는 데이터에 아무 영향없이 접근할 수 있다는 말인듯.







The partitions in the log serve several purposes.




로그안에 파티션은 몇몇의 목적으로 제공된다. ( 파티션 의 필요성)




First, they allow the log to scale beyond a size that will fit on a single server.




첫째, 그들은 단일 서버에 맞는 크기를 넘어 로그를 확장하는것을 허락한다.




Each individual partition must fit on the servers that host it,but a topic may have many partitions so it can handle an arbitrary amount of data.




각각의 개별 파티션은 그 호스트 서버에 적합해야 하지만, 임의의 양의 데이터를 처리 할 수 있도록, 토픽은 많은 파티션을 가질수 있다.




Second they act as the unit of parallelism—more on that in a bit.




두번째, 그들은 병렬 처리의 단위 처럼 행동한다




  








#### Distribution




The partitions of the log are distributed over the servers in the Kafka cluster with each server handling data and requests for a share of the partitions.




로그의 파티션은 파티션의 공유를 위한 데이터 요청을 처리하는 각 서버와 카프카 클러스의 서버에 분배 됩니다.




Each partition is replicated across a configurable number of servers for fault tolerance.




각 파티션은 내 결함성을 위해 설정한 서버의 숫자만큼 복제된다.  








Each partition has one server which acts as the "leader" and zero or more servers which act as "followers".




각 파티션은 리더 역할을 하는 하나의 서버와, 0개 이상의 팔로워 서버를 갖는다.




The leader handles all read and write requests for the partition while the followers passively replicate the leader.




팔로워가 리더를 수동적으로 복제를 하는 동안, 리더는 파티션을 위한 모든 읽기와 쓰기 요청을 처리한다.




If the leader fails, one of the followers will automatically become the new leader.




만약 리더가 실패하면, 팔로워중 하나는 자동적으로 새로운 리더가 될것이다.




Each server acts as a leader for some of its partitions and a follower for others so load is well balanced within the cluster.




각 서버는 파티션의 일부를 위해 리더 역할을 하고, 다른 부하를 위한 팔로워는 클러스터안에 잘 균형 맞춰진다. ?




  








#### Producers




Producers publish data to the topics of their choice.




프로듀서는 데이터를 그들의 선택한 토픽으로 게시한다.




The producer is responsible for choosing which message to assign to which partition within the topic.




프로듀서는 토픽안의 어떤 파티션에 어떤 메시지를 할당할 것인지 선택하는 것을 책임진다.




This can be done in a round-robin fashion simply to balance load or it can be done according to some semantic partition function(say based on some key in the message).




이것은 단순하게 부하를 분산하기 위해 라운드-로빈 방식으로 할 수 있고, 어떤 의미의 분배 함수를 따라 할 수 있다. ( 메시지의 어떤 키 기반를 기준으로 말한다. )




More on the use of partitioning in a second.




잠시후 파티션의 사용법은 자세히한다.




  





#### Consumers




Messaging traditionally has two models: [queuing](http://en.wikipedia.org/wiki/Message_queue) and [publish-subscribe](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern).




메시징은 전통적으로 두가지 모델을 가지고 있다: 큐와 퍼블리쉬-서브스크라이브. ( 게시-구독 )




In a queue, a pool of consumers may read from a server and each message goes to one of them;




큐에서, 컨슈머의 풀은 서버로부터 읽어지고 각 메시지는 그들 중 하나에게 보내질 것이다.




in publish-subscribe the message is broadcast to all consumers.




퍼블리쉬-서브프라이브에서 메시지는 모든 컨슈머에게 뿌려진다.




Kafka offers a single consumer abstraction that generalizes both of these—the _consumer group_.




카프카는 이 두가지를 일반화 하는 단하나의 컨슈머 추상화를 제공한다.  - 컨슈머 그룹  








Consumers label themselves with a consumer group name, and each message published to a topic is delivered to one consumer instance within each subscribing consumer group.




컨슈머는 스스로 컨슈머 그룹 이름 표시하고, 토픽에 게시된 각 메시지는 구독하는 컨슈머 그룹에서 하나의 컨슈머 인스턴스에게 배달된다.




Consumer instances can be in separate processes or on separate machines.




컨슈머 인스턴스는 별도의 프로세스 또는 별도의 머신으로 나눌수 있다.










If all the consumer instances have the same consumer group, then this works just like a traditional queue balancing load over the consumers.




만약 모든 컨슈머 인스턴스가 같은 컨슈머 그룹을 갖는다면, 이것은 단순히 컨슈머의 전통적인 큐 부하 분배처럼 작업한다.










If all the consumer instances have different consumer groups, then this works like publish-subscribe and all messages are broadcast to all consumers.




만약 모든 컨슈머 인스턴스가 다른 컨슈머 그룹을 갖는다면, 이것은 퍼블리쉬-서브크라이브 처럼 작업하고 모든 메시지는 모든 컨슈머에게 뿌려진다.







More commonly, however, we have found that topics have a small number of consumer groups, one for each "logical subscriber".

좀 더 일반적으로, 그럼에도 불구하고, 우리는 토픽이 컨슈머 그룹의 작은 수를 갖는 다는 것을 발견했다. 각 논리적 구독자 위한 하나, (토픽의 컨슈머 그룹의 컨슈머 하나 만이 메시지를 받는다는 말 - 논리적으로는 컨슈머 그룹이 구독자 라고)

Each group is composed of many consumer instances for scalability and fault tolerance.




각 그룹은 확장성과 내결함성을 위해 많은 컨슈머 인스턴스로 구성된다.




This is nothing more than publish-subscribe semantics where the subscriber is cluster of consumers instead of a single process.

이것은 퍼블리쉬-서브크라이브 의미에 불과하고, 여기서 서브크라이버는 단일 프로세스 대신 컨슈머의 클러스터다.

  


_"Apache Kafka의 경우 위에서 언급한 두가지 모델을 모두 포함하는 구조이다._

_Consumer가 같은 consumer group 안에 있게 되면 queue의 구조를 따른다. _

_같은 consumer group은 동일한 queue에서 번갈아가며 데이터를 읽게 된다. _

_하지만 다른 consumer group이라면 consumer들은 각기 전체 데이터를 동시에 읽게 된다.(publish-subscribe) _

**_즉 consumer group은 "logical subscriber"라고 볼 수 있다."_**

  











![](http://kafka.apache.org/images/consumer-groups.png)

  


  





**  
**

**A two server Kafka cluster hosting four partitions (P0-P3) with two consumer groups. Consumer group A has two consumer instances and group B has four.**

두개의 컨슈머 그룹과 네개의 파티션을 호스팅하는 두개 서버의 카프카 클러스터. A 컨슈머 그룹은 두개의 컨슈머 인스턴스를 갖고 그룹 B는 네개를 갖는다.

**  
**







Kafka has stronger ordering guarantees than a traditional messaging system, too.

카프카는 전통적인 메시지 시스템 보다 강한 순서를 보장한다.




A traditional queue retains messages in-order on the server, and if multiple consumers consume from the queue then the server hands out messages in the order they are stored.

전통적인 큐는 메시지를 서버에 들어온 순서대로 유지한다, 그리고 만약 병렬 컨슈머가 큐로부터 소비하면 서버는 그들이 저장한 순서대로 메시지를 전달한다.

However, although the server hands out messages in order, the messages are delivered asynchronously to consumers, so they may arrive out of order on different consumers.

그러나, 비록 서버는 순서대로 메시지를 전달 해도, 메시지는 비동기적으로 소비자에게 전달되어지므로, 그들은 다른 컨슈머에게 순서가 가지 않을 수 있다. ( 다른 소비자에게 순서대로 가지 않는다.)

This effectively means the ordering of the messages is lost in the presence of parallel consumption.

이것은 실제적으로 메시지의 순서가 병렬 소비의 환경에서 손실되는 것을 의미한다.

Messaging systems often work around this by having a notion of "exclusive consumer" that allows only one process to consume from a queue, but of course this means that there is no parallelism in processing.

메시지 시스템은 종종 독점적 컨슈머의 개념을 가지고 이것을 해결하려하는데, 그것은 큐로 부터 오직 하나의 프로세스 만이 소비하는 것을 허락하는 것이다. 그러나 물론 이것은 그것이 병렬 처리가 아니라는 것을 의미한다.







Kafka does it better.

카프카는 더 잘한다.

By having a notion of parallelism—the partition—within the topics, Kafka is able to provide both ordering guarantees and load balancing over a pool of consumer processes.

토픽내의 병렬 처리 개념을 가지고 - 파티션 -, 카프카는 컨슈머 프로세스의 풀을 통해 순서보장과 부하 분산 둘다 제공할 수 있다.

This is achieved by assigning the partitions in the topic to the consumers in the consumer group so that each partition is consumed by exactly one consumer in the group.

이것은 컨슈머 그룹 안의 컨슈머에게 토픽내의 파티션을 할당하는 것을 통해 이루어 지고, 각 파티션은 정확하게 그룹 내의 하나의 컨슈머에 의해 소비되어진다.

By doing this we ensure that the consumer is the only reader of that partition and consumes the data in order.

이렇게 함으로써, 우리는 컨슈머가 해당 파티션의 유일한 리더이고 순서대로 데이터를 소비하고 있는지 확인한다.

Since there are many partitions this still balances the load over many consumer instances.

많은 파티션이 있기 떄문에 여전히 많은 컨슈머 인스턴스를 통해 부하 균형을 맞춘다?  


Note however that there cannot be more consumer instances than partitions.

**_그러나, 파티션보다 더 많은 소비자 인스턴스는 있을 수 없다는것을 주의하라._**







Kafka only provides a total order over messages _within_ a partition, not between different partitions in a topic.

**_카프카는 오직 파티션 내의 메시지에 전체 순서를 제공한다, 다른 토픽 내의 파티션 사이에는 제공하지 않는다._**

Per-partition ordering combined with the ability to partition data by key is sufficient for most applications.

파티션당 순서 결합과 데이터 키에 의한 파티션 능력은 대부분 응용 프로그램에 충분하다.

However, if you require a total order over messages this can be achieved with a topic that has only one partition, though this will mean only one consumer process.

그러나, 만약 당신이 메시지간 전체 순서를 필요하다면 이것은 비록 단일 컨슈머 프로세스를 의미해도, 오직 하나의 파티션을 갖는 토픽으로 이루어 질수있다.

  


카프카는 토픽 내의 파티션 별 순서를 보장하지만, 다른 토픽과 그 파티션 모두에서 전체 순서를 보장하길 원하면, 결국 하나의 컨슈머 프로세스를 구성해야한다는 말.. 인듯..

  








#### Guarantees

At a high-level Kafka gives the following guarantees:

높은 수준에서 카프카는 다음과 같은 것을 보장한다:

  * Messages sent by a producer to a particular topic partition will be appended in the order they are sent. That is, if a message M1 is sent by the same producer as a message M2, and M1 is sent first, then M1 will have a lower offset than M2 and appear earlier in the log.

특정 토픽 파티션에 프로듀서에 의해 보내진 메시지는 그들이 보낸 순서대로 추가된다. 이것은 만약 M1 메시지와 M2 메시지가 같은 프로듀서로 부터 보내졌고, M1 메시지가 먼저 보내졌다면, M1 은 M2 보다 낮은 오프셋을 갖고 로그에 먼저 나타날 것이다.

  * A consumer instance sees messages in the order they are stored in the log.

컨슈머 인스턴스는 그것들이 로그에 저장된 순서대로 메시지를 본다.

  * For a topic with replication factor N, we will tolerate up to N-1 server failures without losing any messages committed to the log.

토픽을 위한 복제 요소 N이 있다면, 우리는 로그 에 커밋 된 메시지 를 잃지 않고 N - 1 서버 의 장애 까지 견딜 수있다.

More details on these guarantees are given in the design section of the documentation.

이러한 보장에 대한 더 자세한 내용은 문서의 디자인 섹션에 있다.

  





###   


  

