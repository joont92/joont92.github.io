---
title: [ddd] 리포지터리와 모델구현(JPA)
date: 2019-05-29 22:14:21
tags:
    - DDD start!
    - 리포지터리
---

데이터 보관소로 RDBMS를 사용할 때 객체 기반의 도메인 모델과 관계형 데이터 모델간의 매핑을 처리하는 기술로 ORM 만한 것이 없다  

# 리파지토리 구현
## 모듈 위치
리포지터리 인터페이스는 애그리거트와 같이 도메인 영역에 속하고, 리포지터리를 구현한 클래스는 인프라스트럭쳐 영역에 속한다  
> 리포지터리 구현 클래스를 domain.impl 같은 패키지에 위치시키는 것은 좋은 설계 방법이 아니다  

## 기본 기능 구현
리포지터리의 기본 기능은 다음 2가지 이다  
- 아이디로 애그리거트 조회
- 애그리거트 저장

이를 제공하는 인터페이스의 형식은 다음과 같다  
```java
interface OrderRepository {
    public Order findById(OrderNo no);
    public void save(Order order);
}
```
- 인터페이스는 애그리거트 루트를 기준으로 작성한다
- 애그리거트를 조회하는 기능의 이름을 지을 때 널리 사용되는 규칙은 `findBy프로퍼티(프로퍼티 값)` 의 형식을 사용하는 것이다  
    - e.g. findById, findByName, findByOrdererId
    - ID가 아닌 값으로 조회할때는 JPQL을 이용한다
- 조회에 해당하는 애그리거트가 존재하면 애그리거트 도메인(e.g. Order)를 반환하고, 존재하지 않는다면 null이나 Optional을 반환한다
- 1건이상 존재하면 List 컬렉션을 반환한다
- JPA는 트랜잭션 범위에서 변경한 데이터를 자동으로 DB에 반영하므로 따로 수정 메서드를 추가할 필요없다
- 삭제 기능을 구현한다면 삭제할 애그리거트 객체를 파라미터로 전달받게끔 한다
    - soft delete로 구현하는 것이 좋다

# 모델(매핑) 구현
## 기본 매핑
![엔티티와 벨류가 한 테이블에 매핑](/temp/엔티티와-벨류가-한-테이블에-매핑.png)  
- 애그리거트 루트는 엔티티이므로 `@Entity`로 매핑한다
    ```java
    @Entity
    class Order {
        // ...
    }
    ```
- 벨류는 `@Embeddable`로 매핑한다
    ```java
    @Embeddable
    class Orderer {
        // ...
    }
    ```
- 밸류를 사용하는 곳에서는 `@Embedded`로 매핑한다
    ```java
    @Entity
    class Order {
        @Embedded
        private Orderer orderer;
    }
    ```
- 벨류가 다른 벨류를 포함할 수 있다
    ```java
    @Embeddable
    class Orderer {
        @Embedded
        private MemberId memberId;
    }
    ```
- 데이터베이스 컬럼명이 다를 경우 `@AttributeOverride(s)` 어노테이션을 사용한다
    ```java
    @Embeddable
    class ShippigInfo {
        @Embedded
        @AttributeOverrides({
            @AttributeOverride(name = "zipcode", column = @Column(name="shipping_zipcode")),
            @AttributeOverride(name = "address1", column = @Column(name="shipping_address1")),
            @AttributeOverride(name = "address2", column = @Column(name="shipping_address2")),
        })
        private Address address;
    }
    ```

## 기본 생성자
생성자는 기본적으로 객체를 생성할 때 필요한 것을 받는 용도로 사용되는 것이다  
> 벨류 오브젝트의 경우 생성 시점에 모든 값을 전달받고, setter를 제공하지 않으므로, 기본 생성자가 필요없다  

하지만 JPA의 `@Entity`나 `@Embeddable`을 사용하려면 기본 생성자를 제공해야한다  
> 기존 객체를 상속한 프록시 객체를 사용하여 지연로딩 기능을 구현하기 때문이다  

이러한 이유 때문에 기본 생성자를 추가해줘야하는데, 알다시피 기본 생성자를 추가하면 객체가 온전하지 못한 상태로 제공될 수 있게된다  
그러므로 기본 생성자를 `protected`로 선언해서 상속한 프록시 객체에서만 사용할 수 있게 해야한다  

## 필드 접근 방식 사용
JPA는 기본적으로 필드와 메서드(프로퍼티)의 두가지 방식으로 매핑을 처리할 수 있다  
- 필드
    ```java
    @Entity
    @Access(AccessType.FIELD)
    class Order {
        @Column(name = "state")
        @Enumberated(EnumType.STRING)
        private OrderState state;
    }
    ```
- 메서드(프로퍼티)
    ```java
    @Entity
    @Access(AccessType.PROPERTY)
    class Order {
        @Column(name = "state")
        @Enumerated(EnumType.STRING)
        public OrderState getState() {
            return state;
        }

        public void setState(OrderState state) {
            this.state = state;
        }
    }
    ```
> 하이버네이트는 `@Access`가 없으면 `@Id`나 `@EmbeddedId`를 보고 접근 방식을 결정한다

공개 getter/setter를 추가하는 `메서드(프로퍼티)` 방식을 사용하게 되면 도메인의 의도가 사라지고, 객체가 아닌 데이터 기반으로 엔티티를 구현할 가능성이 높아지게 된다  
그러므로 가능하다면 `필드` 방식을 사용해서 객체가 제공할 기능 중심으로 구현하게끔 해야한다  

## AttributeConverter를 이용한 벨류 매핑
개발하다보면 가끔 벨류 프로퍼티 하나를 한 개 컬럼에 매핑해야 할 때가 있다  
```java
/**
 * 이 벨류 오브젝트를 DB 컬럼 "WIDTH VARCHAR(20)" 에 매핑
 * e.g. 1000mm
 **/
class Length {
    private int value;
    private String unit;
}
```

이럴땐 JPA 2.1 이후로 추가된 `AttributeConverter`를 이용하면 된다  
```java
interface AttributeConverter<X, Y> { // X = 벨류 타입, Y = DB 타입
    public Y convertToDatabaseColumn(X attribute); // 벨류 타입 -> DB 컬럼 값
    public Y convertToEntityAttribute(Y dbData); // DB 컬럼 값 -> 벨류 타입
}

class MoneyConverter implements AttributeConverter<Money, Integer> {
    @Override
    public Integer converToDatabaseColumn(Money money) {
        return money.getValue();
    }

    @Override
    public Money convertToEntityAttribute(Integer value) {
        return new Money(value);
    }
}
```

작성한 컨버터를 적용시키려면 아래 2가지 방법을 사용하면 된다  
(컨버터를 사용했기 때문에 `@Embedded`를 사용하지 않고 `@Column`으로 직접 매핑한다)  
- 특정 시점에만 적용
    ```java
    class Order {
        @Column(name = "totalAmounts")
        @Converter(converter = MoneyConverter.class)
        private Money totalAmounts;
    }
    ```
    > 리포지터리로 Order 를 handling 할 때 적용된다
- 모든 벨류 오브젝트에 적용
    ```java
    @Converter(autoApply = true)
    class MoneyConverter implements AttributeConverter<Money, Integer> {
        // ...
    }
    ```
    > 모든 `Money` 타입 프로퍼티에 자동 적용된다



<!-- more -->