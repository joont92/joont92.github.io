---
title: [jpa] 한 컬럼에 여러 엔티티 저장하기(feat.@Any)
date: 2019-02-17 20:49:25
tags:
    - '@ManyToOne multiple entities'
    - column polymorphic
    - '@Any'
    - '@AnyMetaDef'
    - '@AnyToMany'
---

여러 테이블에 공용으로 저장하는 테이블을 만들어야 한다고 가정해보자.  

여러 엔티티라고 해서 리스트를 얘기하는 것이 아니라, 다형성을 얘기하는 것이다.  

<http://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#mapping-column-any>  

<https://www.concretepage.com/hibernate/hibernate-any-manytoany-and-anymetadef-annotation-example>  

<!-- more -->