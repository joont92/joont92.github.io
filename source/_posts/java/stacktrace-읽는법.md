---
title: [java] stacktrace 읽는법
date: 2019-01-02 15:06:09
tags:
    - stacktrace
    - causedby
---

<https://okky.kr/article/338405>  

1. 첫번째 줄이 가장 마지막에 실행된 메서드  
2. 밑에 줄은 위의 메서드를 호출한 메서드  
3. CausedBy가 실제 발생한 에러 메세지임.  

```java
Exception in thread "main" java.lang.IllegalStateException: A book has a null property
        at com.example.myproject.Author.getBookIds(Author.java:38)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
Caused by: java.lang.NullPointerException
        at com.example.myproject.Book.getId(Book.java:22)
        at com.example.myproject.Author.getBookIds(Author.java:35)
        ... 1 more
```

이렇게 발생되었다면 보통 아래와 같이 구현된 것임  

```java
try {
....
} catch (NullPointerException e) {
  throw new IllegalStateException("A book has a null property", e)
}
```

<!-- more -->