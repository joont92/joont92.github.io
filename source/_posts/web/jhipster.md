---
title: 'jhipster'
date: 2018-11-25 23:07:08
tags:
    - jhipster example  
    - jhipster-registry
    - jdl
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
jhipster init 할때 kafka를 설정했을 경우, docker로 kafka를 실행한다

```sh
cd {jhipster_project_home}
docker-compose -f src/main/docker/kafka.yml up -d
```

# jdl
jhipster에서 도메인을 나타낼 떄 사용하는 language이다.  
<https://www.jhipster.tech/jdl/>  

작성한 도메인의 관계도는 아래의  
<https://start.jhipster.tech/jdl-studio/>  
에서 가시적으로 확인해볼 수 있고,  
`.jh` 확장자로 따로 저장도 가능하다.  

작성한 `.jh` 파일은 아래의 명령을 통해 jhipster에 적용 갸능하다.  

```sh
jhipster import-jdl <jdl file path>
```

위 명령을 수행하면 작성한 내용을 기준으로 entity, repository를 생성하고 생성된 repository를 사용하여 기본적인 CRUD를 호출하는 controller를 만들어준다.  
(초반 generate 시에 위 파일을 지정해줄수도 있다고 한다)  

하지만 생성된 entity, repository를 그대로 사용할수는 없으니(당연하다)  
초반 와꾸잡는데만 사용하고, 그 시점이후로는 jdl 파일을 건드리진 않는다.  

<!-- more -->