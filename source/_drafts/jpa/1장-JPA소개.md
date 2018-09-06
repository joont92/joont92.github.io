---
title: 1장 - JPA소개
tags:
---

※ ORM은 객체지향과 관계형 데이터베이스의 상위 단계이므로 학습곡선이 높다.
SQL Mapper : Mybatis, JdbcTemplate..
JPA : Hibernate, EclipseLink..

기존 JDBC
엔티티 <> SQL 매핑을 항상 해줘야함.
엔티티는 SQL에 의존적임. 그러므로 엔티티를 신뢰할 수 없다.(항상 결국엔 SQL을 확인해야함)
엔티티는 SQL을 자바로 가져오기 위해 존재하는 도메인 객체 역할 정도밖에 안됨.

그에 반해 JPA는 JPA가 사용하는 API에 엔티티를 넣어주면 자동으로 SQL 쿼리를 만들어 실행해주므로 엔티티를 신뢰할 수 있다.

자바 객체와 RDB의 패러다임 불일치
1) 상속
abstract class Item{
    Long Id;
    String name;
    int price;
}
class Album Extends Item{
    String artist;
}
class Movie Extends Item{
    String director;
}
자바는 위와 같이 상속구조가 가능하나, RDB는 상속구조가 없다.
데이터 모델링의 DTYPE을 이용하면 상속 비슷하게 구현할 수 있긴하다.

※ DTYPE
Table ITEM{
    ID(PK),
    NAME,
    PRICE..
    DTYPE
}
Table ALBUM{     // ITEM과 연결
    ID(PK, FK),
    ARTIST
}

모델링에서 위와 같이 DTYPE을 이용해 구성할 경우
자바에선 ITEM, ALBUM의 CRUD를 이중으로 수행하는 상황이 온다.(ITEM, ALBUM)
이것이 각자의 패러다임을 맞추기 위한 낭비이다.
> JPA는 ALBUM 객체만 전달하면 알아서 수행

2) 연관관계
객체는 참조를 통해 객체끼리 연관관계를 가지고, 해당 참조에 접근해서 연관 객체를 조회한다.
테이블은 외래키를 사용하고, 조인을 이용해 연관 테이블을 조회한다.
> 둘의 패러다임이 완전히 다르다.
   자바는 부모 객체를 통해 자식 객체를 조회 가능하지만 그 반대는 불가능하다. 그러나 테이블은 양쪽 조회가 가능하다.(JOIN)
   ★ 객체는 참조를 통해 연관관계를 가지지만, 테이블은 JOIN이라는 기능을 통한다.
   서로의 패러다임을 맞추기가 힘들다.
   JAVA의 패러다임에 맞추자니 개발자가 직접 변환해줘야 하기도 하고, 굉장히 복잡해진다.
   그렇다고 RDB의 패러다임에 맞추자니 객체지향의 특징을 잃어버린다.
   > JPA는 JAVA의 연관관계를 편하게 사용 가능하다. 참조를 통한 관계를 설정하고, ORM Framework에 전달만 해주면 된다.
   (JAVA의 패러다임에 맞춰 엔티티 설계하고, 복잡한 과정은 JPA가 대신해준다.)

3) 객체 그래프 탐색
Member member = memberDao.find(mid);
member.getTeam();
member.getTeam().getName();
// 이와 같이 참조를 이용해서 파고드는 행위를 객체 탐색이라고 한다.

근데 사실상 아래와 같은 코드들(JPA를 사용하지 않는)
member.getTeam();
member.getOrder().getDelivery();

의 탐색 깊이는 결국 SQL문에 의존적이라는 것이다.
객체가 아무리 연관관계를 가진들 SQL에서 Order 테이블을 JOIN 하지 않았으면 2행은 Null Exception이 발생할 것 아닌가..

그렇다고 조회마다 Member에 관련된 모든 객체를 조회해서 메모리에 올려두는것은 비효율적.
상황에 따라 조회하는 메서드를 만들어야 한다.
> JPA에서는 연관 객체를 사용하는 시점에 select 쿼리를 실행할 수 있다(Lazy Loading)
   게다가 JPA는 이를 투명하게(transparent) 처리하므로 DAO에 따로 코드가 있지 않고
   getOrder() 함수를 호출할 때 지연 로딩을 한다.
   이는 엔티티 설정에서 줄 수 있다. 지연로딩을 하지 않을 경우 애초에 Member 객체를 얻어올떄 연관객체를 조인하는 쿼리가 실행된다.

4) 비교
Java에서는 비교가 동일성, 동등성으로 나뉘기 때문에 복잡해지는 부분이 많다.
class MemberDAO(){
    publid Member findOne(String id){
        // select * from member where id=? 실행

        Member memeber = new Member();
        // resultset을 member에 담음
        member.setXXX(rs.getString(XXX));
        ...

        return member;
    }
}

Member member1 = memberDao.findOne("id1");
Member member2 = memberDao.findOne("id1");

member1 == member2 // false;
> 동일성 비교에서 실패한다. new로 생성된 객체이기 때문이다.
   이처럼 db에서 같은 row를 조회했다 하더라도 위의 연산에서 true를 호출하게 하기가 어렵다.
   하지만 JPA는 위의 상황에서 동일성을 보장한다. (JPA의 조회 메서드를 사용)

- JPA란
위치
Application - JPA - JDBC - DB

JPA는 ORM 기술의 표준이다. ORM은 Object Relation Mapping의 약자이다.
JPA란 자바 ORM 기술에 대한 API 표준명세. 쉽게 말해 인터페이스들을 모아놓은 것이다.
따라서 JPA를 사용하려면 JPA를 구현한 ORM 프레임워크를 선택해야한다. (여러가지가 있으며 하이버네이트가 가장 대중적)
성숙한 ORM 프레임워크의 경우 간단한 CRUD외에 다양한 패러다임 문제까지 해결해주므로 개발자가 객체지향 어플리케이션 개발에 집중할 수 있다.

Why JPA?
1) 생산성 : ORM Framework에 엔티티만 전달하면 되므로, SQL을 개발자가 직접 작성하지 않아도 된다.
2) 유지보수 : 테이블의 명세가 변경되어도 변경할 사항이 많지 않다. 엔티티만 변경해주면 된다. (쿼리가 자동생성이므로)
            ex) 테이블에 컬럼이 추가되거나 삭제될 경우 JPA는 엔티티만 수정하면 되지만, mybatis의 경우 해당 엔티티를 사용하는 쿼리를 전부 수정해줘야 한다.
3) 패러다임 불일치 해결
4) JPA는 application과 jdbc사이에서 동작하기 때문에 시도할 수 있는 성능 최적화가 있다.
    위의 "4) 비교" 의 경우가 그렇다. 똑같은 row를 조회할 때 jdbc에 쿼리를 2번 보내지 않고, 기생성된 인스턴스를 사용한다.
5) 추상화 되어 있으므로 DB변경 또한 용이함. 벤더의 독립성