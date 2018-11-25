---
title: jhipster
date: 2018-11-25 23:07:08
tags:
---

# 정의
<https://inyl.github.io/programming/2016/10/15/JHipster.html>

Spring Boot + Front end + etc.. 등등으로 구성된 프로젝트를 generate 해주는 서비스이다.  
현재는 `yo jhipster` 대신에 `jhipster` 명령어만 입력해서 실행시킬 수 있다.  
이후 나오는 선택사항들을 토대로 프로젝트가 generator 된다(아주 방대한 양의 소스가 generate 된다)  
지원하는 기술이 굉장히 많다.  
msa 프로젝트 초기 구성시에 좋다.  

# msa 프로젝트 generate 예시
1. service type - Microservice Application
2. project name
3. port 
4. package name
5. discovery server - jhipster registry
6. authentication type - jhipster UAA
7. uaa project path
8. database type - SQL(H2, MySQL…)
9. production database - MySQL
10. development database - MySQL
11. cache abstraction - Hazelcast
12. 2nd level cache - Yes
13. build tool - Gradle
14. other technology - OpenAPI generator, Kafka
15. internationalization - Yes
    1. native language - Korean
    2. additional language - English
16. test framework - Cucumber
17. other generator - No  

## msa 설정 시 추가로 해줘야할 부분
### jhipster-registry 설정
msa 구성 시 eureka 서버를 사용해야 하는데 jhipster-registry가 그 역할을 해준다.  
그러므로 클론받고 띄워줘야 한다.  

```sh
git clone https://github.com/jhipster/jhipster-registry

cd jhipster-registry
./mvnw
```

### kafka 설정(선택했을 경우)

<!-- more -->