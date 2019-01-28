---
title: OAS 3.0
date: 2019-01-28 17:29:22
tags:
    - swagger 3.0
---

# document 문서  
git : <https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md>  
swagger : <https://swagger.io/docs/specification/basic-structure/>  


# meta data, server 정보  
```yml
openapi: 3.0.0
info:
  title: Sample API
  description: Optional multiline or single-line description in [CommonMark](http://commonmark.org/help/) or HTML.
  version: 0.1.9
servers:
  - url: http://api.example.com/v1
    description: Optional server description, e.g. Main (production) server
  - url: http://staging-api.example.com
    description: Optional server description, e.g. Internal staging server for testing
```

# data type  
## 기본 타입  
<https://swagger.io/docs/specification/data-models/data-types/>  
- string, number, integer, boolean, array, object 사용가능  
- string 은 date와 file을 포함함

## 

# api 정의  
## path 정의  
`paths` 아래에 정의한다.  

```yml
paths:
  /users:
    get:
        operationId: getUsers
        summary: get user
        description: get user description
    post:
        operationId: createUser
        summary: create user
        description: create given user
  /users/{id}:
    get:
        ...
    patch:
        deprecated: true
```

- path 아래 http method 별로 선언 가능
- summary랑 description 차이가 뭐지?  
- operationId는 나중에 code-gen 에서 메서드명으로 사용됨  
- path templating 할 수 있고, 파라미터에서 받을 수 있음  
- deprecated 할 수 있음  

## parameter 정의  
기본 형태는 아래와 같음  

```yml
paths:
  /users/{id}:
    get:
      parameters:
        - in: path
          name: id  # 상단의 path명과 같아야함
          required: true
          schema:
            type: integer
            minimum: 1
          description: The user ID
```

- 파라미터 형태, 이름, 필수여부 등을 줄 수 있음. 추가적인 정보는 [여기](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#parameterObject) 확인  
    - `in`에는 `path, query, header, cookie` 가 올 수 있음  
    - path의 경우 url에 path templating과 이름을 동일하게 해줘야 한다는 점 빼고는 모두 동일하게 사용 가능  
- schema에서 전달받은 파라미터에 대해 정의함. 타입, 최소값 등을 줄 수 있음. 추가적인 정보는 [여기](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#schemaObject) 확인  


# object 정의  
`components` 아래에 정의한다.  

## schemas
## 공통 parameter 정의 가능


3. Swagger 상속
ItemParameter:
    allOf:
        - $ref: '#/definitions/PageParameter’
        - type: object
          properties:
              name:
                  type: string
              status:
                  $ref: '#/definitions/ItemStatusCode'
              shipping_type:
                  $ref: '#/definitions/ShippingType'
allOf를 사용하면 상속 구현이 가능하다!
4. swagger global consumes 무시하는법
swagger는 global로 consumes가 선언되어 있어도 GET, DELETE에 대해서는 consumes를 붙이지 않는다.
하지만 파라미터로 오브젝트타입(@RequestBody)를 받으면 consume를 그대로 적용하게 된다.
이러면 @RequestBody -> @ModelAttribute 오버라이딩을 하더라도 검색용으로 사용할 수 없게 된다. (GET은 body가 없으니까)
이는 사용하는 api 쪽에서 wildCard를 줘서 해결할 수 있다.(빈 공백은 무시함)
items:
    get:
        operationId: getItems
        description: 상품 목록 조회
        consumes:
            - '*'
     parameters:
            - ...

<!-- more -->