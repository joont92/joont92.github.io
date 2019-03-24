---
title: java list array 변환
date: 2019-03-09 03:27:17
tags:
---

# list -> array
```java
list.toArray(new String[0]);
```
toArray의 인자로 변환하고 싶은 array 타입의 변수를 성생해주면 된다.  
java 1.6 이전에는 인자로 생성하고 싶은 array 개수만큼 사이즈를 주는게 좋았으나,  
1.6 이후로는 0으로 주나 `new String[list.size()]` 하나 동일하다고 한다  

# stream -> array
```java
Stream.toArray(String[]::new)
```

<!-- more -->