---
author: springloops
comments: true
date: 2016-05-13
layout: post
title: 'How to use Gradle 2'
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
- [How to use Gradle 1](/archivers/how-to-use-gradle1): Gradle DefaultTask 의 메소드
- [How to use Gradle 2](#): Gradle DefaultTask 의 속성
- [How to use Gradle 3](/archivers/how-to-use-gradle3): Task Types
- [How to use Gradle 4](/archivers/how-to-use-gradle4): Gradle with Maven


### DefaultTask 의 속성
DefaultTask 는 다음 속성을 갖는다.

* didWork
* enabled
* path
* logger
* logging
* description
* temporaryDir

#### didWork
Task 가 성공적으로 완료되었는지 여부를 Boolean 으로 알려준다.

```
apply plugin: 'java'

task emailMe(dependsOn: compileJava) << {
	if (tasks.compileJava.didWork) {
		println 'send email announcing success'
	}
}

gradle -b didWork.gradle emailMe
:compileJava UP-TO-DATE
:emailMe

BUILD SUCCESSFUL
```
#### enabled
Boolean 속성으로 Task 가 실행될지 여부를 결정한다.

```
task templates << {
	println 'process email templates'
}

task sendEmail(dependsOn: templates) << {
	println 'send email'
}

sendEmail.enabled = false


gradle -b enabled.gradle sendEmail
:templates
process email templates
:sendEmail SKIPPED

BUILD SUCCESSFUL
```

#### path

String 속성으로 Task 의 Fully Qualified Path ( 완전한 경로 ) 를 갖는다.
기본적으로 콜론과 Task이름이 나온다. (ex :taskName)
```
task echoMyPath << {
	println "this task's path is ${path}"
}


gradle -b path.gradle echoMyPath
:echoMyPath
this task's path is :echoMyPath

BUILD SUCCESSFUL
```
첫번째 콜론은 Task의 top-level 빌드 파일의 위치를 가리킨다.
주어진 Build 파일이 top-level 이 아닐 경우, Gradle 은 하위 프로젝트, 중첩 프로젝트를 지원한다.

#### logger
Gradle 내부 오브젝트의 참조.

Gradle logger 는 org.slf4j.Logger 인터페이스의 구현체이고, 몇가지 확장 레벨이 추가되었다.

Logger Level
- DEBUG
- INFO
- LIFECYCLE
- WARN
- QUIET
- ERROR

```
task logLevel << {
  // list 변수 정의
  // 로그 레벨의 내림차순으로 정의된 리스트
	def levels = ['DEBUG', 'INFO', 'LIFECYCLE', 'QUIET', 'WARN', 'ERROR']

  // foreach
	levels.each { level ->
		logging.level = level // 로깅 레벨 정의

    // 로그 레벨에 맞춰 출력
		def logMessage = "SETTING LogLevel=${level}"
		logger.error logMessage
		logger.error '-' * logMessage.size()
		logger.debug 'DEBUG ENABLED'
		logger.info 'INFO ENABLED'
		logger.lifecycle 'LIFECYCLE ENAABLED'
		logger.warn 'WARN ENABLED'
		logger.quiet 'QUIET ENABLED'
		logger.error 'ERROR ENABLED'
		println 'THIS IS println OUTPUT'
		logger.error ' '
	}
}

gradle -b logger.gradle logLevel
10:06:02.908 [LIFECYCLE] [class org.gradle.TaskExecutionLogger] :logLevel
10:06:02.929 [ERROR] [org.gradle.api.Task] SETTING LogLevel=DEBUG
10:06:02.934 [ERROR] [org.gradle.api.Task] ----------------------
10:06:02.935 [DEBUG] [org.gradle.api.Task] DEBUG ENABLED
10:06:02.935 [INFO] [org.gradle.api.Task] INFO ENABLED
10:06:02.936 [LIFECYCLE] [org.gradle.api.Task] LIFECYCLE ENAABLED
10:06:02.936 [WARN] [org.gradle.api.Task] WARN ENABLED
10:06:02.937 [QUIET] [org.gradle.api.Task] QUIET ENABLED
10:06:02.937 [ERROR] [org.gradle.api.Task] ERROR ENABLED
10:06:02.951 [QUIET] [system.out] THIS IS println OUTPUT
10:06:02.951 [ERROR] [org.gradle.api.Task]
SETTING LogLevel=INFO
---------------------
INFO ENABLED
LIFECYCLE ENAABLED
WARN ENABLED
QUIET ENABLED
ERROR ENABLED
THIS IS println OUTPUT

SETTING LogLevel=LIFECYCLE
--------------------------
LIFECYCLE ENAABLED
WARN ENABLED
QUIET ENABLED
ERROR ENABLED
THIS IS println OUTPUT

SETTING LogLevel=QUIET
----------------------
QUIET ENABLED
ERROR ENABLED
THIS IS println OUTPUT

SETTING LogLevel=WARN
---------------------
WARN ENABLED
QUIET ENABLED
ERROR ENABLED
THIS IS println OUTPUT

SETTING LogLevel=ERROR
----------------------
ERROR ENABLED


BUILD SUCCESSFUL
```

#### logging
log level 을 접근할 수 있게 해주는 속성.
logger 예시에서 logging.level 참조.

#### description
description 속성은 단순한 사용자를 위한 메타 정보를 남길 수 있다.

```
task helloWorld(description: 'say hello to the world')  >> {
	println 'hello world'
}
```

다음과 같이 사용할 수도 있다.

```
task helloWorld << {
	println 'hello, world'
}

helloWorld {
	description = 'say hello to the world'
}
```

#### temporaryDir
temporaryDir 속성은 빌드 파일에 속하는 임시 디렉토리를 가리키는 File 객체를 반환한다.

이 디렉토리는 모든 작업 의 중간 결과를 저장하는, 또는 작업의 내부 처리를 위해 임시장소를 필요로하는 작업에 일반적으로 사용할 수 있다.

### Dynamic Properties

다음과 같이 속성을 추가 할 수 있다.

`gradle 2.1 이전 버전...`

```

task copyFiles {
	fileManifest = [ 'data.csv', 'config.json' ]
}

task createArtifact( dependsOn: copyFiles ) << {
	println "FILES IN MANIFEST: ${copyFiles.fileManifest}"
}

```

`gradle 2.1 이후 버전... ext object 를 사용한다.`


```
task copyFiles {
	ext.fileManifest = [ 'data.csv', 'config.json' ]
}

task createArtifact( dependsOn: copyFiles ) << {
	println "FILES IN MANIFEST: ${copyFiles.fileManifest}"
}

gradle -b dynamic.gradle createArtifact
:copyFiles UP-TO-DATE
:createArtifact
FILES IN MANIFEST: [data.csv, config.json]

BUILD SUCCESSFUL

```

`다음과 같이 외부에 ext 선언 후 사용 가능`

```
ext {
	test1 = 'test1'
	test2 = 'test2'
}

task printExtProperties << {

	println "ext properties ${test1}, ${test2}"
}

gradle -b dynamic.gradle printExtProperties
:printExtProperties
ext properties test1, test2

BUILD SUCCESSFUL

```

다음글: [How to use Gradle 3](/archivers/how-to-use-gradle3): Task Types