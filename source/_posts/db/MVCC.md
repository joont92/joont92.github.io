---
title: MVCC
date: 2019-01-06 16:36:05
tags:
    - Real MySQL
    - Multi Version Concurrency Control
---

Multi Version Concurrency Content 의 약자이며, Multi Version이라 함은 하나의 레코드에 대해 여러 버전이 관리된다는 의미이다.  
일반적으로 레코드 레벨의 트랜잭션을 지원하는 DBMS가 제공하는 기능이며, 가장 큰 목적은 잠금을 사용하지 않는 일관된 읽기를 제공하는데 있다.  
(읽는 레코드에 락이 걸리게 되면 외부에서 수정이 불가능하기 때문에 Multi Version으로 관리할 필요가 없기 때문)  
MySQL은 `언두 로그`를 이용해 이 기능을 구현한다.  

가령 아래와 같은 업데이트 문을 실행하고,  

```sql
UPDATE member SET area = '경기' WHERE id = 12
```

아직 COMMIT이나 ROLLBACK을 하지 않은 상태에서 사용자가(다른 트랜잭션에서) 아래와 같이 조회하면 어떻게 될까?  

```sql
SELECT * 
FROM Member
WHERE id = 12
```

정답은 MySQL 초기화 파라미터에 설정된 `격리 수준에 따라 다르다`.  
격리 수준이 `READ_UNCOMMITED`라면 버퍼 풀이나 데이터 파일로부터 데이터를 읽어서 반환하지만,  
격리 수준이 `READ_COMMITED` 이상이라면 버퍼 풀이나 데이터 파일에 있는 데이터를 읽는 대신에  
변경 이전의 내용을 보관하고 있는 `언두 로그` 영역의 데이터를 반환한다.  

![언두 로그](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTMwNDAyOTg4NjImdHlwZT1sJno9MjAxOS8wMS8xMyAxMjo1OQ==)  

(디스크 데이터 파일에 데이터가 업데이트 되어있을 수도 있고, 아닐수도 있기 때문에 `???`라고 표시되어있는데,  
기본적으로 InnoDB는 ACID를 보장하므로 버퍼풀과 데이터파일은 같은 값이라고 봐도 무방하다)  

이처럼 하나의 레코드(id=12)에 대해 Multi Version이 유지되고,  
필요에 따라 보여주는 데이터가 달라지는 구조를 MVCC라고 한다.  

위 상황에서  
커밋을 실행하면 
- 더 이상의 변경작업 없이 지금 상태를 영구적인 데이터로 만들어버린다  
- 언두 로그 영역의 내용을 더 이상 필요로 하는 트랜잭션이 없을 경우, 언두 로그에서 해당 내용을 삭제한다.  

롤백을 실행하면  
- 언두 로그 영역에 있는 데이터를 복구한다  
- 언두 로그에서 해당 내용을 삭제한다.  


<!-- more -->