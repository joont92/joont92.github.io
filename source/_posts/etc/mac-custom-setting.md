---
title: 'mac custom setting'
date: 2018-11-22 21:49:49
tags:
    - mac 한영키
    - mac home/end
    - mac mouse
---

mac을 편리하게 사용하고자 직접 세팅한 내용을 공유하고자 한다.  

# 일반 키보드
레오폴드 키보드 사용  

1. 한영 전환  
아래의 내용을 btt에서 똑같이 사용했다  
<https://jojoldu.tistory.com/345?category=798573>  

1. alt키를 command key로 사용하기  
`환경설정 - 키보드 - 보조키`에서 option, command 키 바꿀 수 있음

1. home, end 사용하기  
KeyBindins/DefaultKeyBinding.dict를 수정하면 된다  
<http://junho85.pe.kr/580>  
> key bindings 값 찾기  
<http://junho85.pe.kr/579>  
기

1. windows의 aero snap 사용하기  
`btt - keyboard`  
Command + Control + 방향키로 `Maximize Window Left, Right, Top Half, Bottom Half` 를 설정했다  

1. 최대화  
`btt - keyboard`  
Command + A = 현재 위치에서 최대화  
Command + S = 다른 모니터(듀얼모니터)로 넘기기  
Command + D = 다른 모니터로 넘기기 + 최대화  

# 일반 마우스
앞으로 가기, 뒤로가기가 있는 버티컬 키보드 사용  

1. 앞으로 가기, 뒤로가기 매핑  
`btt - normal mice`  
button3에 뒤로가기, button4에 앞으로 가기 매핑했다.  
action을 사용하지 않고 Command + [, Command + ]로 매핑했다.  
이렇게 하면 intellij 에서도 마우스로 앞뒤로 왔다갔다 할 수 있다.  

1. swipe 매핑  
`btt - drawings`  
오른쪽에서 왼쪽으로 그리면 right swipe  
왼쪽에서 오른쪽으로 그리면 left swipe  

1. mission control  
`btt - drawings`  
위로 그리면 mission control  

1. 닫기  
`btt - normal mice(chrome)`  
마우스 중간 버튼을 누르면 ctrl + w가 동작하도록 했다  

1. 새로고침  
`btt - drawings(chrome)`  
아래로 길~게 내리면 Command + r 이 동작하도록 했다(스마트폰 처럼)  

# 기타
1. iterm에cmd + 좌우 동작안하고 fn 키로만 됨
<https://stackoverflow.com/questions/6205157/iterm-2-how-to-set-keyboard-shortcuts-to-jump-to-beginning-end-of-line>  
설정에서 hex code 입력으로 바꿔주면 됨

2. 듀얼 모니터
MonitorControl application 으로 밝기, 소리 조절 가능
<https://github.com/the0neyouseek/MonitorControl>  

# 해결하지 못한 부분
1. keyboard로 듀얼 모니터간 포커스 전환  
개발하다 보면 마우스보다 키보드를 많이 쓰는데, 반대편 모니터에 있는 애플리케이션을 사용하려면 결국 마우스로 클릭해야 한다.  
Command + tab으로 하나하나 찾기는 너무 답답하고, 반대편 모니터를 마우스로 찍는 행위만 해주면 좋을텐데..  

2. 오른쪽 command의 본래 기능 없애기  
오른쪽 command를 한영키로 사용하니까 문자키와 한영키를 눌렀을때 command키가 동작해서 난감하다  
q -> ㅂ로 바꾸다가 Command + Q가 되어버리는..  

<!-- more -->
