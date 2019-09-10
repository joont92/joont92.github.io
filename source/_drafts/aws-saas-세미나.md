---
title: 'aws saas 세미나'
tags:
---

SAAS를 운영하기 위해 필요한 리소스들에 대한 소개  

Bare Metal : 가상화를 사용하지 않고 그냥 서버를 사용하는 것   
On-Premise : 클라우드 환경이 아닌 직접 구성한 서버를 사용하는 환경  

# AWS overview
- 아마존에서 가지고 있던 기술을 리패키징하여 클라우드 서비스로 배포
- 클라우드 서비스를 사용하면 비즈니스를 쉽게 테스트해볼 수 있다
- 필요할 떄 마다 서버를 껐다 켜고, 추가하고 뺼 수 있으므로 비용을 절감할 수 있다(기본적인 비용이 싼것은 아님)
- 각 리소스들을 잘 이해하고 있어야 잘 끼워맞추고 합쳐서 좋은 서비스를 구축할 수 있다
- 리소스들을 합쳐서 큰 시너지를 낼 수 있음

- 리전들은 전부 아마존 백본으로 연결되어 있다(해저 케이블 등)
    - AWS Global Backbone
- 복수의 데이터 센터를 하나의 가용영역으로 만들고, 이를 묶어서 리전으로 정의함
- 프로덕션에서는 가용영역을 최소 2중화 이상 하는것이 좋음
- EC2 하이퍼바이저까지 AWS에서 관장
- 인프라를 웹 API로 만든것이 첫 시작(EC2, S3, SQS 등)
- 웹 콘솔보다 CLI이나 SDK 에서 제공하는 기능이 더 많다
- 아마존은 카테고리 하나하나가 전부 마이크로 서비스로 되어있다(약 3000개)

# EC2
- 가상머신이라고 생각하면 된다  
    ```
    게스트1 | 게스트2 | 게스트3
        하이퍼바이저
            호스트
    ```
- AMI(virtual machine 이미지)를 이용하여 인스턴스를 띄움  
    > 아마존 자체 AMI, 벤더 AMI, 사용자 AMI 등  
- EC2에 EBS 스냅샷을 연결?  
- 인스턴스 패밀리에 따라 용도가 각각 정해져 있다  
    - c(인스턴스 패밀리)4(인스턴스 세대).large(인스턴스 크기)
- 구매 방법에 따라 비용을 절감할 수 있다
    - On-Demand : 바로 인스턴스를 떙겨서 쓰는것이므로 요금이 비싸게 나옴
    - Reserved Instances : 일정 수준에 대해 약정을 함(On-Demand 기준에 비해 60프로 저럼?)
    - Spot Instances : 유휴 자원을 일정시간 동안 경매? On-Demand 대비 90프로 저럼
    > On-Demand로 트래픽을 분석한 뒤 구매 방법을 결정하는 것이 좋다
- 추천 : 최소 트래픽에 대해 Reserved Instances 를 설정하고, 동적으로 늘어나는 부분에 대해 On-Demand 로 설정
- CloudWatch를 이용해 성능을 측정한 뒤 EC2 인스턴스 scalable을 달성함
- auto scaling 구성요소
    - Launch Configuration : 어떤 AMI, 어떤 instacne type, 어떤 sg, 어떤 role?
    - Auto Scaling Group : 워크로드를 어디에 어떻게 배포할 것인가?
    - Auto Scaling Policy : 어떤 조건일 떄 배포할 것인가?
- load balancer가 cloud watch에게 정보를 주고, cloud watch가 auto scaling group 에 전달, auto scaling group 에서 load balancer에 전달?
- 윈도우, amazon linux, debian, redhat 등 다양한 운영체제 지원
- stop 동안에는 과금이 되지 않지만, 스토리지 사용량에 대해서는 과금이 된다

- 인스턴스 메타 데이터?
- ECS, EKS, Fargate

# Storage
- 블록 스토리지(EBS), 파일 스토리지(NFS), 오브젝트 스토리지(S3)
- EBS 볼륨
    - 사용한 만큼이 아닌 할당한 만큼 과금됨
    - SSD라 랜덤 IO
    - SSD(I/O 적합), HDD(스루풋 적합)
    - EC2에 EBS 볼륨을 연결하고, 볼륨의 스냅샷을 S3에 생성
- EFS(공유 파일 스토리지)
    - NFS라고 생각하면 됨
- S3(오브젝트 스토리지)
    - 할당한 만큼이 아닌, 사용한 만큼만 지불
    - 높은 안정성
    - static website hosting
        - 웹 서버를 관리할 필요없이 번들된 html이나 js를 올려 서비스
    - Standard(자주 Access) -> IA -> Glacier(조회가 거의 없지만 남겨야하는 데이터) 단위로 데이터 저장공간이 계층화 되어 있다
    - access 하지 않는 기간에 따라 계층이 내려가게 됨(No Access 30일 후 IA 로 내려감 등)
    - Event Notification = 오브젝트 POST, PUT, DELETE 등에 따라 추가적인 행위를 할 수 있게?
    - Intelligent Tiering = 사용자의 패턴을 분석하여 tier를 조정해주는 서비스
    - 데이터베이스의 일종이라고 봐도 될 정도의 기능을 제공한다
- SnowBall : On-Premise 장비의 데이터를 네트워크를 통해 S3로 옮길 때 시간이 너무 오래걸리는 경우가 있어서, 장비를 사무실로 직접 보내 데이터를 넣게끔하고, 아마존에서 받아서 S3로 직접 import
- SnowMobile : SnowBall로 부족해서 트레일러를 보내주는 서비스(100PB까지)

# 네트워크
클라우드지만 가장 기본 설계는 인프라적인 부분부터 시작해야한다  
가장 기본은 네트워크 설계이다  
그 위에 레이어들이 올려져서 서비스를 구성하게 된다  

VPC : 가상 네트워크 공간  
EIP : Ip를 따서 특정 서비스에 지정  
VPN, Direct Connect : VPC를 on premiss와 연결시켜주고 싶을때(논리적, 물리적(전용회선))  
ELB : managed Load Balancer  
Route53 : DNS 서비스  

모든 인프라는 AWS가 관리하는 백본 네트워크와 묶여 있다  
가용영역은 내부적인 LAN 환경이라고 보면 된다(하지만 건물이 여러개일 수 있다)  
이것들을 리전으로 연결함  
Transit 센터를 거쳐서 가용영역의 리소스에 접근하게 됨(평촌 KT 등)  

AZ 여러개로 다중화를 할 수 있고, DNS에서 멀티 에이징? 하게 함  

Global Service : Route53, IAM 등  
Regional Service : 네트워크 자원들. ELB, VPC Peering, VPN 등  
Availability-Zone Service : EC2, storage 등  

인터넷 게이트웨이를 통해야만 VPC가 외부로 통신 가능해진다  
On-Premise 에서 VPC 와 통신을 하려면 IPSec이나 DirectConnection(전용선)을 써야한다  

VPC 내에 있는 서비스들은 전부 private 서비스라고 부른다(private IP만을 가지기 때문이다)  
퍼블릭 서비스들(S3, DynamoDB 등)을 인터넷 게이트 웨이를 통해서 통신할수도 있지만, 비효율적이므로 엔드포인트를 제공해준다  
VPC와 VPC간 통신을 하고 싶다면 VPC peering을 설정해주면 된다  

## VPC
- 역할에 따라 VPC 의 서브넷을 나눔(외부 통신이 필요한 public subnet, 외부 통신이 필요없는 private subnet)  
- 서브넷 자체가 AZ를 넘나들 수 없다  
- 라우트 테이블 : 서브넷 간의 통신 규칙을 만들어 주는 것
- 인터넷 게이트웨이 : 서브넷이 외부 통신을 하기위해
- 각각의 서브넷은 모두 ACL 이라는 방화벽을 가지고 있다

- 네트워크의 특성상 보내는 IP와 받는 IP 대역이 같으면 라우팅이 불가능하다
    - 그러므로 VPC 설정 시 고려해야한다
- 예약된 IP 도 있다
- 각 리소스(엔드포인트) 또한 자체적인 보안을 가지고 있다(Security Group)
    - 모든 리소스가 기본적으로 다 가질수 있는..?
    - 앞에서 방화벽을 통과하더라도 리소스의 SG에서 막힐 수 있다
- 서브넷 별로 방화벽을 둘 경우 NACL
- VPC를 엔드포인트로 잡으면 앞에서 CDN이 많은것을 걸러주므로 ..
- ELB(L4)를 퍼블릭으로 등록하고, 내부에 서버들을 프라이빗으로 등록한 뒤 연결
- ELB에서 오토스캐일링을 위한 트리거도 가능
- private link를 통해 VPC를 다른 곳에 연결시켜줄 수 있음(뭘로 제공되나?)
- 리전간 VPC 피어링 가능?
- DirectConnect Gateway에 전용선을 연결하면 모든 VPC와 연결 가능?
- ALB : L7(컨텐츠를 봐야할 때), NLB : L4(속도가 중요할 때?)

- Route53 : DNS 서비스

## CloudFormation
- 템플릿으로 인프라를 구성하는 것, 우리의 인프라를 템플릿화 하는 것
- 우리 인프라 구성을 메뉴얼로 쓰는것이 아니라, 코드(?)로 작성(json, yaml 포멧 지원)
- AWS Quick Start : 자신의 제품들에 대한 CloudFormation 을 작성해서 제공해주는 것
    - 고객이 선택할수 있도록 옵션을 제공한다
- 입력값에 대한 설정, 파라미터 등 많은 부분 설정 가능
- 함수도 제공한다(getAZ 등)
- Cloud Formation 문서, Quick Start 에 나와있는 부분들을 참고하면 패턴들을 금방 발견할 수 있다
- 인스턴스 내부 스크립트도 지정 가능하다?
- 메타데이터 지정가능
- 파일 액세스 가능
- 설정 파일을 업데이트하고 적용하면, 변경된 부분만 찾아서 변경을 적용함
- 롤백도 가능함
- 스택 삭제시 CloudFormation 으로 생성된 모든 리소스가 다 지워짐
    - 특정 리소스는 지워지지 않도록 설정할수도 있다(데이터는 남겨둬! 등)
- 환경을 구분하여 스택을 나눠서 띄울수도 있다?
- nested stack도 설정 가능하다(stack 안에 stack을 설정)
    - stack 별로 팀을 나눠서 수정관리 할 수 있도록 권한을 나눌수도 있다

# Database
- 동접 100만명 서비스?
- 마이크로 세컨드의 요구사항을 충족하려면 인메모리 데이터베이스를 사용해야한다(레디스)
    - RDB는 최대 밀리세컨드, SSD가 아무리 빨라도 마이크로 세컨드 단위까지 내려갈 수 없다
- 도큐먼트 DB : JSON을 그대로 저장
- 인메모리 데이터 베이스를 하드디스크로 내려서 쓸수도 있지만, 그렇게 잘 쓰지는 않는다
- 그래프 DB : SNS(친구찾기)
- Time series : 시계열 데이터, 주식
- AWS Document DB는 DynamoDB
- RDS 오토스케일링(업)도 지원된다
- RDB에서 확장은 읽기 확장과 쓰기 확장 2가지가 있다(CUD, R의 경계치를 잘 구분해야한다)
    - Read Replica
    - Write Scalable
- Aurora는 글로벌 레플리케이션이 된다
- Master에서 Slave로 데이터를 다 보내야지 커밋으로 인정한다?
- 어떤 경우든 데이터베이스를 1대로만 운영해서는 안된다
- DynamoDB
    - Relation feature가 없다면 dynamoDB를 고려해보는 것이 좋다
    - 파티션키, 소프트키
    - 하나의 키로 샤드를 계속 나눠버리기 때문에 계속해서 같은 50ms 이하의 레이턴시를 제공할 수 있다
    - 어떤것을 샤드키로 쓸지 고민해야한다
- RedShift?
- Redis 정도의 속도를 제공해야 하는데 데이터 영속화를 보장해야 할때?
    - 보통은 Redis의 마스터, 슬레이브 구성만 해도 충분하다
    - 혹은 계속해서 큐(kafka, kinesis)에 보내서 S3에 저장하는 방법도 있다

# 컨테이너
- 많은 엔터프라이즈에서 Kubernetes 기반의 Paas를 많이 개발하고 있다  
- Docker Daemon : 도커를 관리하기 위해 뜨는 데몬, 윈도우나 맥은 가상머신을 띄우고 리눅스를 띄운 뒤 도커 데몬을 띄운다(?)  
- ECR : AWS의 도커 레지스트리
- 도커 오케스트레이션 툴 : **kubernetes**, ECS, swarm, apache mesos 등
    - ECS : 완전 관리형
    - EKS : fully managed는 아님
    - Fargate : 완전 관리형

- 콘솔로 EKS를 컨트롤하면 마스터노드만 만들어준다
- On-Premise나 GCP에서 운영하던것을 EKS로 마이그레이션 하기 쉽다
- 전세계 kubernetes의 51 프로가 AWS 위에서 돌아간다고 한다
- kubernetes cluster 하나 구축하는데 굉장히 많은 시간이 들어가는데, EKS를 사용하면 15분이면 클러스터를 구축할 수 있다
- 각 노드에는 마스터 노드로부터 api 를 받는 kubelet 이 있다
- 마스터 노드와 통신을 하기위한 kube-proxy 가 있다
- 마스터 노드 내의 관리용 컨테이너
    - kube-apiserver
    - etcd : 마스터의 상태 저장?
    - kube-scheduler 
    - kuber-controller-manager
- e.g. 노드에 GPU를 사용하는 노드라고 레이블을 달면 GPU가 필요한 컨테이너들을 해당 노드에 띄울 수 있다
- 쿠버네티스는 RBAC을 사용한다
- EKS를 통해 pod 생성 등을 요청하면 AWS에서 IAM 등을 확인하고 통과되면 명령을 수행한다
- EKS는 config에 베어러토큰이 들어가지 않고 aws command가 적혀있고, 이 command를 통해 토큰을? 받아와서 명령을 수행한다
- 플랫폼 종속성을 뜯기 위해 쿠버네티스를 사용한다
- 서비스에 LoadBalancer 타입 선언하고, ingress랑 포트 맞춰놓으면 eks를 통해 띄울 때 elb를 생성한다?

# 시큐리티
- 고객마다 VPC를 통해 격리된 네트워크를 구성함으로써 보안을 높인다
- EC2 - SG - Subnet - NACL - Route Table - Gateway or ...
- VPC내의 서브넷을 나누고, 리소스를 역할별로 맞게 놔둬야 한다
- 퍼블릭 서브넷에 load balancer, 프라이빗 서브넷에 tomcat, db 등을 놔둬야 한다
- private subnet 안에 있는 서비스들도 yum update 등으로 라이브러리 업데이트 하는 등 인터넷을 써야하는데, 이럴떄는 NAT 게이트웨이를 보도록 해야한다
- Security Group 에 source에 다른 리소스의 id를 적을 수 있다
- bastion : private resource에 접근하기 위한 통로
- NACL은 디폴트로 다 열려있다
- DDOS 공격이 SaaS 서비스로는 생각보다 많이 일어난다
- 작년 서울 리전 사고는 DNS 서버 문제였다(용량이 0으로 수렴하면서 DNS를 사용하는 모든곳이 고장남)

- GuardDuty : 네트워크적인 패턴 뿐 아니라 좀 더 광범위한 위협 탐지 서비스
    - DNS log, VPC flow log, Cloud Trail log 를 읽어서 위협을 찾으면 통보하고, 대응한다(로그를 읽는다!)
    - 들어가서 그냥 켜기만 하면 해당 계정내에 있는 리소스들의 로그를 모니터링 하여 위협을 알려준다
    - 이를 CloudWatch에 연결해서 추가적인 행위를 할 수 있다
    - 해킹이라고 판단되면 EC2 서버의 SG를 모두 deny로 변경해버리고, 해킹 당했을떄의 상태를 스냅샷 뜨고, 포렌식 서버에 보낸다

- SaaS를 만들떄는 웹방화벽에 대한 고민을 해야한다
- WAF(웹방화벽)
    - 기존에는 서드파티 웹 방화벽을 사용했다?
    - 설치형 웹 방화벽은 스케일러블이 되지 않아서 웹 방화벽이 트래픽을 받고 다 죽어버리면 밑에 애들이 트래픽을 하나도 받지 못함
    - 클라우드의 확장성과 스케일러블에 맞는 제품은 WAF이다
    - AWS WAF Security Automation? (+CloudFormation?)
    - managed rule for AWS WAF

- 세상에 모든 DDoS를 막아주는 서비스는 존재하지 않는다
- AWS Shield(DDos 보호 서비스)
    - 굉장히 강력한 DDoS 방어 서비스
    - Standard Protection : layer 3,4 정도에서 들어오는 공격들을 무료로 막아주고, 자동으로 적용되고 있다
    - Advanced Protection : layer 7 까지 막아줌
        - 등록한 AWS 리소스를 등록하면 AWS DDoS 팀에서 이를 매번 확인하고, WAF를 업데이트하거나 해준다
        - 휴먼 리소스가 들어가기 때문에 월 3000달러 부터 시작한다