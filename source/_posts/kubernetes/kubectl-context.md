---
title: '[kubernetes] kubectl context'
date: 2019-07-03 22:10:49
tags:
    - kubectl context change
    - kubectx
---

쿠버네티스 클러스터를 관리하는 cli 도구인 kubectl에는 환경을 바꿔가며 클러스터를 관리할 수 있는 context 라는 기능?이 존재한다  
예를 들어 내 로컬 pc에 설치된 쿠버네티스 클러스터용 context 를 사용하면 kubectl 명령으로 내 로컬 쿠버네티스 클러스터를 컨트롤 할 수 있게되며,  
GCP에 있는 쿠버네티스 클러스터용 context를 사용하면 kubectl로 GCP 쿠버네티스 클러스터를 컨트롤 할 수 있게 되는 것이다  

# 설정
context 는 kubectl 을 깔면 생성되는 파일인 `~/.kube/config` 파일에서 설정할 수 있다  
아래는 내 PC의 config 파일이다(보기쉽게 조금 수정했다)  

```
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://localhost:6443
  name: local-cluster
- cluster:
    certificate-authority-data: ~~~~
    server: https://xxx.xxx.xxx.xxx
  name: gcp-cluster

users:
- name: local-user
  user:
    blah blah
- name: gcp-user
  user:
    blah blah

contexts:
- context:
    cluster: gcp-cluster
    user: gcp-user
  name: gcp-context
- context:
    cluster: local-cluster
    user: local-user
  name: local-context

current-context: local-context

kind: Config
preferences: {}
```
> 크게 clusters, users, contexts 가 있다  

- clusters 
    > 말 그대로 쿠버네티스 클러스터의 정보이다  
    > 내 PC에 설치된 쿠버네티스 클러스터와, GCP에 설치된 쿠버네티스 클러스터가 있음을 볼 수 있다  
    > 각 클러스터의 이름을 local-cluster, gcp-cluster로 수정해서 알아보기 쉽게 해놓았다  
    > 처음 클러스터 생성하면 조금 복잡?한 이름으로 생성되는데, 위처럼 자신이 알아보기 쉽게 바꿔주는 것이 좋다  
- users
    > 클러스터에 접근할 유저의 정보이다  
    > 각 환경마다 필요한 값들이 다르다  
    > 이 또한 알아보기 쉽게 local-user, gcp-user 로 수정해놓았다  
- context
    > cluster와 user를 조합해서 생성된 값이다  
    > cluster의 속성값으로는 위에서 작성한 cluster의 name을 지정했고, user의 속성값 또한 위에서 작성한 user의 name을 지정했다  
    > local-context는 local-user 정보로 local-cluster에 접근하는 하나의 set가 되는 것이다  
- current-context  
    > 현재 사용하는 context 를 지정하는 부분이다  
    > 현재 local-context 를 사용하라고 설정해놓았으므로, 터미널에서 kubectl 명령을 입력하면 로컬 쿠버네티스 클러스터를 관리하게 된다  

# context 조회 및 변경(feat. kubectx)
가장 단순한 방법으로는 `~/.kube/config` 파일을 컨트롤하여 context를 조회하거나 수정하는 방법이 있고,  
그 다음 방법으로는 `kubectl config` 명령을 이용하는 방법이 있다  
```sh
# gcp-context 로 변경
$ kubectl config use-context gcp-context

# context 조회
$ kubectl config get-contexts

CURRENT   NAME            CLUSTER         AUTHINFO     NAMESPACE
*         gcp-context     gcp-cluster     gcp-user
          local-context   local-cluster   local-user
```

하지만 이것보다 `kubectx` 라는 더 최적화된 도구가 있다  
> github : <https://github.com/ahmetb/kubectx>  

간단하게 brew 로 설치할 수 있다  
아래는 간단한 사용법이다  
```sh
# context 조회
$ kubectl
gcp-context # 노란색으로 표시
local-context

# local-context 로 변경
$ kubectx local-context

# 이전 context로 돌아가기
$ kubectx -
```