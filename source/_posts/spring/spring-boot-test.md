---
title: spring boot test
date: 2018-12-16 21:29:37
tags:
---

# 참조하는 application 파일  
`test/resources/config/application.yml` 을 디폴트로 참고한다.

# 트랜잭션 auto rollback
JUnit은 오토 롤백이 디폴트이므로 테스트 메서드에서 db 변경 작업이 있더라도 테스트가 끝나면 항상 롤백한다.  
테스트가 실제 디비에 영향을 주면 안되기 때문이다.  

```java
@Test
@Rollback(false)
public void insetTest(){
    // ...
}
```

위와 같이 설정해서 rollback 하지 않도록 변경할 수 있다.  

<!-- more -->