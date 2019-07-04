---
title: '[kubernetes] 네트워크 구조'
date: 2019-06-30 18:39:30
tags:
---

# Pod Network
![kubernetes-pod-network](https://joont92.github.io/temp/kubernetes-pod-network.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

컨테이너 내의 eth0 이 모두 veth0에 연결  
Pod 띄울 시 pause 라는 컨테이너가 추가로 생성되고, 이 컨테이너가 Pod 으로 묶인 컨테이너들의 네트워크를 담당한다?  
(localhost로 모두 통신할 수 있게끔)  
참고로 Pod으로 묶인 컨테이너들은 localhost로 통신할 수 있으므로 같은 port 를 설정할 수 없다  

그리고 여기서 Pod을 더 띄우게 되면, 지정한 개수만큼 container 가 더 뜨고 pause 컨테이너가 하나 다시 뜨게 된다  
즉 Pod 을 여러개 띄우기는 하지만, 물리적으로는 docker0 과 여러개의 docker container가 뜨는 구조이며, 이를 논리적으로 묶은것이 Pod  
(그리고 이 논리적인 범위에 대해 여러가지를 해주는애가 pause?)  

참고로 docker는 네트워크 타입이 bridge, host, container 등이 있는데 bridge가 기본이고, pod을 띄울떄는 container 형을 사용한다  
container 형을 사용하면 ip가 같게 설정되고, 내부에서 localhost 통신 가능  
(같은 포트로 띄우면 에러 발생함, netstat 등 다 해봐도 바로 됨)  

---

Pod과 Pod 끼리도 통신할 수 있어야 하는데, 같은 node 라면 모두 docker0 과 연결된 컨테이너라 상관없지만 다른 노드에 있다면 어떨까?  
문제가 되는 상황은 각각 다른 노드의 docker0 의 ip 대역대가 같아버리는 문제이다  

그래서 kubernetes 는 이런 문제를 없애기 위해, node 들의 ip 대역을 전방위 적으로 설정해서 겹치지 않게끔 한다  
![kubernetes-pod-network-multi-node](https://joont92.github.io/temp/kubernetes-pod-network-multi-node.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

그리고 이를 게이트웨이의 라우팅 테이블에서 관리한다

# Service Network

POD network : https://coffeewhale.com/k8s/network/2019/04/19/k8s-network-01/
service network : https://coffeewhale.com/k8s/network/2019/05/11/k8s-network-02/

#### 쿠버네티스 로드밸런싱(알아봐야함)
각 노드별로 kube-proxy가 들어가있다  
kube-proxy 내에는 각각 iptables가 있고, 이 iptables 내에 pod들에 대한 정보가 있다?  

서비스는 모든 node에 생성된다?  
모든 노드의 IP(머신일테니까):포트 로 외부에서 접근하면 해당 서비스에 접근할 수 있다  
아마도 L4 스위치를 위한 기능인 것 같다  

<https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview?hl=ko>  
<https://cloud.google.com/blog/products/containers-kubernetes/introducing-container-native-load-balancing-on-google-kubernetes-engine>  

서비스를 NodePort 로 설정하고 앞의 LB에서 VM 단위로 로드밸런싱 하는게 별로 최적화된 방법이 아니라고 하는 듯  
서비스에서 해주는 어플리케이션 레벨의 로드밸런싱이 효과적이다는 말을 하는것 같다  
(어떻게..?)  
VM 간에 연결이 되어야하는..  

VM의 서비스(kube-proxy)로 왔을때, 적절히 다른 Pod로 넘겨준다  
각 Pod들은 여러 VM에 퍼져있다  

Iptables에서 랜덤으로 다른 Pod로 넘긴다  
그러므로 IPvs 로 가라고 했다..?  

https://coffeewhale.com/k8s/network/2019/04/19/k8s-network-01/
https://rootkey.tistory.com/10