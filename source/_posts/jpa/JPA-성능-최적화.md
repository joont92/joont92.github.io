---
title: JPA 성능 최적화
date: 2019-02-15 23:58:36
tags:
    - 자바 ORM 표준 JPA 프로그래밍
    - N+1
    - fetch join
    - '@BatchSize'
---

# N+1 문제  
성능상 가장 주의해야 하는것이 이 N+1 문제이다.  
아래와 같은 엔티티가 있다고 가정한다.  

```java
@Entity
class Member{
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
    private List<Order> orders = new ArrayList<>();
}

@Entity
class Order{
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Member member;
}
```

(참고로 Order에서 Member를 EAGER로 설정해도 동일하게 N+1은 발생한다)  

## 즉시 로딩
```java
List<Member> members = 
    em.createQuery("SELECT m FROM Member", Member.class)
    .getResultList();
```

예전에도 언급했지만, JPA는 fetchType을 전혀 신경쓰지 않고 충실하게 JPQL에 맞춰 SQL을 생성한다.  

따라서 Member를 전체 조회하는 쿼리가 먼저 실행되고,  
Member의 개수만큼 Order를 조회하게 될 것이다(...)  

```sql
SELECT * FROM Member; -- if result is 5
SELECT * FROM Order_ WHERE MEMBER_ID = 1;
SELECT * FROM Order_ WHERE MEMBER_ID = 2;
SELECT * FROM Order_ WHERE MEMBER_ID = 3;
SELECT * FROM Order_ WHERE MEMBER_ID = 4;
SELECT * FROM Order_ WHERE MEMBER_ID = 5;
```

이처럼 `처음 실행한 SQL의 결과 수만큼 추가로 SQL을 실행하는 것`을 N+1 문제라고 한다.  

## 지연 로딩  
즉시로딩을 지연로딩으로 바꿔도 N+1에서 자유로울수는 없다.  
즉시로딩이 아니라서 Member 조회와 동시에 Member 건수만큼 Order를 조회해오진 않겠지만,  
Member에서 Order를 사용하는 시점에는 똑같이 불러오게 된다.  

```java
for(Member member : membres){
    System.out.println(member.getOrders().size());
}
```

members 개수만큼 order 조회 SQL이 실행될 것이다.  
즉, 이것도 결국 N+1 문제다.  

## 해결법  
### 페치 조인  
가장 일반적인 방법이다. SQL 조인을 이용해서 연관된 엔티티를 함께 조회하므로 N+1 문제가 발생하지 않는다.  
JPQL은 아래와 같다.  

```sql
SELECT m FROM Member m JOIN FETCH m.orders
```

JOIN으로 같이 조회해서 Member 엔티티의 orders 속성에 초기화 하였기 때문에 더이상 N+1이 발생하지 않는다.  
참고로 위 예제는 일대다 조인이므로 결과가 늘어날 수 있다. `DISTINCT`를 써줘야한다.  

### 하이버네이트 @BatchSize
하이버네이트가 제공하는 `org.hibernate.annotations.BatchSize` 어노테이션을 이용하면 연관된 엔티티를 조회할 때 지정된 size 만큼 SQL의 IN절을 사용해서 조회한다.  

```java
@Entity
class Member{
    @Id @GeneratedValue
    private Long id;

    @org.hibernate.annotations.BatchSize(size = 5)
    @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
    private List<Order> orders = new ArrayList<>();
}
```

즉시로딩이므로 Member를 조회하는 시점에 Order를 같이 조회한다.  
`@BatchSize`가 있으므로 Member의 건수만큼 추가 SQL을 날리지 않고, 조회한 Member 의 id들을 모아서 SQL IN 절을 날린다.  

```sql
SELECT * FROM
ORDER_ WHERE MEMBER_ID IN(
    ?, ?, ?, ?, ?
)
```

`size`는 IN절에 올수있는 최대 인자 개수를 말한다. 만약 Member의 개수가 10개라면 위의 IN절이 2번 실행될것이다.  

그리고 만약 지연로딩이라면 지연로딩된 엔티티 최초 사용시점에 5건을 미리 로딩해두고, 6번째 엔티티 사용 시점에 다음 SQL을 추가로 실행한다.  
> `hibernate.default_batch_fetch_size` 속성을 사용하면 애플리케이션 전체에 기본으로 `@BatchSize`를 적용할 수 있다.  
> ```xml
> <property name="hibernate.default_batch_fetch_size" value="5" />
> ```

### 하이버네이트 @Fetch(FetchMode.SUBSELECT)
연관된 데이터를 조회할 때 서브쿼리를 사용해서 N+1 문제를 해결한다  

```java
@Entity
class Member{
    @Id @GeneratedValue
    private Long id;

    @org.hibernate.annotations.Fetch(FetchMode.SUBSELECT)
    @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
    private List<Order> orders = new ArrayList<>();
}
```

아래와 같이 실행된다.  

```sql
SELECT * FROM Member;
SELECT * FROM Order_
    WHERE MEMBER_ID IN(
        SELECT ID
        FROM Member
    )
```

즉시로딩으로 설정하면 조회시점에, 지연로딩으로 설정하면 지연로딩된 엔티티를 사용하는 시점에 위의 쿼리가 실행된다.  

> 모두 지연로딩으로 설정하고 성능 최적화가 필요한 곳에는 JPQL 페치 조인을 사용하는 것이 추천되는 전략이다.  

# 읽기 전용 쿼리의 성능 최적화