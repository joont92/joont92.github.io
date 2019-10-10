---
title: '[db] where과 on절 차이'
date: 2018-01-02 23:01:54
tags:
    - where
    - 'on'
---

아래의 두 문장은 어떻게 다를까?  

```sql
select count(*) 
from dept_emp de
left outer join departments d
on de.dept_no = d.dept_no
where d.dept_name = 'Development';

select count(*) 
from dept_emp de
left outer join departments d
on de.dept_no = d.dept_no and d.dept_name = 'Development';
```

select 절의 처리 순서를 찾아보면, where 이 on 보다 먼저 실행되는 것을 볼 수 있다  

첫번째 문장의 경우  
부서명이 Development 인 부서만을 들고온 상태에서 dept_emp 테이블과 조인하므로  
부서명이 Development 가 아닌 부서는 결과에 포함되지 않는다  
즉 `필터 후 조인` 이다  

두번째 문장의 경우  
모든 부서를 들고온 후 전달된 조건(부서번호가 같고 부서명이 Development)으로 조인하는 것이므로  
dept_emp 의 로우가 departments 의 `모든 로우`를 돌면서 결과를 찾게 된다  
이 상태에서 조인의 형태가 left join 이므로 dept_emp 의 모든 데이터가 다 남게된다  
이 쿼리의 결과로는 부서명이 Development 가 아닌 부서도 결과에 포함된다  

이렇게 둘 간의 처리 방식의 차이가 있기 때문에  
left join 시 결과가 다르게 나오는 상황이 발생한다(inner join 시에는 같다)  

아무래도 join 은 on 절에 명시된 조건으로 driving table 이 driven table을 모두 체크하는 구조이다 보니, 첫번째 쿼리처럼 where 로 driven 테이블의 로우를 줄여주고 시작하는 것이 좋다  

<!-- more -->