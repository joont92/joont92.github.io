---
title: 'Swagger와 OAS의 관계'
date: 2019-01-08 18:57:06
tags:
    - swagger
    - OAS
    - difference between swagger and OAS
---

# 누가 이 글에 잘못된 것좀 알려주세요

원래는 기업별로 각각 OpenAPI를 제공하는 명세가 있었다.  
(swagger를 만든 smart bear도 자신들의 OpenAPI specification이 있었다)  

2015년에 이 모든 기업들이 OpenAPI initiative 라는 그룹으로 통합(가입?)하었고, 여기서 공통된 spec인 Open API specification을 제공한다.  
<https://github.com/OAI/OpenAPI-Specification>  

그러므로 swagger도 문서에 보면  
2.0일때는 상단에 `swagger: "2.0"` 이었는데, 3.0부터 `openapi: "3.0"`으로 변경되었다.  
(근데 통합인데 왜 3.0부터 가는건지.. swagger가 주도하나..)  

<!-- more -->