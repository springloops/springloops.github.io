---
author: springloops
comments: true
date: 2015-02-27 08:32:00+00:00
layout: post
link: https://springloops.wordpress.com/2015/02/27/flume-152-user-guide-setup/
slug: flume-152-user-guide-setup
title: '[발번역] Flume 1.5.2 User Guide - Setup'
wordpress_id: 115
categories:
- Open Source/Apache Flume
tags:
- apache flume
- flume
- flume ng
- NG
---

공부하면서 발번역한 내용 공유

  


  








Setup










  








Setting up an agent 에이전트 설정







  





Flume agent configuration is stored in a local configuration file.




플럼 에이전트 설정은 로컬 설정 파일에 저장된다.




This is a text file that follows the Java properties file format.




로컬 설정 파일은 자바 프로퍼티 파일 포멧을 따르는 텍스트 파일이다.




Configurations for one or more agents can be specified in the same configuration file.




하나 또는 여러 에이전트를 위한 설정들은 같은 설정 파일에 명시할 수 있다.




The configuration file includes properties of each source, sink and channel in an agent and how they are wired together to form data flows.




설정 파일들은 에이전트 안의 각각 Source, Sink 그리고 Channel 의 속성과 데이터 흐름으로 부터 에이전트와 그들이 함께 엮이는 방법을 포함하고 있다.




  








Configuring individual components 개별 컴포넌트 설정하기







  





Each component (source, sink or channel) in the flow has a name, type, and set of properties that are specific to the type and instantiation.




흐름안의 각각의 컴포넌트는 이름과 (Source, Sink 그리고 Channel), 속성의 셋은 명확한 타입과 예시를 가지고 있다.




For example, an Avro source needs a hostname (or IP address) and a port number to receive data from.




예를들어, Avro source는 데이터를 받을 곳으로 부터의 hostname 과 port 숫자가 필요하다.




A memory channel can have max queue size (“capacity”), and an HDFS sink needs to know the file system URI, path to create files, frequency of file rotation (“hdfs.rollInterval”) etc.




메모리 채널은 최대 큐 사이즈를 갖을수 있고, HDFS sink 는 파일 시스템의 URI, 파일을 생성할 패스, 파일 순환 빈도를 알아야한다.




All such attributes of a component needs to be set in the properties file of the hosting Flume agent.




모든 이와 같은 컴포넌트의 속성은 플럼 에이전트 호스팅 프로퍼티스 파일안에 설정해야 한다.




  








Wiring the pieces together 조각을 함께 엮기







  





The agent needs to know what individual components to load and how they are connected in order to constitute the flow.




에이전트는 개별 컴포넌트가 적재되고 흐름울 구성하는 순서 방법에 대해 알아야한다.




This is done by listing the names of each of the sources, sinks and channels in the agent, and then specifying the connecting channel for each sink and source.




이것은 에이전트안의 각각의 Sources, Sinks 그리고 Channels 의 이름 목록에 의해 완료되고, 각 Sink 그리고 Source 를 위한 Channel 연결이 지정된다.







For example, an agent flows events from an Avro source called avroWeb to HDFS sink hdfs-cluster1 via a file channel called file-channel.







예를들어, 에이전트는 avroWeb 이라 불리는 Avro Source 로부터 file-channel 이라고 불리는 file channel 를 경유해서 HDFS sink pdfs-cluster 1 에게 이벤트를 흘려보낸다.??







The configuration file will contain names of these components and file-channel as a shared channel for both avroWeb source and hdfs-cluster1 sink.




설정파일은 avroWeb 소스와 HDFS-cluster 1 싱크 모두를 위한 공유 채널로 이러한 구성요소 및 파일 채널의 이름이 포함한다.










#### Starting an agent 에이전트 실행하기




  





An agent is started using a shell script called flume-ng which is located in the bin directory of the Flume distribution.




에이전트는 플럼 배포본의 bin 디렉토리에 있는 flume-ng 라고 불리는 쉘 스크립트를 이용해 시작된다.




You need to specify the agent name, the config directory, and the config file on the command line:




너는 커멘드 라인에서 config 파일과, config 디렉토리, 에이전트 이름을 명시해야한다.









    
    <span style="font-family:AppleGothic;">
    $ bin/flume-ng agent -n $agent_name -c conf -f conf/flume-conf.properties.template
    
    </span>
    







  





Now the agent will start running source and sinks configured in the given properties file.




이제 에이전트는 주어진 속성 파일에 설정된 Source 와 Sink 를 실행할 것이다.










#### A simple example




  





Here, we give an example configuration file, describing a single-node Flume deployment.




여기, 우리는 예제 설정 파일을 준다. single-node 플럼 배포설 명이다.




This configuration lets a user generate events and subsequently logs them to the console.




이 설정은 사용자에게 이벤트를 발생시키게 하고, 로그를 콘솔로 기록한다.









    
    <span style="font-family:AppleGothic;">
    <span style="color:rgb(153,153,136);font-style:italic;"># example.conf: A single-node Flume configuration</span>
    
    <span style="color:rgb(153,153,136);font-style:italic;"># Name the components on this agent</span>
    <span style="color:rgb(0,128,128);">a1.sources</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">r1</span>
    <span style="color:rgb(0,128,128);">a1.sinks</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">k1</span>
    <span style="color:rgb(0,128,128);">a1.channels</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">c1</span>
    
    <span style="color:rgb(153,153,136);font-style:italic;"># Describe/configure the source</span>
    <span style="color:rgb(0,128,128);">a1.sources.r1.type</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">netcat</span>
    <span style="color:rgb(0,128,128);">a1.sources.r1.bind</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">localhost</span>
    <span style="color:rgb(0,128,128);">a1.sources.r1.port</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">44444</span>
    
    <span style="color:rgb(153,153,136);font-style:italic;"># Describe the sink</span>
    <span style="color:rgb(0,128,128);">a1.sinks.k1.type</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">logger</span>
    
    <span style="color:rgb(153,153,136);font-style:italic;"># Use a channel which buffers events in memory</span>
    <span style="color:rgb(0,128,128);">a1.channels.c1.type</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">memory</span>
    <span style="color:rgb(0,128,128);">a1.channels.c1.capacity</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">1000</span>
    <span style="color:rgb(0,128,128);">a1.channels.c1.transactionCapacity</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">100</span>
    
    <span style="color:rgb(153,153,136);font-style:italic;"># Bind the source and sink to the channel</span>
    <span style="color:rgb(0,128,128);">a1.sources.r1.channels</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">c1</span>
    <span style="color:rgb(0,128,128);">a1.sinks.k1.channel</span> <span style="font-weight:bold;">=</span> <span style="color:rgb(187,136,68);">c1</span>
    
    </span>
    







  





This configuration defines a single agent named a1.




이 설정은 a1 라는 이름으로 단일 에이전트를 정의한다.




a1 has a source that listens for data on port 44444, a channel that buffers event data in memory, and a sink that logs event data to the console.




a1 은 44444 포트에서 데이터를 읽기위한 Source , 메모리 기반 이벤트 데이터 버퍼인 channel, 그리고 이벤트 데이터를 콘솔에 기록할 sink를 갖는다.




The configuration file names the various components, then describes their types and configuration parameters.




설정 파일은 다양한 컴포넌트들로 불리고, 그들의 타입과 설정 파라미터들을 설명한다.???




A given configuration file might define several named agents;




주어진 설정 파일에는 몇몇의 이름의 에이전트를 정의할 수 있다;  


when a given Flume process is launched a flag is passed telling it which named agent to manifest.

주어진 수로 프로세스 가 시작될 때 플래그는 에이전트 이름을 정하고 전달할 수 있따;;;;;ㅡㅡ

Given this configuration file, we can start Flume as follows:









    
    <span style="font-family:AppleGothic;">
    $ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
    
    </span>
    







  





Note that in a full deployment we would typically include one more option: --conf=<conf-dir>.




The <conf-dir> directory would include a shell script _[flume-env.sh](http://flume-env.sh)_ and potentially a log4j properties file.




In this example, we pass a Java option to force Flume to log to the console and we go without a custom environment script.




From a separate terminal, we can then telnet port 44444 and send Flume an event:






    
    <span style="font-family:AppleGothic;">
    $ telnet localhost 44444
    Trying 127.0.0.1...
    Connected to localhost.localdomain (127.0.0.1).
    Escape character is '^]'.
    Hello world! <ENTER>
    OK
    </span>
    




The original Flume terminal will output the event in a log message.









    
    <span style="font-family:AppleGothic;">
    <span style="color:rgb(0,128,128);">12/06/19 15</span><span style="font-weight:bold;">:</span><span style="color:rgb(187,136,68);">32:19 INFO source.NetcatSource: Source starting</span>
    <span style="color:rgb(0,128,128);">12/06/19 15</span><span style="font-weight:bold;">:</span><span style="color:rgb(187,136,68);">32:19 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]</span>
    <span style="color:rgb(0,128,128);">12/06/19 15</span><span style="font-weight:bold;">:</span><span style="color:rgb(187,136,68);">32:34 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          Hello world!. }</span>
    
    </span>
    







Congratulations - you’ve successfully configured and deployed a Flume agent! Subsequent sections cover agent configuration in much more detail.










  


Installing third-party plugins

Flume has a fully plugin-based architecture.

플럼은 플러그인 기반의 아키텍쳐를 가지고 있다.

While Flume ships with many out-of-the-box sources, channels, sinks, serializers, and the like, many implementations exist which ship separately from Flume.

플럼은 많은 Source, Channel, Sink, Serializer, 그리고 비슷한 많은 구현체가 존재하지만, 플럼으로 부터 분리되어 포함되있다.

  


While it has always been possible to include custom Flume components by adding their jars to the FLUME_CLASSPATH variable in the [flume-env.sh](http://flume-env.sh) file, Flume now supports a special directory called plugins.d which automatically picks up plugins that are packaged in a specific format.

플럼은 flume-env.sh 파일안에 FLUME_CLASS_PATH 변수에 커스텀 컴포넌트의 jar 를 등록하는 것으로 커스텀 플럼 컴포넌트를 포함할 수있다.

This allows for easier management of plugin packaging issues as well as simpler debugging and troubleshooting of several classes of issues, especially library dependency conflicts.

이것은 플러그인 패키징의 쉬운 관리를 허락하고 , 몇몇 이슈 클래스의 디버깅과 트러블 슈팅, 라이브러리 종속성 충돌을 단순하게 한다.;;; 아 몰라 블라블라;;;




##### The plugins.d directory







  


The plugins.d directory is located at $FLUME_HOME/plugins.d.

plugins.d 디렉터리는 $FLUME_HOME/plugins.d. 여기 위치한다.

At startup time, the flume-ng start script looks in the plugins.d directory for plugins that conform to the below format and includes them in proper paths when starting up java.

시작할 때, flume-ng 시작 스크립트는 다음과 같은 형식을 준수하고 자바를 실행할 때 적절한 경로에 그들을 포함하는 플러그인을 위해 plugins.d 디렉토리 안을 확인한다.







##### Directory layout for plugins




Each plugin (subdirectory) within plugins.d can have up to three sub-directories:




  1. lib - the plugin’s jar(s)
  2. libext - the plugin’s dependency jar(s)
  3. native - any required native libraries, such as .so files



Example of two plugins within the plugins.d directory:









    
    <span style="font-family:AppleGothic;">
    plugins.d/
    plugins.d/custom-source-1/
    plugins.d/custom-source-1/lib/my-source.jar
    plugins.d/custom-source-1/libext/spring-core-2.5.6.jar
    plugins.d/custom-source-2/
    plugins.d/custom-source-2/lib/custom.jar
    plugins.d/custom-source-2/native/gettext.so
    
    </span>
    



















  








  

