---
title: 'intellij 기능 간단 정리'
date: 2019-03-08 23:59:18
tags:
---

maven, gradle
group id = 그룹 아이디, 예를 들면 스프링
artifact id = 모듈 이름, 예를 들면 스프링 시큐리티, 스프링 MVC

액션 검색 : meta + shift + a
새로 만들기 : meta + n
현재포커스 실행 : ctrl + shift + r
이전포커스 실행 : ctrl + r

라이브 템플릿
메인메서드 : psvm
System.out.println : sout

라인 복제하기 : meta + d
라인 삭제하기 : meta + delete
문자열 라인 합치기 : ctrl + shift + j
라인 단위로 옮기기 : 
- 문법 상관하면서 : meta + shift + up/down
- 문법 상관없이 : option + shift + up/down
element 단위로 옮기기(html, xml 속성, 메서드 매개변수 순서 등등) : meta + option + shift + left/right

파라미터 즉시보기 : meta + p
코드 구현부 즉시보기 : option + space
docs 보기 : f1

단어별 이동 : alt + 좌우(선택 : +shift)
라인 첫/끝 : fn + 좌우(선택 : +shift)
page up/down : fn + 위아래

포커스 범위(선택) 한 단계씩 늘리기 : alt + 위아래
포커스 앞/뒤 : meta + [/]
멀티포커스 : alt + alt + 위/아래(리눅스는 ctrl)
오류라인 자동 포커스 : f2

# 리팩토링
- 변수추출  
똑같은 값들을 하나의 변수로 추출하는 과정  
추출하고자 하는 값을 선택한 뒤, command + option + v  

- 파라미터 추출  
command + option + p  
변수가 아니라 파라미터로 추출된다  
extract via overloading method를 사용하면 추출된 메서드를  

- 메서드 추출  
추출하고 싶은 만큼 코드를 선택한 다음 command + option + m  

- 이너클래스 추출  
이너클래스가 여러군데서 사용될 떄 외부클래스로 추출할 수 있다  
f6 번을 누르면 어떻게 이동시킬지 선택할 수 있는 부분이 나오고, 여기서 이동할 패키지를 지정해주면 깔끔하게 클래스가 이동된다  

- 이름 일괄 변경  
shift + f6  
변수 이름 외에, 메서드 이름, 클래스 이름 모두 적용 가능  

- 타입 일괄변경  
파라미터 리턴 타입에 마우스대고 cmd + shift + 6  
반환하는 값에 대해서는 자동 변환시키거나 직접 설정가능  

- 사용하지 않는 import 제거  
ctrl + option + o  
command shitf a + optimize import 부분을 off -> on으로 바꾸면 자동으로 사용하지 않는 import를 정리해준다  
파일을 열떄마다 자동으로 사라지게 해준다  
이 기능을 사용하면 import문을 * 으로 변경하는 경우가 많은데,  
action -> import with * 의 개수를 999로 설정하면 해결된다  

- 정렬되지 않은 코드 정렬  
command + option + l

<!-- more -->