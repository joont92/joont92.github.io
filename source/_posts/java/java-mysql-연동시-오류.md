---
title: [java] java mysql 연동시 오류
date: 2018-12-13 22:07:16
tags:
    - The server time zone value 'KST' is unrecognized or represents more than one time zone
    - public key retrieval is not allowed
---

# The server time zone value 'KST' is unrecognized or represents more than one time zone
<https://yenaworldblog.wordpress.com/2018/01/24/java-mysql-%EC%97%B0%EB%8F%99%EC%8B%9C-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EC%97%90%EB%9F%AC-%EB%AA%A8%EC%9D%8C/>

# public key retrieval is not allowed
mysql 8.x 버전 이후로 발생
jdbc url에 `allowPublicKeyRetrieval=true&useSSL=false` 필요

```
jdbc:mysql://localhost:3306/dev-product?useUnicode=true&characterEncoding=utf8&allowPublicKeyRetrieval=true&useSSL=false
```

<!-- more -->