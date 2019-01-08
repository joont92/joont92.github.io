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

> VO 끼리 상속하지 않도록 한다.   
> Request, Response의 경우 API의 요청과 결과를 담당하는 애들인데, 얘네들 끼리 상속하면 결국 api끼리도 의존성이 생기는 꼴이 될 수 있다.  
> 
> 네트워크 요청 횟수를 줄이기 위해 통합 api(한번에 insert)를 만든 경우에는 상속을 조금 사용했다.  

## Service  
DTO를 사용  
비즈니스 로직을 수행하며 DTO를 자유자제로 변환 가능.  
DAO에 전달할때는 Entity로 변환해서 전달해줘야 함  
> DTO끼리는 자유자재로 상속을 활용했다  
> 잘한짓인지는 모르겠다  

## DAO(Repository)  
Entity를 사용  
실제 database 로직을 수행  

**각 레이어가 사용하는 도메인을 구분함으로써 의존성을 줄임**  

---

# 사용
Controller에서 항상 VO를 사용(request, response)한다고 했지만,  
response 쪽에선 VO와 DTO가 매우 유사하면 그냥 DTO를 사용할 수도 있다.(사실 대부분 DTO를 그대로 내렸다)  

왜 response만 되고 request는 안되느냐 하면  
서버의 입장에서 Request는 매우 중요한데, DTO를 사용해 필요없는 필드까지 받게되면 side-effect가 발생할 확률이 커진다.(받지 말아야 할 데이터를 받아버린다거나)  
반대로 Response의 경우 정보를 더 내려준다고 해서 서버쪽에 side-effect가 발생할 확률이 거의 없다.  

사용자에게 불필요한 정보를 내려주느냐가 논점이 될 수 있는데(뭘 쓰라는거에요? 같은..),  
MSA 구조에서는 BFF가 추가적으로 데이터를 가공해서 클라이언트에게 제공하므로 MSA 구조에서는 이 문제를 어느정도 커버가 가능하다.  

우리는 MSA 구조이므로 DTO를 바로 Response로 주는 구조를 사용했다.  
(어쩌피 BFF에서 추가적으로 가공될 것이기 때문이다.)  
DTO외에 추가적인 정보가 필요할 경우 이를 Response로 감싸서 만들기도 했다.  
> ItemDTO list + page meta data = GetItemResponse  

<!-- more -->