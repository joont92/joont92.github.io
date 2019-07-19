---
title: '[kubernetes] RBAC'
date: 2019-07-14 14:27:49
tags:
---

시크릿  
쿠버네티스 메모리에 암호화해서 저장해놓을 수 있는 환경변수(config map)  
Pod이나 Deployment에서 secretKeyRef 로 작성하면 Pod 내 환경변수등으로 전달할 수 있다  

사용자 어카운트, 서비스 어카운트  
별도 인증 시스템 없음. 외부 계정 시스템을 사용해야하며, 연동을 위해 OAuth나 Webhook 지원  

클라이언트가 쿠버네티스 API에 접근하고자 할 떄 사용되는것은 시스템 == 서비스 어카운트  

Bearer token?  


Role, Binding  
롤/롤바인딩, 클러스터롤/클러스터롤 바인딩  
> 네임스페이스 안에서만 유효한가, 클러스터 전체에서 유효한가?  
> 클러스터롤의 경우 접근할 수 있는 리소스의 범위가 더 크다  

Role에 리소스와 동작 지정  
RoleBinding의 subject에 종류(일반 사용자, 서비스 계정)와 정보를 입력하고, roleRef로 role과 연결시킴  

미리 만들어져있는 롤도 있다  