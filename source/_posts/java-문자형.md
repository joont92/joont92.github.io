---
title: java 문자형
date: 2018-12-08 11:11:42
tags:
---

# char형에 대한 고찰
java에서 char형은 2byte의 크기를 가지며, 음수를 저장하지 않는 unsigned 여서 65,535 크기의 숫자를 저장가능하다.  
내부적으로 문자는 다 숫자로 저장하며, 출력할때는 문자로 보여준다.  
저장할때는 숫자, 문자, 유니코드형 모두 가능하다.  

```java
char a = '\u0061';
char b = 'a';
char c = '97';

/**
출력
a
a
a
**/
```

초기의 문자는 latin 문자 저장을 위한 1byte의 ASCII 코드였는데,  
전세계의 문자를 담기에는 128이라는 숫자가 턱없이 부족해서(...) 유니코드라는 문자셋이 나오게 되었다.  
전세계의 문자는 2byte 내로 전부 담을 수 있고, 이상의 문자들(emoji 등)은 3byte를 넘어간다.  
(유니코드라는 것이 기본적으로 2byte 까지는 말하는 것이 아닌듯? 규격일 뿐이니까?)  

문자열 String은 기본적으로 char 배열로 저장된다.  

http://pickykang.tistory.com/13
https://okky.kr/article/527371



<!-- more -->