---
title: MapStruct
date: 2018-12-16 20:39:25
tags:
    - '@Mapper'
---

# 기본 사용법  
- installation  
<http://mapstruct.org/documentation/stable/reference/html/#setup>  

- 기본 사용법  
<http://mapstruct.org/documentation/stable/reference/html/#basic-mappings>  

클래스간 변환을 간편하게 해주는 라이브러리이다(Car <> CarDTO)  
아래는 Mapper 선언법이다.  

```java
@Mapper
public interface CarMapper {
    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );
 
    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto toDto(Car car);

    @Mapping(target = "id", ignore = true)
    Car toEntity(CarDTO dto);
}
```

(지금은 엔티티와 DTO를 예시로 들었지만 어떤 클래스든 상관없다.)  
install한 library로 빌드하면 `@Mapper` 인터페이스들을 찾아서 `XXXImpl`의 형태로 구현체를 모두 만든다.(빌드 방식 알아봐야함)  

generate된 Impl은  
Car -> CarDTO의 경우 CarDTO를 new로 생성하고(인자없는 생성자),  
setter로 car의 값을 전부 CarDTO에 넣어주는 식으로 generate 된다.  

두 클래스의 모든 필드를 mapping 해주므로,  
두 클래스 필드들이 모두 이름이 같아야한다.  
그렇지 않을 경우 위처럼 `source, target`으로 이름을 변경해주거나,  
`ignore`로 무시할 필드들을 지정해줘야 한다.  

아래는 간단한 사용법이다.  

```java
// some service
public void save(CarDTO carDTO){
    Car car = CarMapper.INSTANCE.toEntity(carDTO);
    
    em.persist(car);
}

public CarDTO select(){
    Car car = em.find(Car.class, 1);

    return CarMapper.INSTANCE.toDTO(car);
}
```





<!-- more -->