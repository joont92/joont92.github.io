---
title: '[linux] 우분투 intellij에서 mac keymap 사용시 이슈'
date: 2018-03-22 18:07:27
tags:
    - ubuntu intellij
---

윈도우 키(대시)
https://reachlabkr.wordpress.com/2014/12/06/ubuntu-%EC%97%90%EC%84%9C-supercommand-key-shortcut-%ED%95%B4%EC%A0%9C/

compiz 설치후에
ubuntu unity plugin
launcher에 젤 상단 dash 부분 사용하지 않음으로 변경
난 super space로 바꿨는데, mac spotlight 느낌이 난다  

윈도우 w키(창 닫기)
창 관리 - 스케일에 가면 있음. super + tab 으로 바꾼건 윈도우 같다고 함

파라미터 보기
https://askubuntu.com/questions/68463/how-to-disable-global-super-p-shortcut

sudo apt-get install dconf-tools
dconf-editor
위처럼 따라간 뒤 xrandr 해제

근데 .. 좀 늦게 눌러야 인식됨 ㅡㅡ