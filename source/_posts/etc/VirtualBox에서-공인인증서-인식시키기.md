---
title: VirtualBox에서 공인인증서 인식시키기
date: 2019-04-24 22:42:53
tags:
    - virtualbox NPKI
---

VirtualBox에서 공인인증서 인식시킨 방식에 대해 공유하고자 함  

# VirtualBox 하드디스크에 위치시키는 방법은 안된다
내 VirtualBox는 윈도우 7이었으므로, 인터넷에서 찾아보고 공인인증서를 `C://Users/{userId}/AppData/LocalLow/NPKI` 로 위치시켰는데도 전혀 인식하지 못했다  
`C://Program Files/NPKI`, `C://Program Files(x86)/NPKI` 또한 마찬가지였다  
VirtualBox라서 경로를 다르게 인식하는 것 같았다  
아마도 mac 파일시스템으로 인식했겠지.. ~/VirtualBox/cdrive 이런식으로..(왜 이 생각을 못했을까 ㅠㅠ)  
어찌됬든 이런 문제때문에 USB로 공인인증서를 불러오게끔 해야한다  

# VirtualBox에서 usb 인식시키기  
<https://imitator.kr/Windows/2826> 여기 따라서 VirtualBox에서 USB를 인식시키게 하면 된다  
> <https://www.virtualbox.org/wiki/Download_Old_Builds> 여기서 자기 VirtualBox 버전에 맞춰 들어간다음, `Extension Pack`을 선택해서 설치한다  

위 블로그대로 수행하면 가상머신안에서도 USB 인식이 가능해지고, 공인인증화면에서 USB로 공인인증서를 찾을 수 있다  
망할놈의 공인인증서 ^_^  

<!-- more -->