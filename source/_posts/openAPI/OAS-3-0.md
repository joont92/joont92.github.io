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
- array는 items가 필수로 와야함. items 아래 type 혹은 $ref

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
<https://swagger.io/docs/specification/describing-parameters/>  
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

## request body
<https://swagger.io/docs/specification/describing-request-body/>  
```yml
paths:
    /pets:
        post:
            summary: Add a new pet
            requestBody:
            description: Optional description in *Markdown*
            required: true
            content:
                application/json:
                schema:
                    $ref: '#/components/schemas/Pet'
    /pets:
        post:
            summary: Add a new pet
            requestBody:
                description: Optional description in *Markdown*
                required: true
                content:
                application/json:
                    schema:
                        type: object
                        properties:

```


# object 정의  
<https://swagger.io/docs/specification/components/>  
`components` 아래에 정의하고, 재사용을 목적으로 한다.  
`$ref` 속성을 이용하고 `#`으로 참조한다.  

```yml
$ref: '#/components/schemas/Item'
```

`#`는 현 위치를 말한다.  

## schemas
일반 오브젝트(DTO 등)  

## parameters
<https://swagger.io/docs/specification/describing-parameters/#common-for-various-paths>  

`Common Parameters for Various Paths` 부분 참조  

## responses
<https://swagger.io/docs/specification/describing-responses/>  

status 별로 선언가능하며 description, object 등을 내려줄 수 있다  

## enum reuse
```yml
paths:
  /products:
    get:
      parameters:
      - in: query
        name: color
        required: true
        schema:
          $ref: '#/components/schemas/Color'
      responses:
        '200':
          description: OK
          
components:
  schemas:
    Color:
      type: string
      enum:
        - black
        - white
        - red
        - green
        - blue
```

<!-- more -->