---
title: OneToOne Lazy Loading
date: 2018-09-06 18:54:14
tags:
---

[하이버네이트] OneToOne 연관 관계 Lazy Fetching이 안 먹어!?
참조 : http://whiteship.me/?p=13301

OneToOne 관계를 맵핑 했을 때, Lazy Fetching이 제대로 적용되는 경우가 있고, 그렇지 않은 경우가 있다. 우선 제대로 동작하는 경우부터 볼까나…

Product 1 –> 1 ProductDetails
Product 1 –> 1 ProductInfo

이렇게 OneToOne 관계가 두 개 있다고 가정하고, 다음과 같이 맵핑했다.

@Entity
public class Product {
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
private BigDecimal price;
private String name;

@OneToOne(fetch = FetchType.LAZY)
private ProductDetails productDetail;
@OneToOne(fetch = FetchType.LAZY)
private ProductInfo productInfo;
//나머지 생략
}

@Entity
public class ProductDetail {
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
private String detail;
}

@Entity
public class ProductInfo {
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
private String info;
}

각 엔티티에 선언한 속성이 의미가 있는건 아니니까 주의깊게 살펴보지 마시고, 연관 관계를 유심히 살펴보는게 좋겠다. 단방향 관계다. 그리고 FetchMode.LAZY다.
Repository는 Spring-Data-JPA 1.0.1.RELEASE를 사용해서 만들었고, 테스트 코드는 다음과 같다.

Product product = productRepository.findOne(1l);
assertThat(product, is(notNullValue()));
assertThat(product.getName(), is("keesun"));

System.out.println("========LAZY LOADING…=========");
product.getProductDetails().getDetails(); // lazy loading


콘솔
Hibernate:
select
product0_.id as id0_0_,
product0_.name as name0_0_,
product0_.price as price0_0_,
product0_.productDetails_id as productD4_0_0_,
product0_.productInfo_id as productI5_0_0_
from
Product product0_
where
product0_.id=?
18:24:53.609 [main] DEBUG org.hibernate.jdbc.AbstractBatcher – about to open ResultSet (open ResultSets: 0, globally: 0)
18:24:53.609 [main] DEBUG org.hibernate.loader.Loader – result row: EntityKey[usecase.snapshot.domain.Product#1]
18:24:53.615 [main] DEBUG org.hibernate.jdbc.AbstractBatcher – about to close ResultSet (open ResultSets: 1, globally: 1)
18:24:53.616 [main] DEBUG org.hibernate.jdbc.AbstractBatcher – about to close PreparedStatement (open PreparedStatements: 1, globally: 1)
18:24:53.618 [main] DEBUG org.hibernate.engine.TwoPhaseLoad – resolving associations for [usecase.snapshot.domain.Product#1]
18:24:53.621 [main] DEBUG org.hibernate.engine.TwoPhaseLoad – done materializing entity [usecase.snapshot.domain.Product#1]
18:24:53.622 [main] DEBUG o.h.e.StatefulPersistenceContext – initializing non-lazy collections
18:24:53.623 [main] DEBUG org.hibernate.loader.Loader – done entity load
========LAZY LOADING…=========
18:24:53.623 [main] DEBUG org.hibernate.impl.SessionImpl – initializing proxy: [usecase.snapshot.domain.ProductDetails#1]
18:24:53.624 [main] DEBUG org.hibernate.loader.Loader – loading entity: [usecase.snapshot.domain.ProductDetails#1]
18:24:53.624 [main] DEBUG org.hibernate.jdbc.AbstractBatcher – about to open PreparedStatement (open PreparedStatements: 0, globally: 0)
18:24:53.625 [main] DEBUG org.hibernate.SQL –
select
productdet0_.id as id1_0_,
productdet0_.details as details1_0_
from
ProductDetails productdet0_
where
productdet0_.id=?


이제 양방향 연관관계로 변경해보겠다.

@Entity
public class Product {
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
private BigDecimal price;
private String name;

@OneToOne(mappedBy="product", fetch = FetchType.LAZY)
private ProductDetails productDetail;
@OneToOne(mappedBy="product", fetch = FetchType.LAZY)
private ProductInfo productInfo;
//나머지 생략
}

@Entity
public class ProductDetail {
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
private String detail;

@OneToOne(fetch = FetchType.LAZY)
private Product product;
}

@Entity
public class ProductInfo {
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
private String info;

@OneToOne(fetch = FetchType.LAZY)
private Product product;
}

양방향으로 변경되면서 스키마가 변경되었다.
처음엔 부모 테이블(Product)에서 자식 테이블(ProductDetail, ProductInfo)을 외래키로 가지고있었고,
현재는 자식테이블에서 부모테이블을 외래키로 가지고 있는 형태이다.

테스트 코드를 실행하여 Lazy 로딩이 되는가 살펴보자.

Product product = productRepository.findOne(~);
syso(lazyloading...)
product.getProductDetail();

lazy 로딩이 안된다. 개떡같은 상황이다.
사실상 나는 밑의 데이터모델링이 좀 더 깔끔하고, 맞는 방식이라 생각되는데..

참조했던 블로그의 글쓴이는 단방향/양방향의 여부를 언급했지만 내가 보기엔 연관관계의 주인 여부인것으로 보인다.

연관관계의 주인이 아닌 쪽에서 Lazy 로딩을 시도하면 되지 않는것이다.
프록시의 한계..?
(1:N의 관계에서는 가능했다.)

해결방법으로는 구조변경, 1:N로 변경(젤 싫음ㅋㅋㅋ) 이 있을 것이고..
다른 방법도 있다한다.
참조했던 블로그에 나와있음! 빌드 타임 인스트런트? finderHandler?

http://stackoverflow.com/questions/17987638/hibernate-one-to-one-lazy-loading-optional-false

<!-- more -->