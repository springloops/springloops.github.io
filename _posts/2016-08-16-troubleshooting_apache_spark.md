---
author: springloops
comments: true
date: 2016-08-16
layout: post
title: 'Apache Spark Troubleshooting'
categories:
- OPEN SOURCE/Apache Spark
tags:
- troubleshooting
- apache spark
- shark-shell
- opensource
- spark
---

# In CDH 5.8.0

## spark-shell

### #java.lang.NoClassDefFoundError: parquet/hadoop/ParquetOutputCommitter

```bash
Spark context available as sc (master = yarn-client, app id = application_1470984222577_0004).
java.lang.NoClassDefFoundError: parquet/hadoop/ParquetOutputCommitter
	at org.apache.spark.sql.SQLConf$.<init>(SQLConf.scala:319)
	at org.apache.spark.sql.SQLConf$.<clinit>(SQLConf.scala)
	at org.apache.spark.sql.SQLContext.<init>(SQLContext.scala:85)
	at org.apache.spark.sql.SQLContext.<init>(SQLContext.scala:77)
	at org.apache.spark.repl.SparkILoop.createSQLContext(SparkILoop.scala:1038)
	at $iwC$$iwC.<init>(<console>:15)
	at $iwC.<init>(<console>:24)
	at <init>(<console>:26)
	at .<init>(<console>:30)
	at .<clinit>(<console>)
	at .<init>(<console>:7)
	at .<clinit>(<console>)
	at $print(<console>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.repl.SparkIMain$ReadEvalPrint.call(SparkIMain.scala:1045)
	at org.apache.spark.repl.SparkIMain$Request.loadAndRun(SparkIMain.scala:1326)
	at org.apache.spark.repl.SparkIMain.loadAndRunReq$1(SparkIMain.scala:821)
	at org.apache.spark.repl.SparkIMain.interpret(SparkIMain.scala:852)
	at org.apache.spark.repl.SparkIMain.interpret(SparkIMain.scala:800)
	at org.apache.spark.repl.SparkILoop.reallyInterpret$1(SparkILoop.scala:857)
	at org.apache.spark.repl.SparkILoop.interpretStartingWith(SparkILoop.scala:902)
	at org.apache.spark.repl.SparkILoop.command(SparkILoop.scala:814)
	at org.apache.spark.repl.SparkILoopInit$$anonfun$initializeSpark$1.apply(SparkILoopInit.scala:133)
	at org.apache.spark.repl.SparkILoopInit$$anonfun$initializeSpark$1.apply(SparkILoopInit.scala:124)
	at org.apache.spark.repl.SparkIMain.beQuietDuring(SparkIMain.scala:305)
	at org.apache.spark.repl.SparkILoopInit$class.initializeSpark(SparkILoopInit.scala:124)
	at org.apache.spark.repl.SparkILoop.initializeSpark(SparkILoop.scala:64)
	at org.apache.spark.repl.SparkILoop$$anonfun$org$apache$spark$repl$SparkILoop$$process$1$$anonfun$apply$mcZ$sp$5.apply$mcV$sp(SparkILoop.scala:974)
	at org.apache.spark.repl.SparkILoopInit$class.runThunks(SparkILoopInit.scala:160)
	at org.apache.spark.repl.SparkILoop.runThunks(SparkILoop.scala:64)
	at org.apache.spark.repl.SparkILoopInit$class.postInitialization(SparkILoopInit.scala:108)
	at org.apache.spark.repl.SparkILoop.postInitialization(SparkILoop.scala:64)
	at org.apache.spark.repl.SparkILoop$$anonfun$org$apache$spark$repl$SparkILoop$$process$1.apply$mcZ$sp(SparkILoop.scala:991)
	at org.apache.spark.repl.SparkILoop$$anonfun$org$apache$spark$repl$SparkILoop$$process$1.apply(SparkILoop.scala:945)
	at org.apache.spark.repl.SparkILoop$$anonfun$org$apache$spark$repl$SparkILoop$$process$1.apply(SparkILoop.scala:945)
	at scala.tools.nsc.util.ScalaClassLoader$.savingContextLoader(ScalaClassLoader.scala:135)
	at org.apache.spark.repl.SparkILoop.org$apache$spark$repl$SparkILoop$$process(SparkILoop.scala:945)
	at org.apache.spark.repl.SparkILoop.process(SparkILoop.scala:1064)
	at org.apache.spark.repl.Main$.main(Main.scala:31)
	at org.apache.spark.repl.Main.main(Main.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.SparkSubmit$.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:731)
	at org.apache.spark.deploy.SparkSubmit$.doRunMain$1(SparkSubmit.scala:181)
	at org.apache.spark.deploy.SparkSubmit$.submit(SparkSubmit.scala:206)
	at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:121)
	at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
Caused by: java.lang.ClassNotFoundException: parquet.hadoop.ParquetOutputCommitter
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 52 more

<console>:16: error: not found: value sqlContext
         import sqlContext.implicits._
                ^
<console>:16: error: not found: value sqlContext
         import sqlContext.sql
                ^
```

CDH 기반 환경에서 local에서 spark-shell 로 CDH spark 접근시, 위 worning 이 발생할 때

[CDH tarball](http://archive.cloudera.com/cdh5/cdh/5/parquet-1.5.0-cdh5.8.0.tar.gz?_ga=1.209492112.343771408.1470976811) 페이지에서 Apache Parquet 을 내려 받고, 

/usr/local/parquet/current/parquet-hadoop/target/parquet-hadoop-1.5.0-cdh5.8.0.jar 파일을 

spark-defaults.conf 의 spark.driver.extraClassPath 옵션으로 추가 하면 해결됨.

- - -

### #Pending Spark shell job with logging that "Failed to connect to driver xxx at retrying ..."

```bash
ERROR yarn.ApplicationMaster: Failed to connect to driver at 192.168.99.1:55546, retrying ...
ERROR yarn.ApplicationMaster: Failed to connect to driver at 192.168.99.1:55546, retrying ...
ERROR yarn.ApplicationMaster: Failed to connect to driver at 192.168.99.1:55546, retrying ...
ERROR yarn.ApplicationMaster: Uncaught exception: 
org.apache.spark.SparkException: Failed to connect to driver!
	at org.apache.spark.deploy.yarn.ApplicationMaster.waitForSparkDriver(ApplicationMaster.scala:484)
	at org.apache.spark.deploy.yarn.ApplicationMaster.runExecutorLauncher(ApplicationMaster.scala:345)
	at org.apache.spark.deploy.yarn.ApplicationMaster.run(ApplicationMaster.scala:187)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anonfun$main$1.apply$mcV$sp(ApplicationMaster.scala:653)
	at org.apache.spark.deploy.SparkHadoopUtil$$anon$1.run(SparkHadoopUtil.scala:69)
	at org.apache.spark.deploy.SparkHadoopUtil$$anon$1.run(SparkHadoopUtil.scala:68)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1693)
	at org.apache.spark.deploy.SparkHadoopUtil.runAsSparkUser(SparkHadoopUtil.scala:68)
	at org.apache.spark.deploy.yarn.ApplicationMaster$.main(ApplicationMaster.scala:651)
	at org.apache.spark.deploy.yarn.ExecutorLauncher$.main(ApplicationMaster.scala:674)
	at org.apache.spark.deploy.yarn.ExecutorLauncher.main(ApplicationMaster.scala)
INFO yarn.ApplicationMaster: Final app status: FAILED, exitCode: 10, (reason: Uncaught exception: org.apache.spark.SparkException: Failed to connect to driver!)
INFO util.ShutdownHookManager: Shutdown hook called
```

위와 같이 Yarn ApplicationMaster(이하 AM) 가 spark-shell 이 실행된 local IP 가 아닌 다른 내부 IP 로 접근을 하려고 시도하지만, spark-shell 이 실행된 네트워크 인터페이스가 아니기 때문에, 결국 연결 실패로 종료되는 경우.

맥에서 Docker 를 실행하거나, VM 툴 들을 사용할 경우, 가상의 네트워크 인터페이스들이 생성된다.

spark-shell 이 그 가상 네트워크 인터페이스의 IP를 사용하여 AM 에 접속하여, Spark Job 이 Yarn 에서 pending 되는 상황이다.

```bash
export SPARK_LOCAL_IP=<Real IP Address>
```

위와 같은 설정으로, local IP 를 고정 시킬수 있음.

bash_profile 정도에 등록하면 해결된다.


- - -