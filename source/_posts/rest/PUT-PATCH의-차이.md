---
title: 'PUT, PATCH의 차이'
date: 2018-11-18 19:52:27
tags:
    - rest put patch
---

# 정의
<https://stackoverflow.com/questions/28459418/rest-api-put-vs-patch-with-real-life-examples>  

PUT은 전체 엔티티를 전달해줘야하고, PATCH는 변경하고자 하는 속성만 전달해주면 된다.  

```json
// PUT
{
    "name" : "joont",
    "age" : 27,
    "sex" : "male",
    // send all data
}

// PATCH
{
    "name" : "joont92" // send only data you want to change
}
```

PUT에 전달한 엔티티에 일부 속성이 누락될 경우 해당 속성은 API 호출 후 값이 유실된다.(매우 중요)  
PUT은 대체한다는 개념으로 보면 된다. 없으면 생성도 된다.  

# 사례  
```
API : item/options
content : option list
행위 : 기존 item에 있는 option들을 전부 지우고, 전달받은 option들로 전부 다시 인서트함
```

위처럼 자식의 내용을 다 지우고 다시 인서트 하는 형태의 API의 경우 PUT이 좀 더 바람직하다.  
`교체`의 개념과 딱 맞기 떄문이다.  

근데 여기서 전달받은 options의 element에 식별자가 있으면 update 한다고 할 경우, 이건 PUT이 맞을까?  

<!-- more -->