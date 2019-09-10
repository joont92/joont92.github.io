---
title: '[db] jdbc'
date: 2018-01-13 18:00:52
tags:
    - java mysql
    - preparedstatement
    - sql injection
---

# JDBC  
java에서 mysql을 사용하기 위한 규약(?)  
JDBC는 그냥 껍데기(interface)일 뿐이고, DBMS 벤더들이 그 규약을 구현한 Driver 들을 제공한다.  
JDBC URL이 있다. 이것도 JDBC 규약에 있는 표준이다. jdbc:mysql:// 스트링은 고정이다.  

요즘은 커넥션 풀을 사용하므로 아래의 코드가 익숙하지는 않곘지만..  
Class.forName(driverClass).newInstance(); 를 통해 드라이버를 JVM으로 로드하고,  
DriverManager.getConnection()으로 커넥션을 얻는다.  
사용한 커넥션은 바로바로 반납해주는 것이 좋다.  

# Statement VS PreparedStatement
기본적으로 JDBC 프로그래밍을 하면 Statement가 실행되는 과정은 아래와 같다.  

```
요청 -> 쿼리 분석 -> 최적화 -> 권한요청 -> 실행  
```

쿼리 분석과 최적화에서 시간이 꽤 걸리는데,  
PreparedStatement는 이 과정을 미리 메모리에 저장해두고 다음 요청에서 바로 사용함으로써 시간을 많이 줄일 수 있게 된다.  
(PreparedStatement는 파라미터 바인딩을 지원하므로 완벽하게 똑같은 쿼리가 아니어도 상관없다.)  

또한 PreparedStatement는 전달되는 파라미터 앞 뒤로 SQL injection에 문제될 문자들을 검사하고 escape 처리하기 때문에 개발자가 직접 처리해줘야 하는 번거로움이 없다.  

<!-- more -->