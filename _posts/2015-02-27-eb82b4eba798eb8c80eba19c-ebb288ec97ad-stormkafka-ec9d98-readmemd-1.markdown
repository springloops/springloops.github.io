---
author: springloops
comments: true
date: 2015-02-27 05:41:47+00:00
layout: post
link: https://springloops.wordpress.com/2015/02/27/%eb%82%b4%eb%a7%98%eb%8c%80%eb%a1%9c-%eb%b2%88%ec%97%ad-stormkafka-%ec%9d%98-readmemd-1/
slug: '%eb%82%b4%eb%a7%98%eb%8c%80%eb%a1%9c-%eb%b2%88%ec%97%ad-stormkafka-%ec%9d%98-readmemd-1'
title: '[내맘대로 번역] Storm-kafka 의 readme.md'
wordpress_id: 114
categories:
- Open Source/Apache Kafka
tags:
- Kafka
- spout
- Storm
- storm spout
- storm-kafka
---

  
Kafka의 ConsumerGroup 은 쓰기 싫고, SimpleConsumer Api 는 아직 보지도 못했고, (왠지 보기 싫고..)

  


SimpleConsumer 를 구현한, Storm-kafka 로 때워볼까해서..

  


당장 급하게 필요해서, 대충 번역.

  


  


  


원문 : [https://github.com/apache/storm/tree/master/external/storm-kafka#using-storm-kafka-with-different-versions-of-scala](https://github.com/apache/storm/tree/master/external/storm-kafka#using-storm-kafka-with-different-versions-of-scala)

  


  


Kafka 를 위한 Spout 은 Trident 와 core Spout 둘다 지원해, 얘 들을 사용하려면,

  


BrokerHost 인터페이스 와 kafkaConfig 를 구현해야하는데,

  


BrokerHost 는 카프카 브로커와 파티션 매핑을 추적하고

  


kafkaConfig는 카프카 관련된 파라미터를 제어하지.

  


### BrokerHosts

너의 spout/emitter 를 순차적으로 초기화 하려면, BrokerHost 마커 인터페이스를 구축해야하는데,

스톰 애덜은 두가지 구현방법을 제공해.

  


**ZkHosts**

**동적으로 kafka partition mapping 을 추적하기 원한다면 사용해야할 인터페이스**
    
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> ZkHosts(<span style="box-sizing:border-box;">String</span> brokerZkStr, <span style="box-sizing:border-box;">String</span> brokerZkPath) 
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> ZkHosts(<span style="box-sizing:border-box;">String</span> brokerZkStr)
    

kafka 가 사용하는 zookeeper 의 정보가 필요하지요..

brokerZkPath는  카프카 파티션이 사용하는 모든 토픽의 루트 패스이고...

  


브로커 - 파티션 매핑을 확인 하기 위해 스톰은 기본적으로 60초에 한번씩 갱신하는데, 이거 바꾸고 싶으면,

host.refreshFreqSecs 에 원하는 값을 넣으면돼.

  


**StaticHosts**

**정적인 브로커 파티션 매핑을 하려면 아래처럼 직접 입력해**
    
        <span style="box-sizing:border-box;">Broker</span> brokerForPartition0 <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">Broker</span>(<span style="box-sizing:border-box;color:rgb(223,80,0);"><span style="box-sizing:border-box;">"</span>localhost<span style="box-sizing:border-box;">"</span></span>);<span style="box-sizing:border-box;color:rgb(150,152,150);">//localhost:9092</span>
        <span style="box-sizing:border-box;">Broker</span> brokerForPartition1 <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">Broker</span>(<span style="box-sizing:border-box;color:rgb(223,80,0);"><span style="box-sizing:border-box;">"</span>localhost<span style="box-sizing:border-box;">"</span></span>, <span style="box-sizing:border-box;color:rgb(0,134,179);">9092</span>);<span style="box-sizing:border-box;color:rgb(150,152,150);">//localhost:9092 but we specified the port explicitly</span>
        <span style="box-sizing:border-box;">Broker</span> brokerForPartition2 <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">Broker</span>(<span style="box-sizing:border-box;color:rgb(223,80,0);"><span style="box-sizing:border-box;">"</span>localhost:9092<span style="box-sizing:border-box;">"</span></span>);<span style="box-sizing:border-box;color:rgb(150,152,150);">//localhost:9092 specified as one string.</span>
        <span style="box-sizing:border-box;">GlobalPartitionInformation</span> partitionInfo <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">GlobalPartitionInformation</span>();
        partitionInfo<span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>addPartition(<span style="box-sizing:border-box;color:rgb(0,134,179);">0</span>, brokerForPartition0);<span style="box-sizing:border-box;color:rgb(150,152,150);">//mapping form partition 0 to brokerForPartition0</span>
        partitionInfo<span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>addPartition(<span style="box-sizing:border-box;color:rgb(0,134,179);">1</span>, brokerForPartition1);<span style="box-sizing:border-box;color:rgb(150,152,150);">//mapping form partition 1 to brokerForPartition1</span>
        partitionInfo<span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>addPartition(<span style="box-sizing:border-box;color:rgb(0,134,179);">2</span>, brokerForPartition2);<span style="box-sizing:border-box;color:rgb(150,152,150);">//mapping form partition 2 to brokerForPartition2</span>
        <span style="box-sizing:border-box;">StaticHosts</span> hosts <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">StaticHosts</span>(partitionInfo);
    

GlobalPartitionInformation 인터페이스를 구축해야해.

  


### KafkaConfig

  


다음으로 필요한건 KafkaConfig 여..
    
    <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> KafkaConfig(<span style="box-sizing:border-box;">BrokerHosts</span> hosts, <span style="box-sizing:border-box;">String</span> topic)
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> KafkaConfig(<span style="box-sizing:border-box;">BrokerHosts</span> hosts, <span style="box-sizing:border-box;">String</span> topic, <span style="box-sizing:border-box;">String</span> clientId)
    

BrokerHosts는 위에서 얘기한 것들 아무거나 쓸수 있고,  토픽은 카프카 토픽의 이름이야.

옵션으로 ClientId 는 스파우트가 현재 소비한 offset 이 저장되어진, zookeeper 경로의 한 부분으로 사용되.

  


storm-kafka 0.7 버전 그니까 3년 전 문서를 보면, ([https://github.com/nathanmarz/storm-contrib/tree/master/storm-kafka](https://github.com/nathanmarz/storm-contrib/tree/master/storm-kafka))

스톰의 주키퍼에 사용한 파티션의 오프셋을 저장하는데 그 경로에 대해 다음과 같이 표현하고 있어.
    
    <code style="box-sizing:border-box;font-family:Consolas, 'Liberation Mono', Menlo, Courier, monospace;font-size:13.60000038147px;padding:0;border-radius:3px;word-break:normal;border:0;max-width:initial;overflow:initial;word-wrap:normal;background:transparent;">{root path}/{id}/0
    {root path}/{id}/1
    {root path}/{id}/2
    {root path}/{id}/3
    ...</code>

  


아마도 저기 {id} 같은 형태의 메타 정보를 저장한다는 얘기 같아.

  


현재 사용하는 KafkaConfig 의 두가지 확장법이 있어.

  

    
    <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> SpoutConfig(<span style="box-sizing:border-box;">BrokerHosts</span> hosts, <span style="box-sizing:border-box;">String</span> topic, <span style="box-sizing:border-box;">String</span> zkRoot, <span style="box-sizing:border-box;">String</span> id);
    <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> SpoutConfig(<span style="box-sizing:border-box;">BrokerHosts</span> hosts, <span style="box-sizing:border-box;">String</span> topic, <span style="box-sizing:border-box;">String</span> id);
    

  


zkRoot 는 소비자의 메타 정보를 저장할 주키퍼의 루트 경로이고, id 는 소비자 id 야.

그러니까, zkRoot 밑에 소비한 offset 을 저장하고, id 는 유니크한 spout id 가 되는거야.

  


추가적으로 KafkaSpout 가 어떻게 행동하는지 제어할수 있도록, SpoutConfig 가 포함하는 파라미터가 있는데...

  

    
     <span style="box-sizing:border-box;color:rgb(150,152,150);">// setting for how often to save the current kafka offset to ZooKeeper</span>
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">long</span> stateUpdateIntervalMs <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">2000</span>;
    
        <span style="box-sizing:border-box;color:rgb(150,152,150);">// Exponential back-off retry settings.  These are used when retrying messages after a bolt</span>
        <span style="box-sizing:border-box;color:rgb(150,152,150);">// calls OutputCollector.fail().</span>
        <span style="box-sizing:border-box;color:rgb(150,152,150);">// Note: be sure to set backtype.storm.Config.MESSAGE_TIMEOUT_SECS appropriately to prevent</span>
        <span style="box-sizing:border-box;color:rgb(150,152,150);">// resubmitting the message while still retrying.</span>
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">long</span> retryInitialDelayMs <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">0</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">double</span> retryDelayMultiplier <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">1.0</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">long</span> retryDelayMaxMs <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">60</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">*</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">1000</span>;
    

  


Core KafkaSpout 만이 SpoutConfig의 인스턴스를 받아 들인대..

  


TridentKafkaConfig 는 KafkaConfig 의 다른 확정인데, TridentKafkaEmitter 만이 TridentKafkaConfig 를 받아들인대

  


  


  


KafkaConfig 클래스는 공개된 변수 들을 가지고 있는데, 너의 어플의 행동을 제어하지.
    
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">int</span> fetchSizeBytes <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">1024</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">*</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">1024</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">int</span> socketTimeoutMs <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">10000</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">int</span> fetchMaxWait <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">10000</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">int</span> bufferSizeBytes <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">1024</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">*</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">1024</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;">MultiScheme</span> scheme <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">RawMultiScheme</span>();
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">boolean</span> forceFromStart <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">false</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">long</span> startOffsetTime <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;">kafka.api<span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>OffsetRequest</span><span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>EarliestTime();
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">long</span> maxOffsetBehind <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;">Long</span><span style="box-sizing:border-box;color:rgb(0,134,179);"><span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>MAX_VALUE</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">boolean</span> useStartOffsetTimeIfOffsetOutOfRange <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">true</span>;
        <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">int</span> metricsTimeBucketSizeInSecs <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(0,134,179);">60</span>;
    

  


MultiScheme 빼고, 이름을 보면 머하는 애들인지 알수 있대.. 난 모르겠다만...

  


할튼,

  


**MultiScheme**

**  
**

MultiScheme 은 카프카에서 스톰 튜플 안으로 변환되어 가져온 바이트 배열을 (byte[]) 어떻게 소비할 것인지 구술한 인터페이스래.

  


그것은 또한 너의 출력 필드 (output field) 의 이름을 제어한대.

  


볼트 (Bolt ) 에서 사용하려면, 중요하겠지???
    
      <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">Iterable<<span style="box-sizing:border-box;">List<<span style="box-sizing:border-box;color:rgb(51,51,51);">Object</span>></span>></span> deserialize(<span style="box-sizing:border-box;color:rgb(167,29,93);">byte</span>[] ser);
      <span style="box-sizing:border-box;color:rgb(167,29,93);">public</span> <span style="box-sizing:border-box;">Fields</span> getOutputFields();
    

  


기본적으로 RawMultiScheme 은 단순히 바이트 배열 (byte[]) 을 가져오고, 바이트 배열로 리턴한대. 출력 필드 (output field) 이름은 “bytes” 래..

대안으로, SchemeAsMultiScheme 와 KeyValueSchemeAsMultiScheme 같은 구현체가 있는데, 이것들은 byte[] 를 문자열로 변환할 수 있대.

  


중요하겠지???

  


예제는 아래 있고,

### Examples

#### Core Spout
    
    <span style="box-sizing:border-box;">BrokerHosts</span> hosts <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">ZkHosts</span>(zkConnString);
    <span style="box-sizing:border-box;">SpoutConfig</span> spoutConfig <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">SpoutConfig</span>(hosts, topicName, <span style="box-sizing:border-box;color:rgb(223,80,0);"><span style="box-sizing:border-box;">"</span>/<span style="box-sizing:border-box;">"</span></span> <span style="box-sizing:border-box;color:rgb(167,29,93);">+</span> topicName, <span style="box-sizing:border-box;color:rgb(0,134,179);">UUID</span><span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>randomUUID()<span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>toString());
    spoutConf<span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>scheme <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">SchemeAsMultiScheme</span>(<span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">StringScheme</span>());
    <span style="box-sizing:border-box;">KafkaSpout</span> kafkaSpout <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">KafkaSpout</span>(spoutConfig);
    

#### Trident Spout
    
    <span style="box-sizing:border-box;">TridentTopology</span> topology <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">TridentTopology</span>();
    <span style="box-sizing:border-box;">BrokerHosts</span> zk <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">ZkHosts</span>(<span style="box-sizing:border-box;color:rgb(223,80,0);"><span style="box-sizing:border-box;">"</span>localhost<span style="box-sizing:border-box;">"</span></span>);
    <span style="box-sizing:border-box;">TridentKafkaConfig</span> spoutConf <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">TridentKafkaConfig</span>(zk, <span style="box-sizing:border-box;color:rgb(223,80,0);"><span style="box-sizing:border-box;">"</span>test-topic<span style="box-sizing:border-box;">"</span></span>);
    spoutConf<span style="box-sizing:border-box;color:rgb(167,29,93);">.</span>scheme <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">SchemeAsMultiScheme</span>(<span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">StringScheme</span>());
    <span style="box-sizing:border-box;">OpaqueTridentKafkaSpout</span> spout <span style="box-sizing:border-box;color:rgb(167,29,93);">=</span> <span style="box-sizing:border-box;color:rgb(167,29,93);">new</span> <span style="box-sizing:border-box;">OpaqueTridentKafkaSpout</span>(spoutConf);
    

## 

  


스칼라의 다른 버전과 스톰-카프카를 쓸경우 메이븐 종속성 설정 하는 방법과...

[https://github.com/apache/storm/tree/master/external/storm-kafka#using-storm-kafka-with-different-versions-of-scala](https://github.com/apache/storm/tree/master/external/storm-kafka#using-storm-kafka-with-different-versions-of-scala)

  


그 아래에, 토폴로지 의 부분으로 카프카에 데이터를 전송할 때에 대한 방법..

[https://github.com/apache/storm/tree/master/external/storm-kafka#writing-to-kafka-as-part-of-your-topology](https://github.com/apache/storm/tree/master/external/storm-kafka#writing-to-kafka-as-part-of-your-topology)

이 있지만, 지금 당장 안급하니 링크만 남기는 걸로...

  


  

