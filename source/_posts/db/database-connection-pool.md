---
title: database connection pool
date: 2018-01-18 16:25:35
tags:
    - connection pool
---

### database connection pool 정의
<https://www.holaxprogramming.com/2013/01/10/devops-how-to-manage-dbcp/>

데이터베이스 작업에서 가장 비용이 많이 발생하는 부분은 커넥션을 생성하여 맺는 부분  
db 작업이 일어날 때 마다 이렇게 커넥션을 생성하고 반납하면 매우 낭비가 크므로,  
이러한 커넥션들을 미리 생성해서 풀에 담아두고 필요할 때 마다(db 요청이 있을 때 마다) 꺼내쓰도록 함  
이를 커넥션 풀이라고 하고, Apache Commons DBCP를 사용  

커넥션 풀 관련해서 여러가지 속성들(maxActive, maxIdle 등)이 있지만   
이것보다 실제 커넥션 최대 개수가 성능상 가장 중요한 이슈이다  

커넥션 개수를 설정하는 방법은  
DBMS가 수용할 수 있는 커넥션 개수를 측정한 다음에, WAS 하나가 사용할 커넥션 풀 개수를 구하면 된다.  
그리고 기본적으로 WAS의 Thread 수는 DBMS의 Connection 수보다 많아야 한다.  
모든 작업이 DB를 필요로 하진 않기 때문이다.  
쓰레드가 DBCP보다 한 10개 정도 많게 하면 적당하다.  

<!-- more -->