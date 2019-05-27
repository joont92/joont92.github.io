---
title: java8 날짜/시간 api
date: 2019-03-06 00:48:20
tags:
    - java date
    - LocalDate
    - LocalTime
    - LocalDateTime
    - ZonedDateTime
    - Instant
---

UTC : <https://ko.wikipedia.org/wiki/%ED%98%91%EC%A0%95_%EC%84%B8%EA%B3%84%EC%8B%9C>  
ISO 8601, 표기법 : <https://ohgyun.com/416>

# 기존 자바 시간 API의 문제
java의 기존 Date 에 문제가 많았음  
- 불변 객체가 아니다
- 상수를 남용한다(e.g. `월` 파라미터에 1~12가 아닌 값이 들어가도 문제없음. 상수이기 때문이다)
- `월`이 상수 0부터 시작한다

그래서 Calendar가 나왔는데, 여전히 문제가 있음  
- 대부분의 날짜 유틸이 Date 위주임  
- Date로 변하기 위한 중간객체 수준밖에 안됨(생성 비용도 비쌈)
- 0 부터 시작하는 월 은 변경되지 않음

좋은 API는 오용하기 어려워야 하고, 문서가 없어도 쉽게 사용할 수 있어야 한다  
그러나 Java의 기본 API는 문서를 열심히 보기 전까지는 제대로 사용하기 어렵다  

그래서 jodaTime을 대부분 많이 사용했는데, 자바 8 부터 jodaTime의 유용한 기능들을 java.time 패키지에 넣어서 새로 배포함  

이와 관련된 자세한 내용은 아래의 글에 나와있다  
<https://d2.naver.com/helloworld/645609>  

# java 8 시간 API
상세하게 설명되어 있는 블로그  
<https://perfectacle.github.io/2018/09/26/java8-date-time/>  

## LocalDate, LocalTime, LocalDateTime  
Timezone을 가지지 않는 시간  

여기서 불변객체라 함은 setter 등으로 변경할 수 없는 객체를 말한다  
LocalDate는 날짜를 표현하는 불변객체  
LocalTime은 시간을 표현하는 불변객체  
LocalDateTime은 둘을 합쳐서 표현  

`of` static method로 연,월,일,시간 등을 받아서 생성할 수 있음  
`now` 메서드로 현재 시간 생성 가능  
`parse`로 문자열을 그대로 받아 사용 가능. 여기에는 DateTimeFormatter 인스턴스를 전달할 수도 있음  

## ZonedDateTime
<http://www.daleseo.com/java8-zoned-date-time/>  
LocalDateTime에 타임존이나 시차가 추가되었다고 보면 된다  
ZonedId나 ZonedOffset 값을 주면 타임존이나 시차를 적용할 수 있고, 값을 주지 않을 경우 로컬의 기본 타임존 값을 사용한다.  

> ZonedId 이 부모 클래스, ZonedOffset, ZonedRegion 이 하위 클래스임  
> 시차를 쓰는 곳은 많지않고, ZonedOffset 같은 경우 Summer time 등을 처리하지 못하므로 ZonedId를 쓰는 것이 좋다  
> `ZoneId seoulZone = ZoneId.of("Seould/Asia")`  

LocalDate, LocalDateTime, Instant를 ZonedDateTime으로 변환할 수 있다  

```java
LocalDate localDate = LocalDate.now();
ZonedDateTime zd1 = localDate.atStartOfDay(seoulZone);

LocalDateTime localDateTime = LocalDateTime.now();
ZonedDateTime zd2 = localDateTime.atZone(seoulZone);

Instant instant = Instant.now();
ZonedDateTime zd3 = instant.atZone(seoulZone);
```

이것보다는 OffsetDateTime이 더 선호된다고 함  

## Instant  
컴퓨터가 알아보기 쉽게 표현하기 위한 형태이다  
유닉스 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC)를 기준으로 특정 지점까지를 초로 표현한 것이다.  
나노초(10억분의 1)까지 표현 가능하다  

ZonedId를 이용해서 LocalDateTime을 Instant로 바꿀 수 있다  
```java
Instant instant = LocalDateTime.now().toInstant(seoulZone);
```

## Period, Duration  


<!-- more -->