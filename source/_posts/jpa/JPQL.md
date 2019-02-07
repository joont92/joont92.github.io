---
title: JPQL
date: 2019-01-01 21:24:46
tags:
    - 자바 ORM 표준 JPA 프로그래밍
    - 객체지향 쿼리 언어
    - JPQL 문법  
    - 네이티브 쿼리  
    - 벌크 연산  
---

JPA에서 현재까지 사용했던 `검색`은 아래와 같다.  
- 식별자로 조회 EntityManager.find()  
- 객체 그래프 탐색 e.g. a.getB().getC()

하지만 현실적으로 이 기능만으로 어플리케이션을 개발하기에는 무리이다.  
그렇다고 모든 엔티티를 메모리에 올려두고 어플리케이션 내에서 필터하는 것은 현실성이 없는 소리이다.  
즉, 데이터베이스에서 필터해서 조회해올 무언가가 필요하고, 그게 `객체지향 쿼리 언어(JPQL)`이다.  

JPQL은 엔티티 객체를 조회하는 객체지향 쿼리 언어이다.  
문법은 SQL과 비슷한데, 실제론 SQL을 추상화 한것이기 때문에 특정 데이터베이스에 의존하지 않는 특징이 있다.  

SQL과 비슷하게 SELECT, UPDATE, DELETE 문을 사용할 수 있다.  
(참고로 엔티티 저장은 그냥 entityManager.persist를 사용하면 되므로 INSERT 문은 없다.)  
JPQL에서 UPDATE, DELETE 문은 벌크 연산이라고 해서 뒤에서 따로 설명할 것이므로, SELECT 문만 작성하겠다.  

# 기본 문법  
기본 형태는 아래와 같다.  

```sql
SELECT m FROM Member AS m WHERE m.username = 'Hello'
```

1. 대소문자 구문  
    - 엔티티와 속성은 대소문자를 구분한다. Member와 member는 다르고 username과 USERNAME은 다르다.  
    - SELECT, FROM 같은 JPQL 키워드는 대소문자를 구분하지 않는다.   
2. 엔티티 이름  
    - FROM 이후에 오는 대상은 테이블 이름이 아니라 엔티티 이름이다.  
    - 기본값인 클래스명을 엔티티명으로 사용하는 것을 추천한다.  
3. 별칭은 필수  
    - JPQL은 별칭을 필수로 사용해야 한다. AS 뒤에 `m`이 Member의 별칭이다.  
    - AS는 생략 가능하다.  

## TypedQuery, Query  
작성한 JPQL을 실행시키기 위해 만드는 `쿼리 객체`이다.  
JPQL이 반환할 타입을 명확하게 지정할 수 있으면 TypedQuery를 사용하고, 명확하게 지정할 수 없으면 Query를 사용하면 된다.  

```java
// 조회대상이 정확히 Member 엔티티이므로 TypedQuery 사용 가능
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);

// 조회대상이 String, Integer로 명확하지 않으므로 Query 사용
Query query = em.createQuery("SELECT m.username, m.age FROM Member m");
```

TypedQuery로 실행된 쿼리는 두번쨰 인자로 주어진 클래스를 반환하고,  
Query의 경우 예제처럼 조회 컬럼이 1개 이상일 경우 `Object[]`, 1개일 경우 `Object`를 반환한다.  
> 참고로 String 타입의 필드만 조회하고 TypedQuery<String[]> 를 사용하는 방식은 안된다.  
> 하나하나 다 체크하기에는 너무 많으니까..  

## 결과 조회  
쿼리 객체에서 아래의 메서드들을 사용해 JPQL을 실행한다.  
- query.getResultList()  
    > 결과를 컬렉션으로 반환한다. 결과가 없으면 빈 컬렉션이 반환된다. 1건이면 1건만 들어간 컬렉션이 반환된다.  
- query.getSingleResult()  
    > 결과가 정확히 1건 일때 사용한다.  
    > 결과가 없으면 javax.persistence.NoResultException, 결과가 1건 이상이면 javax.persistence.NonUniqueResultException이 발생한다.  
    > 근데 얘는 `Optional`을 반환해야 하지 않을까?  

# 파라미터 바인딩  
아래와 같은 `이름 기준 파라미터 바인딩`을 지원한다.  

```java
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m WHERE m.username = :username", Member.class)
    .setParameter("username", "joont1"); // JPQL은 대부분 메서드 체인 방식으로 되어있어서 이렇게 연속해서 작성하는 것이 가능하다

List<Member> result = query.getResultLst();
```

username은 Member 클래스에 정의된 프로퍼티 이름이다. 앞에 `:`를 붙여서 바인딩한다.  
username 에 joont1 이 바인딩 될 것이다.  

참고로 아래와 같이 위치 기준 파라미터 바인딩도 지원하기는 한다.  

```java
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m WHERE m.username = ?1", Member.class)
    .setParameter(1, "joont1");
```

이것보다는 전자가 더 명확하다.  

참고로 LIKE 연산처럼 `%` 같은 특수문자가 필요할 경우 전달하는 파라미터에 붙여서 사용하면 된다.  
```java
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m WHERE m.username LIKE :username", Member.class)
    .setParameter("username", "%joont%"); // 이런식으로
```

> **파라미터 바인딩 방식은 선택이 아닌 필수이다**  
> - JPQL에 직접 문자를 더하면 SQL Injection을 당할 수 있다  
> - JPA에서 파라미터만 다를 뿐 같은 쿼리로 인식하므로, JPQL을 SQL로 파싱한 결과를 재사용할 수 있다  
> - SQL 내에서도 같은 쿼리는 결과를 재사용한다  

# 프로젝션  
조회할 대상을 지정하는 것을 프로젝션이라고 한다.  
`SELECT [프로젝션 대상] FROM` 으로 대상을 지정한다.  
대상은 엔티티 타입, 임베디드 타입, 스칼라 타입이 있다.  

## 엔티티 프로젝션  
```sql
SELECT m FROM Member m // member

SELECT m.team FROM Memher m // team
```

둘 다 엔티티를 프로젝션 대상으로 사용했다.  
참고로 이렇게 조회한 엔티티는 영속성 컨텍스트에서 관리된다.  

## 임베디드 타입 프로젝션  
엔티티를 통해서 조회한다.  

```java
Address address = 
    em.createQuery("SELECT m.address FROM Member m", Address.class)
    .getSingleResult();
```

임베디드 타입은 엔티티 타입이 아닌 값 타입이므로  
이렇게 조회한 임베디드 타입은 영속성 컨텍스트에서 관리되지 않는다.  

## 스칼라 타입 프로젝션  
```java
// 이름조회
TypedQuery<String> query = em.createQuery("SELECT m.username FROM Member m", String.class);
List<String> resultList = query.getResultList();

// 이름조회(중복제거)
TypedQuery<String> query = em.createQuery("SELECT DISTINCT m.username FROM Member m", String.class);
List<String> resultList = query.getResultList();

// 통계 쿼리
TypedQuery<Double> query = em.createQuery("SELECT AVG(o.orderAmount) FROM Order o", Double.class);
List<Double> resultList = query.getResultList();
```

조회되는 컬럼이 1건이라 TypedQuery를 사용하였다. 보다시피 통계 쿼리도 스칼라 타입으로 조회할 수 있다.  

## 여러 값 조회  
아래와 같이 여러값으로 조회했을 때는 TypedQuery를 사용할 수 없고, Query만 사용할 수 있다.  

```java
Query query = em.createQuery("SELECT m.username, m.age, m.team FROM Member m");
List<Object[]> resultList = query.getResultList();

for(Object[] row : resultList){
    String username = (String)row[0];
    Integer age = (Integer)row[1];
    Team team = (Team)row[2];
}
```

물론 아때도 조회한 엔티티는 영속성 컨텍스트에서 관리된다.  

## NEW 명령어  
NEW 명령어를 사용하면 Object[] 대신 바로 객체로 생성해서 받아볼 수 있다.  

```java
TypedQuery<UserDTO> query = 
    em.createQuery("SELECT NEW com.joont.dto.UserDTO(m.username, m.age, m.team) FROM Member m", UserDTO.class);

List<UserDTO> resultList = query.getResultList();
```

기존이라면 하나하나 번거롭게 변환했어야 했을 작업을 NEW 명령어를 사용해서 간단하게 처리했다.  
NEW 명령어를 사용하려면 아래 2가지를 주의해야 한다.  
- 패키지명을 포함한 클래스명을 입력해야 한다.  
- 순서와 타입이 일치하는 생성자가 필요하다.  

직접 쓰라고 있는 기능은 아닌 것 같다.  
라이브러리들이 적절히 구현해라고 만들어놓은 기능인듯(QueryDSL 등)  

# 페이징 API
JPA는 데이터베이스들의 페이징들을 아래의 두 API로 추상화했다.  
(페이징은 데이터베이스마다 문법이 다 다르다)  
- setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작)  
- setMaxResult(int maxResult) : 조회할 데이터 수  

```java
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);

query.setFirstResult(10);
query.setMaxResult(20);
query.getResultList();
```

11번쨰 데이터부터 시작해서 20개를 조회한다. 즉 11~30번 데이터를 조회하게 된다.  
지원하는 모든 데이터베이스를 추상화했기 때문에 데이터베이스가 바껴도 방언만 바꿔주면 된다.  

# 집합과 정렬  
## 집합 함수  
함수 | 설명 | 리턴타입
---|----|-----
COUNT | 결과 수를 구한다 | Long
MAX, MIN | 최대, 최소값을 구한다 | 대상에 따라 다름
AVG | 평균값을 구한다. 숫자타입만 사용할 수 있다. 숫자가 아니면 0을 리턴한다. | Double | 
SUM | 합을 구한다. 숫자타입만 사용할 수 있다. | 정수합 : Long<br/>소수합 : Double<br/>  

> **집합 함수 사용 시 참고사항**  
> - 통계를 계산할 때 NULL값은 무시된다(`COUNT(*)`은 제외).  
> - 값이 없을 때 SUM, AVG, MAX, MIN를 사용하면 NULL을 리턴한다. COUNT는 0을 리턴한다.  
> - DISTINCT를 집합 함수안에 사용하면 중복된 값을 제거하고 집합을 구한다.  

## 그룹핑
GROUP BY, HAVING도 사용할 수 있다.  

```sql
SELECT t.name, COUNT(m.age), SUM(m.age), AVG(m.age), MAX(m.age), MIN(m.age)
FROM Member m LEFT JOIN m.team t
GROUP BY t.name  
HAVING AVG(m.age) >= 10
```

(team의 이름으로 그룹화한 뒤 나이의 평균이 10살 이상인 그룹에 대해서 집합을 구했다.)  
문법은 아래와 같다.  
> group by절 : GROUP BY {단일값 경로 | 별칭}  
> having절 : HAVING 조건식  

이런식의 통계 쿼리는 보통 전체 데이터를 기준으로 사용하므로 실시간으로 사용하기에는 부담이 많다.  

## 정렬  
ORDER BY도 사용할 수 있다.  

```sql
SELECT t.name, COUNT(m.age) AS cnt  
FROM Member m LEFT JOIN m.team t  
GROUP BY t.name  
ORDER BY t.name ASC, cnt DESC
```

문법은 아래와 같다.  
> order by절 : ORDER BY {상태필드 경로 | 결과변수 [ASC | DESC]}

상태필드는 m.name 같이 객체의 상태를 나타내는 필드를 말하고,  
결과변수는 SELECT 절에 나타나는 값을 말한다. 위의 예제에서는 cnt가 결과변수이다.  

# 조인  
## 내부 조인  
```sql
SELECT m FROM Member m INNER JOIN m.team t
```

보다시피 일반적인 SQL 조인과 조금 다르다.  
가장 큰 특징은 `연관 필드`를 사용해서 조인한다는 점이다.  
즉, 조인을 사용하려면 엔티티에 연관관계 명시는 필수적으로 되어있어야 한다.  
(위와 같이 작성하면 Member의 team 필드에서 관계의 정보를 얻은 뒤 조인할 것이다)  

아래는 잘못 작성된 JPQL 조인이다.  

```sql
SELECT m FROM Member m INNER JOIN Team t // 잘못된 조인
```

만약 조인한 두 개의 엔티티를 조회하려면 다음과 같이 작성하면 된다.  

```sql
SELECT m,t
FROM Member m INNER JOIN m.team t
```

서로 다른 타입의 두 엔티티를 조회했으므로 TypedQuery는 사용할 수 없다.  

```java
List<Object[]> list = em.createQuery(jpql).getResultList(); // 위에서 작성한 쿼리

for(Object[] o : list){
    Member m = (Member)o[0];
    Team t = (Team)o[1];
}
```

## 외부 조인  
키워드만 바꿔주면 된다.  

```sql
SELECT m FROM Member m LEFT JOIN m.team t
```

## 컬렉션 조인  
일대다 관계나 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것을 말한다.  
아래와 같이 `컬렉션 값 연관 필드`를 사용하면 된다.  

```sql
SELECT t, m FROM Team t LEFT JOIN t.members m
```

## 세타 조인  
CROSS JOIN을 말한다.  
CROSS JOIN이란 일반적으로 INNER JOIN에 ON절을 주지 않을 것을 말한다.  
그러므로 JPQL에서는 CROSS JOIN으로 외부 조인을 사용할 수 없다.  
(조인시 ON절이 자동 생성되므로)  

## ON절  
JPA 2.1부터 조인할 때 ON 절을 지원한다.  

```sql
SELECT m, t 
FROM Member m LEFT JOIN m.team t
ON t.name = 'A'
```

실행 결과는 아래와 같다.  

```sql
SELECT m.*, t.*
FROM Member m
LEFT JOIN Team t ON m.team_id = t.id AND t.name = 'A'
```

ON 절을 사용하면 조인 대상을 필터링 하고 사용할 수 있다.  

# 패치 조인  
패치조인은 SQL에 있는 개념은 아니고 JPQL에서 성능 최적화를 위해 제공하는 기능이다.  
간단하게 말해서 연관된 엔티티나 컬렉션을 한번에 같이 조회한 뒤에 대상 객체의 필드에 set 해서 내려주는 것이다.  
문법은 아래와 같다.  
> fetch join : `[ LEFT [ OUTER ] | INNER ] JOIN FETCH 조인경로`  

## 엔티티 패치 조인  
```sql
SELECT m 
FROM Member m INNER JOIN FETCH m.team
```

실행해보면 연관 엔티티의 값을 채워주기 위해 `M.*, T.*`의 형태로 연관된 팀까지 함께 조회한다.  
그리고 기존의 INNER JOIN 에서 Object[] 로 받아야 했던것 과는 달리, Member의 team 변수에 값이 다 채워진 상태로 리턴된다.  
즉, `객체 그래프를 그대로 유지하면서 받을 수 있는 방법`이다. 그러므로 성능 최적화를 위해 제공화는 기능이라고 하는 것이다.  

```java
List<Member> list = 
    em.createQuery(jpql /* 위에서 작성한 쿼리 */, Member.class).getResultList();

for(Member m : list){
    System.out.println(m.getTeam().getName()); // LAZY 로딩 발생 안함
}
```

(Member의 Team은 fetch가 LAZY라고 가정한다)  
패치 조인을 통해 이미 연관된 팀을 같이 조히했으므로 위와 같이 수행해도 LAZY 로딩이 발생하지 않는다.  

## 컬렉션 패치 조인  
일대다 관계에서도 패치 조인을 사용할 수 있다.  

```sql
SELECT t
FROM Team t INNER JOIN FETCH t.members
```

이것 또한 SELECT 절에 t 만 명시했음에도 불구하고, `T.*, M.*`의 형태로 연관된 회원까지 함께 조회된다.  
근데 여기서 주의할 점이 있는데, 쿼리의 결과가 증가해서 그런지 위 jpql의 결과를 리스트로 받아보면 Team의 개수가 Member의 개수와 동일함을 볼 수 있다.  

```java
List<Team> list = em.createQuery(jpql, Team.class).getResultList(); // 위에서 작성한 쿼리

for(Team t : list){
    System.out.println(t);

    for(Member m : t.getMembers()){
        System.out.println("-> " + m);
    }
}
```

> Team@0x100
> -> Member@0x200
> -> Member@0x300
> Team@0x100
> -> Member@0x200
> -> Member@0x300

이렇듯 일대다 조인은 결과가 증가할 수 있음에 주의해야 한다.  

## DISTINCT  
JPQL의 DISTINCT는 SQL에 DISTINCT를 추가하는 것은 물론이고, 어플리케이션에서 한번 더 중복을 제거한다.  
이 특징을 이용해서 위의 컬렉션 패치 조인에서 리스트가 중복되서 나오는 문제를 해결할 수 있다.  

```sql
SELECT DISTINCT t
FROM Team t INNER JOIN FETCH t.members
```

이렇게 작성하면 먼저 SQL에 DISTINCT가 적용된다.  
하지면 지금은 로우의 데이터가 다르므로 DISTINCT는 효과가 없다.  

다음으로 어플리케이션에서 DISTINCT 명령을 보고 중복된 데이터를 걸러낸다.  
`SELECT DISTINCT t`는 Team 엔티티의 중복을 제거하라는 의미이므로, 여기에서 중복이 제거되고, 예상했던 결과를 받아볼 수 있게된다.  

## 패치조인과 일반조인의 차이  
위에서도 언급했지만, 일반조인의 경우 결과를 반환할 때 연관관계까지 고려하지 않는다.  
단지 SELECT 절에 지정한 엔티티만을 조회하고, 연관된 엔티티에 대해서는 프록시나 컬렉션 래퍼를 반환한다.  

```java
List<Team> list = em.createQuery(jpql, Team.class).getResultList(); // 일반 조인 쿼리

for(Team t : list){
    System.out.println(t.getMembers.get(0)); // ?
}
```

그러므로 위와 같이 조회하면  
fetchType이 LAZY일 경우 `?` 부분에서 LAZY 로딩이 발생할 것이고,  
fetchType이 EAGER일 경우 회원 컬렉션을 즉시 로딩하기 위해 쿼리를 한번 더 실핼하게 된다.  

## 패치조인의 특징과 한계  
패치조인은 글로벌 전략보다 우선한다. (글로벌 전략 : 엔티티에 직접 적용하는 로딩 전략 e.g. `fetch=FetchType.LAZY`)  
그러므로 최적화를 위해 글로벌 로딩 전략을 즉시 로딩으로 설정하기 보다는,  
글로벌 전략은 지연 로딩으로 설정하고 최적화가 필요한 곳에서 패치조인을 사용하는 것이 전체적으로 봤을때 훨씬 효과적이다.  

물론 이런 좋은 패치조인에도 한계가 있다.  
- 패치조인 대상에는 별칭을 줄 수 없다  
> 별칭을 줄 수 없다는 말인 즉 SELECT, WHERE, 서브쿼리에 패치조인 대상을 사용할 수 없음을 말한다.  
> 하이버네이트를 포함한 몇몇 구현체들은 패치조인에 별칭을 지정하는 것을 허용해주나 잘못 사용하면 무결성이 깨질 수 있으므로 조심해서 사용해야 한다.  

- 둘 이상의 컬렉션을 패치할 수 없다  
> 컬렉션 * 컬렉션의 카테시안 곱이 만들어지므로 주의해야한다.  
> 하이버네이트를 사용할 경우 `MultipleBagFetchException`이 발생한다.  
> (근데 컬렉션이 Set이면 왜 될까?)  

- 컬렉션을 패치 조인하면 페이징 API를 사용할 수 없다  
> `HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!`  
> 와 같은 경고로그를 남기면서 메모리에서 페이징한다.  
> 데이터가 적으면 상관없겠지만 많으면 성능 이슈가 발생할 수 있어서 위험하다.  
>> driven 엔티티의 중복이 제거된 다음 페이징을 하려고 메모리에서 하는건데.. 왜 그런걸까?  
>> DISTINCT 없는 패치조인일 때는 그냥 다 뿌려줬으면서.. 이것도 그냥 다 뿌려진 데이터에 페이징을 넣으면 되는 것 아닌지?  

이렇듯 패치조인으로 모든것을 해결할수는 없다. 필요할 때만 사용하여 성능 최적화를 꾀하는 것이 좋다.  

# 경로 표현식  
경로 표현식이란 `.(점)`을 찍어 그래프를 탐색하는 것을 말한다.  

```sql
SELECT m.username
FROM Member m
    INNER JOIN m.team t
    INNER JOIN m.orders o
WHERE t.name = 'TeamA';
```

여기서 `m.username, m.team, m.orders, t.name` 모두 경로 표현식이다.  

아래는 경로 표현식의 종류와 특징들이다.  
- 상태필드 : 단순히 값을 저장하기 위한 필드. 일반적인 자바 기본 타입의 컬럼들을 말한다.  
    - `m.username`, `t.name`이 해당한다.  
    - 더는 탐색할 수 없다
- 연관필드 : 연관관계를 위한 필드, 임베디드 타입
    - 단일 값 연관 필드 : 대상이 엔티티인것을 말한다. (`@ManyToOne`, `@OneToOne`)  
        - `m.team`이 해당된다.  
        - 묵시적으로 내부 조인이 일어난다.  
        - 계속 탐색할 수 있다  
        - 임베디드 타입도 단일 값 연관 필드이지만 연관관계가 없으므로 조인이 일어나지 않는다.  
    - 컬렉션 값 연관 필드 : 대상이 컬렉션것을 말한다. (`@OneTomany`, `@ManyToMany`)  
        - `m.order`가 해당된다.  
        - 묵시적으로 내부 조인이 일어난다.  
        - 기본적으로 더는 탐색할 수 없으나, FROM 절에서 별칭을 얻으면 별칭으로 탐색할 수 있다.  

## 단일 값 연관 경로 탐색 예제  
```sql
SELECT o.member from Order o
```

위 JPQL은 아래와 같이 변환된다.  

```sql
SELECT m.*
FROM Order_ o
    INNER JOIN Member m ON o.member_id = m.id
```

위처럼 JPQL에 JOIN을 적어주지 않았는데 JOIN이 발생하는 것을 `묵시적 조인`이라고 하고, JOIN을 직접 적어주는 것을 `명시적 조인`이라고 한다.  
묵시적 조인은 `내부 조인만 가능` 하다. 외부 조인을 하고 싶으면 `명시적 조인`을 사용해야 한다.  

## 컬렉션 값 연관 경로 탐색  
컬렉션 값에서는 경로 탐색이 불가능하다(가장 많이 하는 실수)  

```sql
SELECT t.members.username FROM Team t // 실패
```

만약 경로 탐색을 하고 싶으면 명시적 조인을 사용해서 외부 별칭을 획득해야 한다.  

```sql
SELECT m.username
FROM Team t  
INNER JOIN t.members m
```

참고로 컬렉션을 컬렉션의 크기를 구할 수 있는 `size`라는 특별한 기능을 제공한다.  

```sql
SELECT t.member.size FROM Team t
```

는 COUNT 함수를 사용하는 함수로 적절히 변환된다.  

> 기본적으로 쿼리에서 조인이 성능상 차지하는 부분은 아주 크다.  
> 단순하면 별로 문제될 것 없으나, 복잡하고 성능이 중요하면 분석이 용이하도록 명시적 조인을 사용하는 것이 좋다.  

# 서브쿼리
JPQL에서는 서브쿼리를 WHERE, HAVING 절에서만 사용할 수 있다.  
SELECT, FROM 절에서는 사용할 수 없다.  
아래는 간단한 서브쿼리 예시이다.  

```sql
// 회원들의 평균 나이를 넘는 회원 조회
SELECT m
FROM Member m
WHERE m.age > (SELECT AVG(m2.age) FROM Member m2)
```

## 서브쿼리 함수  
- **EXISTS**  
    - 문법 : [NOT] EXISTS {subquery}
    - 설명 : 서브쿼리가 결과에 존재하면 참이다(NOT은 반대)  

    ```sql
    // teamA에 소속인 회원
    SELECT m 
    FROM Memner m 
    WHERE EXISTS (
        SELECT t
        FROM m.team t
        WHERE t.name = 'teamA'
    )
    ```

- **ALL | ANY | SOME**  
    - 문법 : { ALL | ANY | SOME } {subquery}
    - 설명 : 비교 연산자와 같이 사용한다  
        - ALL : 조건을 모두 만족하면 참  
        - ANY, SOME : 둘은 같은 의미임. 조건을 하나라도 만족하면 참  

    ```sql
    // 전체 상품 각각의 재고보다 주문량이 많은 주문들  
    SELECT o
    FROM Order o
    WHERE o.orderAmoun > ALL(
        SELECT p.stockAmoun from Product p // o.p가 아니고?
    )

    // 어떤 팀이든 팀에 소속된 회원  
    SELECT m
    FROM Member m
    WHERE m.team = ANY(
        SELECT t
        FROM Team t // 이것도 좀 이상한데...
    )
    ```

- **IN**  
    - 문법 : [NOT] IN {subquery}  
    - 설명 : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참이다. IN은 서브쿼리가 아닌 곳에서도 사용할 수 있다.  

# 조건식  
## 타입 표현  
종류 | 설명 | 예제 | 
---|----|----|
문자 | 작은 따옴표 사이에 표현.<br/>작음 따옴표를 표현하고 싶으면 작은 따옴표 2개('') 사용 | 'HELLO'<br/>'She''s ' | 
숫자 | L(Long 타입 지정)<br/>D(Double 타입 지정)<br/>F(Float 타입 지정) | 10L<br/>10D<br/>10F<br/> | 
날짜 | DATE {d 'yyyy-mm-dd'}<br/>TIME {t 'hh:mm:ss'}<br/>TIMESTAMP {ts 'yyyy-mm-dd hh:mm:ss.f} | m.createDate = {d '2012-03-24'} | 
Boolean | TRUE, FALSE | 
Enum | 패키지명을 포함한 전체 이름 | com.joont.MemberType.Admin | 
엔티티 타입 | 엔티티의 타입을 표현함. 주로 상속과 관련해 사용. | TYPE(m) = Member

## 연산자 우선 순위  
1. 경로 탐색 연산 : `.`  
2. 수학 연산 : `+(단항 연산), -(단항 연산), *, /, +, -`  
3. 비교 연산 : `=, >, >=, <, <=, <>,`  
`[NOT] BETWEEN, [NOT] LIKE, [NOT] IN,`  
`IS [NOT] NULL, IS [NOT] EMPTY, [NOT] MEMBER [OF], [NOT] EXISTS`  
4. 논리연산 : NOT, AND, OR  

`IS [NOT] EMPTY`, `[NOT] MEMBER [OF]` 만 뺴면 사용법은 일반적인 SQL과 동일하다.  
이 두개는 `컬렉션 식`으로써 JPA에서 제공하는, 컬렉션에만 사용가능환 특별 기능이다.  

## 컬렉션 식  
컬렉션에만 사용될 수 있음에 주의해야 한다. 컬렉션이 아닌 곳에 사용하면 오류가 발생한다.  
- **빈 컬렉션 비교 식**  
    - 문법 : {컬렉션 값 연관 경로} IS [NOT] EMPTY  
    - 설명 : 컬렉션에 값이 비었으면 참  

    ```sql
    SELECT m
    FROM Member m
    WHERE m.orders IS EMPTY
    ```

    는 아래와 같이 실행된다.  

    ```sql
    SELECT m.*
    FROM Member m
    WHERE EXISTS (
        SELECT o.id
        FROM Order_ o
        WHERE o.member_id = m.id
    )
    ```

- **컬렉션 멤버 식**  
    - 문법 : {엔티티나 값} [NOT] MEMBER [OF] {컬렉션 값 연관경로}  
    - 설명 : 엔티티나 값이 컬렉션에 포함되어 있으면 참  

    ```sql
    // 전달된 멤버가 포함되어 있는 팀 조회  
    SELECT t
    FROM Team t
    WHERE :memberParam MEMBER OF t.members
    ```

## 스칼라 식  
숫자, 문자, 날짜, case, 엔티티 타입 같은 가장 기본적인 타입들을 스칼라 타입이라고 한다.  

- **문자함수**   
    함수 | 설명 | 예제 | 
    ---|----|----|-
    CONCAT(문자1, 문자2) | 문자를 합한다 | CONCAT('A', 'B') = AB | 
    SUBSTRING(문자, 위치[, 길이]) | 위치부터 시작해 길이만큼 문자를 구한다. 길이 값이 없으면 나머지 전체 길이를 뜻한다 | SUBSTRING('ABCDEF', 2, 3) = BCD | 
    TRIM([[LEADING | TRAILING | BOTH] [트림 문자] FROM] 문자) | `LEADING : 왼쪽만`, `TRAILING : 오른쪽만`, `BOTH : 양쪽` 다 트림 문자를 제거한다. 기본값은 `BOTH`이고, 트림 문자의 기본값은 `공백(space)`이다. | TRIM(' ABC ') = ABC<br/>TRIM(LEADING 'A' FROM 'ABC') = BC | 
    LOWER(문자) | 소문자로 변경 | LOWER('ABC') = abc | 
    UPPER(문자) | 대문자로 변경 | UPPER('abc') = ABC | 
    LENGTH(문자) | 문자 길이 | LENGTH('ABC') = 3 | 
    LOCATE(찾을 문자, 원본 문자[, 검색 시작 위치]) | 검색위치부터 문자를 검색한다. 1부터 시작하고 못찾으면 0을 반환한다. | LOCATE('DE', 'ABCDEFG') = 4 | 

- **수학함수**  
    함수 | 설명 | 예제 | 
    ---|----|----|-
    ABS(식수학식) | 절대값을 구한다 | ABS(-10) = 10 | 
    SQRT(수학식) | 제곱근을 구한다 | SQRT(4) = 2.0 | 
    MOD(수학식, 나눌 수) | 나머지를 구한다 | MOD(4, 3) = 1 | 
    SIZE(컬렉션 값 연관 경로식) | 컬렉션의 크기를 구한다 | SIZE(t.members) | 
    INDEX(별칭) | LIST 타입 컬렉션의 위치값을 구함. 단 컬렉션이 @OrderColumn을 사용하는 LIST 타입일 때만 사용할 수 있다 | t.members m where INDEX(m) > 3 | 

- **날짜함수**  
    함수 | 설명 | 
    ---|----|-
    CURRENT_DATE | 현재 날짜 | 
    CURRENT_TIME | 현재 시간 | 
    CURRENT_TIMESTAMP | 현재 날짜 + 시간 | 

    하이버네이트는 날짜 타입에서 년,월,일,시,분,초 값을 구하는 기능을 지원한다 
    (YEAR,MONTH,DAY,HOUR,MINUTE,SECOND)  
    ```sql
    SELECT YEAR(m.createdDate), MONTH(m.createdDate), DAY(m.createdDate) FROM Member;
    ```

## CASE 식  
- **기본 CASE**  
    - 문법 :  
        ```
        CASE  
            {WHEN <조건식> THEN <스칼라식>}+  
            ELSE <스칼라식>  
        END
        ```

- **심플 CASE**  
    - 문법 :  
        ```
        CASE <조건대상>  
            {WHEN <스칼라식1> THEN <스칼라식2>}+
            ELSE <스칼라식>
        END
        ```

- **COALESCE**  
    - 문법 : `COALESCE(<스칼라식>, {,<스칼라식>}+)`  
    - 설명 : 스칼라식을 차례대로 조회해서 null이 아니면 반환한다. IFNULL과 약간 비슷하다.  

    ```sql
    SELECT COALESCE(m.usernae, 'nobody') 
    FROM Member m
    ```

- **NULLIF**  
    - 문법 : `NULLIF(<스칼라식>, <스칼라식>)`  
    - 설명 : 두 값이 같으면 null 반환, 다르면 첫번째 값을 반환한다.  

# 다형성 쿼리  
상속관계(`@Inheritance`)로 구성된 엔티티를 JPA에서 조회하면 그 자식 엔티티도 같이 조회한다.  
이건 기존과 동일하다.  

## TYPE  
상속 구조에서 조회 대상을 특정 타입으로 한정할 때 사용한다.  

```sql
SELECT i
FROM Item i
WHERE TYPE(i) IN(Book, Movie)
```

는 아래와 같이 실행된다  

```sql
SELECT i.*
FROM Item i
WHERE i.DTYPE IN('B', 'M')
```

## TREAT  
상속 구조에서 부모 타입을 특정 타입으로 다룰 때 사용한다.(자바의 타입 캐스팅과 비슷하다)  
JPA 표준은 FROM, WHERE절에서만 사용 가능하고, 하이버네이트의 경우 SELECT에서도 가능하다.  

```sql
SELECT i
FROM Item i
WHERE TREAT(i as Book).author = 'kim'
```

Item을 자식 타입인 Book으로 다뤘다. 그래서 Book의 필드인 author에 접근할 수 있다.  

# 사용자 정의 함수 호출(since JPA2.1)
JPA 2.1부터 사용자 정의 함수를 지원한다.  
> 문법 : `FUNCTION(function_name {, function_arg}*)` 

```sql
SELECT FUNCTION('group_concat', i.name)
FROM Item i
```

하이버네이트를 사용할 경우 아래와 같이 방언 클래스를 상속해서 사용할 데이터베이스 함수를 미리 등록해야 한다.  

```java
public class MyH2Dialect extends H2Dialect{
    public MyH2Dialect(){
        registerFunction(
            "group_concat", 
            new StandardFunction("group_concat", StandardBasicTypes.STRING)
        );
    }
}
```

`registerFunction`의 두번째 인자로는 하이버네이트의 `SQLFunction` 구현체를 주면 된다.  
지금은 기본 함수를 사용하겠다는 의미로 `StandardFunction`을 사용하였고,  
첫번째 인자로 `함수 이름`, 두번째 인자로 `리턴 타입`을 주고 있는 모습이다.  

상속한 Dialect는 아래와 같이 등록하면 되고,  

```xml
<property name="hibernate.dialect" value="com.joont.dialect.MyH2Dialect" />
```

하이버네이트를 사용하면 기본 문법보다 축약해서 사용할 수 있다.  

```sql
SELECT group_concat(i.name)
FROM Item i
```

# 엔티티 직접 사용  
객체 인스턴스는 참조 값으로 식별하고 테이블 로우는 기본 키 값으로 식별하기 때문에  
JPQL에서 엔티티 객체를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.  
몇 가지 예시를 보자.  

```sql
SELECT COUNT(m)
FROM Member m
```

은 아래와 같이 변환된다.  

```sql
SELECT COUNT(m.id)
FROM Member m
```

- 기본키 비교에 엔티티 사용  
    ```java
    List<Member> result = 
        em.createQuery("SELECT m FROM Member m WHERE m = :member")
        .setParameter("member", member) // 엔티티 객체 직접 사용
        .getResultList();
    ```

    member가 영속성 컨텍스트에 있을 필요는 없다. 그냥 식별자만 가지고 있으면 된다.  
    실행되는 sql은 아래와 같다.  

    ```sql
    SELECT m.*
    FROM Member m 
    WHERE m.id = ? -- member 파라미터 id 값
    ```

- 외래키 비교도 마찬가지다  
    ```java
    List<Member> result = 
        em.createQuery("SELECT m FROM Member m WHERE m.team = :team")
        .setParameter("team", team) // 엔티티 객체 직접 사용
        .getResultList();
    ```

    실행되는 sql은 아래와 같다.  

    ```sql
    SELECT m.*
    FROM Member m 
    WHERE m.team_id = ? -- team 파라미터 id 값
    ```

    MEMBER 테이블은 이미 TEAM의 식별자 값을 가지고 있기 때문에 묵시적 조인은 일어나지 않는다.  

# Named 쿼리(정적 쿼리)  
`em.createQuery("select ... ")` 처럼 JPQL을 직접 문자로 넘기는 것을 동적 쿼리라고 하고,  
미리 정의한 쿼리에 이름을 부여해서 해당 이름으로 사용하는 것을 `Named 쿼리(정적 쿼리)`라고 한다.  

Named 쿼리는 어플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱해두므로 오류를 빨리 확인할 수 있고, 사용하는 시점에는 파싱된 결과를 재사용하므로 성능상 이점도 있다.  

Named 쿼리는 `@NamedQuery` 어노테이션을 사용해서 자바 코드에 작성하거나 XML 문서에 작성할 수 있다.  

## 어노테이션에 정의  
```java
@Entity
@NamedQueries({
    @NamedQuery(
        name = "Member.findByUsername",
        query = "SELECT m FROM Member WHERE m.username = :username"
    ),
    @NamedQuery(
        name = "Member.count",
        query = "SELECT COUNT(m) FROM Member m"
    )
})
class Member{
    // ...
}
```

위처럼 엔티티에 `@NamedQuery`, `@NamedQueries` 어노테이션을 사용해서 직접 정의해주면 된다.  
(Named 쿼리의 이름에 있는 `Member`가 뭔가 기능적으로 하는게 있는 것은 아니다. 그냥 관리의 편의성을 위함이다.  
그리고 Named 쿼리는 영속성 유닛 단위로 관리되므로 충돌을 방지하기 위해 이름으로 구분한 것이기도 하다)  
그리고 아래와 같아 사용해주면 된다.  

```java
List<Member> result = 
    em.createNamedQuery("Member.findByName", Member.class)
    .setParameter("username", "joont")
    .getResultList();
```

## XML에 정의  
사실상 자바로 멀티라인 문자를 다루는 것은 상당히 귀찮은 일이므로, Named 쿼리를 작성할 때는 XML을 사용하는 것이 더 편리하다.  

```xml
<!--xml version="1.0" encoding="UTF-8"?-->
<entity-mappings xmlns="http://java.sun.com/xml/ns/persistence/orm" version="2.0">

    <named-query name="Member.findByUserName">
        <query>
            select m 
            from Member m 
            where m.username = :username
        </query>
    </named-query>

    <named-query name="Member.findByAgeOver">
        <query><![CDATA[
            select m 
            from Member m 
            where m.age > :age
        ]]></query>
    </named-query>

    <named-native-query name="Inter.findByAlal" result-class="sample.jpa.Inter">
        <query>select a.inter_seq, a.inter_name_ko, a.inter_name_en from tb_inter a where a.inter_name_ko = ?</query>
    </named-native-query>

</entity-mappings>
```

XML에서 `&, <, >,`는 예약문자어 이므로 `&amp;, &lt;, &gt;`를 사용해야 한다.  
`<![CDATA[ ]]>`를 사용하면 그 사이에 있는 문자를 그대로 출력하므로 예약 문자도 사용할 수 있다.  

# 기타  
- `Enum은 = 비교연산만 지원한다` 는 JPQL 명세이고, 하이버네이트에서는 아래가 가능하다  
    ```java
    Delivery delivery = 
        em.createQuery("select d from Delivery d where d.deliveryStatus like '%CO%'", Delivery.class)
        .getSingleResult();
    ```
- `임베디드 타입은 비교를 지원하지 않는다` 또한 JPQL 명세이고, 하이버네이트에서는 아래가 가능하다  
    ```java
    Delivery foundDelivery = 
        em.createQuery("select d from Delivery d where d.address = :address", Delivery.class)
        .setParameter("address", new Address("seoul", "새마을로", "1111-1111"))
        .getSingleResult();
    ```
> 하이버네이트에서만 지원하는건가?  

- JPA는 `''`를 길이 0인 Empty String으로 정했지만 데이터베이스에 따라 `''`를 `null`로 사용하는 곳이 있으니 확인하고 사용해야 한다.  
- **NULL 정의**  
    - 조건을 만족하는 데이터가 하나도 없으면 `NULL` 이다  
    - NULL은 알수 없는 값이다. NULL과의 모든 수학적 연산은 NULL이다.  
    - JPA 표준명세에서 정하는 NULL과의 논리연산은  
    NULL과 False를 AND 연산하면 False.  
    NULL과 True를 OR 연산하면 True이다.  


# 네이티브 SQL
JPA는 표준 SQL이 지원하는 대부분의 SQL 문법과 함수들을 지원하지만,  
특정 데이터베이스만 지원하는 함수나 문법, SQL 쿼리 힌트 같은 것들은 지원하지 않는다.  
이런 기능을 사용하기 위해선 `네이티브 SQL`을 사용해야 한다.  
네이티브 SQL이란 JPA에서 일반 SQL을 직접 사용하는 것을 말한다.  

실제 데이터베이스 쿼리를 사용한다는 점 외에는 JPQL을 사용할때와 거의 비슷하다.  
(원래는 위치기반 파라미터만 지원하지만 하이버네이트는 이름기반 파라미터까지 지원한다)  

## 엔티티 조회
`Query createNativeQuery(String sqlString, Class resultClass)` 를 말한다.  
> 반환타입을 줘도 TypedQuery가 아닌 Query를 반환하는 이유는,  
> JPA 1.0에서 규약이 그렇게 정의되어 버렸기 때문에 그렇다고하니 신경쓰지 않아도 된다.  

- 이 메서드로 조회해온 엔티티는 영속성 컨텍스트에서 관리된다.  
- 그러므로 모든 필드를 다 조회하는 SQL을 실행해야 한다  
- 특정 필드만 조회해오면 오류가 발생한다.  

```java
String sql = "SELECT * FROM Member WHERE id = 1";

Member memberFromNative = (Member)em.createNativeQuery(sql, Member.class).getSingleResult();
Member memberFromJPQL = em.find(member.class, 1);

assertSame(memberFromnNative, memberFromJPQL); // success
```

## 값 조회  
`Query createNativeQuery(String sqlString)` 를 말한다.  

```java
String sql = "SELECT id, name, age FROM Member";

List<Object[]> list = em.createNativeQuery(sql).getResultList();

for(Object[] row : list){
    Integer id = row[0];
    String name = row[1];
    Integer age = row[2];
}
```

## 결과 매핑 사용  
네이티브 쿼리에서 여러값들이 나올 때 결과를 여러 엔티티나 엔티티+스칼라 형태로 적절히 합치는건데.. 굳이 이것까지..  

## Named 네이티브 쿼리  
어노테이션의 경우 `@NamedNativeQuery` 사용하면 되고,  
XML의 경우 `<named-native-query>` 사용하면 된다.  

> 될수 있으면 JPQL을 사용하고, 기능이 부족하면 HQL 등을 사용해보고, 그래도 안되면 네이티브 SQL을 사용하자  

# 스토어드 프로시저  
JPQL에서 사용 가능하지만(Named 스토어드 프로시저도 가능) MySQL에서 성능 이점이 그리 많지 않아 잘 사용되지 않으니 패스  

# 벌크 연산(UPDATE, DELETE)  
JPQL로 여러 건을 한번에 수정하거나 삭제할 떄 사용한다.  
아래는 UPDATE 벌크 연산이다.  

```java
String sql = "UPDATE Product p " +
    "SET p.prce = p.price * 1.1 " +
    "WHERE p.stockAmount < :stockAmount";

int resultCount = em.createQuery(sql)
        .setParameter("stockAmount", 10)
        .executeUpdate();
```

`executeUpdate` 메서드를 사용한다. 벌크 연산으로 영향을 받은 엔티티 건수를 반환한다.  
아래는 DELETE 벌크 연산이다.  

```java
String sql = "DELETE FROM Product p " +
    "WHERE p.price < :price";

int resultCount = em.createQuery(sql)
        .setParameter("price", 100)
        .executeUpdate();
```

## 벌크 연산시 주의사항  
벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다는 특징이 있으므로 주의해야 한다.  
아래는 발생가능한 문제 상황이다.  

```java
Product product = em.find(Product.class, 1);
assertThat(product.getPrice(), is(1000));

String sql = "UPDATE Product p " +
    "SET p.prce = p.price * 1.1";
em.createQuery(sql).executeUpdate();

assertThat(product.getPrice(), is(1100)); // FAIL!!
```

벌크 연산은 영속성 컨텍스트와 2차 캐시를 무시하고 데이터베이스에 직접 쿼리한다.  
따라서 위와 같이 영속성 컨텍스트와 데이터베이스 간에 데이터 차이가 발생할 수 있는 것이다.  
이를 해결하기 위한 방법은 아래와 같다.  

- `em.refrest(entity)` 사용  
    > 벌크 연산 직후에 `em.refresh(entity)`를 사용하여 데이터베이스에서 다시 상품을 조회하면 된다.  
- 벌크 연산 먼저 실행  
    > 벌크 연산을 가장 먼저 실행하면 이미 변경된 내용을 데이터베이스에서 가져온다.  
    > 가장 실용적인 해결책이다.  
- 벌크 연산 수행 후 영속성 컨텍스트 초기화  
    > 영속성 컨텍스트가 초기화되면 데이터베이스에서 다시 조회해오기 때문에 이것도 방법이다.  

# 영속성 컨텍스트와 JPQL  
영속성 컨텍스트에 이미 있는 엔티티를 JPQL로 다시 조회해올 경우 어떻게 처리될까?  

```java
Member member1 = em.find(Member.class, 1);

List<Member> list = 
    em.createQuery("SELECT m FROM Member m", Member.class)
    .getResultLst();
```

이미 영속성 컨텍스트에 들어있는 1번 member가 JPQL에 의해 다시 한번 조회되는 상황이다.  
결과부터 말하자면 JPQL 쿼리는 쿼리대로 다 날라가고, 조회한 엔티티를 영속성 컨텍스트에 다 저장한다.  
여기서 중요한 점은 1번 member의 경우 영속성 컨텍스트에 이미 들어있으므로, JPQL로 조회해온 1번 member는 그냥 버려진다는 것이다.  

![JPQL 조회시 영속성 컨텍스트](/temp/JPQL-조회시-영속성-컨텍스트.jpeg)  

보다시피 조회해온 member들 중 1번 member는 영속성 컨텍스트에 이미 있으므로 그 결과가 버려진다.  
영속성 컨텍스트에 없는 2번 member의 경우 영속성 컨텍스트에 저장된다.  

새로 조회해온 결과를 기존 영속성 컨텍스트에 덮어쓰지 않는 이유는 `영속 상태인 엔티티의 동일성을 보장해야하기 때문`이다.  
> find로 들고오든, JPQL로 들고오든 동일한 엔티티를 반환해야 한다.  

```java
Member member1 = em.find(Member.class, 1);
Member member2 = 
    em.createQuery("SELECT m FROM Member WHERE m.id = :id", Member.class)
    .setParameter("id", 1)
    .getSingleResult();

assertSame(member1, member2); // SUCCESS
```

![JPQL 실행시 플로우](/temp/JPQL-실행시-플로우.jpeg)  

보다시피 영속성 컨텍스트에 1번 member 엔티티가 있더라도 무조건 SQL을 실행해서 조회해온다.  
(JPQL을 분석해서 영속성 컨텍스트를 조회하는 것은 너무 힘들기 때문이다.)  
그리고 조회해온 엔티티를 영속성 컨텍스트에 넣을 때, 이미 있는 엔티티일 경우 결과를 버린다.  

## JPQL과 플러시 모드  
플러시 모드는 `FlushMode.AUTO(Default)`, `FlushMode.COMMIT`이 있다.  
이때까지 `FlushMode.AUTO` 는 트랜잭션이 끝날때나 커밋될 때만 플러시를 호출하는 것으로 알고 있었으나, 사실은 시점이 하나 더 있다. JPQL 쿼리를 실행하기 직전이다.  

```java
Member member1 = em.find(Member.class, 1);
member1.setName("modified name");

Member member2 = 
    em.createQuery("SELECT m FROM Member WHERE m.id = :id", Member.class)
    .setParameter("id", 1)
    .getSingleResult();

assertThat(member1.getName(), member2.getName());
```

변경감지는 플러시 될때 발생하므로, JPQL에서 아직 변경되지 않은 name 값을 가진 1번 member를 가져올 것이라고 생각할 수 있지만,  
`FlushMode.AUTO`는 영속 상태인 엔티티의 동일성을 보장하기 위해 `JPQL 실행 전에 플러시를 수행한다`.  
그러므로 위의 테스트는 성공한다.  
> 어떻게 동작하는지 정확히는 모르겠으나, 영속성 컨텍스트에 있는 엔티티에 대해 JPQL을 실행할 떄만 플러시를 수행한다.  
> 즉, 위의 상황에서 JPQL로 Team을 조회해올 경우 플러시가 발생하지 않는다.  

여기서 플러시 모드를 `FlushMode.COMMIT`으로 설정하면 쿼리전에 플러시를 수행하지 않으므로 위의 테스트가 실패하게 된다.  
이때는 직접 `em.flush`를 호출해주거나, Query 객체에 직접 플러시 모드를 설정해주면 된다.  

```java
em.setFlushMode(FlushMode.COMMIT); // 커밋시에만 플러시

Member member1 = em.find(Member.class, 1);
member1.setName("modified name");

em.flush(); // 1. em.flush 직접 호출

Member member2 = 
    em.createQuery("SELECT m FROM Member WHERE m.id = :id", Member.class)
    .setParameter("id", 1)
    .setFlushMode(FlushMode.AUTO) // 2. setFlushMode 설정
    .getSingleResult();

assertThat(member1.getName(), member2.getName());
```

FlushMode.COMMIT은 너무 잦은 플러시가 일어나는 경우, 플러시 횟수를 줄여서 성능을 최적화하고자 할 때 사용할 수 있다.  
> 한 트랜잭션 안에서 특정 엔티티에 대한 insert, update, delete를 수행하고  
> 그 뒤에서 JPQL로 전혀 다른 엔티티의 값을 읽어올 경우 불필요한 flush가 날라가게 된다.  

<!-- more -->