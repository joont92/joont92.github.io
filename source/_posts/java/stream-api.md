---
title: stream api
date: 2018-12-16 21:27:40
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

<!-- more -->