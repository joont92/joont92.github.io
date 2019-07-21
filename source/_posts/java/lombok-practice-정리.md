---
title: [java] lombok practice 정리
date: 2018-12-13 23:47:54
tags:
    - lombok
    - '@Builder'
---

# 상속 클래스에 builder 적용
<https://reinhard.codes/2015/09/16/lomboks-builder-annotation-and-inheritance/>  
또는 @SuperBuilder를 사용할 수 있음  
intellij 구조적 문제로 에러로 표시된다는 특징이 있다.. 컴파일은 잘 된다.  

# Builder에서 필수값, 선택값 구분하기  
빌더 패턴은 필수값과 선택값을 구분할 수 있다는 장점도 가지고있는데, 클래스 위에 그냥 `@Builder`를 선언하면 그 장점을 누리지 못하게 된다.  
아래와 같이 작성해서 필수값, 선택값을 구분하게끔 할 수 있다.  

```java
@Builder
@AllArgsConstructor
public class Member {
    private String name;
    private Integer age;
    private String nickname;
    private String address;

    public static MemberBuilder builder(String name, Integer age){
        return new MemberBuilder()
            .name(name)
            .age(age);
    }
}
```

사용은 아래와 같이 할 수 있다.  

```java
Member.builder("joont",28)
    .address("somewhere in seoul")
    .build();
```

<!-- more -->