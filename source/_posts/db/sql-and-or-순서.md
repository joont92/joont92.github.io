---
title: sql and or 순서
date: 2019-10-11 00:31:12
tags:
  - and
  - or
---

다들 알다시피 SQL 에서 소괄호 없이 AND OR 을 사용하면 AND -> OR 의 순서로 처리된다  
그렇다면 아래와 같은 쿼리는 어떻게 처리되는걸까?  

```sql
select count(*) 
from employees 
where year(hire_date) = '1998' or year(hire_date) = '1999' and gender = 'M';
```

이 쿼리는 and 조건 먼저 실행되고 그다음 or 조건이 실행된다  
이 쿼리는 아래의 두 쿼리를 합친것과 동일하다  

```sql
select count(*) from employees 
where year(hire_date) = '1999' and gender = 'M';

select count(*) from employees 
where year(hire_date) = '1998';
```

> 결론은 그냥 알아보기 너무 힘드니까 입닥치고 괄호로 감싸주자  

<!-- more -->