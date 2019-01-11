---
title: QueryDSL
date: 2019-01-10 23:36:34
tags:
---

JPQL을 편하게, 동적으로 작성할 수 있도록 JPA에서 공식 지원하는 Creteria 라는것이 있다.  
하지만 큰 단점이 있는데, 너무 불편하다는 것이다.  

그에 반해 JPA에서 공식 지원하지는 않지만  
쿼리를 문자가 아닌 코드로 작성해도 쉽고 간결하며, 모양도 쿼리와 비슷하게 개발할 수 있는 QueryDSL 이라는 것이 있다.  
QueryDSL은 오픈소스 프로젝트이며, 이름 그대로 데이터를 조회하는데 기능이 특화되어 있다.  
> `최범균`님이 번역한 공식 한국어 문서를 제공한다.  
> <http://www.querydsl.com/static/querydsl/4.0.1/reference/ko-KR/html_single/>  

# 설정  
- 필요 라이브러리  
    ```xml
    <dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>${querydsl.version}</version>
    <scope>provided</scope>
    </dependency>

    <dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>${querydsl.version}</version>
    </dependency>
    ```

    - querydsl-jpa : QueryDSL JPA 라이브러리  
    - querydsl-apt : 쿼리 타입(Q)를 생성할 때 사용하는 라이브러리  

- 쿼리 타입  
    엔티티를 기반으로 생성된 쿼리용 클래스를 말한다.  
    ```gradle
    buildscript {
        repositories {
            mavenCentral()
            maven {
                url "https://plugins.gradle.org/m2/"
            }
        }
        dependencies {
            classpath('net.ltgt.gradle:gradle-apt-plugin:0.18')
        }
    }
    repositories {
        mavenCentral()
    }
    apply plugin: "net.ltgt.apt"
    apply plugin: "net.ltgt.apt-idea”

    compile "org.projectlombok:lombok:${lombok_version}"
    annotationProcessor "org.projectlombok:lombok:${lombok_version}"

    compile "com.querydsl:querydsl-jpa:${querydsl_version}"
    compile "com.querydsl:querydsl-core:${querydsl_version}"
    compile "com.querydsl:querydsl-apt:${querydsl_version}"
    annotationProcessor "com.querydsl:querydsl-apt:${querydsl_version}:jpa"
    annotationProcessor "org.hibernate.javax.persistence:hibernate-jpa-2.1-api:${hibernate_jpa_api_version}"
    ```
    
    빌드하면 지정한 `outputDirectory에 지정한 target/generated-sources` 위치에 `QMember.java` 처럼 Q로 시작하는 쿼리 타입들이 생성된다.  

# 사용  
기본 구조는 아래와 같다.  

```java
```

<!-- more -->