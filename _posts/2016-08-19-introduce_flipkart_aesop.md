---
author: springloops
comments: true
date: 2016-08-19
layout: post
title: 'Introduce Flipkart Aesop'
categories:
- OPEN SOURCE/Flipkart Aesop
tags:
- flipkart Aesop
- opensource
- tutorial
- introduce
- databus
---

# Flipkart Aesop 소개.

aesop 은 LinkedIn의 [Databus](/archivers/linkedin-databus-introduce-home)를 사용하여, 변경된 데이터를 발견하여 로그를 수집하는 방식을 제공한다.

추가적으로 Aesop 은 NGDATA Hbase-sep 기반으로 Event Producer 를 구현하여, HBase 데이터 저장소의 변화 탐지에 대한 지원을 확장한다.

MySQL의 경우, Aesop은 데이터 버스에서 사용할 수 있는 producer 구현을 생성한다.

이 글에서는 Aesop의 내부적으로 어떻게 Application 을 생성하고, 동작하는지 가볍게 설명한다.


## Aesop Archtecture

Aesop 은 Databus 를 확장하는 CDC 로, Databus와 마찬가지로 Relay, Client, Consumer, Bootstrap, Bootstrap Server 컴포넌트를 제공한다.

![Aesop logical archtecture](https://github.com/Flipkart/aesop/raw/master/docs/Overall_architecture.png)
	그림과 함께 보자.


### Relays

1. Databus 소스로 부터 소스 데이터베이스의 변경된 rows 를 읽어와, 메모리 버퍼에 Databus의 데이터 변경 이벤트로 직렬화 한다. ( Avro 이용 )
2. Databus Client 로부터 요청을 듣고, 새로운 Databus 데이터 변경 이벤트로 변환한다.
3. 더 자세한 내용 -> [Databus 2.0 Relay](https://github.com/linkedin/databus/wiki/Databus-2.0-Relay)

### Clients

1. Relay 에서 새로운 데이터 변경 이벤트를 체크하고 명시적인 콜백 비즈니스 로직을 실행한다.
2. 만약 relay 에서 부터 너무 멀리 떨어질 경우, 부트스트랩 서버의 격차해소(catchup) 쿼리를 실행한다.
3. 새로운 Databus Client 는 부트스트랩 서버의 부트스트랩 쿼리를 수행하고, 최신 데이터 변경 이벤트를 위해 Relay 를 변경한다.
4. 하나의 클라이언트는 전체 데이터 버스 스트림을 처리 할 수도 있고, 클러스터의 일부만 소비할 수도 있다.
5. 더 자세한 내용 -> [Databus 2.0 Client](https://github.com/linkedin/databus/wiki/Databus-2.0-Client)

### Bootstrap Producers
1. 특별한 종류의 Databus Client
2. Relay 에서 새로운 데이터 변경 이벤트를 체크한다.
3. Mysql 에 이벤트를 저장한다.
4. Mysql은 Client 가 Bootstrap 과 격차해소(catchup)를 하기 위해 사용된다.

### Bootstrap Servers
1. Databus Client 로 부터의 요청(request)을 듣고, 시작(bootstrapping) 과 격차해소(catchup)을 위해 긴 look-back 데이터 변환 이벤트를 반환한다.


## Trooper 에 의해 실행 관리 되는 Aesop

Aesop 은 어플리케이션 컨테이너를 구축하는 플랫폼인 [Trooper](https://github.com/regunathb/Trooper/wiki) 에 의해 각각의 container의 Component 로 정의 되고, Trooper에 의해 실행/관리된다.

솔직히 Trooper가 뭔지도 잘 모르겠고 왜 써야 하는지 모르겠다만, Aesop을 사용하기 위해, 정의하는 방법/실행 과정만 좀 보자.

Trooper 는 Spring Framework 기반의 App 이고, 아래 정의된 beans 를 기반으로 Trooper의 BootstrapLauncher 클래스를 통해 실행된다. 

* bootstrap.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN" "http://www.springframework.org/dtd/spring-beans-2.0.dtd">
<beans>
    <bean id="bootstrapInfo" class="org.trpr.platform.runtime.spi.bootstrap.BootstrapInfo">
        <property name="projectsRoot" value="$RUNTIME_CONFIG_PATH/../../" />
        <property name="applicationName" value="Aesop Sample Client Server" />
        <property name="runtimeNature" value="server" />
        <property name="container" ref="containerImpl" />
    </bean>

    <bean id="containerImpl" class="org.trpr.platform.runtime.impl.container.spring.SpringContainerImpl">
	    <property name="componentContainers">
		    <list>
		        <bean id="clientComponentContainer" class="com.flipkart.aesop.runtime.spring.ClientRuntimeComponentContainer"/>
		    </list>
		</property>
    </bean>
</beans>
```

### Aesop 에서 제공하는 Trooper ComponentContainer 

Trooper 의해 관리되기 위한 Aesop 에서 제공하는 Container는 다음과 같다.

* RuntimeComponentContainer
	* ClientRuntimeComponentContainer
	* RelayRuntimeComponentContainer
	* BlockingBootstrapRuntimeComponentContainer
	* BootstrapRuntimeComponentContainer
	* BootstrapProducerRuntimeComponentContainer

Aesop 이 구현한 Trooper Runtime Component Container 들의 초기화 수행 로직은 대부분 RuntimeComponentContainer에 정의되어 있고,

RuntimeComponentContainer를 상속한 하위 Container 들은 자신에 맞는 속성만 추가적으로 가지고 있다.
 

### Trooper 에 의한 Aesop Runtime Container 초기화 과정

#### **Trooper 의 Bootstrap 초기화**

TrooperLauncher 클래스는 Bootstrap 클래스를 통해 위 정의된 Spring Beans 정의 xml을 읽어 들이고, 해당 Container 들의 초기화 및 실행 관리 한다.

그 흐름은 다음과 같다.

![trooper sequence diagram](https://raw.githubusercontent.com/springloops/springloops.github.io/master/imgs/_for_posts/trooper_seq.png)

위 sequence diagram 에서 보듯 BootStrap 은 정의된 ComponentContainer 를 초기화하고 자신을 새로운 Thread 로 등록하고, BootstrapLauncher Thread 는 종료 된다.

여기서 Bootstrap 이 초기화 한 ComponentContainer 가 Aesop 에서 구현한, 위에 나열한 ComponentContainer 들이고,

예시로 보인 bootstrap.xml 에서는 ClientRuntimeComponentContainer 가 된다.


#### **Aesop ComponentContainer 초기화**

Aesop ClientRuntimeComponentContainer 는 RuntimeComponentContainer 를 상속하고 있다.

Aesop 이 제공하는 ComponentContainer 는 모두 RuntimeComponentContainer 를 상속하고, 실제 로직은 RuntimeComponentContainer 가 처리하고, 각 Component 별 필요한 설정 값만 extended 된다.

때문에 ComponentContainer 의 초기화 과정을 보려면, RuntimeComponentContainer 를 봐야 한다.

위 Sequence diagram 에서 보듯이, Container class 는 ComponentContainer 의 init() 메소드를 호출한다.

ComponentContainer 는 bootstrap.xml 에 정의된 ComponentContainer 목록을 로드하고, iterator 로 변환, 반복하며 등록된 ComponentContiner 의 init() 을 호출한다.

* Aesop Client Component Container 정의 파일

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

	<bean id="sampleClient"
		class="com.flipkart.aesop.runtime.clusterclient.DefaultClusterClientFactory">
		<property name="clientClusterConfig" ref="clientClusterConfig" />
		<property name="clusterRegistrations">
			<list>
				<bean class="com.flipkart.aesop.runtime.config.ClusterRegistration">
					<property name="clusterName" value="Person_Cluster" />
					<property name="consumerFactory">
						<bean class="com.flipkart.aesop.clusterclient.sample.consumer.ConsumerFactory" />
					</property>
					<property name="logicalSources">
						<list value-type="java.lang.String">
							<value>pz.or_test.person</value>
						</list>
					</property>
				</bean>
			</list>
		</property>
	</bean>

	<bean id="clientClusterConfig" class="com.flipkart.aesop.runtime.config.ClientClusterConfig">
		<property name="clientProperties">
			<bean id="clientPropertiesFactory"
				class="org.springframework.beans.factory.config.PropertiesFactoryBean">
				<property name="singleton" value="true" />
				<property name="properties">
					<props>
						<prop key="databus.client.container.httpPort">11125</prop>
						<prop key="databus.client.container.jmx.rmiEnabled">false</prop>
						<prop key="databus.client.connectionDefaults.pullerRetries.initSleep">1</prop>
						<prop key="databus.client.checkpointPersistence.clearBeforeUse">false</prop>
						<prop
							key="databus.client.connectionDefaults.enablePullerMessageQueueLogging">false</prop>
					</props>
				</property>
			</bean>
		</property>
		<property name="relayClientConfigs">
			<list>
				<bean class="com.flipkart.aesop.runtime.config.RelayClientConfig">
					<property name="relayId" value="412" />
					<property name="relayHost" value="localhost" />
					<property name="relayPort" value="25021" />
					<property name="relayLogicalSourceNames">
						<list value-type="java.lang.String">
							<value>pz.or_test.person</value>
						</list>
					</property>
				</bean>
			</list>
		</property>
		<property name="clusterInfoConfigs">
			<list>
				<bean class="com.flipkart.aesop.runtime.config.ClusterInfoConfig">
					<property name="id" value="1" />
					<property name="clusterName" value="Person_Cluster" />
					<property name="zkAddr" value="zookeeper IP:POSRT" />
					<property name="numPartitions" value="3" />
					<property name="quorum" value="1" />
					<property name="zkSessionTimeoutMs" value="3000" />
					<property name="zkConnectionTimeoutMs" value="3000" />
					<property name="checkpointIntervalMs" value="300000" />
				</bean>
			</list>
		</property>

		<property name="checkpointDirectoryLocation" value="../../../sandbox/client_checkpoint_directory" /> <!-- This is relative to projects root -->
	</bean>
</beans>
```

ComponentContainer 의 init 메소드에서는 위와 같이 정의된 Spring Bean 을 초기화 한다.

위 정의파일에서는 Client cluster 설정, relay 정보, sampleClient Bean 을 등록하고 있다.


## Client & Consumer

위 Spring beans 정의 문서에서, 사용자가 작업하기 위해 중요한 부분은, 

ClientFactory 로 Databus의 변경 이벤트를 받아 오는 Client 를 생성하고, consumerFactory로 사용자 정의 ConsumerFactory를 등록하고 있다는 것이다.

실제로 사용자가 처리해줘야 할 부분은 저 부분 뿐이다. (Avro 의 namespace 정의와 함께...)

ConsumerFactory 는 이벤트 콜백에 대한 행위를 하는 클래스를 생성하는 Factory 로, Client 가 Relay로 부터 변경 이벤트를 받으면, ConsumerFactory에 구현된 consumer 로 이벤트를 전달한다.

실제 구현된 consumer 는 AbstractMySqlEventConsumer 클래스를 상속하고, 다음과 같은 processEvent 를 구현하고 있다.

```java
@Override
public ConsumerCallbackResult processEvent(MysqlBinLogEvent mysqlBinLogEvent)
{
    LOGGER.debug("Event : " + mysqlBinLogEvent.toString());
    return ConsumerCallbackResult.SUCCESS;
}
```

Client 정의에 대해 대충 정리 함. 

머리속에 맴도는 셀들을 하나하나 엮어, 정리하고 표현하는 것이 이렇게 힘들다;;;;; 귀차니즘 보다 능력 밖인듯...

client & consumer 관련 더 자세한 내용은 Databus 글을 참고하시라..

[LinkedIn Databus Client Design](http://springloops.github.io/archivers/linkedin-databus-introduce-client-design)

## Relays

Aesop Relays 의 Trooper 에 의한 구동은 위와 동일하다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">

    <bean id="sampleRelay" class="com.flipkart.aesop.runtime.relay.DefaultRelayFactory">
        <property name="relayConfig" ref="relayConfig"/>
        <property name="producerRegistrationList">
            <list>
                <bean class="com.flipkart.aesop.runtime.config.InitBackedProducerRegistration">
                    <property name="properties">
                        <bean id="productRegistrationPropertiesFactory" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
                            <property name="singleton" value="true"/>
                            <property name="properties">
                                <props>
                                    <prop key="databus.relay.dataSources.sequenceNumbersHandler.file.initVal">4294967300</prop>
                                </props>
                            </property>
                        </bean>
                    </property>
                    <property name="eventProducer" ref="mysqlPersonProducer"/>
                    <property name="physicalSourceConfig" ref="physicalSourceConfig"/>
                </bean>
            </list>
        </property>
    </bean>

    <bean id="mysqlPersonProducer" class="com.flipkart.aesop.runtime.producer.MysqlEventProducer">
        <property name="physicalSourceConfig" ref="physicalSourceConfig"/>
        <property name="eventsMap" ref="eventsMapRef"/>
        <property name="binLogEventMappers" ref="binLogEventMappersRef"/>
        <property name="schemaChangeEventProcessor">
            <bean class="com.flipkart.aesop.runtime.producer.schema.eventprocessor.impl.NopSchemaChangeEventProcessor" />
        </property>
    </bean>

    <bean id="orToAvroMapper" class="com.flipkart.aesop.runtime.producer.mapper.impl.ORToAvroMapper"/>

    <util:map id="eventsMapRef" key-type="java.lang.Integer" value-type="com.flipkart.aesop.runtime.producer.eventprocessor.BinLogEventProcessor">
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).QUERY_EVENT}" value-ref="mutatingQueryEventProcessor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).ROTATE_EVENT}" value-ref="rotateEventProcessor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).XID_EVENT}" value-ref="commitEventProcessor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).TABLE_MAP_EVENT}" value-ref="tableMapEventProcessor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).WRITE_ROWS_EVENT}" value-ref="insertEventProcessor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).UPDATE_ROWS_EVENT}" value-ref="updateEventProcessor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).DELETE_ROWS_EVENT}" value-ref="deleteEventProcessor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).WRITE_ROWS_EVENT_V2}" value-ref="insertEventV2Processor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).UPDATE_ROWS_EVENT_V2}" value-ref="updateEventV2Processor"/>
        <entry key="#{T(com.flipkart.aesop.runtime.producer.constants.MySQLConstants).DELETE_ROWS_EVENT_V2}" value-ref="deleteEventV2Processor"/>
    </util:map>

    <bean id="mutatingQueryEventProcessor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.MutatingQueryEventProcessor"/>
    <bean id="rotateEventProcessor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.RotateEventProcessor"/>
    <bean id="commitEventProcessor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.CommitEventProcessor"/>
    <bean id="tableMapEventProcessor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.TableMapEventProcessor"/>
    <bean id="insertEventProcessor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.InsertEventProcessor"/>
    <bean id="updateEventProcessor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.UpdateEventProcessor"/>
    <bean id="deleteEventProcessor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.DeleteEventProcessor"/>
    <bean id="insertEventV2Processor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.InsertEventV2Processor"/>
    <bean id="updateEventV2Processor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.UpdateEventV2Processor"/>
    <bean id="deleteEventV2Processor" class="com.flipkart.aesop.runtime.producer.eventprocessor.impl.DeleteEventV2Processor"/>

    <util:map id="binLogEventMappersRef" key-type="java.lang.Integer" value-type="com.flipkart.aesop.runtime.producer.mapper.BinLogEventMapper">
        <entry key="40" value-ref="defaultBinLogEventMapper"/>
    </util:map>
    <bean id="defaultBinLogEventMapper" class="com.flipkart.aesop.runtime.producer.mapper.impl.DefaultBinLogEventMapper">
        <property name="orToAvroMapper" ref="orToAvroMapper"/>
    </bean>

    <bean id="physicalSourceConfig" class="com.linkedin.databus2.relay.config.PhysicalSourceConfig">
        <property name="id" value="1"/>
        <property name="name" value="personPhysicalSource"/>
        <property name="uri" value="mysql://or_test%2For_test@localhost:3306/12345/mysql-bin"/>
        <property name="sources">
            <list>
                <bean id="logicalSourceConfig" class="com.linkedin.databus2.relay.config.LogicalSourceConfig">
                    <property name="id" value="41"/>
                    <property name="name" value="com.flipkart.aesop.events.ortest.Person"/>
                    <property name="uri" value="or_test.person"/>
                    <property name="partitionFunction" value="constant:1"/>
                </bean>
            </list>
        </property>
    </bean>

    <bean id="relayConfig" class="com.flipkart.aesop.runtime.config.RelayConfig">
        <property name="relayProperties">
            <bean id="relayPropertiesFactory" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
                <property name="singleton" value="true"/>
                <property name="properties">
                    <props>
                        <prop key="databus.relay.container.httpPort">25021</prop>
                        <prop key="databus.relay.container.jmx.rmiEnabled">false</prop>
                        <prop key="databus.relay.eventBuffer.allocationPolicy">MMAPPED_MEMORY</prop>
                        <prop key="databus.relay.eventBuffer.queuePolicy">OVERWRITE_ON_WRITE</prop>
                        <prop key="databus.relay.eventLogReader.enabled">false</prop>
                        <prop key="databus.relay.eventLogWriter.enabled">false</prop>
                        <prop key="databus.relay.schemaRegistry.type">FILE_SYSTEM</prop>
                        <prop key="databus.relay.eventBuffer.maxSize">10240000</prop>
                        <prop key="databus.relay.eventBuffer.readBufferSize">10240</prop>
                        <prop key="databus.relay.eventBuffer.scnIndexSize">10240000</prop>
                        <prop key="databus.relay.eventBuffer.restoreMMappedBuffers">true</prop>
                        <!-- <prop key ="databus.relay.dataSources.sequenceNumbersHandler.file.initVal">4294967300</prop> -->
                    </props>
                </property>
            </bean>
        </property>
        <property name="schemaRegistryLocation" value="schemas_registry"/>
        <property name="mmappedDirectoryLocation" value="/tmp/sandbox/mmapped_directory"/> <!-- This is relative to projects root -->
        <property name="maxScnDirectoryLocation" value="/tmp/sandbox/maxscn_directory"/> <!-- This is relative to projects root -->
    </bean>

</beans>
```

Relays 정의한 위 spring Beans 는 실제로 physicalSourceConfig Bean만 확인하면 된다.

접속할 Mysql 과 테이블과 매핑될 Avro Schema 정의면 된다.

Relays 관련 더 자세한 내용은 Databus 글을 참고하시라...

[LinkedIn Databus Relay Design](http://springloops.github.io/archivers/linkedin-databus-introduce-relay-design)


## Avro Schema Generator 제공.

[Avro Schema Genrator](https://github.com/Flipkart/aesop/wiki/Utilities-Examples)



