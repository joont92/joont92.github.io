---
title: [java] stream example
date: 2019-01-22 17:04:45
tags:
---

# stream 도는 오브젝트에 추가적인 행위를 하기
```java
list.stream()
    .map(mapper::toEntity)
    .peek(e -> e.setParent(parent))
    .forEach(childRepository::save);
```

peek을 이용하여 중간중간 메서드들을 호출 가능하다.  

# Stream을 이용하여 노출순서 정렬하기  
```java
AtomicInteger i = new AtomicInteger(0);
SomeDTOList.stream()
    .sorted(Comparator.comparingInt(SomeDTO::getDisplayOrder))
    .peek(dto -> dto.setDisplayOrder(i.incrementAndGet()))
    .peek(dto -> {
        AtomicInteger j = new AtomicInteger(0);
        dto.getChildren().stream()
            .sorted(Comparator.comparingInt(SomeChildDTO::getDisplayOrder))
            .forEach(c -> c.setDisplayOrder(j.incrementAndGet()));
    })
    .map(itemOptionMapper::toEntity)
    .forEach(repository::save);
```

# 자식들의 속성들까지 들고와 하나의 리스트로 합치기  
```java
Stream.of(category)
    .flatMap(c -> c.getChildCategories().stream())
    .map(Category::getId)
    .collect(Collectors.toList())
```



<!-- more -->