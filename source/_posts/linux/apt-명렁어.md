---
title: apt 명렁어
tags:
---

# 패키지란
커널 및 라이브러리 버전의 배포판 환경에 맞추어 빌드한 실행파일을 압축한 것.

# apt란
우분투에서 쓰이는 데비안 계열의 패키지를 관리하는데 쓰이는 도구이다.  
> 패키지 저장소 리스트 : /etc/apt/sources.list  

이 패키지 저장소 덕분에 일일히 홈페이지를 검색하며 들어가는 등의 수고를 할 필요가 없어지게 된다.  

> 하지만 모든 프로그램이 우분투 공식 패키지 저장소에 들어갈 순 없다.  
그래서 PPA(Personal Package Archivce)라는 개인 패키지 저장소를 이용한다.  
(공식 패키지 저장소가 느릴때 PPA를 사용하기도 한다. 공식 저장소에는 몇만개의 패키지가 있으므로..)  
해당 패키지 저장소를 apt 패키지 저장소에 추가해주면 해당 패키지 저장소를 통해 패키지를 내려받을 수 있게 된다.  
저장한 PPA 목록은 /etc/apt/sources.list.d 에서 확인할 수 있다.
```shell
#추가
sudo add-apt-repository '저장소이름'
#삭제
sudo add-apt-repository --remove '저장소이름'

#이후작업
sudo apt-get update # 저장된 패키지 저장소를 토대로 패키지 목록을 업데이트 한다
sudo apt-get install '패키지명' # 패키지를 다운로드한다.
```

### 패키지 리스트 업데이트
```
apt-get update
```
\> 실제 패키지를 업그레이드 하는 것이 아니라 사용가능한 패키지 리스트의 정보를 업데이트

### 패키지 업데이트
```
apt-get upgrade
```
\> 실제 설치되어 있는 패키지들을 최신 버전으로 업그레이드

### 패키지 설치
```
apt-get install [패키지명]
```

### 패키지 재설치
```
apt-get --reinstall install [패키지명]
```

### 패키지 삭제
```
apt-get remove [패키지명] # 설정파일은 지우지 않음
apt-get purge [패키지명] # 설정파일까지 지움
```

### 패키지 검색
```
apt-cache search [패키지명]
```

### 패키지 정보
```
apt-cache show [패키지명]
```

apt 명령어를 이용해 설치한 패키지는 \/var\/cache\/apt\/archives 에 설치된다.