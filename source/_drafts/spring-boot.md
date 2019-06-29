---
title: spring-boot
tags:
---

스프링을 사용하기 편하게 해주는 일종의 도구  
= 개발자 -> 스프링 부트 -> 스프링  

# 스타터 
스타터는 여러 의존관계들을 묶어서 하나로 제공해주며, 버전관리까지 해준다  
부트의 버전과 스타터 라이브러리의 버전은 같이 간다  
= 부트 버전 2.1에 spring-boot-data-jpa-starter 라고 명시하면 starter 버전 2.1이 다운로드 된다(아마도)  
spring-boot repository의 spring-boot-dependencies 에서 각 버전별 사용하는 라이브러리 버전을 볼 수 있다  

# 그레들
기본구성
```
gradle/
    gradle-wrapper.jar
    gradle-wrapper.properties
gradlew
gradle.bat
settings.gradle
```
gradler wrapper를 사용함으로써 개발환경마다 그레들 버전이 달라서 생기는 오류를 방지한다  
(gradlew, gradle.bat은 사용하는 PC의 그레들이 아니라 wrapper를 사용한다)  
VCS로 버전관리도 가능하다  
그레들 버전을 올리고 싶다면 properties 를 수정하면 된다  

그레들 멀티모듈을 사용하고 싶을 경우 New Module로 생성하고, settings.gradle에 include 해주면 된다  

# 설정파일
싱글파일 : application.yml 내에 --- 로 구분  
멀티파일 : application-[env].yml 로 구분  
         [env] 파일이 1순위로 적용되고, application.yml이 2순위로 적용된다
jar 파일 실행 시 -D 옵션에 spring.profiles.active 값으로 주면 된다

# @Value
```yml
test:
    property: test-property
testProperty: test-property
testProperties: test-property1, test-property2
```

```java
@Value("${test.property}")
private String test;

@Value("${testProperty}")
private String test;

@Value("${testProperties}")
private String[] tests;

@Value("#{'${property.test}'.split(',')}")
private List<String> tests;
```