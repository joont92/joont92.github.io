---
title: docker-compose로 nginx + spring-boot + mysql 구성하기
date: 2019-06-16 11:10:49
tags:
    - docker-compose
    - docker
    - nginx
    - spring boot
    - mysql
---

알다시피 일반적으로 시스템은 단일 어플리케이션 만으로 구성되지 않는다  
다른 어플리케이션 서버나 미들웨어 등과 서로 통신하며 하나의 시스템이 구성된다  
이번에는 가장 일반적인 구조인 `리버스 프록시(nginx) + 어플리케이션 서버(spring-boot) + 데이터 스토어(mysql)` 를 구성해보겠다  

각각의 구성요소는 서로 의존관계가 있다  
서로 통신할 수 있어야하고 각 구성요소가 올라오는 순서도 맞춰줘야 한다  
물론 도커로 다 설정할 수 있긴하지만, 사람이 매번 수작업으로 맞춰줘야 하기 때문에 번거롭고 실수하기도 쉽다  
그래서 이런 컨테이너의 실행을 한번에 관리할 수 있게 해주는 `docker-compose`를 사용할 것이다  

# 사전 구성
spring boot로 간단히 사용자를 저장하고, 조회하는 로직을 작성한다  

```java
// TestController.java
@RequiredArgsConstructor
@RequestMapping("/users")
@Controller
public class TestController {
    private final UserRepository userRepository;

    @PostMapping
    public ResponseEntity<Void> createUser(@RequestBody UserRequest request) {
        User user = userRepository.save(request.toEntity());

        return ResponseEntity.created(URI.create("/users/" + user.getId())).build();
    }

    @GetMapping
    public ResponseEntity<List<User>> getUsers() {
        return ResponseEntity.ok(
            userRepository.findAll()
        );
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(
            userRepository.findById(id)
                .orElseThrow(IllegalStateException::new)
        );
    }
}

// UserRequest.java
@Getter
@Setter
public class UserRequest {
    private String name;

    private Integer age;

    public User toEntity() {
        return new User(name, age);
    }
}

// User.java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(name = "user")
public class User {
    @Getter
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Getter
    @Column(name = "name")
    private String name;

    @Getter
    @Column(name = "age")
    private Integer age;

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}

// UserRepository.java
public interface UserRepository extends JpaRepository<User, Long> {
}
```
보다시피 user 를 조회하고 저장하는 기능을 가진 간단한 API 서버이다  

이를 도커 컨테이너로 띄우기 위해 `Dockerfile`을 작성한다  
```dockerfile
FROM openjdk:8-jdk

COPY ./test-application /test-application
WORKDIR /test-application

CMD ["./gradlew", "bootRun"]
```

`test-application`은 spring-boot 어플리케이션이 있는 디렉토리이다  
간단하게 spring-boot 어플리케이션이 있는 디렉토리 전체를 컨테이너로 복사한 뒤, `./gradlew bootRun`으로 어플리케이션을 실행시킨다  

# spring-boot, mysql 띄우기(feat. docker-compose.yml)
이제 위 파일을 docker image로 만들어주면 되는데, 알다시피 저 `Dockerfile`을 사용해 docker image를 만들어봐야 사용하지 못한다  
db가 없기 때문이다  
mysql을 docker container 로 띄운 뒤 spring-boot 를 docker container로 띄우면 되긴 하지만, 번거로우므로 이를 같이 해줄 수 있는 `docker-compose` 설정 파일을 작성한다  

```yml
version: "3"
services:
    test_database:
        # 컨테이너 이름을 주고 싶다면 작성한다
        # container_name: test_database
        image: mysql:5.7
        environment:
          MYSQL_DATABASE: test_db
          MYSQL_ROOT_PASSWORD: root
          MYSQL_ROOT_HOST: '%'
        ports:
          - 3306:3306        

    test_application:
        build: .
        ports:
          - 8080:8080
        depends_on:
          - test_database
```
mysql과 spring-boot 컨테이너 2개를 띄워주는 `docker-compose.yml` 파일이다  
mysql의 경우 이미지명을 지정해 레지스트리에서 땡겨오도록 했고, spring-boot의 경우 현 위치에 있는 `Dockerfile`을 참조하여 만든 image를 컨테이너로 띄우게끔 했다  
(`docker-compose.yml`과 spring-boot `Dockerfile`은 같은 위치에 있다)  
여기서 중요한 것은 `depends_on` 속성인데, 이렇게 해놓으면 mysql 컨테이너가 다 뜬 다음 spring-boot 컨테이너가 뜨게끔 설정된다  

이제 spring-boot에서 mysql을 바라볼 수 있도록 `application.properties`에 접속 정보를 작성한다  
```properties
spring.datasource.url=jdbc:mysql://test_database:3306/test_db?useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.database-platform=org.hibernate.dialect.MySQL5Dialect
spring.jpa.generate-ddl=true
spring.jpa.show-sql=true
```
여기서 특별한 부분은 datasource url로 ip 대신 `test_database` 라고 준 부분이다  
이는 `docker-compose.yml`에 써놓은 mysql의 서비스명과 동일한데,  
`docker-compose.yml` 내에 작성한 컨테이너들은 모두 같은 네트워크에서 동작하므로, 서비스 이름으로 서로를 참조할 수 있기 때문이다  
> 기존에는 `links` 라는 속성으로 지정해줘야 했으나, docker-compose 설정파일 v3 부터 이를 작성할 필요없게 되었다  
> 근데 이 기능을 뭐라고 부를까..?  

이제 작성이 끝났으니, docker-compose 로 컨테이너들을 실행시켜보자  
```sh
$ docker-compose up
```
> docker-compose는 기본적으로 명령을 실행한 위치에 있는 `docker-compose.yml` 파일을 참조하여 실행한다  
> 만약 다른 경로에 있거나 다른 파일명을 사용하고 싶을 경우 `-f` 옵션으로 docker-compose 파일을 지정해주면 된다  

mysql과 spring-boot가 차례대로 실행되는것을 볼 수 있다  
> `-d` 옵션을 주면 백그라운드로 실행시킬 수 있다  

postman으로 `localhost:8080`으로 API를 호출해보면, 잘 동작함을 볼 수 있다  

# nginx 추가  
이제 리버스 프록시인 nginx를 추가해보자  
`docker-compose.yml` 파일을 수정한다  

```yml
version: "3"
services:
    test_web:
        image: nginx
        ports:
          - 80:80
        volumes:
          - ./nginx/conf.d:/etc/nginx/conf.d
        depends_on:
          - test_application

    test_database:
        image: mysql:5.7
        environment:
          MYSQL_DATABASE: test_db
          MYSQL_ROOT_PASSWORD: root
          MYSQL_ROOT_HOST: '%'
        ports:
          - 3306:3306

    test_application:
        build: .
        expose:
          - 8080
        depends_on:
          - test_database
```
spring-boot 컨테이너가 뜬 다음에 레지스트리에서 nginx를 땡겨와서 컨테이너로 띄우게끔 했다  
nginx 를 작성한 부분에 `volumes` 라는 부분이 보이는데, 이는 호스트의 `nginx/conf.d` 폴더를 컨테이너의 `/etc/nginx/conf.d` 폴더로 마운트 해주겠다는 의미이다  
이렇게 작성한 이유는 호스트쪽에 작성해놓은 nginx 설정 파일을 nginx 컨테이너가 뜨면서 읽게하기 위함이다  
> nginx는 `/etc/nginx/conf.d` 내에 들어있는 모든 `.conf` 파일을 include 한다  

아래는 conf.d 안에 작성한 `app.conf` 파일이다  
```conf
server {
    listen 80;
    access_log off;

    location / {
        proxy_pass http://test_application:8080;
	    proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

```

이제 작성이 끝났으니, `docker-compose`를 down 했다가 다시 up 한다  
앞에 리버스 프록시를 두었으니 url `localhost`로 변경했을때 잘 작동함을 볼 수 있다  
> 참고로 `test_application`에 대한 포트포워딩 설정이었던 `ports`가 `expose`로 바뀌었는데, 이는 컨테이너 내부에서만 해당 포트를 인식하게끔 하는 속성이다  
> 즉, 다른 컨테이너에서는 8080으로 통신이 가능하지만, 외부에서는 8080으로 더이상 접근할 수 없다  

github url : <https://github.com/joont92/docker-study/tree/master/step02>  
참고 : <https://github.com/hellokoding/hellokoding-courses/blob/master/docker-examples/dockercompose-springboot-mysql-nginx/docker-compose.yaml>  

<!-- more -->