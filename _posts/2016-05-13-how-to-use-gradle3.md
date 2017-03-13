---
author: springloops
comments: true
date: 2016-05-13
layout: post
title: 'How to use Gradle 3'
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
- [How to use Gradle 2](/archivers/how-to-use-gradle2): Gradle DefaultTask 의 속성
- [How to use Gradle 3](#): Task Types
- [How to use Gradle 4](/archivers/how-to-use-gradle4): Gradle with Maven

### Task Types
Gradle 에는 DefaultTask 외에 copying, archiving, executing 프로그램 등 다양한 task가 있다.

- Copy
- Jar
- JavaExec

#### Copy
Copy Task 는 복사

```
task copyFiles(type: Copy) {
	from 'resource'
    into 'target'
    include '**/*.xml', '**/*.txt', '**/*.properties'
}
```
target 이 없을 경우, 디렉토리를 생성한다.

위 예제에서는 resource 디렉토리 아래 모든 xml, txt, properties 파일을 target 으로 복사한다.

#### Jar
Jar Task 는 source 파일로 부터 Jar 파일을 만든다.
