---
title: '[git] github을 이용하는 전체 흐름'
date: 2019-01-03 14:27:08
tags:
    - pull request
    - 코드리뷰
    - 오픈소스
---

1편 : <https://blog.outsider.ne.kr/865>  

2편 : <https://blog.outsider.ne.kr/866>  

1. repository fork 
2. local clone 
3. 개발하면서 local에 commit  
> 브랜치는 맞춰주는 것이 좋고, 원본 저장소를 따로 라모트로 추가해서 개발 중 변경사항을 주기적으로 pull 받아 맞춰주는 것이 좋다  
4. 개발이 완료되면 본인 remote repository push  
5. 원본 repository로 pull request 생성  
> 오픈소스 프로젝트마다 Pull Request를 받아주는 약속이 다르므로 보내기 전에 이를 먼저 확인해야 한다.  
> 소스 수정이 잘 되었더라도 이 약속을 제대로 지키지 않으면 받아주지 않는다.  
> 탭에서 Commits나 Files Changed를 클릭하면 Pull Request를 보내는 커밋과 변경사항이 제대로 되었는지 확인할 수 있다.  

Pull Request를 이용한 개발 흐름 : <https://blog.outsider.ne.kr/1199>  
1. 작업 시작과 동시에 PR을 올려서 내 작업 진행과정을 투명하게 공유한다
> 다른 작업자들이 내 작업 진행과정을 알 수 있어서 나중에 merge 작업이나 새로운 작업 시작시에 편리함. 코드 리뷰를 할 수 있다는 장점도.  
2. PR Comment에 할 일 목록을 만들고 체크리스트로 표시하도록 함. 작업중일떄는 WIP, 작업이 끝났을 경우 DONE

<!-- more -->