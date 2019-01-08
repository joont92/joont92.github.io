---
title: git-crypt
date: 2019-01-08 14:27:56
tags:
    - git-crypt 적용
---

# git-crypt란?
git-crypt를 사용하면 특정 파일 또는 폴더를 `원격 repository에 올릴때는 encrypt하고, 로컬로 내려받을때는 decrypt 하는 식`으로 관리할 수 있다.  

git-crypt repostiroy <https://github.com/AGWA/git-crypt>  
git-crypt 적용법 <https://s-opensource.org/2017/07/18/introduction-gpg-encryption-git-crypt/>  


# 적용법
```shell
brew install git-crypt

cd /path/to/repository

git-crypt init
```

하면 .git/git-crypt가 추가되고, git-crypt 할 수 있게 됨  

그리고 repository root 경로애 `.gitattributes` 파일을 만들고 내부에 git-crypt가 적용될 파일을 적어주면 된다.  

```
config/production/** filter=git-crypt diff=git-crypt
# more...
```

원하는 패턴이 더 있을 경우 밑에 나열해주면 된다.(`.gitignore`과 비슷하게 작성한다)  

기본적으로 초기에 git-crypt를 설정한 사람은 올리고 내릴때 자동으로 encrypt decrypt가 되지만,  
다른 collaborator들은 적용되지 않으므로 추가적인 설정이 필요하다.  

1. gpg 적용  
    > 상단의 `git-crypt 적용법` 보고 따라하면 될 듯
    1. 추가하고 싶은 collaborator의 pc에서 gpg 생성
    2. project 생성자가 `git-crypt add-gpg-user` 로 collaborator의 gpg를 추가
    3. pull 받은 collaborator는 `git-crypt unlock` 으로 간단하게 unlocking 가능  
    4. 테스트 안해봄  

2. key export import
    > encrypt 할 때 사용한 key를 export하고, collaborator는 해당 key로 unlock 하는 방법  
    1. repository의 git-crypt key값을 export 하여 collaborator에게 전달하고,  
        ```
        git-crypt export-key /path/to/key
        ```
    2. 전달받은 collaborator는 해당 key 파일을 사용하여  
        ```
        git-crypt unlock /path/to/key
        ```
        의 형태로 decrypt 한다.  

# 기타  
1. 만약 키를 공유하고 싶으면?  
한쪽 레파지토리에서 생성된 `.git/git-crypt/keys/default`를 다른 레파지토리에서 경로 그대로 복사해서 사용하면 됨  
근데 이 방식으로 `git-crypt unlock`이 바로 되지는 않는다. 번거롭게 key 파일 위치를 지정해줘야 한다.  

<!-- more -->