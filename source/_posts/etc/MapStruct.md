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
@Mapper(componentModel = "spring", uses = {
    OwnerMapper.class // 3
})
public interface CarMapper {
    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );
 
    @Mapping(source = "numberOfSeats", target = "seatCount") // 1
    CarDto toDto(Car car);

    @Mapping(target = "id", ignore = true) // 2
    Car toEntity(CarDTO dto);
}
```

보다시피 entity를 DTO로 변환해주는 작업을 한다(지금은 엔티티와 DTO를 예시로 들었지만 어떤 클래스든 상관없다).  
아래의 generate 된 코드를 보면 알 수 있곘지만,  
변환될 클래스는 setter가 필요하고, 변환대상 클래스는 getter가 필요하다.  

1. source와 target의 필드 이름이 다를 경우 직접 지정할 수 있다  
2. ignore를 통해 특정 필드는 변환되지 않도록 설정할 수 있다. target만 신경쓰면 된다.  
3. 클래스에서 다른 클래스를 참조할 경우 다른 Mapper를 사용하여 변환시킬수도 있다. 이를 지정하지 않으면 좀 비효율적인 방법으로 변환을 시도한다.  

install한 library로 빌드하면 `@Mapper` 인터페이스들을 찾아서 `XXXImpl`의 형태로 구현체를 모두 만든다.(빌드 방식 알아봐야함)  
생성된 구현체는 아래와 같다. 간단하다.  

```java
@Override
public CarDTO toDto(Car entity) {
    if ( entity == null ) {
        return null;
    }

    CarDTO carDTO = new CarDTO();

    carDTO.setName( entity.getName() );
    carDTO.setSeatCount( entity.numberOfSeats() );
    carDTO.setOwner( ownerMapper.toDto(entity.getOwner() ) );

    return carDTO;
}
```

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