---
title: [jpa] JPA 예외
date: 2019-02-15 22:10:36
tags:
    - 자바 ORM 표준 JPA 프로그래밍
    - jpa exception
    - PersistenceException
---

JPA의 표준 예외들은 `javax.persistence.PersistenceException`의 자식 클래스이다.  
그리고 이 클래스는 `RuntimeException`의 자식이다. 즉, JPA의 예외는 모두 언체크 예외이다.  

# JPA 표준 예외
JPA 표준 예외는 크게 아래의 2가지로 나뉜다.  
난 실행단계에서 아래 2개를 어떻게 구분해야할지 잘 모르겠다  

## 트랜잭션 롤백을 표시하는 예외  
심각한 예외이므로 복구해선 안되는 예외들이다.  
이 예외가 발생하면 트랜잭션을 강제로 커밋해도 커밋되지 안혹 `javax.persistence.RollbackException`이 발생한다.  

예외 | 설명 | 
------- | ------- | 
javax.persistence.EntityExistsException | `EntityManager.persist(..)` 호출 시 같은 엔티티가 있으면 발생 | 
javax.persistence.EntityNotFoundException | `EntityManager.getReference(..)` 호출하고 실제 사용 시 엔티티가 존재하지 않으면 발생. `refresh(..)`, `lock(..)` 에서도 발생 | 
javax.persistence.OptimisticLockException | 낙관적 락 충돌시 | 
javax.persistence.PessimisticLockException | 비관적 락 충돌시 | 
javax.persistence.RollbackException | `EntityTransaction.commit()` 실패 시 발생. 롤백이 표시되어 있는 트랜잭션 커밋시에도 발생 | 
javax.persistence.TransactionRequiredException | 트랜잭션이 필요할 떄 트랜잭션이 없으면 발생. 트랜잭션 없이 엔티티를 변경할 떄 주로 발생 | 

## 트랜잭션 롤백을 표시하지 않는 예외  
심각한 예외가 아니다.  
개발자가 트랜잭션을 커밋할지 롤백할지 판단하면 된다.  

예외 | 설명 | 
---|----|
javax.persistence.NoResultException | `Query.getSingleResult()` 호출 시 결과가 하나도 없을 때 발생 | 
javax.persistence.NonUniqueResultException | `Query.getSingleResult()` 호출시 결과가 둘 이상일 떄 발생 | 
javax.persistence.LockTimeoutException | 비관적 락에서 시간 초과시 발생한다 | 
javax.persistence.QueryTimeException | 쿼리 실행 시간 초과시 발생  

# 스프링 프레임워크의 JPA 예외 변환  
서비스 계층에서 데이터 접근 기술에 직접 의존하는 것은 좋은 설계가 아니다.  
이것은 예외도 마찬가지다. 서비스 계층에서 위 JPA 예외에 직접 의존하면 결국 JPA에 의존하게 되는것이다.  
스프링 프레임워크는 이런 문제를 해결하기 위해 JPA 예외를 추상화해서 제공한다.  

blah blah..  

## 스프링 프레임워크에 JPA 예외 변환기 적용  
위처럼 JPA예외를 스프링 예외로 변경해서 받으려면 `PersistenceExceptionTranslationPostProcessor`를 스프링 빈으로 등록하면 된다.  
이것은 `@Repository` 어노테이션을 사용한곳에 예외 변환 AOP를 적용해서 JPA 예외를 스프링 추상화 예외로 변환해준다.  

```java
@Bean
public PersistenceExceptionTranslationPostProcessor exceptionTranslation(){
    return new PersistenceExceptionTranslationPostProcessor();
}
```

아래 예외는 변환되어서 던져질 것이다.  

```java
@Repository
public class NoResultExceptionTestRepository{
    @PersistenceContext
    private EntityManager em;

    public Member findMember(){
        // 조회결과 없음
        return em.createQuery("SELECT m FROM Member WHERE m.id = :id", Member.class)
            .setParameter("id", 999)
            .getSingleResult();
    }
}
```

원래라면 `NoResultException`이 발생해야 하지만, 등록한 AOP 인터셉터가 동작해서 `EmptyResultDataAccessException`으로 변환해서 반환한다.  

만약 예외를 변환하지 않고 그대로 반환하고 싶다면 반환할 JPA 예외나 JPA 예외의 부모 클래스를 직접 명시하면 된다.  

```java
@Repository
public class NoResultExceptionTestRepository{
    public Member findMember() throws javax.persistence.NoResultException{
        return em.createQuery("SELECT m FROM Member WHERE m.id = :id", Member.class)
            .setParameter("id", 999)
            .getSingleResult();
    }
}
```

예외의 최상위 클래스인 Exception을 throws 하면 예외를 아예 변환하지 않을 것이다.  

# 트랜잭션 롤백 시 주의사항  
트랜잭션을 롤백하는 것은 데이터베이스의 반영사항만 롤백하는 것이지, 수정한 자바 객체까지 원상태로 복구해주지는 않는다.  
이 말인 즉 트랜잭션이 롤백되었다고 영속성 컨텍스트도 롤백되는 것은 아니라는 것을 의미하는데,  
기본 전략인 `트랜잭션당 영속성 컨텍스트` ㅈ

<!-- more -->