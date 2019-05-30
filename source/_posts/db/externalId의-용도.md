---
title: externalId의 용도
date: 2018-12-16 21:23:44
tags:
    - externalId
---

# 사용방법
PK를 그냥 key로 사용할 경우 외부에서 마스터 데이터의 양을 어느정도 알 수 있기 때문에 externalId 컬럼을 따로 만들어주고, 이것을 외부에 노출시키는 것이 좋다.  

externalId는 PK와 동일하게 unique key가 되어야 한다(unique constraints 필요)  
내부적인 룰을 통해 이 값을 지정하거나 특정한 룰이 없다면 UUID로 생성한다  

# 주의  
externalId를 PK로 잡으면 클러스터드 인덱스의 특징상 미친듯한 재정렬이 일어나므로, 추가 컬럼으로 사용하는 것이 좋다.  

<!-- more -->