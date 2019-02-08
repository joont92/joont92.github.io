---
title: mysql port 파라미터 안될 때  
date: 2019-02-08 14:12:58
tags:
    - mysql port doesn't work
---

docker로 3307 포트로 mysql을 띄웠는데 포트 변경해서 접속하는 옵션인 -P 를 아무리 줘도 무시되는 상황이 발생했다.  

찾아보니 -h 가 localhost일 경우 소켓을 사용해서 포트 옵션이 무시되므로, 127.0.0.1을 써줘야 한다고 한다  
<https://serverfault.com/questions/306421/why-does-the-mysql-command-line-tool-ignore-the-port-parameter/306423?newreg=1c57059c005942bfbf0735b86bc570d6>  

```
mysql -u root -h 127.0.0.1 -P 3307  
```

<!-- more -->