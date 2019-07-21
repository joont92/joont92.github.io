---
title: [jpa] 'jpa entity equals, hashcode'
date: 2019-02-13 19:01:42
tags:
    - jpa equals hashcode
    - equals
    - hashcode
---

JPA는 영속성 컨텍스트의 키로 엔티티의 식별자를 사용한다.  
식별자가 primitive 타입일 경우 `==`, reference 타입일 경우 `equals`를 사용한다(아마도 당연히)  
기본적인 wrapper 타입의 경우 equals가 잘 구현되어 있기 때문에 상관없으나  
`@EmbeddedId`의 경우 equals, hashCode를 구현해주지 않으면 두 엔티티가 같다는 것을 보장해줄 수 없다.  

```java
@Entity
class Member{
    @EmbeddedId
    private MemberPK id;

    public Member(MemberPK id){
        this.id = id;
    }
}

@Embeddable
class MemberPK{
    private Integer memberId;
    private String name;
}
```

```java
Member member1 = new Member(new MemberPK(1, "joont"));
Member member2 = new Member(new MemberPK(1, "joont")); 

em.persist(member1);
em.persist(member2);
```

MemberPK에 대해 eqauls, hashCode를 구현했다면 두 엔티티가 같다고 보장되므로 Member가 하나만 들어가지만,  
equals, hashCode를 구현하지 않았다면 똑같은 식별자를 가졌음에도 불구하고 영속성 컨텍스트에 두번 들어가게 된다.  
그러므로 `@EmbeddedId`에 대해서는 꼭 equals, hashCode를 구현해줘야 한다.  

아래는 또 다른 사용 사례에 관한 것이다.  
<https://jojoldu.tistory.com/134>


<!-- more -->