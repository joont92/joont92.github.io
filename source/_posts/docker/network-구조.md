---
titke: [docker] network 구조
date: 2019-06-29 17:30:30
tags:
    - docker
    - docker network
    - docker0
---

도커를 공부하면서 네트워크 부분에서 알게된 부분은 아래와 같았다  
- 도커 컨테이너를 띄울때마다 매번 동일한 네트워크 대역의 IP가 할당된다
- 각 컨테이너들은 서로 통신이 가능하다
- docker-compose로 컨테이너를 띄우면 다른 네트워크 대역의 IP가 할당된다

이런것을 알게되다 보니 도커의 네트워크 구성이 궁금해졌고, 찾아가며 공부한 내용을 정리하고자 한다  
(`참고`에 있는 글들에 훨씬 잘 설명되어있으니 그걸 읽는게 더 낫다.. 난 그냥 개인 정리용..ㅎ)  

---

# veth interface, NET namespace
도커의 네트워크 구조를 이해하기 위해선 먼저 리눅스의 `NET namespace`와 `veth interface`에 대해 알아야한다  

- veth interface 
    > 간단히 말해 랜카드에 연결된 실제 네트워크 인터페이스가 아니라, 가상으로 생성한 네트워크 인터페이스이다  
    > 일반적인 네트워크 인터페이스와는 달리 패킷을 전달받으면, 자신에게 연결된 다른 네트워크 인터페이스로 패킷을 보내주는 식으로 동작하기 때문에 항상 쌍(pair)로 생성해줘야 한다  

- NET namespace
    > 리눅스 격리 기술인 namespace 중 네트워크와 관련된 부분을 말한다  
    > 네트워크 인터페이스를 각각 다른 namespace에 할당함으로써 서로가 서로를 모르게끔 설정할 수 있다  

(자세한 내용은 <https://bluese05.tistory.com/28> 를 참조한다.. 100배 설명이 더 잘되어 있다)

# 도커 네트워크 구조
도커는 위에서 언급한 veth interface와 NET namespace 를 사용해 네트워크를 구성한다  
그림으로 보면 아래와 같다  

![docker-network](https://joont92.github.io/temp/docker-network.png)  

1. 생성되는 도커 컨테이너는 namespace 로 격리되고, 그 상태에서 통신을 위한 네트워크 인터페이스(eth0)를 할당받는다
2. host PC의 veth interface 가 생성되고 도커 컨테이너 내의 eth0 과 연결한다
    > 컨테이너의 네트워크 격리를 달성하기 위해 선택한 방법인 것 같다
3. host PC의 veth interface 는 `docker0` 이라는 다른 veth interface 와 연결된다
4. 이 과정이 컨테이너가 추가될 때 마다 반복된다

여기서 나오는 `docker0`은 docker 실행 시 자동으로 생성되는 `가상 브릿지` 이다  
컨테이너가 생성될 때 마다 가상 인터페이스가 생성되고, 이 브릿지에 바인딩 되는 형태라고 보면 된다  
즉, 모든 컨테이너는 외부로 통신할 때 이 `docker0 브릿지`를 무조건 거쳐가야 되는 것이다  
이러한 특징 때문에 도커 컨테이너끼리 서로 통신이 가능한 것이다  
> `docker0` 브릿지에 할당된 ip 대역에 맞춰 컨테이너들도 ip가 할당된다  
> e.g. docker0 = 172.17.0.1/16, container1 = 172.17.0.2, container2 = 172.17.0.3 ...  
> IP는 veth가 가지는걸까, eth0 이 가지는걸까?(가상 네트워크 인터페이스에 대한 이해가 조금 더 필요하다)  

아래는 컨테이너 2개를 띄웠을 떄의 모습이다  
```sh
$ ip link ls

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:d8:72:d9 brd ff:ff:ff:ff:ff:ff
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:c0:e5:8c:93 brd ff:ff:ff:ff:ff:ff
4: vethf3bf38b@if34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 02:13:50:cf:5c:48 brd ff:ff:ff:ff:ff:ff link-netnsid 1
5: veth1680438@if36: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether f2:b2:f7:eb:a5:55 brd ff:ff:ff:ff:ff:ff link-netnsid 2
```

그리고 아래는 브릿지에 연결되어 있는 veth interface 를 조회한 모습이다  
```sh
$ brctl show docker0
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242c0e58c93	no		veth1680438
							            vethf3bf38b
```

> 참고로 mac이나 window에서는 이 `docker0` 브릿지나 `veth interface` 들을 볼 수 없다  
> VM 안으로 감쳐줘 있기 때문이다  

## 사실 docker container의 네트워크 모드는 총 4개이다
bridge, host, container, none 으로 총 4개가 존재한다  
위에서 살펴본 구조가 bridge 네트워크로 생성한 container의 동작방식이며, default 값이다  
kubernetes의 pod을 생성할때는 container 모드를 사용한다  
자세한 내용은 <https://bluese05.tistory.com/38>를 참조한다  

## docker-compose 로 띄우면 다른 네트워크 대역을 가진다
docker-compose 를 공부할 때 docker-compose 로 띄운 컨테이너들은 모두 같은 네트워크에 속한다는 말이 있었고, 실제로도 그랬다  
단독으로 띄운 도커 컨테이너와 docker-compose 를 통해 띄운 도커 컨테이너들을 들어가서 ip를 확인해보면 각자 네트워크 대역대가 다름을 알 수 있다  

이는 docker-compose 로 컨테이너를 띄우면 compose 로 묶은 범위에 맞춰 브릿지를 하나 더 생성하기 때문이다  
```sh
# 이렇게 docker-compose로 컨테이너를 띄우고
$ docker-compose -f docker-compose.yml up
# 네트워크 인터페이스를 확인해보면
$ ip link ls

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:d8:72:d9 brd ff:ff:ff:ff:ff:ff
# docker0 브릿지와 아까 띄워놓은 컨테이너들
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:c0:e5:8c:93 brd ff:ff:ff:ff:ff:ff
4: vethf3bf38b@if34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 02:13:50:cf:5c:48 brd ff:ff:ff:ff:ff:ff link-netnsid 1
5: veth1680438@if36: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether f2:b2:f7:eb:a5:55 brd ff:ff:ff:ff:ff:ff link-netnsid 2
# 여기서부터 docker-compose로 띄운 부분
# bridge가 하나 더 생겼다!
6: br-776e18676383: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:e0:f3:47:6b brd ff:ff:ff:ff:ff:ff
7: vethc059d77@if39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-776e18676383 state UP mode DEFAULT group default
    link/ether 0a:f7:37:0c:cd:64 brd ff:ff:ff:ff:ff:ff link-netnsid 3
8: veth0fea37d@if41: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-776e18676383 state UP mode DEFAULT group default
    link/ether b2:95:86:71:c6:43 brd ff:ff:ff:ff:ff:ff link-netnsid 4
```

이런 상태이므로 물론 docker-compose로 띄운 컨테이너와 일반 컨테이너간의 통신은 불가능하다(서로 경유하는 브릿지가 다르므로)  

# 외부와 통신은 어떻게 할까?
우리는 컨테이너를 띄울 때 아래와 같이 포트포워딩을 설정하여 외부에 컨테이너를 공개할 수 있다(+expose)  
```sh
# 포트포워딩 설정하여 컨테이너 생성
$ docker container run -p 8080:80 nginx
# 포트 listen 확인
$ netstat -nlp | grep 8080

tcp6       0      0 :::8080                 :::*                    LISTEN      26113/docker-proxy-
```
근데 보다시피 `docker-proxy` 라는 프로세스가 해당 포트를 listen 하고 있음을 볼 수 있다  
이는 간단히 docker host 로 들어온 요청을 해당하는 컨테이너로 넘기는 역할만을 수행하는 프로세스이다  
컨테이너에 포트포워딩이나 expose를 설정했을 경우 같이 생성되는 프로세스이다  

그렇지만 중요한것은, 실제로 포트포워딩을 이 docker-proxy가 담당하는 것이 아니라, host PC iptables 에서 관리한다는 점이다  
```sh
$ iptables -t nat -L -n

Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
MASQUERADE  tcp  --  172.17.0.4           172.17.0.4           tcp dpt:80

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.17.0.4:80
```
보다시피 모든 요청을 DOCKER Chain 으로 넘기고, DOCKER Chain 에서는 DNAT를 통해 포트포워딩을 해주고 있음을 볼 수 있다  
(이 iptables 룰은 docker daemon이 자동으로 한다)  

docker-proxy 는 iptables가 어떠한 이유로 NAT를 사용하지 못하게 될 경우 사용된다고 한다  

참고 :  
- NET namespace, veth interface : <https://bluese05.tistory.com/28>
- 도커 네트워크 구조 : <https://bluese05.tistory.com/15>
- 도커 네트워크 외부 통신 : <https://bluese05.tistory.com/53>