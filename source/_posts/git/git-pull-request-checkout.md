---
title: '[git] git pull request checkout'
date: 2019-05-01 16:55:27
tags:
    - github PR
    - pull request
---

# 직접 checkout
git config 추가
```sh
$ git config --add remote.origin.fetch "+refs/pull/*/head:refs/remotes/origin/pr/*"
```
fetch로 가져올 때 origin 말고 pr도 가져올 수 있게함

```sh
$ git fetch origin
$ git checkout origin/pr/[PR number]
```

# intellij 에서 checkout
<https://blog.jetbrains.com/idea/2018/10/intellij-idea-2018-3-eap-github-pull-requests-and-more/>  
shift 2번 눌러서 search everywhere 실행시킨 뒤 `view pull request` 클릭  
생성된 pr에서 보고 싶은 pr을 선택하여  
local branch 만들고 checkout  

<!-- more -->