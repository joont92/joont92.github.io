---
title: 'git fetch, pull 차이'
date: 2018-01-03 11:50:41
tags:
    - fetch
    - pull
---

<https://backlog.com/git-tutorial/kr/stepup/stepup3_2.html>  

fetch는 원격 저장소의 변경 내역을 가져오는 것이고 직접 로컬 branch에 반영하진 않는다  
push는 fetch 한 내역을 로컬 branch에 merge까지 한다.  

그러므로 pull은 branch를 지정해야 가져올 수 있고,  
fetch는 branch를 지정해도 되고, 원격 저장소를 지정해도 된다.  

fetch로 가져온 내용은 checkout할 수 있다.  
```
git checkout origin/develop
git checkout another-origin/develop
```

위처럼 말고도 fetch_head 로도 checkout 할 수 있는데, 정확히 무슨 기준으로 checkout fetch_head가 결정되는지는 모르겠다(1개 이상의 branch를 fetch 했을 경우)  

fetch 한 상태에서 `git merge` 혹은 `git pull` 입력 시 기존의 git pull과 동일한 행위를 하게 된다. 
(branch를 입력하지 않을 경우 config를 참조하게 됨)  

<!-- more -->