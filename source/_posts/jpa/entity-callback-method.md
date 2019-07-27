---
title: '[jpa] entity callback method'
date: 2019-01-25 12:15:59
tags:
    - '@PostLoad'
    - '@PrePersist'
    - '@PostPersist'
    - '@PreUpdate'
    - '@PostUpdate'
---

<http://www.thejavageek.com/2014/05/23/jpa-lifecycle-callback-methods/>
lifecycle call method

<http://www.thejavageek.com/2014/05/24/jpa-entitylisteners/>
클래스에 적용하는 법

class 단위 범용적인 life cycle을 적용하고 싶다면
EntityListener class 사용  

- entity에 접근할 수 있어야 하므로 callback method에서 단일파라미터로 엔티티를 받을 수 있다
- 엔티티리스너 클래스는 public no args 생성자가 있어야 한다  

엔티티리스너를 엔티티에 붙이려면 아래와 같이 해야함  
- @EntityListeners 어노테이션을 엔티티에 선언해줘야 함. 여러개 붙일 수 있음  
- 라이프사이클 이벤트가 발생하면 @EntityListeners에 선언된 애들 순서대로 생성되고 실행된다  
- 이벤트리스너 내의 callback 메서드를 실행하면서 entity를 리스너에 전달한다(콜백 메서드가 있다면)  



<!-- more -->