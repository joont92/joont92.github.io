---
title: Naming Convention
date: 2018-11-25 21:31:57
tags:
    - Naming Convention
---

# API operation id
`action + resources`로 함  
복수형, 단수형은 꼭 구분하도록 함  

목록(GET)
> getItems, listItems

상세(GET)
> getItem, detailItem  

등록(POST)
> CreateItem

대체(PUT)
> ChangeItem

수정(PATCH)
> UpdateItem

삭제(DELETE)
> deleteItem

# VO name
API operation id와 동일하게 HTTP method에 맞춰 지정한다.  

request
> CreateItemRequest  
> ChangeItemRequest

response
> 일반적으로 DTO를 쓸 예정이나 혹시나 response가 필요할 때 사용한다.  
> 기존 목록에 추가적인 정보가 필요할 때 사용할 수 있다(e.g. page meta data)  
>
> getItemsResponse

<!-- more -->