---
title: '[kubernetes] 네트워크 구조'
date: 2019-06-30 18:39:30
tags:
---

쿠버네티스 서비스 타입들이 이해가 안되서 찾아보다가 도커 네트워크를 공부했고, 쿠버네티스 네트워크까지 넘어왔다  
참고한 글들에 다 너무 잘 나와있지만 개인적으로 한번 더 정리해봤다  

# Pod Network
Pod은 내부의 컨테이너들끼리 IP와 port를 공유한다는 특징이 있었는데, 어떻게 구성되어 있길래 그럴까?  
아래는 Pod 의 네트워크를 잘 설명해주는 하나의 그림이다  

![kubernetes-pod-network](https://joont92.github.io/temp/kubernetes-pod-network.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

기본적으로 도커 컨테이너를 하나 생성하면, 해당 컨테이너를 namespace로 격리한 뒤 eth0 interface를 생성하고, 노드에 veth interface를 생성해서 연결시키는 구조로 동작한다  
하지만 위의 그림은 보다시피, 컨테이너별로 각기 다른 veth interface에 연결된 것이 아닌, 공통된 하나의 veth interface에 연결되어 있음을 볼 수 있다  

이는 쿠버네티스가 Pod 내에서 컨테이너를 생성할 때 도커 컨테이너를 bridge 모드(default)가 아닌 container 모드로 띄웠기 때문이다  
container 모드는 아래와 같이 다른 컨테이너를 지정함으로써 띄울 수 있는데, 이렇게 생성함으로써 생성된 도커 컨테이너는 지정한 도커 컨테이너와 네트워크를 공유하는 형태로 생성될 수 있게 된다  
```sh
$ docker run --net=container:{container_id} -d {image_name}
```
> 자세한 내용은 <https://bluese05.tistory.com/38>를 참조한다  

이렇게 생성된 컨테이너들을 각각 들어가서 ip 정보를 확인해보면, 모두 ip가 같음을 볼 수 있을 것이다  
즉 결론은, Pod 내의 도커 컨테이너들은 모두 위처럼 container 모드로 실행되었기 때문에 아래와 같은 특성이 만족되는 것이다  
- 외부에서 같은 IP로 접근할 수 있다(각 컨테이너들의 IP가 같음)
- 컨테이너간 localhost 로 통신할 수 있다
- 같은 port는 사용 불가능

> 그림에는 있지만 언급하지 않은 `pause` 컨테이너가 있는데, 이는 위에서 언급한 일련의 기능들을 수행하기 위해 Pod 내에 추가적으로 생성되는 컨테이너이다  
> Pod 이 실행될 때 마다 이 컨테이너가 먼저 실행되고, 이 컨테이너의 namespace를 다른 컨테이너들이 공유하고, 이 컨테이너가 다른 pod 들을 container 형태로 띄우는 건가?  

## 노드가 다른 Pod 끼리의 통신
위의 모델의 경우 같은 `docker0` 브릿자 아래에서는 Pod 간의 통신이 전혀 문제될 것이 없지만,  
쿠버네티스 클러스터의 경우 보통 1개 이상의 노드를 관리하며, Pod 는 매번 다른 노드에 배포되는 특성이 있다  

`docker0` 브릿지는 각 노드마다 생성되는데, 중요한 점(문제되는 점)은 이 `docker0` 브릿지의 ip 대역대가 겹칠 수 있다는 것이다  
`docker0` 아래의 컨테이너들은 전부 `docker0` 브릿지의 네트워크 대역을 따라가는데, 만약 `docker0` 브릿지의 네트워크 대역이 겹친다면 결국 Pod의 IP가 겹치는 문제가 발생한다  

![kubernetes-pod-network-multi-node1](https://joont92.github.io/temp/kubernetes-pod-network-multi-node1.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

이런식으로 말이다  
이 상태에서는 Pod 간에 서로 통신할 수가 없다. src와 dest의 IP가 같기 때문이다  

노드 안에서 생성되는 `doccker0` 브릿지의 경우 default 네트워크 대역이 정해져있고,  
만약 default 값을 사용하지 않는다고 해도 다른 노드의 `docker0` 브릿지가 어떤 네트워크 대역을 사용하고 있는지 알지 못하기 때문에 문제가 해결되지는 않는다  

쿠버네티스는 이러한 문제점을 해결하기 위해 2가지 방법을 사용했다  
- `docker0` 브릿지들의 전체 네트워크 대역을 아우르는 주소 대역을 할당한다
- 들어오는 패킷들이 어떤 브릿지로 가야하는지에 대해 라우팅 테이블을 설정한다

이를 적용하면 아래와 같은 모습이 된다  

![kubernetes-pod-network-multi-node2](https://joont92.github.io/temp/kubernetes-pod-network-multi-node2.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

(docker에서 사용하는 `docker0` 브릿지로는 이 문제를 해결할 수 없기 때문에 이를 커스텀한 `cbr` 브릿지를 사용했다)  

이제 Pod은 항상 라우팅 테이블을 거쳐 다른 Pod 이 존재하는 브릿지를 찾아갈 수 있기 때문에, 다른 노드에 있는 Pod 끼리도 원할하게 통신이 가능해진다!  

# Service Network
위에서 봤듯이 Pod 의 네트워크 메카니즘 때문에 다른 노드에 있는 Pod 끼리도 서로 통신이 가능하다  
근데 문제는, Pod는 상황에 따라 늘어나거나 삭제되는 둥 변경이 많은 리소스라는 점이다  
즉 새롭게 배치되면서 노드가 변경되고, 그에따라 IP가 변경되는 경우가 많다는 의미인데, 이런 상태에서는 Pod에서 다른 Pod로 통신하는 것이 사실상 불가능해진다  

쿠버네티스는 이러한 문제를 해결하기 위해 `서비스` 라는 새로운 리소스를 등장시켰는데, 개념은 대충 아래와 같다  
![kubernetes-service-network1](https://joont92.github.io/temp/kubernetes-service-network1.png)  
> 출처 : <https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0>  

보다시피 scalable 한 Pod 들을 서비스 라는 리소스 하나로 묶고(label을 이용하여 묶는다), 이 리소스를 통해 Pod에 도달할 수 있도록 한다    
아래는 생성된 서비스의 모습이다  
```sh
$ kubectl get services

NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
echo         ClusterIP   10.3.241.152   <none>        8888/TCP   23s
```

다른 Pod에서 서비스에 부여된 ClusterIP(내부 IP)로 요청을 보냄으로써 서비스에 포함된 Pod 들과 통신할 수 있게 되는 것이다  
이렇게 함으로써 Pod의 IP 변경에 대해 신경쓰지 않아도 되고, 서비스가 요청을 각 파드별로 적절히 로드밸런싱 해주는 효과까지 얻을 수 있게 된다  

## 어떻게 처리되는 걸까?(feat. kube-proxy)
그렇다면 서비스 IP를 Pod 의 IP로 바꿔주는 역할은 어디서 수행하게 되는걸까?  
실제로 서비스에 부여된 ClusterIP의 경우 어디에도 연결되어 있는 네트워크 인터페이스가 없다  
이는 어딘가 다른데서 이를 처리해주고 있음을 의미하는데, 그곳이 어디일까?  

이 역할은 각 노드마다 하나씩 떠 있는 kube-proxy 라는 도커 컨테이너가 담당한다  

서비스는 결국 `ClusterIP : Pod IP 들` 의 형태로 이루어진 iptables로 보면 된다  
iptables에서 로드밸런싱 기능을 구현하여 서비스의 load balancing 을 달성했으며, 이 iptables 정보는 kube-proxy에 의해 계속해서 관리된다  

Pod가 서비스 IP를 요청하면 cbr0 까지 올라가고 노드의 네트워크 인터페이스를 통해 나갈떄 iptables에 의해 변경되어서 나간다  
그러므로 갔다가 다시 똑같은 노드로 돌아올 수도 있다  

노드에 해당 포트로 들어왔을 때 해당 서비스로 연결하는 것이 노드포트이다  
NodePort로 31111 지정했다는 것은, 해당 노드의 31111 포트로 들어왔을 때 서비스 ClusterIP로 포워딩 됨을 의미한다  
해당 가상 IP로 연결되면 다시 위의 구조를 타서 Pod로 연결된다  


서비스의 loadBalancer 타입은 클라우드에 로드밸런서를 하나 생성해주고,  
이 로드밸런서를 클러스터 내의 서비스들에게 NodePort 타입으로 연결한다  
<https://www.bogotobogo.com/DevOps/Docker/Docker_Kubernetes_NodePort_vs_LoadBalancer_vs_Ingress.php>  
<https://medium.com/google-cloud/kubernetes-from-load-balancer-to-pod-3f2399637b0c>  
(서비스 자체는 Pod 처럼 실제 네트워크 인터페이스가 연결된 실체가 아니므로, NodePort 를 통해서만 접근할 수 있다. 자체적으로 외부 IP를 가질 수 없다)    
기본적인 동작은 이렇게 설명되지만, 이는 타입은 철저하게 클라우드 서비스 구현에 의존한다  
즉, 클라우드 서비스 마다 이 기능이 다를 수 있으며, 어떤 클라우드는 이 기능을 제공하지 않을수도 있다  
그리고 그림을 다시 보면, 외부 LoadBalancer에 cloud-controller 라는 애가 추가적으로 도움을 주고 있다  
사실상 Node가 늘어나고 줄어들수도 있고, service port도 변경될 수 있으니 이런애가 없으면 운영이 불가능했을 것이다  

ingress는 내부에 있다?  
L7 레벨까지 지원하는 로드밸런서이다  
외부에서 인그레스로는 어떻게 들어오는가?  
ingress는 어디에 있는가? 마스터 노드에 있는가? 클러스터 내에 있다면 왜 NodePort 모드로 생성되어야 하는가? 그냥 ClusterIP 모드로 생성되면 안되는가?  
인그레스까지 오는 것은 마찬가지로 로드 밸런서를 탄다(?)  

라우팅과 로드밸런싱의 차이?  

kube proxy : https://arisu1000.tistory.com/27839
service network : https://coffeewhale.com/k8s/network/2019/05/11/k8s-network-02/

# External Network(feat. Ingress)
라우터의 IP를 치고 들어오는것이 아닌..가..  
기본적인 DNS와 Route에 대한 개념이 필요하다


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