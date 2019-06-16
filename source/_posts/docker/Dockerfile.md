---
title: Dockerfile
date: 2019-06-07 21:07:24
tags:
    - 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문
    - docker
    - docker image
---

# Dockerfile이란?
도커 이미지를 만들 때 꼭 필요한 설정파일이다  
이 파일내에 작성된 인스트럭션들을 참조하여 이미지가 만들어진다  

기본으로 `Dockerfile` 이라는 이름을 사용하고, 이름을 변경하고 싶다면 이미지 빌드시에 추가 옵션을 줘야한다(-f)  

# 인스트럭션
`Dockerfile` 내에 있는 명령어들을 말한다  
아래는 각 인스트럭션에 대해 간단히 나열한 것이며, 자세한 내용이 궁금하면 <https://docs.docker.com/engine/reference/builder/> 를 참조한다  

## 자주 사용하는 인스트럭션
예시로 사용할 자바 어플리케이션과 Dockerfile 이다  

Test.java
```java
public class Test {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

Dockerfile
```
FROM openjdk:8-jdk

COPY Test.java .
RUN javac Test.java

CMD ["java", "Test"]
```

### FROM
해당 도커 이미지의 바탕이 될 베이스 이미지를 지정하는 인스트럭션이다  
openjdk를 베이스 이미지로 땡겨왔기 때문에 javac, java 명령어가 실행 가능함을 볼 수 있다  
> openjdk의 Dockerfile을 따라가보면 FROM에 ubuntu를 사용하고 있다  
> 이런식으로 이미지가 포장되고, 포장되어 빌드되는 형식이다  
> 어찌됬든 젤 안쪽에는 운영체제를 베이스 이미지로 땡겨야 할 것이다  

openjdk 는 이미지명이며, 8-jdk는 태그명이다  
레지스트리를 따로 지정하지 않았기 때문에 도커 허브에서 땡겨온다  

### COPY
호스트 머신의 파일이나 디렉터리를 도커 컨테이너 안으로 복사하는 인스트럭션이다  
이미지가 빌드될 때 1번만 실행되는 명령어이다  

컨테이너안의 현재 디렉토리로 호스트 머신의 Test.java 이 복사된다  

### RUN
도커 이미지를 `실행할 컨테이너 안에서` 실행할 명령을 정의한다  
이 또한 이미지가 빌드될 때 1번만 실행되는 명령어이다  

복사된 Test.java를 javac로 컴파일하고 있다  

### CMD
컨테이너 안에서 실행할 프로세스(명령)를 지정한다  
이는 이미지가 컨테이너화 될 때(실행될 때)마다 실행되는 명령어이다  

컴파일된 Test.class 파일을 실행하고 있다  

작성법이 조금 특이한데, 총 3가지 작성법을 제공한다  
- CMD command param1 param2 [...]
    - 가장 익숙한 형태이다  
    - FROM 으로 설정한 이미지에 포함된 쉘 파일을 사용하여 명령을 실행한다  
    - 쉘 스크립트 구문을 사용할 수 있다  
- CMD ["executable", "param1", "param2" [, ...]]
    - 쉘 없이 바로 실행하면서 매개변수를 던져주는 형태이다  
    - 도커에서 권장하는 형태이다
    - 쉘 스크립트 구문을 사용할 수 없다
        ```dockerfile
        CMD ["echo", "Hello, $name"]

        $ Hello $name
        ```
    - 만약 쉘 스크립트를 사용하고 싶다면 쉘을 실행시키면서 인자로 전달해줘야 한다
        ```dockerfile
        CMD ["/bin/bash", "-c", "echo Hello, $name"]
        ```
- CMD ["param1", "param2" [, ...]]
    - ENTRYPOINT에 지정된 명령에 사용할 인자를 전달한다

#### CMD는 Dockerfile 내에 하나만 작성할 수 있다
만약 CMD를 여러개 작성한다면 가장 앞부분껀 전부 무시되고 가장 마지막에 있는 명령만이 실행된다  
```dockerfile
CMD ["javac", "Test.java"]
CMD ["java", "Test"]
```
했다가 안되서 찾아봤음...  

#### CMD 명령 오버라이드도 가능하다
위와 같이 선언된 상황에서 
```sh
$ docker container run javatest:latest echo joont92
$ joont92 # 출력
```
CMD 명령이 무시됨을 볼 수 있다  

## 그 외 인스트럭션
### ENTRYPOINT
CMD와 마찬가지로 컨테이너 안에서 실행될 프로세스(명령)를 지정하는 인스트럭션이다  
CMD와 다른점은 조금 기준점(?) 이 되는 프로세스를 지정하는 것이랄까..  
ENTRYPOINT를 입력하면 CMD에 전달된 인자들은 전부 ENTRYPOINT의 인자로 전달된다  
```dockerfile
FROM openjdk:jdk-8

ENTRYPOINT ["java"]
CMD ["version"]
```

또한 아래와 같이 사용해서 컨테이너의 용도를 어느정도 제한 할수도 있다  
```dockerfile
FROM golang:1.10

ENTRYPOINT ["go"]
CMD [""]
```

자동으로 go 가 입력된 상태라고 보면 된다  
go 이후의 명령어만 인자로 넘기면 된다  
```sh
$ docker container run gotest:latest version
$ go version go1.10.3 linux/amd64 # 출력
```

### LABEL
이미지를 만든 사람의 이름 등을 적을 수 있다  
```dockerfile
LABEL maintainer="joont92@github.com"
```

### ENV
도커 안에서 사용할 환경 변수를 지정한다  
```dockerfile
ENV CLASSPATH=/workspace/javatest

CMD ["java", "Main"]
```

### ARG
이미지 빌드할 떄 환경변수를 정의하여 사용할 수 있다  
```dockerfile
ARG classpath=.
ENV CLASSPATH=${classpath}

CMD ["java", "Main"]
```

외부에서 환경변수를 전달 받을수도 있다
`--build-arg`를 통해 인자를 전달한다  
```sh
$ docker image build --build-arg classpath=/workapce/javatest -t javatest:latest .
```
> 이미지 빌드시에만 사용할 수 있다  
> 컨테이너 생성시에 클래스패스를 바꾸는 것은 불가능하다  

참고 : [야마다 아키노리, 『도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문』, 심효섭 옮김, 위키북스(2019)](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158391447&orderClick=LEA&Kc=)

<!-- more -->