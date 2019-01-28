---
title: REST API Best Practice
date: 2018-12-16 18:40:07
tags:
    - REST
    - REST API example
---

# REST API 좋은 예제(읽어봐야함)
<https://developer.github.com/v3/>  
REST API는 Github API가 좋은 에제이다.  
구글이나 페이스북의 경우 사람이 많아서 오히려 관리가 잘 안되고 있댜.  
그에 비해 Github은 300명 정도로 운영하기 때문에, 좀 더 낫다고 함.  

# Rest API 좋은 가이드라인(읽어봐야함)
<https://allegro-restapi-guideline.readthedocs.io/en/latest/>  
<https://docs.microsoft.com/ko-kr/azure/architecture/best-practices/api-design>  

# 리스트에 대한 안티패턴  
```json
[
    {
        ...
    }
    ...
]

```

추가적인 정보는 대부분 헤더에 내려주긴 하지만, 가끔씩 헤더에 넣기 애매한 애들이 있다(page meta information 등).  
그런 애들은 응답 바디에 넣어주는 것이 좋은데, 위와 같이 api 설계를 하면 추가적인 정보를 내려줄 수 없게된다.  

그러므로 아래와 같이 해주는 것이 좋다.  
```json
{
    "data": [
        {
            ...
        },
        ...
    ]
}

```

<!-- more -->