---
title: stream example
date: 2019-01-22 17:04:45
tags:
---

1. Stream을 이용하여 노출순서 정렬하기  
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

<!-- more -->