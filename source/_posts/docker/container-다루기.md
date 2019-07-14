---
titke: [docker] container 다루기
date: 2019-06-08 15:58:27
tags:
    - 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문
    - docker container
---

빌드한 도커 이미지를 실행시키면 도커 컨테이너가 된다  
언급했듯이 도커 이미지는 하나의 템플릿이므로, 하나의 도커 이미지로 여러개의 도커 컨테이너를 만들 수 있다  

# 도커 컨테이너 생명주기
도커 컨테이너는 도커 이미지를 `실행`시킨 것이기 때문에 상태를 가지고 있다  
- 실행 중
    > ENTRYPOINT, CMD에 있는 어플리케이션이 실행된 상태를 말한다  
    > 명령행 도구 등의 컨테이너는 이 상태가 길게 유지되지 않는다  
- 정지
    > 실행 중 상태에 있는 컨테이너를 명시적으로 종료하거나, 컨테이너에서 실행된 어플리케이션이 정상/오류를 막론하고 종료된 경우의 상태를 말한다  
    > 정지되어도 그 상태 그대로 디스크에 남아있기 때문에(docker container ls 로 확인 가능) 다시 실행할 수 있다
- 파기
    > 정지된 컨테이너를 명시적으로 삭제하면 파기 상태가 된다(docker container ls 로 확인 불가능)  
    > 파기된 컨테이너는 다시 정지된 컨테이너와 같은 상태로 돌아갈 수 없으므로 주의해서 실행해야 한다

---

아래는 도커 컨테이너를 관리하면서 자주 사용하는 명령어들과 그에 대한 간단한 설명이다  
추가적인 명령어나 옵션이 궁금하다면 <https://docs.docker.com/engine/reference/commandline/container/> 를 참조한다  

# docker container run
```sh
$ docker container run [options] 이미지명[:태그명] [명령] [명령인자..]
$ docker container run [options] 이미지ID [명령] [명령인자..]
```
도커 이미지로부터 컨테이너를 생성+실행 하는 명령이다  

유용한 옵션이 많으니 활용하면 좋다  
- -d : 백그라운드로 실행
- -p : 포트포워딩 
    > `-p 9000:8080` == 호스트 포트 9000번을 컨테이너 포트 8080으로 포워딩
- --name : 컨테이너에 이름 부여
    > 매번 컨테이너ID를 조회하는 것이 번거로우므로 이름을 지정할 수 있다  
    > 컨테이너 이름은 중복될 수 없기 때문에 개발환경 외에는 잘 사용되지 않는다(운영은 많은수의 컨테이너를 추가/삭제 하므로)
- -i, -t : -i는 표준 입력을 받을지 여부이고(파이프라이닝, 키보드 입력), -t는 가상터미널을 제공할지 여부이다  
    ```sh
    $ docker container run -it ubuntu:16.04
    ```
    > 보통 위처럼 -it를 같이 사용하고, 이러면 컨테이너에 쉘에 직접 접근할 수 있게 된다(가상 터미널로 표준 입력을 하는것이 되므로)  
    >
    > -t 없이 -i 만 사용해도 의미가 있다(파이프라이닝으로 입력하는 방법이 있으므로)  
    > -i 없이 -t 만 사용하는건 의미가 없다(입력을 받지 못하는 상태라 가상터미널을 열어봐야)  
    > -d와 -it를 같이 사용할 수 없다(백그라운드라 입력을 대기하거나 가상터미널을 제공하는것이 불가능하다)
- --rm : 컨테이너를 종료할 때 컨테이너를 파기하는 옵션
    > 같은 이름으로 컨테이너를 실행시킬 수 없기 때문에 이 옵션을 사용해주는것이 좋다  
    > 보통 `--name` 과 같이 사용한다

> 컨테이너에 할당할 자원도 어느정도 제어할 수 있다 <https://jungwoon.github.io/docker/2019/01/13/Docker-6/>

# docker container ls
```
$ docker container ls [options]
```
컨테이너의 목록을 보여줌  

- -a : 종료된 컨테이너의 목록도 같이 보여줌(원래는 실행 중인 컨테이너의 목록만 보여줌)
- -q : 컨테이너 ID만 추출함
    ```sh
    $ docker container stop $(docker container ls -q)
    ```
    처럼 사용할 수 있음
- --filter : 컨테이너 목록 필터링
    ```sh
    $ docker container ls --filter "name=container_name" # 컨테이너명
    $ docker container ls --filter "ancestor=image_name" # 이미지명
    ```

# docker container stop
```sh
$ docker container stop 컨테이너ID_또는_컨테이너명
```
실행중인 컨테이너를 종료할 떄 사용한다

# docker container restart
```sh
$ docker container restart 컨테이너ID_또는_컨테이너명
```
정지한 컨테이너를 재시작할 때 사용한다  
> start 와 차이가 뭘까?  

# docker container rm
```sh
$ docker container rm 컨테이너ID_또는_컨테이너명
```
정지된 컨테이너를 완전히 파기할 떄 사용한다
실행중인 컨테이너를 강제로 삭제하고 싶다면 `-f` 옵션을 사용한다  

# docker container logs
```sh
$ docker container logs [options] 컨테이너ID_또는_컨테이너명
```
컨테이너의 표준 출력으로 출력된 내용을 보여준다  
`-f` 옵션을 주면 새로 출력되는 표준 출력을 계속 볼 수 있다  

보통은 이 내용을 수집해 웹 브라우저나 명령행 도구를 통해 보여주므로 이 명령을 사용할 일은 많이 없다  

# docker container exec
```sh
$ docker container exec [options] 컨테이너ID_또는_컨테이너명 컨테이너에서_실행할_명령
```
실행 중인 컨테이너에서 원하는 명령을 실행할 수 있다  

```
$ docker container exec ubuntu_docker pwd
$ docker container exec ubuntu_docker ls
```

아예 컨테이너로 접속하고 싶다면 아래와 같이 입력하면 된다  
```
$ docker container exec -it ubuntu_docker sh
```

# docker container cp
```sh
$ docker container cp [options] 컨테이너ID_또는_컨테이너명:원본파일 호스트_대상파일
$ docker container cp [options] 호스트_원본파일 컨테이너ID_또는_컨테이너명:대상파일
```
실행중인 컨테이너에 파일을 복사하거나 복사해 올 때 사용한다  
> Dockerfile의 CP는 이미지 빌드시에 호스트 -> 컨테이너 방향으로의 복사만 가능하다  

```sh
$ docker container cp test_file ubuntu_docker:/tmp/test_file
```

참고 : 
- [야마다 아키노리, 『도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문』, 심효섭 옮김, 위키북스(2019)](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158391447&orderClick=LEA&Kc=)

<!-- more -->