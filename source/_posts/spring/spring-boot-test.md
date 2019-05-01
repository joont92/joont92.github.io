---
title: spring boot test
date: 2018-12-16 21:29:37
tags:
    - '@SpringBootTest'
---

<https://meetup.toast.com/posts/124>

spring boot test 모듈은 아래의 2개가 존재함
- spring-boot-test
- spring-boot-test-autoconfigure

대부분 spring-boot-starter-test로 충분함(위의 2개를 다 포함하고 있나?)  

# @SpringBootTest
@ContextConfiguration의 발전된 기능?  
테스트에 사용할 ApplicationContext를 쉽게 조작할 수 있음  
반드시 @RunWith(SpringRunner.class)와 함께 사용해야함

- Bean 설정
    - classes 속성을 통해 할 수 있음
    - 설정한 클래스들만 빈으로 등록함
    - @Configuration을 설정할 경우 내부의 빈들도 전부 등록됨

- @TestConfiguration
    - 기존에 정의한 @Congifuration을 재정의하고 싶을 경우
    - @TestConfiguration에 정의된 bean으로 override 된다
    - 테스트 클래스내에 선언할수도 있지만, 이러면 @ComponentScan시에만 감지되므로 classes를 명시했을 경우 사용할 수 없다는 단점이 있다
    - @Import를 써서 직접 사용하는 것이 더 좋다
        - 이렇게 하면 여러클래스에서도 사용할 수 있다
    
- @MockBean
    - @MockBean 어노테이션을 사용하면 mock 객체를 빈드로 등록할 수 있다
    - @Autowired 등으로 주입받는 객체가 @MockBean으로 선언된 객체라면, 해당 mock 객체가 주입된다
    - @MockBean으로 선언한 객체가 이미 빈에 등록되어져 있다면 override 된다

- properties
    - 스프링은 테스트시에 기본적으로 class path의 application.properties(yml)을 참조한다
    - properties 속성으로 별도의 테스트를 위한 설정파일을 지정할 수 있다
    - test classpath의 application.yml이 있으면 그걸 먼저 참조..하나?

# TestRestTemplate
- @SpringBootTest와 RestTestTemplate를 같이 사용한다면 편리하게 웹 통합테스트가 가능하다
- @SpringBootTest에서 Web Environment를 설정했다면 TestRestTemplate는 그에맞춰 자동으로 빈으로 생성된다
- MockMvc는 servlet container를 생성하지 않고, TestRestTemplate는 servlet container를 생성함
    - 그러므로 실제 동작되는 것처럼 테스트를 수행할 수 있다
    - 클라이언트가 수행하는 것 처럼 테스트할 수 있다

# 트랜잭션
- @Test 어노테이션과 @Transactional 어노테이션을 함꼐 사용했을 경우 테스트가 끝나면 rollback 됨
    - 기존의 spring-test와 동일
    - 만약 롤백하고 싶지 않다면 아래와 같이 함
        ```java
        @Test
        @Rollback(false)
        public void insetTest(){
            // ...
        }
        ```
- 하지만 webEnvironment의 RANDOM_PORT나 DEFINED_PORT로 테스트를 설정하면 테스트가 별도의 스레드에서 수행되기 때문에 rollback이 수행되지 않음

# @JsonTest
- json serialize, deserialize를 편하게 테스트해볼 수 있다

# @WebMvcTest
- 서버사이드 API Test?

# Async web Test
- async 테스트?

# @DataJpaTest
- memory db를 사용하고 테스트가 끝날떄마다 롤백됨
- @Transactional 어노테이션을 포함하고 있음
- 테스트를 위한 TestEntityManager 클래스가 빈으로 등록된다
    - Spring Data JPA를 쓰지 않을 경우 유용할까?

# @JdbcTest
- @DataJpaTest와 유사하게 순수 JDBC를 테스트하고 싶을 경우 사용
- 테스트를 위한 JdbcTemplate가 생성된다

# @DataMongoTest
- mongodb

# @RestClientTest
- 좀 더 리얼환경에 가깝게 api 테스트를 할 수 있음
- RestTemplate에 반응하는 가상의 mock 서버라고 생각하면 된다?

<!-- more -->