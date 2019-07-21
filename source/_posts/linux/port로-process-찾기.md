---
title: [linux] port로 process 찾기
date: 2019-03-13 12:46:46
tags:
---

<https://stackoverflow.com/questions/3855127/find-and-kill-process-locking-port-3000-on-mac>  
mac, centos7 에서 아래의 명령어로 찾을 수 있다고 함

```sh
netstat -vanp tcp | grep 9999
```

뒤에서 4번째 컬럼이 process 번호이다  

<!-- more -->