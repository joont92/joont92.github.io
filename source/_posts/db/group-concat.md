---
title: [db] group_concat
date: 2019-03-12 12:08:29
tags:
    - mysql
    - group_concat separator
    - group_concat 구분자
---

지정한 컬럼을 구분자 묶어서 출력해줌  

```sql
select team, group_concat(id) from user group by team;
```

```
A_team | 1,3,5
B_team | 2,4,6
C_team | 7,8,9
...
```

기본 구분자는 `,`이고, 변경가능  

```sql
select team, group_concat(id separator '|') from user group by team;
```

앞뒤로 문자를 붙일수도 있음  
```sql
-- 1__, 2__, 3__
select team, group_concat(id, '__') from user group by team;

-- __1__, __2__, __3__ 
select team, group_concat('__', id, '__') from user group by team;
```

<!-- more -->