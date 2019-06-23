---
title: volume container 추가하기
date: 2019-06-16 22:41:01
tags:
    - 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문
    - docker
    - volume container
---

# volume container란?
일반적으로 docker container는 컨테이너 내부에 데이터를 관리하므로, 컨테이너가 파기되면 데이터가 모두 날라가게 된다  
이는 mysql 같은 데이터 스토리지를 사용할 경우 위험하게 되는데, 이를 방지하기 위해 따로 볼륨을 설정해서 데이터를 저장해줘야 한다  
호스트OS 디렉토리를 마운트시켜서 데이터를 관리할 수도 있지만, 호스트쪽 디렉토리에 의존이 생기고 만약 이 디렉토리의 데이터를 잘못 손대면 애플리케이션에 부정적 영향을 미칠 수 있기 때문에 이 방식은 사용하지 않는것이 좋다  

그래서 이에 대한 대안으로 추천되는것이 `볼륨 컨테이너`이다  
볼륨 컨테이너는 말 그대로 데이터를 저장하는 것이 목적인 컨테이너이다  
기본적으로 우리는 `Dockerfile` 작성 시 아래와 같이 볼륨을 설정할 수 있다  
```dockerfile
FROM mysql

VOLUME /var/lib/mysql

# ...
```

위처럼 작성하게 되면 컨테이너 내의 `/var/lib/mysql` 디렉터리가 호스트 PC의 `/var/lib/docker/volumes/${volume_name}/_data`에 마운트 된다  
(볼륨 이름은 임의의 해쉬값으로 생성된다)  
> 참고로 mac이나 windows의 경우 `/var/lib/docker/volumes` 디렉토리가 없는데,  
> 이는 mac이나 windows의 경우 docker를 바로 실행할 수 없으므로 VM을 하나 띄운 뒤, docker를 실행하기 때문이다  
> 즉, `/var/lib/docker/volumes` 디렉토리는 mac과 docker 사이에 띄워진 VM 내에 감춰져있다  
> 참고 : <https://forums.docker.com/t/var-lib-docker-does-not-exist-on-host/18314>  

volume container는 볼륨의 이러한 특징을 사용한 것이다  
컨테이너 자체를 볼륨을 관리하는 애로 만들어서 캡슐화하고, 이를 다른 컨테이너의 볼륨에 매핑해서 결합을 느슨하게 하는 것이다  

만약 아래와 같은 볼륨 컨테이너를 작성하고,  
```dockerfile
FROM busybox # 최소한의 운영체제 기능만 제공

VOLUME /var/lib/mysql
VOLUME /var/log
```
> `/var/lib/mysql`, `/var/log` 디렉토리에 대한 볼륨 2개가 생성되어 각각 `/var/lib/docker/volumes/${volume_name}/_data` 에 마운트 된다  

빌드하고 컨테이너로 띄운 뒤,
```sh
$ docker image build -t volume_container:latest .
$ docker container run -d volume_container:latest
```
> 참고로 볼륨 컨테이너를 띄우면 바로 종료되는데, 이렇게 종료된 컨테이너를 사용해도 상관없다

`--volumes-from` 옵션으로 다른 컨테이너에 연결한다면  
```sh
$ docker container run --volumes-from volume_container mysql:5.7
```

아래와 같은 형태가 되는 것이다  
```
mysql container -> volume_container -> /var/lib/docker/volumes/${/var/lib/mysql's volume_name}/_data
                                    -> /var/lib/docker/volumes/${/var/log's volume_name}/_data
```

# docker-compose.yml 에 volume container 추가  
위의 방식처럼 컨테이너를 직접 만들고 다른 컨테이너 실행 시 `--volumes-from` 속성으로 연결해주는 방법도 있지만, docker-compose를 사용 시 좀 더 간단한 방법을 제공해준다  
아래처럼 작성하면 된다  
```yml
version: "3"
services:
    test_database:
        image: mysql:5.7
        environment:
          MYSQL_DATABASE: test_db
          MYSQL_ROOT_PASSWORD: root
          MYSQL_ROOT_HOST: '%'
        ports:
          - 3306:3306
        volumes:
          - test_volume:/var/lib/mysql

    test_application:
        build: .
        expose:
          - 8080
        depends_on:
          - test_database

volumes:
    test_volume:
```

보다시피 `test_volume` 이라는 볼륨을 생성하고, 사용하는 쪽에서 `${volume_name}:${mount를 원하는 디렉토리}` 의 형태로 지정해주면 된다  
docker-compose 설정파일 v2 의 형태로 보면 좀 더 직관적으로 이해가 갈 것이다  
```yml
version: "2"
services:
    test_database:
        image: mysql:5.7
        environment:
          MYSQL_DATABASE: test_db
          MYSQL_ROOT_PASSWORD: root
          MYSQL_ROOT_HOST: '%'
        ports:
          - 3306:3306
        volumes:
          - test_volume

    test_application:
        build: .
        expose:
          - 8080
        depends_on:
          - test_database

    test_volume:
        image: busybox
        volumes:
            - /var/lib/mysql
            - /var/log
```
v3의 경우 컨테이너를 따로 생성하지 않아도 된다는 장점이 있다  

볼륨 컨테이너는 충분히 좋은 기능이지만, 그래도 범위가 같은 도커 호스트 안이라는 사실은 변하지 않는다  
이렇듯 데이터 이식 면에서는 아직 개선할 부분이 많이 남아있다

참고 :  
- <https://darkrasid.github.io/docker/container/volume/2017/05/10/docker-volumes.html>  
- <https://stackoverflow.com/questions/45494746/docker-compose-volumes-from-usage-example>

<!-- more -->