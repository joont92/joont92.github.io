---
title: apt 명렁어
tags:
---

# 패키지란
커널 및 라이브러리 버전의 배포판 환경에 맞추어 빌드한 실행파일을 압축한 것.

# apt란
우분투에서 쓰이는 데비안 계열의 패키지를 관리하는데 쓰이는 도구이다.

저장소 리스트 : /etc/apt/sources.list

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