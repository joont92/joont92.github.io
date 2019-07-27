---
title: 'swagger 2.0 문법'
date: 2018-11-18 18:53:52
tags:
    - swagger
    - swagger 2.0
    - swagger 문법
---

# 문법
## 기본 구조  
<https://swagger.io/docs/specification/2-0/basic-structure/>  

paths에 api url,  
definitions에 object를 정의해서 서로 사용 가능  

## data types
<https://swagger.io/docs/specification/data-models/data-types/>  

기본적으로 string, number, integer, boolean, array, object 타입을 가지며  
2.0에서 저 많은 옵션들을 다 지원하는지는 모르곘다.  
array는 항상 items 옵션을 가진다.  

## request  
<https://swagger.io/docs/specification/2-0/describing-parameters/>  

path, query, formData type들을 받을 수 있으며  
required, default, minimum 등 여러가지 옵션을 줄 수 있다.  

<https://swagger.io/docs/specification/2-0/describing-request-body/>  

body 형태의 파라미터도 받을 수 있다.  
path 파라미터를 제외하고 하나만 받을 수 있다.  

## response
<https://swagger.io/docs/specification/2-0/describing-responses/>  

response code, header, response data 등을 줄 수 있다.  
response data로 definitions에 정의한(definitions는 그냥 이름을 뿐임) object를 줄 수 있다.  

## enum
<https://swagger.io/docs/specification/2-0/enums/>  

기본적으로 enum은 string array 형태로 작성한다.  
그리고 definitions 처럼 따로 선언해서 재사용하게 할 수 있다.  
parameters에서는 `&ENUM`을 사용하지만 object 내에서는 `#/definitions/ENUM`형태로 선언해야 한다.  

```yaml
CategoryType:
    type: string
    enum: &CATEGORYTYPE
        - MAIN
        - SUB

CategoryDTO:
    title: CategoryDTO
    type: object
    properties:
        externalId:
            type: string
        type:
            enum: *CATEGORYTYPE
```

이런식으로 선언할 경우 code-gen에서 이상한 TypeEnum 형태의 inner enum을 generate 한다.  

<!-- more -->