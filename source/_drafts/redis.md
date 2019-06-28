---
title: redis
tags:
---

NoSQL
- 비정형 구조라 데이터 저장이 유연함
- 읽기와 쓰기가 빨라서 빅데이터 처리에 효과적
- key-value, column-family, document, graph 형식이 있음
- 선택시 고려사항
    - 파티셔닝, 샤딩이 가능한가?(빠른 쓰기와 읽기의 핵심)
    - 복제(Replication)이 가능한가?(failover)
    - 트랜잭션이 지원되는가?

Redis
- 다른 NoSQL DB와 비교했을때 매우 경량임
- key-value 형태
    - 데이터 표현에 한계가 있음
- 인메모리 db라 속도가 매우 빠름
    - 선택적으로 파일로 저장 가능하지만, 관리가 어려워서 무결성 보장을 위해 사용하기는 힘듦
- String, Set, Sorted Set, Hash, List, HyperLogLogs 유형 저장 가능
- Master/Slave replication 가능
- 파티셔닝 가능
- 데이터에 expired time 설정 가능
- 캐싱을 통한 빠른 쓰기/읽기 작업이 요구되는 곳에 사용
- 