---
title: 12장-Spring Data JPA
date: 2018-09-06 18:50:11
tags:
---

스프링 프레임워크에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트
인터페이스만 작성하면 데이터 엑세스 계층을 만들 수 있다.

- setting
1) pom.xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>1.10.6.RELEASE</version>
</dependency>

2) 설정파일
·  xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/util
    http://www.springframework.org/schema/util/spring-util.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/jdbc
    http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd
    http://www.springframework.org/schema/data/jpa
    http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="org.hsqldb.jdbc.JDBCDriver" />
        <property name="url" value="jdbc:hsqldb:mem:testdb" />
        <property name="username" value="sa" />
        <property name="password" value="" />
    </bean>

    <!-- JPA settings -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory" />
    </bean>

    <!-- JPA 예외를 스프링 예외로 변환 -->
    <bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />
     
    <!-- Hibernate 설정 -->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="packagesToScan" value="com.khjeon" /> <!-- @Entity 탐색 시작 위치 -->

        <property name="jpaVendorAdapter"> <!-- 하이버네이트 구현체 사용 -->
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter" />
        </property>

        <property name="jpaProperties">
            <!-- 하이버네이트 상세 설정 -->
            <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.HSQLDialect</prop> <!-- 방언 -->
            <prop key="hibernate.show_sql">true</prop> <!-- SQL 보기 -->
            <prop key="hibernate.format_sql">true</prop> <!-- SQL 정렬해서 보기 -->
            <prop key="hibernate.use_sql_comments">true</prop> <!-- SQL 코멘트 보기 -->
            <prop key="hibernate.id.new_generator_mappings">true</prop> <!-- 새 버전의 ID 생성 옵션 -->
            <prop key="hibernate.hbm2ddl.auto">create</prop> <!-- DDL 자동 생성 -->
            </props>
        </property>
    </bean>

    <!-- Jpa repository -->
    <jpa:repositories base-package="com.khjeon" entity-manager-factory-ref="entityManagerFactory" />

</beans>

· java config
@Configuration
@EnableJpaRepositories(basePackages="repository package.. ")
public class AppConfig{}

> base package 내부에 있는 Repository 인터페이스를 찾아서 동적으로 클래스를 생성하고 빈으로 등록한다.

3) 리파지토리
빈으로 등록하기 위해 상단에 @Repository 어노테이션을 선언한다.
그리고 CRUD에서 공통으로 처리하는 JpaRepository 인터페이스를 상속하면 Spring data jpa를 사용할 수  있는 리파지토리가 된다.

@Repository
public interface <Entity명>Repository extends JpaRepository<<Entity명> , <Entity 기본키 타입> > {}
> generic에는 위와 같이 선언한다.

- 기본 메서드
1) save(Entity) :
전달받은 entity의 기본키가 null이면 insert, null이 아니면 update를 수행한다.(새로운 엔티티는 저장, 아닐경우 수정)
update시에 cache를 위해 전체 필드에 대해 update 하므로, 전달받은 Entity의 값이 null 일 경우를 주의해야 함
개인적인 생각으로 update는 영속성 컨텍스트를 이용하는것이 좋다고 생각함.
2) findAll : 모든 엔티티를 조회한다. 정렬(Sort)나 페이징(Paging)을 파라미터로 제공할 수 있다.
3) findOne : 엔티티 하나를 조회. 내부에서 EntityManager.find()를 호출
4) getOne : 엔티티를 프록시로 조회. 내부에서 EntityManager.getReference() 호출
5) delete : 엔티티 하나를 삭제. 내부에서 EntityManager.remove() 호출

-- 쿼리 생성(cglib)
1) 기본
findBy~~ : 조건절 추가
countBy~~ : count 쿼리
deleteBy~~ : 삭제 쿼리

2) Like 쿼리
Using Like: select ... like :username
> List<User> findByUsernameLike(String username);

StartingWith: select ... like %:username
> List<User> findByUsernameStartingWith(String username);

EndingWith: select ... like :username%
> List<User> findByUsernameEndingWith(String username);

Containing: select ... like %:username%
>List<User> findByUsernameContaining(String username);

3) 접근방법 예시2
like : findByValContains(String str)
in : findByValIn(Collection col)
entity에 Address entity가 있고, 내부에 ZipCode가 있는경우
> findByAddress_ZipCode(String zipCode)
현테이블과 Address 테이블이 Left Join되서 나온다..

- JPQL
@Query 메서드로 선언
입력하는 쿼리에 맞춰 반환형도 제대로 맞춰줘야 한다. 아니면 오류 발생

- native Query
@Query(쿼리, [nativeQuery=true])
변수 : 숫자 = ?0 부터 시작, 이름 = :이름

- Spring-Data 벌크성 수정 쿼리
@Modifying
@Query("update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount)
int bulkPriceUp(@Param("stockAmount") String stockAmount);
> @Query의 파라미터와 매핑. 순서로도 지정 가능. :stockAmount 대신 ?0


리스트도 가능. ","로 연결되어 들어간다.
※ 배열은 안됨
@Query("SELECT p FROM postContent WHERE p.postNo IN(:postNoList)
public List<PostContent> postList(@Param("postNoList") List<Long> postNoList);
> 1,2,3,4...

-반환 타입
결과 1건 이상 : 컬렉션 인터페이스
단건 : 반환타입 지정 ex)Member
결과 없을 경우 : 컬렉션=빈 컬렉션, 단건=null

- 정렬
org.springrframework.data.domain.Pageable : 페이징 기능(내부에 Sort포함)

Pageable은 인터페이스이므로 호출할때는 구현체인 PageRequest를 사용한다.
인자로는 현재 페이지수, 보여질 개수를 전달한다. Sort 오브젝트도 추가로 전달 가능하다.
> new PageRequest(page, size, ....)
size * page로 limit의 시작값이 정해짐

※ Pageable을 사용하면 스프링 데이터는 페이징 기능을 위한 count 쿼리를 추가로 호출하고
반환형으로 Page 오브젝트를 사용한다.
반환형이 Page가 아닐 경우 count 쿼리를 추가로 호출하지 않는다.
스프링 데이터에서 기본 제공하는 검색 메서드에서는 Pageable을 사용할경우 반환형이 무조건 Page이나,
사용자 정의 메서드에서는 반환형이 Page가 아니게 설정할 수 있다.
이때는 Pageable을 사용했으나 count 쿼리가 추가로 발생하지 않는다.
> 즉 반환형이 Page일때만 count 쿼리를 추가로 호출한다.

반환타입인 Page 오브젝트는 페이징 처리에 다양한 메서드를 제공한다(현재 페이지, 전체 개수 등)

Repository를 사용하지 않고 반환형으로 Page를 줘야할 경우가 있다.
그럴 경우 PageImpl 객체를 사용한다.
new PageImpl(List<T>) : 페이지 객체로 변환
new PageImpl(List<T>, Pageable, int) : 페이징할것도 아니면서 Pageable 왜 쳐받는지 모르겠다. 그냥 정보제공용이다.
                                                          애초에 List에 Paging되는걸 던져야한다


org.springrframework.data.domain.Sort : 정렬
그냥 사용할 수도 있고, PageRequest에 넣어서 사용할 수도 있다.
new Sort(~~)의 형태로 전달한다.

ex) new Sort(Direction.DESC, "변수명");
> 정렬할 변수명으로 앞의 Direction이 적용되어 정렬된다.
변수명에는 엔티티 필드명 전달한다. table alias는 전달하지 않아도 된다.
계산된 컬럼의 alias 또한 전달할 수 있다. 리스트도 전달할 수 있다(Direction은 일괄적용)

- QueryDSL 사용하기
QueryDslRepositorySupport 클래스를 상속한 QueryDSL용 클래스를 만듬
해당 클래스에서 구현할 메서드는 따로 인터페이스를 만든다.

e.g.

public interface PostContentCustom{
}

public class PostContentRepositoryImpl extends QueryDslRepositorySupport implements PostContentRepositoryCustom{
    // 생성자 필요
    public PostContentRepositoryImpl(){
        super(PostContent.class);
    }

          // PostContentRepositoryCustom에 있는 메서드 구현
    @Override
    public Page<PostContent> search(Map<String, Object> paramMap, Pageable page){
        QPostContent postContent = new QPostContent("p");

        JPQLQuery<PostContent> query = from(postContent);

        BooleanBuilder condition = new BooleanBuilder();

        query.where(postContent.title.contains("test"));
        
        // paging
        query = super.getQuerydsl().applyPagination(page, query);
        QueryResults<PostContent> result = query.fetchResults(); // 결과

        return new PageImpl<PostContent>(result.getResults(),page,result.getTotal()); // 이런식으로 리턴 가능
    }
}

<!-- more -->