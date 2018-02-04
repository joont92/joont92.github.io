---
title: Session 동작방식
date: 2018-02-03 16:50:55
tags:
---

### 개념
http 프로토콜은 매 접속마다 새로운 요청이 이뤄지는, 상태를 저장하지 않는 stateless 형식의 프로토콜이다.  
그래서 서버쪽에서 클라이언트의 상태를 기억하기 사용하는 것이 session이다.  
서버는 클라이언트의 정보를 내부에 저장하고, 이 저장된 정보에 접근하기 위해 세션 ID(JSESSIONID)를 이용한다.  
여기에는 여러가지 정보(사용자 정보 등)를 담을 수 있고, 이 정보는 사용자가 일치하는 세션 ID를 가지고 들어오지 않는 한, 설정한 session timeout 만큼 유지된다.  
쿠키와 달리 서버쪽에 저장되는 정보이므로 보안에 좀 더 안전하다.  

### FLOW
아래는 session의 요청 방식을 잘 보여주는 그림이다.  
![image](https://user-images.githubusercontent.com/18513953/35765190-489a7a4e-0902-11e8-9cd0-fe4e264ee831.png)  
1. 클라이언트가 처음 서버로 http요청을 시도한다.
2. 서버쪽에서는 전달받은 세션 ID(JSESSIONID)가 없으므로 생성한다.  
    > 1. 
    > 2. 저장되는 쿠키는 세션종료시 같이 소멸되는 Memory cookie가 사용된다.(expire date가 session)
3. 생성한 세션 ID를 쿠키에 넣어 클라이언트에게 전달된다. 여기서 만약 클라이언트가 쿠키를 사용하지 않을 경우에는 URL에 붙여서 전달한다. ex) http://joont92.github.io;JSESSIONID=~~  
> 서버입장에서는 JSESSIONID를 전달받는 방식에서(request header냐 URL이냐) 사용자가 쿠키를 사용하느냐 안하느냐의 여부를 알 수 있다.  
초반에는 어디서든 JSESSIONID를 전달받지 않으므로 두 가지 방식으로 모두 보내본다.
3. 클라이언트쪽에서는 받은 세션 ID를 쿠키에 저장하고(), 앞으로 요청마다 해당 세션 ID를 request header에 전달하게 된다.
4. 이후 요청은 3번에서 얘기헀듯이 세션 ID를 계속 request header에 넣어서 전달하게 되고, 서버는 해당하는 세션 ID에 맞는 정보를 내려주며 상태를 유지할 수 있게 되는것이다.