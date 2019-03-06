---
title: java8 날짜/시간 api
date: 2019-03-06 00:48:20
tags:
---

Date 에 문제가 많았음  
> UTC가 아닌 CET 기준이라던가, 0부터 시작하는 월 이라던가  

그래서 Calendar가 나왔는데, 여전히 문제가 있음  
> 0 부터 시작하는 월 은 변경되지 않음. 대부분의 날짜 유틸이 Date 위주임  

그래서 jodaTime을 대부분 많이 사용했는데, 자바 8 부터 jodaTime의 유용한 기능들을 java.time 패키지에 넣어서 새로 배포함  

# LocalDate, LocalTime, LocalDateTime  
여기서 불변객체라 함은 setter 등으로 변경할 수 없는 객체를 말한다  
LocalDate는 날짜를 표현하는 불변객체  
LocalTime은 시간을 표현하는 불변객체  
LocalDateTime은 둘을 합쳐서 표현  

`of` static method로 연,월,일,시간 등을 받아서 생성할 수 있음  
`now` 메서드로 현재 시간 생성 가능  
`parse`로 문자열을 그대로 받아 사용 가능. 여기에는 DateTimeFormatter 인스턴스를 전달할 수도 있음  

# ZonedDateTime
<http://www.daleseo.com/java8-zoned-date-time/>  
LocalDateTime에 타임존이나 시차가 추가되었다고 보면 된다  
ZonedId나 ZonedOffset 값을 주면 타임존이나 시차를 적용할 수 있고, 값을 주지 않을 경우 로컬의 기본 타임존 값을 사용한다.  

> ZonedId는 타임존, ZonedOffset은 시차이다.  
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

# Instant  
컴퓨터가 알아보기 쉽게 표현하기 위한 형태이다  
유닉스 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC)를 기준으로 특정 지점까지를 초로 표현한 것이다.  
나노초(10억분의 1)까지 표현 가능하다  

ZonedId를 이용해서 LocalDateTime을 Instant로 바꿀 수 있다  
```java
Instant instant = LocalDateTime.now().toInstant(seoulZone);
```

Period, Duration  


<!-- more -->