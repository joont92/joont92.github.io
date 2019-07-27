---
title: 'redis'
tags:
---

NoSQL
- 비정형 구조라 데이터 저장이 유연함
- 읽기와 쓰기가 빨라서 빅데이터 처리에 효과적
- key-value, column-family, document, graph 형식이 있음
- 선택시 고려사항
    - 파티셔닝, 샤딩이 가능한가?(빠른 쓰기와 읽기의 핵심)
    - 복제(Replication)이 가능한가?(failover)
    - 스케일 아웃이 가능한가?
    - 트랜잭션이 지원되는가?

Redis
- 다른 NoSQL DB와 비교했을때 매우 경량임
- key-value 형태
    - 데이터 표현에 한계가 있음
- 인메모리 db라 속도가 매우 빠름
    - 선택적으로 파일로 저장 가능하지만, DBMS를 통해 자동으로 관리되는것이 아니기때문에 데이터 유실의 리스크가 있음
    - 이러한 특징 때문에 메인 DB 로 사용하는 것은 한계가 있음
- String, Set, Sorted Set, Hash, List, HyperLogLogs 유형 저장 가능
- Master/Slave replication 가능
- 파티셔닝 가능
- 데이터에 expired time 설정 가능
- 캐싱을 통한 빠른 쓰기/읽기 작업이 요구되는 곳에 사용

# 명령어
```sh
# 데이터 저장
set 1111 "Value"

# 데이터 검색
get 1111

# 데이터 범위 검색
keys *
keys *2
keys 2*

# 데이터 삭제
del 1111

# 모든 데이터 삭제
flushall

# 키 존재 여부 체크
exists 1112

# expired time 설정
setex 1111 30 "Value"

# 여러 값 한번에 저장
mset 1111 "Value1" 1112 "Value2"

# 여러 값 한번에 조회
mget 1111 1112
```

# 데이터타입

---

- redis는 제약조건 기능이 제공되지 않는다

https://www.joinc.co.kr/w/man/12/REDIS/IntroDataType

- Redis 자료 구조 활용
- 모델링 방법
- key는 object:id 의 형태를 취한다
- keys 대신 scan 을 사용한다
    - 되도록이면 키 값을 직접 지정해서 탐색하느 것이 좋다
- Replication
- master/slave