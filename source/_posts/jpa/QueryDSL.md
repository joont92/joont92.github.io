---
title: [jpa] QueryDSL
date: 2019-01-10 23:36:34
tags:
    - QueryDSL 문법  
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

# 사용(4.1.3 버전 기준)  

## 기본 사용법
동적으로 생성할 쿼리는 `JPAQuery`를 사용하여 만들 수 있는데, 이것보단 `JPAQueryFactory`를 사용하는게 권장된다고 한다.  
> JPQLQuery 인터페이스가 queryDSL 동적 쿼리 생성의 기준이 되는 인터페이스이고,  
> JPAQuery는 JPQLQuery를 구현한 클래스이다. 근데 왜 이름이 JPAQuery일까?  

```java
JPAQueryFactory queryFactory = new JPAQueryFactory(em);
QMember member = QMember.member;

Member foundMember = 
    queryFactory.selectFrom(member) // select + from
    .where(customer.username.eq("joont"))
    .fetchOne();
```

대충 위의 형태로 사용할 수 있다.  

### 결과반환  
- fetch : 조회 대상이 여러건일 경우. 컬렉션 반환  
- fetchOne : 조회 대상이 1건일 경우(1건 이상일 경우 에러). generic에 지정한 타입으로 반환  
- fetchFirst : 조회 대상이 1건이든 1건 이상이든 무조건 1건만 반환. 내부에 보면 `return limit(1).fetchOne()` 으로 되어있음  
- fetchCount : 개수 조회. long 타입 반환  
- fetchResults : 조회한 리스트 + 전체 개수를 포함한 QueryResults 반환. count 쿼리가 추가로 실행된다.  

### 프로젝션  
프로젝션을 지정한다.  

```java
List<Member> foundMembers = 
    queryFactory.select(member)
    .from(member, order)
    .fetch();
```

(아직 나오진 않았지만 from 절에 위처럼 쿼리 타입을 연속으로 줄 경우, 두 엔티티가 조인된다.)  

member와 order가 조인된 상태에서 member 엔티티의 속성만 가져온다.  
(select를 생략하면 기본적으로 from의 첫번째 엔티티를 프로젝션 대상으로 쓴다)  

### from
쿼리할 대상을 지정한다.  

```java
List<Member> foundMembers = 
    queryFactory.from(member)
    .fetch();
```

member 테이블을 전체 조회하게 된다. 프로젝션 지정(select)가 빠졌지만 위와 동일하게 from의 첫번째 엔티티를 사용한다.  

> from과 select를 나누기 보단 `selectFrom` 절을 쓰는것이 더 낫다.  

### 조인  
join, innerJoin, leftJoin, rightJoin 을 지원한다.  
개인적으로 from절에 multiple arguments를 주는것보다 이게 더 좋다.(SQL에서도...)  

```java
QTeam team = QTeam.team;

List<Member> foundMembers = 
    queryFactory.selectFrom(member)
    .innerJoin(member.team, team)
    .fetch();
```

join의 첫번쨰 인자로는 join할 대상, 두번쨰 인자로는 join할 대상의 쿼리 타입을 주면 된다. on 절은 자동으로 붙는다.  

- 추가적인 on 절도 사용할 수 있다.  
    ```java
    List<Member> foundMembers = 
        queryFactory.selectFrom(member)
        .innerJoin(member.team, team)
        .on(member.username.eq("joont"))
        .fetch();
    ```

### 조건  
```java
List<Member> foundMembers = 
    queryFactory.selectFrom(member)
    .where(member.username.eq("joont")) // 1. 단일 조건  
    .where(member.username.eq("joont"), member.homeAddress.city.eq("seoul")) // 2. 복수 조건. and로 묶임  
    .where(member.username.eq("joont").or(member.homeAddress.city.eq("seoul"))) // 3. 복수 조건. and나 or를 직접 명시할 수 있음  
    .where((member.username.eq("joont").or(member.homeAddress.city.eq("seoul"))).and(member.username.eq("joont").or(member.homeAddress.city.eq("busan"))))
    .fetch();
```

`(E1 and E2) or (E3 and E4)` 같은 형태도 가능하다. 그냥 괄호로 묶어주면 된다.  

```java
List<Member> foundMembers = 
    queryFactory.selectFrom(member)
    .where((member.username.eq("joont").or(member.homeAddress.city.eq("seoul"))).and(member.username.eq("joont").or(member.homeAddress.city.eq("busan"))))
    .fetch();
```

두가지 조건이 괄호로 묶이게 되었을때, or 이면 합집합이고 and 이면 교집합이다.  
참고로 `(E1 and E2) or (E3 and E4)` 는 괄호가 생략되고 `(E1 or E2) and (E3 or E4)` 는 잘 동작한다.  

### 그룹핑
group by도 가능하다.  

```java
List<String> foundCities = 
    queryFactory.from(member)
    .select(member.homeAddress.city)
    .groupBy(member.homeAddress.city)
    .fetch();
```

city로 group by 한 뒤 city만 출력하게 된다.  

- having도 가능하다. 집계함수도 쓸 수 있다.  
    ```java
    List<String> foundItems = 
        queryFactory.select(item.category) // category가 그냥 String이라고 가정
        .from(item)
        .groupBy(item.category)
        .having(item.price.avg().gt(1000)) // 집계함수 사용
        .fetch();
    ```

### 정렬
```java
List<Member> foundMembers = 
    queryFactory.selectFrom(member)
    .orderBy(member.id.asc(), member.username.desc())
    .fetch();
```

### 페이징  
시작 인덱스를 지정하는 `offset`,  
조회할 개수를 지정하는 `limit`,  
두개를 인수로 받는 QueryModifiers를 사용하는 `restrict`를 지원한다.  

근데 실제로 페이징 처리를 하려면 전체 데이터 개수를 알고 있어야하므로, fetchResults()를 사용해야 한다.  

```java
QueryResults<Member> result = 
    queryFactory.selectFrom(member)
    .offset(10)
    .limit(10)
    .fetchResults();

List<Member> foundMembers = result.getResults(); // 조회된 member
long total = result.getTotal(); // 전체 개수  
long offset = result.getOffset(); // offset
long limit = result.getLimit(); // limit
```

## 다중 결과 반환   
다중 프로젝션 할 경우 Tuple 클래스로 받을 수 있다.  

```java
List<Tuple> foundMembers = 
    queryFactory.select(member.username, member.homeAddress.city)
    .from(member)
    .fetch();

System.out.println(founeMembers.get(0));
System.out.println(founeMembers.get(1));
```

리턴되는 클래스가 `class com.querydsl.core.types.QTuple$TupleImpl` 인데, 이것보단 아래 빈 생성(bean population)을 쓰는게 더 나아보인다.  

## 빈 생성  
자바빈을 말한다(스프링 빈 아님).  
출력되는 다중 결과를 빈으로 변경해서 리턴할 수 있다.  

```java
List<MemberDTO> foundMembers = 
    queryFactory.select(Projections.bean(UserDTO.class, member.username, member.homeAddress.city))
    .from(member)
    .fetch();
```

위의 `bean` 메서드를 호출하면 전달받은 인자와 동일하게 UserDTO의 setter를 호출한다.  
`field` 메서드를 사용하면 필드에 직접 접근하고(private도 가능),  
`constructor` 메서드를 사용하면 생성자를 사용한다. 지정한 프로젝션과 파라미터 순서가 같은 생성자가 필요하다.  

엔티티의 필드명과 빈의 필드명이 다를 경우 아래와 같이 사용할 수 있다.  
```java
queryFactory.select(Projections.bean(UserDTO.class, member.username.as("name"), member.homeAddress.city))
    ....
```

Member 엔티티 필드 username을 MemberDTO의 name에 전달하게 된다.  

## 서브쿼리  
JPAExpression 을 사용하면 된다.  

```java
QMember member = QMember.member;
QMember subQueryMember = new QMember("subQueryMember"); // 추가로 생성해줘야 함  

List<Tuple> foundMembers = 
    queryFactory.select(member.name, member.homeAddress.city)
        .from(member)
        .where(member.name.in(
                JPAExpressions.select(memberForSubquery.name)
                .from(memberForSubquery)
        ))
        .fetch();
```

## 동적 조건  
`com.querydsl.core.BooleanBuilder` 를 사용하면 동적 조건을 생성할 수 있다.  

```java
BooleanBuilder builder = new BooleanBuilder();
if(param.getId() != null){
    builder.and(member.id.eq(param.getId()));
}
if(param.getName() != null){
    builder.and(member.name.contains(param.getName()));
}

List<Member> list = 
    queryFactory.selectFrom(member)
    .where(booleanBuilder)
    .fetch();
```

## 수정, 삭제, 배치 쿼리  
- **update**  
    `JPAUpdateClause` 클래스를 통해 실행할 수 있다.(인터페이스는 `UpdateClause`이다)  
    `JPAQueryFactory`의 `update` 메서드를 통해 생성할 수 있다.  
    ```java
    QCustomer customer = QCustomer.customer;
    // rename customers named Bob to Bobby
    queryFactory.update(customer).where(customer.name.eq("Bob"))
        .set(customer.name, "Bobby")
        .execute();
    ```

- **delete**  
    `JPADeleteClause` 클래스를 통해 실행할 수 있다.(인터페이스는 `DeleteClause`이다)  
    `JPAQueryFactory`의 `delete` 메서드를 통해 생성할 수 있다.  
    ```java
    QCustomer customer = QCustomer.customer;
    // delete all customers
    queryFactory.delete(customer).execute();
    // delete all customers with a level less than 3
    queryFactory.delete(customer).where(customer.level.lt(3)).execute();
    ```

<!-- more --> 