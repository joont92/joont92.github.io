---
title: MultipleBagException
date: 2019-01-31 18:58:51
tags:
    - queryDSL
    - JPQL
    - 1:N multiple join
---

JPQL은 기본적으로 1 -> N 방향으로 조인이 1번 까지밖에 안된다  
2번 이상하게 되면 `MultipleBagException` 이 발생한다.  

```java
// item아래 재고단위, 색상 등이 1:N으로 있다고 가정
query.join(item.stockUnits, stockUnit).fetchJoin()
    .join(item.colors, color).fetchJoin() // MultipleBagException 발생  
```

카테시안 곱이 일어나기 때문이라는데..(너무 많은 로우가 생기므로)  
@OneToMany 관계를 List가 아닌 Set으로 바꿔주면 위 에러가 발생하지 않는다  
<https://www.thoughts-on-java.org/hibernate-tips-how-to-avoid-hibernates-multiplebagfetchexception/>  

이유는 모르겠다 젠장..  

<!-- more -->