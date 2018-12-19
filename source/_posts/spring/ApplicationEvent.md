---
title: ApplicationEvent
date: 2018-11-25 21:59:42
tags:
---

# Spring ApplicationEvent
<http://wonwoo.ml/index.php/post/1070#comment-7184>

event를 발생시키는 publisher(ApplicationEventPublsh),  
event를 받는 lister(ApplicationEventLister),
event가 있음(ApplicationEvent).  

publisher에서 event를 생성해서 발생시키면  
lister에서 자신이 구현한 event가 맞을 경우 그 이벤트를 받을 수 있다   

스프링 4.2 부터는 어노테이션으로 가능하고,  
많은 옵션들(spel 등)을 줄 수 있다.  

<!-- more -->