---
title: travis ci로 github blog 자동 배포
date: 2019-05-01 10:16:33
tags:
    - github
    - travis ci
    - hexo travis ci
---

1. github app에서 travis CI 설치
2. github setting - developer setting에서 personal access token 발급
3. .travis_ci, _config.yml 설정
4. push하고 travis에서 빌드 잘 되는지 확인

- theme
    > 테마는 submodule로 저장해줘야하고, 테마에서 _config.yml을 수정할 것이므로 theme fork한것을 submodule에 넣어야함

- travis ci app 설치 및 진행 방식  
    > <https://okky.kr/article/515352>

- .travis.yml, _config.yml은 여기를 참조하면 됨  
    > <https://medium.com/@changjoopark/travis-ci%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-github-pages-hexo-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-6a222a2013e6>


<!-- more -->