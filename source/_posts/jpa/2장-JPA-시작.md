---
title: 2장 - JPA 시작
tags:
---

- H2 데이터베이스
따로 설치할 필요없이 자바만 설치되어 있으면 사용할 수 있는 경량(1.7M) 데이터베이스이다.
(h2database.com에서 다운로드 가능)
JVM 메모리 안에서 실행되는 임베디드 모드와 별도의 서버를 띄우는 서버모드가 있다.

※ package가 javax.persistence인것은 JPA 표준이므로 특정 구현체에 종속되지 않으나
hibernate로 시작되는건 hibernate만 사용 가능

- JPA 설정
1) 어노테이션
@Entity : 이 엔티티는 테이블과 매핑한다고 JPA에게 알려줌
@Table : 엔티티에 매핑될 테이블 정보를 알려줌(name 속성을 사용). 생략 시 클래스 이름을 사용
@Id : 기본키 매핑
@Column : 컬럼 매핑. name 속성에 테이블의 속성명을 써준다. 생략시 필드명을 사용

2) persistence.xml
JPA 설정정보이다. META-INF 안에 있으면 자동으로 인식한다
\<persistence\> 태그로 시작. namespace와 version 지정
\<persistence-unit name="~~"\> 이라는 영속성 유닛 지정. 일반적으로 DB하나당 하나의 영속성 유닛을 등록한다.

필수 속성
1) JPA 표준 속성
javax.persistence.jdbc.driver
javax.persistence.jdbc.user
javax.persistence.jdbc.password
javax.persistence.jdbc.url
2) 하이버네이트 속성
hibernate.dialect
> dialect는 데이터베이스 방언을 의미한다. 방언은 특정 데이터베이스만의 고유한 기능을 뜻한다.
   JPA에서 특정 데이터베이스의 기능은 여기 방언에 지정한 클래스로 처리한다.
   즉, DB가 교체되어도 이 방언만 수정해주면 된다.

이 외에 optional한 속성들이 있다(쿼리 로그, 쿼리 포맷, 주석 등등)

- 실습
Hibernate의 기능을 사용하려면 EntityManager 라는 클래스를 사용해야 한다.(mybatis에서 sqlSession 이었던 것처럼)
EntityManager를 얻으려면 먼저 EntityManagerFactory를 생성해야 한다.

EntityManagerFactory emf = Persistence.createEntityManagerFactory("영속성유닛");

persistence.xml의 내용을 읽어 EntityManagerFactory를 생성한다.
중요한점은 위의 엔티티 매니저 팩토리를 생성하는 비용이 매우 크다는 것이다.
그러므로 엔티티 매니저 팩토리는 딱 1번만 생성하고 공유해서 사용해야 한다.

EntityManager em = emf.createEntityManager();

엔티티 매니저 팩토리에서 엔티티 매니저를 생성한다.JPA의 대부분 기능은 이 EntityManager에서 제공한다(CRUD)
하나의 엔티티 매니저는 하나의 데이터소스를 유지하면서 통신한다.
데이터베이스 커넥션과 밀접한 관련이 있으므로 스레드간에 공유하거나 재사용하면 안된다.

예시)
※ JPA를 사용해서 데이터를 변경하려면 항상 **트랜잭션 안에서** 수행해야한다. 아니면 예외가 발생한다.
Member member = new Member();
member.setXXX("XXX");
....

em.persist(member); // 저장

member.setXXX("XXX"); // 수정.  JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있다.

em.remove(member); // 삭제
// Member Entity에 @Id 컬럼으로 지정한 부분을 조건으로 삭제를 진행한다.

em.find(Member.class, id); // select * from member where id=?
// @Id 컬럼으로 지정한 부분을 조건으로 조회하는 아주 단순한 조회 메서드이다.

- JPQL
ORM의 한계를 위해 존재하는 부분.
select의 기본 문법을 지원함. 그러나 대상이 RDB의 테이블이 아니라 Entity이다.
> TypedQuery<T> query = em.createQuery(JPQL, return Type);
   T t = query.getReulstList();
ex) em.createQuery("select m from Member m", Member.class);
// Member는 MEMBER테이블이 아니라 엔티티 Member이다.