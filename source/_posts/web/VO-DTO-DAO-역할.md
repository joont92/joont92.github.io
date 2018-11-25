---
title: 'VO,DTO,DAO 역할'
date: 2018-11-18 19:34:12
tags:
    - VO
    - DTO
    - DAO
---

논란이 많은 부분이라고 생각함  
이건 그냥 같이 일하는 분과 정한 방식일 뿐이다  

# 정의
VO : immutable한 객체  
DTO : 전송을 위한 객체. 값의 수정이 자유로움  
Entity : 데이터베이스와 직접적으로 관련있는 객체  

## Controller  
VO(Request, Response)를 사용  
서비스에 전달할때는 VO를 DTO로 변환해서 전달해줘야 함  
> POST는 CreateXXXRequest, PUT은 ChangeXXXRequest, PATCH는 UpdateXXXRequest로 사용하도록 함(최대한 분리시킴)  
> 하지만 현재는 POST 메서드가 많이 존재하지 않는 상황이라 CreateXXXRequest가 ChangeXXXRequest를 상속하는 구조로 만들어 두었다.  
> 이후로 POST 메서드가 늘어나면 분리할 생각이다.  
> 현재시점에서 하는건 뭔가 오버엔지니어링(?) 느낌..  

## Service  
DTO를 사용  
비즈니스 로직을 수행하며 DTO를 자유자제로 변환 가능.  
DAO에 전달할때는 Entity로 변환해서 전달해줘야 함  

## DAO(Repository)  
Entity를 사용  
실제 database 로직을 수행  

**각 레이어가 사용하는 도메인을 구분함으로써 의존성을 줄임**  

---

# 사용법
Controller에서 항상 VO를 사용(request, response)한다고 했지만,  
response 쪽에선 VO와 DTO가 매우 유사하면 그냥 DTO를 사용할 수도 있다.  

왜 response만 되고 request는 안되느냐 하면  
서버의 입장에서 Request는 매우 중요하므로, DTO를 사용해 일괄적으로 받게되면 side-effect가 발생할 확률이 커진다.(받지 말아야 할 데이터를 받아버린다거나)  
반대로 Response의 경우 정보를 더 내려준다고 해서 서버쪽에 side-effect가 발생할 확률이 거의 없다.  

사용자에게 불필요한 정보를 내려주느냐가 논점이 될 수 있는데, MSA 구조에서는 BFF가 추가적으로 데이터를 가공해서 클라이언트에게 제공하므로 MSA 구조에서는 이 문제를 어느정도 커버가 가능하다.  

<!-- more -->