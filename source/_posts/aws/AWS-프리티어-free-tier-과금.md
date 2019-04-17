---
title: AWS 프리티어(free-tier) 과금
date: 2019-04-04 12:38:25
tags:
    - AWS
    - AWS 프리티어
    - AWS 프리티어 과금
---
asdasd
어제 밤에 갑자기 AWS에서 29달러 정도가 결제되어서 급하게 확인했더니, 프리티어 기간이 끝나서 과금이 되었었다... 아놔..  
그래서 그 내용을 정리하고자 한다  

# 프리티어?  
AWS는 가입시 1년간 특정 리소스들을 무료로 사용할 수 있는 `프리 티어` 권한을 가질 수 있다  
프리티어에서 1년간 무료로 사용가능한 리소스들은 아래와 같다  
<https://aws.amazon.com/ko/free/?awsf.Free%20Tier%20Typeasdasds=categories%2312monthsfree&awsm.page-all-free-tier=2>  

# 프리티어인지 확인하는 법  
`AWS 로그인 - 내 결제 대시보드` 로 접속한다  

1. `청구서` 탭에서 날짜 드롭박스를 보고 언제 가입했는지 알 수 있다. 여기서 1년이 아직 안 지났는지를 체크해보면 된다.  
2. 대시보드 홈에서 `경고 및 알람` 부분에 free tier 관련 내용이 써져있으면 아직 free tier라는 의미이다  
    <https://docs.aws.amazon.com/ko_kr/awsaccountbilling/latest/aboutv2/free-tier-eligibility.html>  

# 과금을 해결하는법  
프리티어 기간이 지났으면(혹은 프리티어라도 일정 수준 넘으면 과금될 수 있음) 과금이 될 리소스를 정리해야함  
위의 1년간 무료로 사용할 수 있는 리소스와 아래 정보들을 조합해서 과금 대상인 리소스들을 다 정리해야함  
<https://docs.aws.amazon.com/ko_kr/awsaccountbilling/latest/aboutv2/checklistforunwantedcharges.html>  

## 내 경우
EC2와 RDS에서 과금이 되고 있었음  
RDS 들어가서 해당 데이터베이스를 삭제시켰고, EC2에서 인스턴스를 terminated 시켰음  

<!-- more -->