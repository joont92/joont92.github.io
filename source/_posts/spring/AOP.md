---
title: [spring] AOP
date: 2019-05-06 22:05:21
tags:
    - spring aop
---

# 메서드 전반적으로 공통기능을 넣고싶다
특정 메서드들에 공통된 기능을 넣고 싶을 경우(로깅, 트랜잭션 등등)가 있다  
해당 기능들을 직접 메서드에 넣으면 클래스의 관심사가 1개 이상이 되는 문제가 있으니 이를 분리해야 한다  

<https://jojoldu.tistory.com/69?category=635883>  
상속 적용  
1. 비즈니스 로직을 추상메서드로 가지는 부모클래스를 생성함
2. 추상클래스 앞 뒤로 before(), after() 같은 메서드들을 실행함
3. 하위 클래스에서 추상 메서드를 오버라이드 한다
    - 단일 책임 원칙을 지킬 수 있게 됨

괜찮은 방법이긴 하지만 상속의 고질적인 문제점을 그대로 가져가게 됨  

<https://jojoldu.tistory.com/70?category=635883>  
데코레이터 패턴 적용  
1. 인터페이스를 하나 만듦(e.g. UserService)
2. UserService를 구현하고, 비즈니스 로직을 구현하는 구현체를 만듦(e.g. UserServiceImpl)
3. UserService를 구현하고, 공통 기능을 위한 구현체를 만듦(e.g. UserPerformenceServiceImpl)
4. 3번 객체의 인자로 2번 객체를 가짐
    - UserPerformanceServiceImpl에서 UserServiceImpl을 DI 받음
5. UserService로 DI받으면 UserPerformanceImpl이 injection 됨
6. UserController -> UserService(UserPerformanceServiceImpl -> UserServiceImpl) 의 형태로 수행됨

이후 여러가지 방식이 시도됨  

# AOP란?
<https://jojoldu.tistory.com/71?category=635883>  
AOP = Aspect Oriented Programming(관점 지향 프로그래밍)  
> 애플리케이션 전체에 걸쳐 사용되는 기능을 재사용하도록 지원하는 것  

부가기능(로깅, 트랜잭션)이라는 관점에서 어플리케이션을 바라보고, 횡단 관심사를 잘라냄(크로스 컷팅)  
그리고 이를 모듈화하여 재사용하고자 함  

## AOP 용어
- 타겟
    > 부가기능을 부여할 대상
- 애스펙트
    > 부가기능 모듈을 뜻함  
    > 어드바이스 + 포인트컷  
- 어드바이스
    > 실질적인 부가기능을 가지고 있는 모듈  
    > `무엇을`, `언제`를 포함하고 있음  
    > 특정 오브젝트에 종속되지 않음  
- 포인트컷
    > 부가기능(어드바이스)가 적용될 대상을 선정하는 룰
- 조인포인트
    > 어드바이스가 적용될 위치  
    > 스프링은 기본적으로 메서드 조인포인트만 지원함(프록시 방식이기 떄문)  
    > 다른 조인포인트를 사용하고 싶다면 AspectJ 같은 것을 써야함(byte instrument?)  
- 위빙
    > 타겟에 애스펙트가 적용되어 프록시 객체가 생성되는 과정  

## 적용
```java
@Aspect
public class Performance {

    @Around("execution(* com.blogcode.board.BoardService.getBoards(..))")
    public Object calculatePerformanceTime(ProceedingJoinPoint proceedingJoinPoint) {
        Object result = null;
        try {
            long start = System.currentTimeMillis();
            result = proceedingJoinPoint.proceed();
            long end = System.currentTimeMillis();

            System.out.println("수행 시간 : "+ (end - start));
        } catch (Throwable throwable) {
            System.out.println("exception! ");
        }
        return result;
    }
}
```
> 스프링 부트에서는 몇가지 설정을 추가한 후 이렇게 애스펙트를 정의하면 간단히 적용가능하다  

### 어드바이스
1. @Before (이전)
2. @After (이후)
3. @AfterReturning (정상적 반환 이후)
4. @AfterThrowing (예외 발생 이후)
5. @Around (메소드 실행 전후)
    - procceed()를 꼭 호출해줘야함  

### 포인트컷
종류는 위 글 마지막 부분 참조  

expression 사용법은 아래와 같다  
```
execution([접근제한자] 리턴타입 [클래스타입(패키지포함).]메서드이름(타입 | "..", ...) [throws 예외])
```

메서드의 풀 시그니처를 문자열로 비교한다고 보면 된다.  
`getMethod`를 통해 메서드 풀 시그니쳐를 뽑아보면 아래와 같다.  
```
public int springbook.learningtest.spring.pointcut.Target.minus(int,int) throws java.lang.RuntimeException
```

## 확장
<https://jojoldu.tistory.com/72?category=635883>  
@PointCut 어노테이션을 이용해서 어노테이션을 메서드로 정의하고, 재사용할 수 있음  

<!-- more -->