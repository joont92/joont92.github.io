---
title: 신규 엔티티에 기존 엔티티의 id를 넣어줄때 발생하는 일
date: 2018-12-27 16:22:47
tags:
    - orphanRemoval
    - entity child diff
    - merge
---

JPA를 개발하다 보면, 아래와 같은 행위를 할떄가 가끔씩 있다.  
(Spring Data JPA를 사용한다고 가정)  

```java
Member newMember = newMemberDTO.toEntity();
newMember.setId(1); // 이렇게 id를 직접 넣어주는 행위

// ...

memberRepositroy.save(newMember);
```

이는 간단하다.  
Repository를 만들때 implements하는 JpaRepository의 구현체인 SimpleJpaRepository의 save 메서드를 보면 아래와 같다.  
```java
// repository.save
@Transactional
public <S extends T> S save(S entity) {
    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}

// entityInformation.isNew
public boolean isNew(T entity) {

    ID id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
        return id == null;
    }

    if (id instanceof Number) {
        return ((Number) id).longValue() == 0L;
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s!", idType));
}
```

보다시피 엔티티의 식별자가 전달되었다면 merge, 그렇지 않다면 persist를 수행한다.  
위와 같이 식별자를 강제로 넣어서 전달하면 해당 엔티티에 대해서 `em.merge가 발생하게 되는 것`이다.  
merge니까, 해당 식별자로 member 테이블을 조회하고, 엔티티가 존재하면 영속성 컨텍스트에 넣게 된다.  
그리고 flush 되면 변경감지에 의해 update 문이 발생하게 될 것이다.(newEntity로 완전 대체되었기 때문에)  
만약 해당 식별자에 해당하는 엔티티가 없다면, 새로 insert 될 것이다.  
PUT REST API를 작성하기에 적절한 방법이다.(틀릴수도..)  

그렇다면 아래와 같이 하면 어떻게 될까?  

```java
Member newMember = newMemberDTO.toEntity();

newMember.setId(1); // 기존 member의 id값을 그대로 유지
// oldItem의 값들이 몇개 필요해서 들고와서 세팅
Member oldMember = memberRepository.findById(1);
newMember.setXXX(oldMember.getXXX());   

oldItem.setXXX(~~~); // 이렇게 하면 어떻게 되지?  

memberRepository.save(newMember);
```

newItem으로 대체하는데 oldItem의 값들이 몇개 필요해서 oldEntity를 조회한 다음 해당 값을 사용하는 케이스이다.  
이럴 경우, find에 의해 처음 영속성 컨텍스트로 들고오면 아래와 같을 것이다.  
key | entity
----|-------
1 | oldEntity

이 상태에서 아래의 save에 의해 merge가 수행될 것이고, 결과적으로 아래처럼 변할것이다.  
key | entity
----|-------
1 | newEntity

즉 oldEntity는 아예 영속성 컨텍스트에 의해 관리되지 않는 준영속 상태가 되어버리므로, 위처럼 oldItem에 무언가 변경을 수행해도 아무일도 일어나지 않는다.  
(당연한가?)  

# 자식이 있을 경우
해당 엔티티에 속하는 자식 엔티티들이 있을 경우 조금 복잡해진다.(안 복잡할수도 있다)  

```java
class Member{
    @OneToMany(mappedBy = "member", cascade = CascadeType.PERSIST)
    private List<Order> orderList = new ArrayList<>();
}
```

위와 같다고 할 때, 아래와 같이 작성하면 문제가 발생한다.  

```java
Member newMember = newMemberDTO.toEntity(); // orderList를 가지고 있음

newMember.setId(1); // 기존 member의 id값을 그대로 유지

memberRepository.save(newMember);
```

이때는, newMember의 orderList에 대해 연쇄적으로 persist를 수행하고 끝나버릴 것이므로,  
만약 기존의 1번 member에 해당하는 order가 있었다고 할 경우, 해당 order는 그대로 있고 신규로 전달된 order들이 저장되게 될 것이다.  
그러므로 만약 기존 1번 member에 해당하는 order가 2개 있고, 전달된 order가 2개인 상태에서 위의 메서드를 실행하게 되면 order가 총 4개가 되어버린다.  

이럴땐 `orphanRemoval = true` 속성을 주면 해결할 수 있다.  
```java
class Member{
    @OneToMany(mappedBy = "member", cascade = CascadeType.PERSIST, orphanRemoval = true)
    private List<Order> orderList = new ArrayList<>();
}
```

사실상 이건 명확한 교체가 아니므로, 
의도한 상황이라면 상관없는데, 그게 아니라면 `orphanRemoval = true`를 사용하여 존재하지 않는 엔티티에 대해 삭제하게끔 해야한다.  

```java
Member newMember = newMemberDTO.toEntity(); // orderList를 가지고 있음
newMember.setId(1); // 기존 member의 id값을 그대로 유지

memberRepository.save(newMember);
```

이렇게 하면 기존 1번 member에 있던 order는 모두 삭제되고, newMember에 있는 order가 전부 insert 된다.  
만약 기존 order에 새로 전달받은 order를 덧붙이고 싶으면 newMember의 orderList에 살려놓을 order들을 추가해줘야 한다.  

```java
Member newMember = newMemberDTO.toEntity(); // orderList를 가지고 있음
newMember.setId(1); // 기존 member의 id값을 그대로 유지

Member oldMember = memberRepository.findById(1);
List<Order> aliveOrder = 살려놓을_주문_구하기(oldMember.getOrder());

newMember.getOrderList().addAll(aliveOrder);

memberRepository.save(newMember);
```

<!-- more -->