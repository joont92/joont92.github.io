---
title: 3장 스프링 웹 기술과 스프링 MVC
date: 2018-01-21 23:17:45
tags:
---

### 스프링의 웹 프레젠테이션 계층 기술  
스프링은 의도적으로 웹 어플리케이션의 컨텍스트를 2가지로 분리해놓았다.  
웹과 관련되지 않은 설정인 루트 어플리케이션 컨텍스트, 웹과 관련된 설정인 서블릿 어플리케이션 컨텍스트.  
이렇게 2가지로 분리해 둔 이유는 통쨰로 다른 기술로 대체할 수 있도록 하기 위해서이다.  

> **스프링 기반이 아닌 웹 프레임워크에서 어플리케이션 컨텍스트를 사용하는 법**  
스프링을 사용하지 않는 웹 프레젠테이션 계층(e.g. jsp model1)에서 루트 컨텍스트를 사용하는 방법은 아래와 같다.  
```java
ApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(request.getSession().getServletContext());

TestBean bean = context.getBean(TestBean.class);
```
> ㅁㄴㅇㅁㄴㅇ