---
title: '[java] interface에 static method'
date: 2019-03-14 23:30:38
tags:
    - interface static method
---

interface에 선언한 static method는 일반적으로 우리가 정의하는 메서드와는 다르다.  
1. body가 있어야 한다  
2. implements 한곳에서 override가 불가능하다  

```java
interface SomeInterface {
    public static String doSomething(){
        // blah blah
    }

    public String doNothing(); // have to override
}
```

마치 java8의 default method와 약간 비슷하다. 하지만 static method는 override가 불가능하다는 것이 특징이다.  

<!-- more -->