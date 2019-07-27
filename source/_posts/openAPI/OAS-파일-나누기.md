---
title: 'OAS 파일 나누기'
date: 2019-02-12 15:36:55
tags:
    - OAS file split
    - swagger file split
---

참고로 이 방식은 OAS 3.0 이상에서만 된다.  

방식은 간단하다. `#` 앞에 파일 경로만 써주면 된다.  

```yml
# api.yml
items/{itemId}:
    get:
        operationId: getItem
        summary: 아이템 조회
        parameters:
            - $ref: 'items.yml#/parameters/itemId'
        responses:
            '200':
                description: OK
                content:
                    application/json:
                        schema:
                            $ref: 'item.yml#/schemas/Item'

# item.yml
parameters:
    itemId:
        in: path
        name: itemId
        required: true
        schema:
            type: integer
schemas:
    Item:
        title: Item
        type: object
        properties:
            # ...
```

- 파일명 뒤에 `#`만 붙여주면 바로 접근 가능하다.  
- item.yml은 api.yml과 같은 위치에 있다(상대경로로 접근했음)  
- 폴더로 나누고 `definitions/item.yml#schemas/Item` 의 형태로 선언해도 된다.  

보다시피 `item.yml` 파일에 최상위 레벨이 없다. 없어도 되기 때문이다.  
단일파일에 작성할 때 `components` 아래 작성했던 부분을 파일로 나눴다고 생각하면 된다.  

<!-- more -->