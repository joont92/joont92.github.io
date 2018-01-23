---
title: hexo와 github pages로 블로그 만들기
tags:
---

이번엔 앞서 작성한 github-pages에 블로그 서비스를 하기 위해 

정적 사이트 생성 도구인 hexo에 대해 알아보겠습니다.

<br/>

# Hexo

블로그 형태의 정적사이트를 생성하는데 사용되는 도구입니다.

hexo는 사용자가 작성한 포스트(markdown 등)을 읽어서, 

정적파일 생성기를 통해 웹서버에 바로 서비스 할 수 있는 형태의 정적 웹사이트를 만들어냅니다.

대표적인 것으로 jekyll이 있지만 hexo가 좀 더 편해보이고 테마도 맘에 들어서 hexo를 사용하기로 했습니다 ㅎㅎ

<br/>

## 설치

> 사전준비 : Node.js,npm,git

바로 설치하고 초기화 해보겠습니다.

```bash
npm install -g hexo-cli
hexo init '폴더명'
```

'폴더명'에 입력한 폴더를 만들고 그 폴더에 hexo 관련 파일을 초기화합니다.

(폴더를 지정하지 않으면 현재 폴더에 초기화하는데, 현재 폴더가 비어있는 상태여야 합니다.)

아래는 초기화 후 폴더 모습입니다!

![image](https://user-images.githubusercontent.com/18513953/30768882-f384f9ba-a049-11e7-8e1c-66bb64603b72.png)

빨간색으로 표시해둔 **_config.yml**에서 블로그에 대한 대부분의 설정을 할 수 있습니다.
<br/>
<br/>

초기화가 완료되면 간단하게 로컬에서 테스트 해보도록 할까요

해당 폴더로 이동하여

```bash
hexo server
```

라고 입력하면, 
> INFO  Start processing

> INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

라는 메시지와 함께 **http://localhost:4000** 으로 접속 가능합니다.

기본 테마로 생성된 정적 블로그 페이지를 볼 수 있을 것입니다 ㅎㅎ

<br/>

## 테마 적용

하지만 이대로 사용할 순 없으니 테마를 한번 적용해보도록 하죠.

적용방법은 매우 간단합니다.

<https://hexo.io/themes/index.html>

위의 주소에 접속한 뒤, 마음에 드는 테마를 고르시면 됩니다.

각 테마의 github 페이지에 들어가면 테마 적용 방법에 대한 상세한 설명이 있으니 별로 어려움 없으실 겁니다 ㅎㅎ

제가 고른 테마는 Material Flow 라는 테마입니다.

> gitgub : <https://github.com/stkevintan/hexo-theme-material-flow>

보시다시피 매우 간단합니다. 소스를 clone받고 _config.yml에서 해당 테마로 지정해주기만 하면 됩니다.

```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: material-flow
```

설정이 다 되었으면 

```bash
hexo clean
hexo generate # 정적 리소스 생성
```

와 같이 입력하여 정적 리소스를 생성해주면 됩니다.

간혹 제대로 되지 않는 경우도 있기 떄문에 clean도 한번 해줬습니다.

이제 다시 hexo server 입력 후 들어가보시면 테마가 잘 적용되어 있음을 보실 수 있습니다!

<br/>

## 글을 써보자

블로그를 만들었으니 글을 써야겠네요.

```bash
hexo new post [post_name]
# ex) hexo new post 'first post'
# ex) hexo new post first-post
```

과 같이 입력하면, ```source/_post``` 폴더에 아래와 같은 형태로 markdown 파일이 하나 생성됩니다.

```md
---
title: first post
date: 2017-09-23 10:51:08
tags:
---
```

각종 폴더나 카테고리에 대한 설정도 **_config.yml**에서 할 수 있으니 각자 설정하시면 됩니다 ㅎㅎ

<br/>
<br/>

## 배포

이제 github에 배포해보도록 하겠습니다 ㅎㅎ

먼저 **_config.yml**에 deploy 관련 설정을 해 줍니다.

```yml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/joont92/joont92.github.io.git
  branch: master
```

저장한 뒤

```bash
hexo clean

hexo generate # 정적파일 생성하고
hexo deploy # 배포!

# hexo deploy --generate 로도 가능
```

와 같이 해주면 끝입니다. 매우 간단하죠??

<br/>

#### 배포시 아래와 같은 메시지와 함께 배포가 되지 않는 경우

```
ERROR Deployer not found: git
```

**hexo-deployer-git** 플러그인을 설치해주면 됩니다.

```
npm install hexo-deloyer-git --save
```

<br/>

여기까지입니다 ㅎㅎ

블로그에 markdown을 사용할 수 있고, git의 형상관리를 블로그에 사용할 수 있다니 매우 좋은것 같네요.

들려주셔서 감사합니다~~