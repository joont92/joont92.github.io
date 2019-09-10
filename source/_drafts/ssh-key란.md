---
title: 'ssh key란?'
date: 2019-06-15 18:01:58
tags:
---

ssh 접속시 키 암호화 방식을 사용한다  
public key를 서버에 .ssh/authorized_keys 에 저장해두면 외부에서 public key를 가지고 접속할 수 있다  
ssh key는 유닉스 계열일 경우 ssh-keygen 명령으로 생성할 수 있다  
id_rsa 가 private key고, id_rsa.pub가 public key이다  

암복호화는 어떻게 할까?  
ssh 통신을 하려면 무조건 ssh key 를 생성해야 하는것인가?
그냥 로그인으로 할수도 있지 않나?  
보낼떄 private key로 암호화하고, 받는쪽에서 public key로 복호화하는 것인가?
그렇다면 서버에는 무조건 public key가 저장되어 있어야 하지 않는가?

known_hosts 이 파일은 뭔가?  

.htaccess, .htpasswd ??  

ssl, https

<!-- more -->