---
title: 'PUT, PATCH의 차이'
date: 2018-11-18 19:52:27
tags:
    - rest put patch
---

### 관련 글
<https://stackoverflow.com/questions/28459418/rest-api-put-vs-patch-with-real-life-examples>  

PUT은 전체 엔티티를 전달해줘야하고, PATCH는 변경하고자 하는 속성만 전달해주면 된다.  
PUT에 전달한 엔티티에 일부 속성이 누락될 경우 해당 속성은 API 호출 후 값이 유실된다.  
PUT은 대체한다는 개념으로 보면 된다. 없으면 생성도 된다.  

<!-- more -->