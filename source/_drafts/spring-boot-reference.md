---
title: 'spring boot reference'
date: 2019-04-26 18:03:54
tags:
---

스프링 부트스프링부트 2.x는 자바 8이상 필요(11까지 호환)
메이븐 3.3, 그레들 4.4 이상
서블릿 3.1이상(어떠한 was도 상관없다고함)

메이븐도 플러그인이 있다
스프링 부트는 메이븐용 parent pom을 제공한다. 
(spring-boot-starter-parent)
spring-boot-run 태스크가 등록된다
이 태스크를 통해 스프링 부트 어플리케이션을 실행시킬 수 있다

스프링 부트 cli를 이용하면 가장 빠르게 스프링 부트 어플리케이션을 init할 수 있다

@EnableAutoConfiguration은 클래스패스에 있는 라이브러리들을 참조하여 스프링 설정을 진행한다. (starter dependency들과 잘 동작하도록 디자인되었다)
하지만 꼭 starter dependency를 사용하지 않아도 다운받은 라이브러리들을 참조하여 auto-configiration을 매우 잘 수행해준다

spingboot application의 run 메서드를 실행하면 springapplication 클래스로 행동을 위임한다
springapplication은 우리 어플리케이션을 만들고, 설정하고, 자동 설정된 톰캣을 띄운다
run에 전달된 인수들이 springapplication 으로 그대로 전달된다(?)

스프링 부트에서 만든 실행가능한 jar에는 컴파일된 니 코드들과 필요한 external library들이 합쳐서 압축된다
이를 만드려면 maven 이나 gradle 플러그인을 추가해야한다
이후 mvn package를 입력하면 executable jar가 만들어진다. 이는 10mb 정도 된다 보통.
(이 파일은 만들어진 jar를 스프링 부트가 리패키징 한 파일이다. 내부를 보고 싶다면 tar tvf을 입력해서 해제하면 .original 파일이 나온다)
'java -jar jar파일명' 을 입력하면 스프링 부트 어플이케이션이 실행된다.

스프링 부트는 필요한 의존성들을 직접 관리해준다.스프링 부트를 업대이트하면 관련된 라이브러리 버전도 같이 올라가므로, 신경쓸 필요없다. 물론 내가 라이브러리 버전을 오버라이딩 할수도 있다

스타터 라이브러리를 사용하면 관련된 라이브러리들을 한번에 얻을 수 있다
메이븐 pom.xml에서는 spring-boot-starter-* 형태로 입력하면 라이브러리들을 바로 볼 수 있다. 스프링부트에서 제공되는 스타터 라이브러리들은 전부 org.springframework.boot 패키지 아래 있다
starter logging, startet tomcat 같은것을 사용하면 특정(default?) 기술을 바꿔서 사용할수있다

ㅡㅡㅡㅡㅡㅡㅡ

클래스안에 package 선언이 따로 없으면 이 클래스는 디폴트패키지로 사용될수있고, 이는 특정 문제를 발생시킬수있다(compnentscan, entityscan등)
그러므로 자바에서 추천하는 naming convention을 사용하는것이 좋다(com.example.project 처럼 도메인 형태로)

메인 클래스는 다른 클래스들보다 한단계 위에 있는 루트 패키지에 넣을것을 추천한다
이 위치는 암묵적으로 무언가를 찾을때(scanning) 기준점으로 사용된다

스프링부트는 단 하나의@configuration 자바 config를 사용할것을 추천한다. 메인 메서드가 하나의 @configuration이 되기에 좋은 클래스이다.
(외부에는 xml로 작성된 스프링 프로젝트들이 매우 많은데, 가능하면 자바 config로 작성하라고 한다. @EnableXXX 라고 검색해보는게 좋은 시작점이다)

꼭 하나의 class로 configuration을 구성하지 않아도된다
@Import 를 사용하면 다른 클래스를 임포트할 수 있고, 혹은 @Configuration 클래스가 component scan에 걸리게하는 법도 있다
xml 설정의 경우 @ImportResource를 사용하면 된다

스프링 부트는 클래스패스에 있는 라이브러리를 보고 자동으로 프로잭트 설정을 한다. 만약 h2 라이브러리가 클래스패스에 있다면 따로 meorydb 세팅을 할 필요가없다(어떻게 되는걸까?)
@SpringBootApplication이나 @EnableAutoConfiguration을 @Configuration 클래스에 하나 추가해주면 된다

auto configure은 코드에 직접적으로 들어가는게 아니기 때문에 어디서든 auto configue 설정 값을 바꿀 수 있다(다른 datasource 빈 등록 등)
어떤 라이브러리들이 사용되는지 보고싶다면 --debug 모드로 보면 된다(?)  

# AutoConfiguration
- AutoConfiguration을 disable 허고 싶을 경우 exclude option을 사용한다
    ```java
    @EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
    ```

# Spring Bean DI
루트 클래스에 @EnableAutoConfiguration을 설정하면 @ComponentScan을 생략할 수 있다  
(스프링의 모든 @Component 클래스를 빈으로 등록한다)  

# @SpringBootApplication
이 어노테이션은 아래 3개 기능을 포함한다(default option으로 사용한것과 같음)  
- @EnableAutoConfiguration : spring auto-configuration 전략을 사용하게 함
- @ComponentScan : application이 위치한 곳부터 Component scan함
- @Configuration : 외부 빈들을 선언할 수 있고, 다른 빈들을 import 할 수 있음  

Application is just like any other Spring Boot application `except` that @Component ~~ and that ~~  
> Application은 Spring Boot Application과 같다 ~와 ~를 `제외하고는`  


<!-- more -->