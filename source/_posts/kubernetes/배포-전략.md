---
title: '[kubernetes] 배포 전략'
date: 2019-07-13 18:49:46
tags:
  - 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문
  - RollingUpdate
  - Blue-Green
---

쿠버네티스는 2가지 방법으로 무중단 배포를 수행할 수 있다  

# 롤링 업데이트
Deployment 속성중에 `.specs.strategy.type` 값을 통해 Pod 교체전략을 지정할 수 있는데, 여기서 `RollingUpdate` 를 사용하는 방법이다  
RollingUpdate는 우리가 잘 알다시피, Pod를 하나씩 죽이고 새로 띄우면서 순차적으로 교체하는 방법이다  
> RollingUpdate, Recreate 2개의 값이 존재하며 Recreate의 경우 기존 파드를 모두 삭제한 다음 새로운 파드를 생성하는 방법이다(이 방식은 무중단이 아니다)  
> 기본값은 RollingUpdate 이다  

RollingUpdate를 테스트하기 위해 먼저 deployment 와 service를 정의한다  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name : test-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: test-pod
  template:
    metadata:
      labels:
        app: test-pod
    spec:
      containers:
        - name: test-pod
          image: joont92/echo-version:1.0 # 단순히 요청을 받으면 version을 리턴해주는 서버이다
          ports:
            - containerPort: 8080
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: test-pod
```
> deployment의 기본 속성이 RollingUpdate 이므로 따로 RollingUpdate 부분을 명시하지 않았다  

이제 다른 Pod에서 위 서비스명으로 호출하면 `1.0` 이라는 버전을 리턴해주게 된다  
이를 `2.0`을 리턴해주는 이미지로 바꿀건데, RollingUpdate 를 이용하여 바꿔보도록 할 것이다 

먼저 위의 서비스 명으로 호출하는 Pod를 하나 만들어야한다  
```yml
apiVersion: v1
kind: Pod
metadata: 
  name: update-checker
spec:
  containers:
    - name: update-checker
      image: joont92/update-chekcer:latest # curl 이 깔려있는 alpine 리눅스
      command:
        - sh
        - -c
        - |
          while true
          do
            echo "[`date`] curl -s http://test-service/"
            sleep 1
          done
```

이 Pod가 뜨게되면 1초마다 test-service를 호출하게 되고, test-service 에서 사용하는 Pod 가 받아서 0.1.0 이라는 값을 리턴해주게 될 것이다  
```
[Sat Jul 13 14:08:27 UTC 2019] APP_VERSION=0.1.0
[Sat Jul 13 14:08:28 UTC 2019] APP_VERSION=0.1.0
...
```

이제 Deployment 내 Pod의 이미지를 바꿔서 적용시켜보겠다  
먼저 변경할 내용을 작성하고,  
```yml
spec:
  template:
    spec:
      containers:
        - name: echo-version
          image: joont92/echo-version:0.2.0 # 버전 변경
```

기존의 Deployment 에 적용한다  
```sh
$ kubectl patch deplyment test-deployment -p "$(cat patch-deployment.yaml)"
```

참고로 아래처럼 할수도 있다  
```sh
$ kubectl set image deployment test-deployment test-pod=joont92/echo-version:0.2.0
```

변경사항을 적용하자, 컨테이너가 하나씩 삭제되고 생성되는 것을 볼 수 있다  
```sh
$ kubectl get pod -l app:test-pod -w
NAME                               READY   STATUS              RESTARTS   AGE
test-deployment-6b8d4f7967-9gwxl   0/1     Terminating         0          46s
test-deployment-6b8d4f7967-f9nlf   1/1     Running             0          48s
test-deployment-6b8d4f7967-knrws   1/1     Running             0          48s
test-deployment-6b8d4f7967-wkd4l   0/1     Terminating         0          46s
test-deployment-86658bfcfd-4kf2z   1/1     Running             0          3s
test-deployment-86658bfcfd-ckxvx   0/1     ContainerCreating   0          1s
test-deployment-86658bfcfd-d4znq   0/1     ContainerCreating   0          3s
test-deployment-86658bfcfd-d4znq   1/1     Running             0          4s
test-deployment-6b8d4f7967-knrws   1/1     Terminating         0          49s
test-deployment-86658bfcfd-s96t6   0/1     Pending             0          0s
test-deployment-86658bfcfd-s96t6   0/1     Pending             0          0s
test-deployment-86658bfcfd-s96t6   0/1     ContainerCreating   0          1s
test-deployment-6b8d4f7967-wkd4l   0/1     Terminating         0          49s
test-deployment-6b8d4f7967-wkd4l   0/1     Terminating         0          49s
test-deployment-86658bfcfd-ckxvx   1/1     Running             0          4s
test-deployment-6b8d4f7967-f9nlf   1/1     Terminating         0          51s
test-deployment-6b8d4f7967-knrws   0/1     Terminating         0          52s
test-deployment-6b8d4f7967-knrws   0/1     Terminating         0          52s
test-deployment-6b8d4f7967-knrws   0/1     Terminating         0          52s
test-deployment-86658bfcfd-s96t6   1/1     Running             0          4s
test-deployment-6b8d4f7967-f9nlf   0/1     Terminating         0          54s
test-deployment-6b8d4f7967-f9nlf   0/1     Terminating         0          54s
test-deployment-6b8d4f7967-f9nlf   0/1     Terminating         0          54s
test-deployment-6b8d4f7967-9gwxl   0/1     Terminating         0          53s
test-deployment-6b8d4f7967-9gwxl   0/1     Terminating         0          53s
```

## 롤링 업데이트 동작 제어
Deployment의 `.specs.strategy` 속성을 설정하여 롤링 업데이트의 동작을 제어할 수 있다  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name : test-deployment
spec:
  replicas: 4
  strategy: # 요기
    type: RollingUpdate
    rollingUpdate:
      maxUnvailable: 3
      maxSurge: 4
  selector:
    matchLabels:
      app: test-pod
  template:
    metadata:
      labels:
        app: test-pod
    spec:
      containers:
        - name: test-pod
          image: joont92/echo-version:0.1.0
          ports:
            - containerPort: 8080
```
`maxUnvaliable`은 롤링 업데이트시 동시에 삭제할 수 있는 파드의 최대 개수를 의미하고, `maxSurge`는 동시에 생성될 수 있는 파드의 최대 개수를 의미한다  
(둘 다 기본값은 replicas 값의 25%이다)  
이 값을 적절히 수정하면 RollingUpdate 시간을 단축할 수 있으나, 갑자기 한 Pod에 부하가 몰리거나 서버의 리소스가 잠깐 급증하는 사이드이팩트도 있으니 잘 설정해줘야 한다  

# 블루-그린
롤링 업데이트는 강력하지만, 구버전과 새버전이 공존하는 시간이 발생한다는 단점이 있다  
이 문제를 해결하기 위해 사용되는게 이 블루-그린 배포 인데, 이는 서버를 새버전과 구버전으로 2세트를 마련하고, 이를 한꺼번에 교체하는 방법이다  

기존에 아래와 같이 Service, Deployment가 생성되어 있었다고 치자  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name : test-deployment-blue
spec:
  replicas: 4
  selector:
    matchLabels:
      app: test-pod
      color: blue
  template:
    metadata:
      labels:
        app: test-pod
        color: blue
    spec:
      containers:
        - name: test-pod
          image: joont92/echo-version:0.1.0 # 구버전
          ports:
            - containerPort: 8080
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: test-pod
    color: blue
```

blue-green 배포를 하기 위해, 새로운 버전이 적용된 Deployment 세트를 하나 더 준비한다  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name : test-deployment-green
spec:
  replicas: 4
  selector:
    matchLabels:
      app: test-pod
      color: green
  template:
    metadata:
      labels:
        app: test-pod
        color: green
    spec:
      containers:
        - name: test-pod
          image: joont92/echo-version:0.2.0 # 신버전
          ports:
            - containerPort: 8080
```

아래가 블루-그린 배포를 하기 위한 핵심 부분이다  
```yml
spec:
  selector:
    color: green
```

```sh
$ kubectl patch service test-service -p "$(cat patch-service.yaml)"
```

test-service의 label selector가 업데이트 됨으로써, test-service는 새로 배포된 Pod 들만을 바라보게 변경되었다  
이처럼 구버전과 신버전이 공존하는 텀이 없이 바로 신버전으로 전환할 수 있다(!!)  
게다가 만약 새로 배포된 버전에 문제가 발생한다면, 다시 test-service의 label을 blue로 돌려줌으로써 쉽게 롤백도 가능하다  

기존에 물려있던 트래픽에 대해서는.. 롤링배포와 블루그린 배포에서 차이점이 뭘까?

참고 :  
- [야마다 아키노리, 『도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문』, 심효섭 옮김, 위키북스(2019)](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158391447&orderClick=LEA&Kc=)