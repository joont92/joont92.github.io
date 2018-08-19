---
title: 'String, StringBuffer, StringBuilder'
date: 2018-08-14 21:02:47
tags:
    - String
    - StringBuffer
    - StringBuilder
---

셋 다 문자열 처리를 위한 클래스이나, 그 처리 방법에서 차이를 보인다.  
실제로 사용하는 상황에 따라 성능차이가 발생하니 이를 확실히 정리하고자 한다.  

# String
`String`은 기본적으로 변경이 불가능한 `immutable` 클래스이다.  
이말인 즉 `String`에 추가적인 연산을 하게 될 경우, 기존 `String` 클래스의 값이 변경되는 것이 아니라 항상 새로운 클래스가 생성된다는 뜻이다.  

```java
String str1 = "AAA" + "BBB";
String str2 = "CCC".concat("DDD");
```

`str1`의 경우 힙 영역에 `AAA`, `BBB`, `AAABBB`가 각각 생기게 되는 것이고,  
`str2`의 경우 힙 영역에 `CCC`, `DDD`, `CCCDDD`가 생기게 되는 것이다.  
`String`이 가지고 있는 각종 `mutable`해 보이는 연산(`substring()`, `toLowerCase()`) 등 은 모두 위와 같이 처리된다.  
> JDK 5.0 미만 버전 한정이다. JDK 5.0 이상부터는 `String` 연산도 내부적으로 `StringBuilder`로 변환된다.  

`String`을 이렇게 디자인 한 이유는 프로그램 기본 문자열 클래스로 사용하기 위해서이다.  
프로그램 작성 시 문자열을 생성하고 참조하는 경우는 많으나, 변경하는 일은 그리 많지 않다.  
이럴 경우 위와 같은 `immutable` 형태로 선언하게 되면 많은 효과를 누릴 수 있다.  
일단 `thread-safe` 하기 때문에 여러 쓰레드 내에서 자유롭게 참조할 수 있고,  
한번 생성한 문자열은 같은 문자열에 대해서는 변경을 가하지 않으므로,  
요청 시 똑같은 주소값을 반환해 주기 때문에 성능상으로 많은 이점을 누릴 수 있다.  
> 실제로 아래의 연산이 성립한다.  

```java
String str1 = "AAA";
String str2 = "AAA";
assertTrue(str1 == str2);
```

두 문자열이 같은 힙 영역을 공유하기 때문이다.  

# StringBuffer
이에 반해 `StringBuffer` 클래스는 `mutable` 클래스이다.  
위처럼 기존의 문자열에 추가적인 연산을 하게 되면, `String`처럼 새로운 문자열이 생성되는 것이 아니라 기존의 문자열에 추가적인 연산을 하게 된다.  
그러므로 당근 문자열의 연산이 많을 경우에는 `String` 클래스보다 훨씬 효율적이다.  
그러면 `아! 그럼 문자열 연산이 많을 때는 무조건 StringBuffer 써야겠구나!` 라고 할 수 있지만,  
`StringBuffer`는 `thread-safe`를 위해 내부적으로 `synchronized` 연산을 수행하게 되므로 문자열의 변경이 잦지 않을 경우는 `String` 보다 나쁜 성능을 보인다.  
그러므로 연산이 많을 경우에 사용하도록 하자~~  

# StringBuilder
JDK 5.0 부터 나왔다.  
`StringBuffer`와 동일하나 `thread-safe` 하지 않다는 점이 차이점이다.  
위에서 언급했듯이 JDK 5.0 이후로는 `String` 연산이 내부적으로 이 `StringBuilder`로 변환되어 처리된다.  

# 그럼 뭘 쓸까?
문자열 연산이 잦지 않은 경우는 `String`을 사용하는 것이 좋고,  
연산이 잦을 경우에는 `thread-safe` 여부를 따져서 `StringBuffer`나 `StringBuilder`를 사용하면 된다.  
어쩌피 JDK 5.0 이후로 연산 시 `String`이 `StringBuilder`로 변환된다지만, 문자열을 더할 떄 까지 객체를 계속 추가해야 한다는 사실은 변함이 없으므로, 연산이 많으면 `StringBuilder`나 `StringBuffer`를 사용하는 것이 좋다.  