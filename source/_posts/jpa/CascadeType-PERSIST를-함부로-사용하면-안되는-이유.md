---
title: CascadeType.PERSIST를 함부로 사용하면 안되는 이유
date: 2018-12-27 11:43:03
tags:
    - CascadeType.PERSIST
    - CascadeType.ALL
    - add list insert
    - delete doesn't work
    - orphanRemoval doesn't work
---

엔티티의 자식에 CascadeType.PERSIST를 지정할 경우 JPA에서 추가적으로 수행하는 동작이 있고,  
이 때문에 예상치 못한 사이드 이펙트가 발생할 수 있으므로 이를 남겨두고자 한다.  

일단 기본적으로 cascade(영속성 전이)는 간단하다.  
EntityManager를 통해 `영속성 객체에 수행하는 행동이 자식까지 전파`되는 것이다.    

```
em.persist(parent);
==
em.persist(parent);
em.persist(child1);
em.persist(child2);
```

# 변경 감지에서의 CascadeType.PERSIST
근데 여기 [JPA 2.2 specification](https://download.oracle.com/otndocs/jcp/persistence-2_2-mrel-eval-spec/index.html) 문서의 3.2 장 `Entity Instance's Life Cycle`에
변경감지 부분인 3.2.4 Synchronization to the Database에 보면 아래와 같은 내용이 추가적으로 있음을 볼 수 있다.  

> The semantics of the flush operation, applied to an entity X are as follows:
> • If X is a managed entity, it is synchronized to the database.
> • For all entities Y referenced by a relationship from X, if the relationship to Y has been annotated with the cascade element value cascade=PERSIST or cascade=ALL, the persist operation is applied to Y.  

flush가 발생할 때 `CascadeType.PERSIST`나 `CascadeType.ALL`이 있을 경우 자식에 연쇄적으로 `persist operation`이 발생한다는 의미이다.  

이 특징을 기반으로 아래의 행위들을 설명할 수 있다.  
Member와 Order의 관계는 아래와 같다고 가정한다.  
```java
class Member{
    @OneToMany(mappedBy = "member", cascade = CascadeType.PERSIST)
    private List<Order> orderList = new ArrayList<>();
}
class Order{
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;
}
```

1. em.persist  

```java
Member member = new Member();
Order order1 = new Order();
Order order2= new Order();

member.addOrder(order1);
member.addOrder(order2);

em.persist(member);

Order order3 = new Order();
member.addOrder(order3);

// order1, order2, order3 insert 됨
```

member를 persist할 때 order1, order2 까지 연쇄적 persist가 발생하고,  
트랜잭션이 끝나고 flush 될 때 자식들에 대해 다시 persist operation을 수행하게 된다.  
spec에 보면 persist operation은 아래와 같다.  
> • If X is a new entity, it becomes managed. The entity X will be entered into the database at or before transaction commit or as a result of the flush operation.  
> • If X is a preexisting managed entity, it is ignored by the persist operation. // ...  

즉 member의 orderList 3개에 대해 모두 persist operation이 발생하고,  
앞의 2개는 이미 존재하던 것이므로 무시되고, 마지막 order3는 추가적으로 insert 되는 것이다.  

2. em.merge  

```java
Member member = new Member();
Order order1 = new Order();
Order order2 = new Order();

member.addOrder(order1);
member.addOrder(order2);

member = em.merge(member);

Order order3 = new Order();
member.addOrder(order3);

// order1, order2, order3 insert 됨
```

CascadeType.PERSIST 이므로 em.merge 할 때 자식까지 연쇄적으로 merge가 발생하지는 않는다.  
하지만 flush 될 때 CascadeType.PERSIST에 의해 member 3개에 대해 모두 persist operation이 발생한다.  

3. em.find  

```java
Member member = em.find(Member.class, 1);
Order order1 = new Order();
Order order2 = new Order();

member.addOrder(order1);
member.addOrder(order2);

// order1, order2 insert 됨
```

이 또한 flush 될 떄 CascadeType.PERSIST에 의해 자식 order1, order2에 대해 persist operation이 수행된다.  

즉 모든 행위는 `flush의 CascadeType에 대한 특징` 때문이다.  

이러한 특징으로 봤을때, 우리가 의문을 가졌던 아래 코드 또한 설명이 된다.  

```java
class Member{
    @OneToMany(mappedBy = "member", cascade = CascadeType.MERGE)
    private List<Order> orderList = new ArrayList<>();
}

Member member = new Member();
Order order1 = new Order();
Order order2= new Order();

member.addOrder(order1);
member.addOrder(order2);

member = em.merge(member);

Order order3 = new Order();
member.addOrder(order3); 
// order1, order2 insert 됨
```

반면에 `CascadeType.MERGE`의 경우 flush와 관련이 없기 떄문에, `em.merge` 메서드에 전달한 엔티티까지만 연쇄적으로 merge가 되고, 아래는 그냥 무시되었던 것이다.  

# persist operation의 대상  
위에서 언급했다시피 CascadeType.ALL, CascadeType.PERSIST 어노테이션이 추가된 자식에 대해 모두 persist operation을 발생시킨다. (소스를 정확히 본것은 아니므로 틀릴 수 있음)  
그러므로 아래의 두 코드에서 발생하는 insert가 동일하게 된다.  

```java
public void cascadeTest(){
    Member member = em.find(Member.class, 1); // 5개의 orderList가 있다고 가정

    member.addOrder(order1);
    member.addOrder(order2);
}

// ==

public void cascadeTest(){
    Member member = em.find(Member.class, 1); // 5개의 orderList가 있다고 가정
    member.getOrderList().clear(); // 기존의 애들을 다 지워버려도

    member.addOrder(order1);
    member.addOrder(order2);
}
```

첫번쨰의 경우 총 7개의 order에 대해 persist operation을 수행하여 5개는 무시되고, 2개가 insert 된것이고,  
두번쨰의 경우 clear로 날려버렸기 때문에 총 2개의 order에 대해 persist operation이 수행되어 2개가 insert 된 것이다.(기존에 있던 것을 삭제하고 싶으면 `orphanRemoval = true`를 줘야한다)  
그러므로 위의 두 행위는 결과적으로 데이터베이스에 동일한 행위를 수행하게 되는 것이다.  

# 예상치 못한 동작1 
```java
class Member{
    @OneToMany(mappedBy = "member", cascade = CascadeType.PERSIST)
    private List<Order> orderList = new ArrayList<>();
}
class Order{
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;
}

public void deleteTest(){
    Member member = em.find(Member.class, 1);

    List<Order> orderList = member.getOrderLsit();

    for(Order order : orderList){
        em.remove(order);
    }
}
```

(지금은 예제가 간단하지만, 위와 같은 상황은 얼마든지 나올 수 있음)  
order가 삭제될 것이라고 예상할 수 있지만, flush시에 orderList에 남아있는 모든 order에 대해 persist 연산을 수행하므로 결과적으로 delete 메서드가 날라가지 않는 현상이 발생한다.  
그러므로 `CascadeType.PERSIST`를 사용하고자 할 경우 삭제하는 order에 맞춰 orderList에서 요소를 삭제해주거나,  
`orphanRemoval = true`를 사용해 orderList에서 삭제되면 자동으로 delete 가 날라가게끔 해야한다. 

# 예상치 못한 동작2


<!-- more -->