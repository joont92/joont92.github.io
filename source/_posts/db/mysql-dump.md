---
title: mysql dump
date: 2018-04-10 13:02:20
tags:
    - mysql dump
    - mysql 덤프
---

# 덤프뜨기

## 전체 database 덤프
```
# -A 또는 --all-databases 옵션
mysqldump -u[아이디] -p[패스워드] -A > 경로/파일명.sql
mysqldump -u[아이디] -p[패스워드] --all-databases > 경로/파일명.sql
```

## 특정 database 또는 특정 table만 덤프
```
# 특정 database
mysqldump -u[아이디] -p[패스워드] [database명] > 경로/파일명.sql

# 특정 table
mysqldump -u[아이디] -p[패스워드] [database명] [table] > 경로/파일명.sql
```

## 테이블 스키마만 덤프
데이터 없이 테이블 구조(스키마)만 받을 때 사용한다  

```
# -d 또는 --no-data 옵션
mysqldump -u[아이디] -p[패스워드] -d [database명] > 경로/파일명.sql
mysqldump -u[아이디] -p[패스워드] --no-data [database명] > 경로/파일명.sql
```

## 특정 조건으로 덤프
특정 database의 특정 table에서 원하는 값만 덤프받고 싶을 경우 사용한다.  
ex) `test` db의 `employees` 테이블에서 `emp_no`이 `1` 이상 `10`이하인 값만 덤프를 받고자 할 때  

```
# -w 옵션을 사용한다. 조건은 ''로 묶어줘야 한다
mysqldump -u[아이디] -p[패스워드] [database명] [table명] -w 'emp_no >= 1 and emp_no <= 10'
```

# 복구하기
```
mysql -u[아이디] -p[패스워드] [데이터베이스명] < 덤프파일명.sql
```

덤프파일내에 database 생성 구문이 있을 경우 지정된 데이터베이스는 무시된다.  

<!-- more -->