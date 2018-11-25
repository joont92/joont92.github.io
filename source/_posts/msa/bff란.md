---
title: bff란?
date: 2018-11-18 17:26:40
tags:
    - bff
---

# 정의
<http://cyberx.tistory.com/101>

`backend for frontend`의 약자  
msa api들은 외부에 종속적이지 않은, 자신의 도메인에만 충실한 api를 작성하는데  
ui 쪽에서 이 api들을 조합해서 사용하기란 쉽지 않다.  
ui에서 사용하기 쉽도록 msa의 api들을 조합해주는, frontend와 msa api들의 사이에 있는 계층이다.  

# 구조
client(react, vue) - BFF - MSA APIs

<!-- more -->