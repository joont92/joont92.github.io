---
title: ''Java classpath, package''
tags:
---

https://effectivesquid.tistory.com/entry/%EC%9E%90%EB%B0%94-%ED%81%B4%EB%9E%98%EC%8A%A4%ED%8C%A8%EC%8A%A4classpath%EB%9E%80

java 런타임은 classpath에 지정된 경로를 모두 검색해서 class 파일을 찾는다  
클래스 패스를 뒤져 가장 먼저 찾은 class 파일을 사용한다  
classpath를 지정하는 방법은 CLASSPATH 변수를 사용하거나 java runtime에 classpath 플래그를 사용하는 방법이다

https://stackoverflow.com/questions/8227682/whats-the-default-classpath-when-not-specifying-classpath

java default classpath = .