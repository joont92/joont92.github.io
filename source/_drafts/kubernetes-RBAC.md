---
title: '[kubernetes] RBAC'
date: 2019-07-14 14:27:49
tags:
---

일반 사용자, 서비스 계정  

Role, Binding  
롤/롤바인딩, 클러스터롤/클러스터롤 바인딩  
> 네임스페이스 안에서만 유효한가, 클러스터 전체에서 유효한가?  

Role에 리소스와 동작 지정  
RoleBinding의 subject에 종류(일반 사용자, 서비스 계정)와 정보를 입력하고, roleRef로 role과 연결시킴  

