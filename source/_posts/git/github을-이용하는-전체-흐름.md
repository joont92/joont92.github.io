---
title: github을 이용하는 전체 흐름
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

<!-- more -->