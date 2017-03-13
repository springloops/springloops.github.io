---
author: springloops
comments: true
date: 2016-05-13
layout: post
title: 'How to use Gradle 1'
categories:
- Gradle
tags:
- gradle
- builder
- tools
- opensource
- java
---

[Building_and_Testing_with_Gradle](https://gradle.org/books/){:target="_blank"} 책을 빠르게 보며 요약한 내용입니다. **번역본이 아닙니다.**

언제나 그렇듯 원본을 참고하시기 바랍니다.

>
- [How to use Gradle 1](#): Gradle DefaultTask 의 메소드
- [How to use Gradle 2](/archivers/how-to-use-gradle2): Gradle DefaultTask 의 속성
- [How to use Gradle 3](/archivers/how-to-use-gradle3): Task Types
- [How to use Gradle 4](/archivers/how-to-use-gradle4): Gradle with Maven

# Gradle 설치

Mac 에서 Homebrew 를 이용한 설치

`
sudo brew install gradle
`

# Task 정의

## 선언과 할당을 분리

` << 는 append, doLast() 메소드 같은 기능`

``` gradle
task hello	// task 를 변수처럼 정의만 하고 작업은 할당하지 않은 상태.

hello << {
	print 'hello, '
}

hello << {
	println 'world'
}

결과 > hello, world
```

## Task Configuration blocks
``` gradle
앞 숫자는 실행 순서

3. task initializeDatabase
4. initializeDatabase << { println 'connect to database' }
5. initializeDatabase << { println 'update database schema' }
1. initializeDatabase { print 'configuring ' }
2. initializeDatabase { println 'database connection' }

결과>

configuring database connection
:initializeDatabase
connect to database
update database schema

BUILD SUCCESSFUL
```

## Task 는 Object 다.
모든 Task 는 아무런 action을 하지 않는 **DefaultTask** 를 상속 받는다.

하지만 Task 가 가지고 있어야할 method 의 인터페이스를 정의하고 있다.

### DefaultTask 의 메소드

DefaultTask 는 다음 메소드를 갖는다.

* dependsOn(task)
* doFirst(closure)
* doLast(closure)
* onlyIf(closure)


#### **dependsOn(task)**

`하나의 dependancy 만 갖을 경우 아래와 같은 방법으로 작성.`

``` gradle
task loadTestData {
    dependsOn createSchema
}

task loadTestData {
    dependsOn << createSchema
}

task lodaTestData {
    dependsOn 'createSchema'
}

// 명시적 호출
task loadTestData
loadTestData.dependsOn createSchema

// 단축형
task loadTestData(dependsOn: createSchema)
```

`하나 이상의 dependancy 를 갖을 경우 아래와 같이 작성.`

``` gradle
task loadTestData {
    dependsOn << compileTestClasses
    dependsOn << createSchema
}

task world {
    dependsOn compileTestClasses, createSchema
}

// 명시적 호출
task world
world.dependsOn compileTestClasses, createSchema

// 단축형
task world(dependsOn: [ compileTestClasses, createSchema ])
```

#### **doFirst(closure)**

`task 액션이 시작될 때 실행되는 블럭`

``` gradle
task setupDatabaseTests << {
	println 'load test data'
}

setupDatabaseTests.doFirst {
	println 'create schema'
}

gradle -b doFirstTest.gradle setupDatabaseTests

결과>
:setupDatabaseTests
create schema
load test data

// doFirst 호출 되고, << 로 append
```
`task.doFirst{ ... }` 는 `task { doFirst { ... } }` 와 같기 때문에, task configuration 블럭이 먼저 호출되고, 이때 doFirst 가 호출 되기 때문에 순서가 바뀐다.

```
task setupDatabaseTests << {
	println 'load test data'
}

setupDatabaseTests {
	doFirst {
		println 'create schema'
	}
}

gradle -b doFirstTest.gradle setupDatabaseTests

결과>
:setupDatabaseTests
create schema
load test data

BUILD SUCCESSFUL
```
반복되는 doFrist 호출은 추가된다.

```
task setupDatabaseTests << {
	println 'load test data'
}

setupDatabaseTests.doFirst {
	println 'create database schema'
}

setupDatabaseTests.doFirst {
	println 'drop database schema'
}

setupDatabaseTests << {
	println 'loaded test data'
}

gradle -b doFirstTest.gradle setupDatabaseTests

결과 >
:setupDatabaseTests
drop database schema
create database schema
load test data
loaded test data

BUILD SUCCESSFUL
```

다음과 같이 refectoring 가능함.

```
task setupDatabaseTests << {
	println 'load test data'
}

setupDatabaseTests {
	doFirst {
		println 'create database schema'
	}

	doFirst {
		println 'drop database schema'
	}
}
```

#### **doLast(closure)**

`액션의 마지막에 실행되는 것을 제외하고 doFirst 와 비슷한 메소드.`

```
task setupDatabaseTests << {
	println 'create database schema'
}

setupDatabaseTests.doLast {
	println 'load test data'
}

결과 >
:setupDatabaseTests
create database schema
load test data

BUILD SUCCESSFUL
```

doFirst 와 같이, 반복된 호출할 경우 추가된다.

```
task setupDatabaseTests << {
	println 'create database schema'
}

setupDatabaseTests.doLast {
	println 'load test data'
}

setupDatabaseTests.doLast {
	println 'update version table'
}

결과 >
:setupDatabaseTests
create database schema
load test data
update version table

```

#### **onlyIf(closure)**

`task 를 실행할지 말지를 결정한다.`

```
task createSchema << {
	println 'create database schema'
}

task loadTestData(dependsOn: createSchema) << {
	println 'load test data'
}

loadTestData.onlyIf {
	System.properties['load.data'] == 'true'
}

gradle -b onlyIf.gradle loadTestData

결과 >
:createSchema
create database schema
:loadTestData SKIPPED

BUILD SUCCESSFUL

gradle -b onlyIf.gradle -Dload.data=true loadTestData

결과 >
:createSchema
create database schema
:loadTestData
load test data

BUILD SUCCESSFUL
```

다음글 : [How to use Gradle 2](/archivers/how-to-use-gradle2): Gradle DefaultTask 의 속성
