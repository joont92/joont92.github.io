---
title: '[jpa] 엔티티 값을 변환해서 저장하기(@Converter)'
date: 2019-02-09 01:39:07
tags:
    - '@Converter'
    - AttributeConverter
---

`@Converter`를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장할 수 있다.  
예를 들면 엔티티에는 boolean, 데이터베이스에는 YN 값을 저장하고 싶을 경우 정도가 있겠다.  
(기본적으로 boolean 타입으로 지정하면 0,1로 저장된다)  

# 기본 사용법
```java
@Entity
class Member{
    @Id @GeneratedValue
    private Integer id;

    @Convert(converter=BooleanToYNConverter.class)
    private boolean useYn;
}

@Converter
class BooleanToYNConverter implements AttributeConverter<Boolean, String>{
    @Override
    public String convertToDatabaseColumn(Boolean attribute){
        return (attribute != null && attribute) ? "Y" : "N";
    }

    @Override
    public Boolean convertToEntityAttribute(String dbData){
        return "Y".eqauls(dbData);
    }
}
```

보다시피 간단하다.  
`AttributeConverter`를 구현하면서 제네릭으로 `엔티티 컬럼 타입, 데이터베이스 컬럼 타입`을 주고,  
아래 2개의 메서드를 오버라이드 해주면 된다(엔티티->데이터베이스, 데이터베이스->엔티티)  

# 클래스 레벨 설정
클래스 레벨에도 설정할 수 있다. 단 이때는 attrbuteName 속성을 사용해서 어떤 필드에 컨버터를 적용할 지 명시해야한다.  

```java
@Entity
@Converter(converter = BooleanToYNConverter.class, attributeName = "useYn")
class Member{
    // ...
}
```

# 글로벌 설정
모든 Boolean 타입에 설정하고 싶을 경우 아래와 같이 직접 컨버터에 명시해주면 된다.  

```java
@Converter(autoApply = true)
class BooleanToYNConverter implements AttributeConverter<Boolean, String>{
    // ...
}
```

이렿게 하면 엔티티에 따로 컨버터를 지정해주지 안항도 자동으로 컨버팅 된다.  

<!-- more -->