---
title: DispatcherServlet Flow
date: 2018-01-21 23:17:45
tags:
    - DispatcherServlet
    - MVC
    - Front Controller
    - DispatcherServlet flow
    - 토비의 스프링
---

스프링 웹 기술은 `MVC 아키텍쳐`를 근간으로 한다.  
> **MVC 아키텍쳐**  
정보를 담은 `모델(M)`, 화면 출력 로직을 담을 `뷰(V)`, 제어 로직을 담은 `컨트롤러(C)`가 서로 협력하여 하나의 웹 요청을 처리하고 응답을 만들어내는 구조이다.  

MVC 아키텍처는 기본적으로 **프론트 컨트롤러 패턴**과 함께 사용된다.  
> **프론트 컨트롤러 패턴**  
클라이언트의 요청을 먼저 받고 공통적인 작업을 수행 후, 나머지 작업을 세부 컨트롤러로 위임해주는 방식  

스프링은 **DispatcherServlet** 이라는 프론트 컨트롤러를 사용하며, 이는 스프링 MVC의 핵심이다.  

---

# DispatcherServlet flow
![DispatcherServlet Flow](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwMzQzNTEmdHlwZT1sJno9MjAxOC8wOS8xMyAyMjoxMQ==#width80)  
## 1) DispatcherServlet의 HTTP 요청 접수
들어오는 http 요청이 `DispatcherServlet`에 할당된 것이라면 이를 받는다.  
대체로 아래와 같이 `web.xml`에 정의되어 있다.  
```xml
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>
```
요청을 받은 `DispatcherServlet`은 공통적으로 진행해야 할 전처리 작업이 있다면 이를 먼저 수행한다.  

## 2) DispatcherServlet에서 컨트롤러로 HTTP 요청 위임
`DispatcherServlet`은 들어온 http 요청 정보(URL, parameter, method 등)을 참고로 해서 어떤 컨트롤러에게 작업을 위임할 지 결정한다.  
`DispatcherServlet`은 프론트 컨트롤러이므로 세부 컨트롤러를 직접 찾지않고 [HandlerMapping, HandlerAdapter](/spring/HandlerMapping-HandlerAdapter-HandlerInterceptor)라는 전략에 해당 행위를 위임한다.(자세한 내용은 위 링크에서 확인)  
지금은 그냥 전달받은 `HttpServletRequest`, `HttpServletResponse`를 넘겨주면 세부 컨트롤러를 찾아준다 정도만 기억해도 된다.  

## 3) 컨트롤러의 모델 생성과 정보 등록
컨트롤러는 사용자 요청을 해석하여 비즈니스 로직을 수행하고 결과를 받아온 뒤, 모델에 넣는다.  
(모델은 뷰에 뿌려줄 정보를 담은 key, value 형태의 맵이다.)  
> MVC 패턴의 장점은 모델과 뷰가 분리되었다는 것이다. 같은 모델이라도 뷰만 바꿔주면 전혀 다른 방식으로 모델의 정보를 출력시킬 수 있다.  
예를 들어 jsp뷰를 선택하면 html, 엑셀뷰를 선택하면 엑셀, pdf뷰를 선택하면 pdf로 모델정보를 출력할 수 있다.  

## 4) 컨트롤러의 결과 리턴 : 모델과 뷰
모델이 준비되었으므로 뷰를 선택한다.  
뷰 오브젝트를 직접 선택할 수도 있지만 보통은 뷰 리졸버를 사용므로 뷰의 논리적인 이름만을 리턴하면 된다.  
컨트롤러가 최종적으로 리턴해주는 정보는 모델과 뷰, 2가지이다.  
이를 리턴해주고 컨트롤러의 역할은 끝이난다.

## 5-6) DispatcherServlet의 뷰 호출과 모델 참조
다시 `DispatcherServlet으로` 넘어왔다.  
뷰 오브젝트에 모델을 넘겨주며 최종 결과물을 생성해달라고 요청한다.  
생성된 최종 결과물은 `HttpServletResponse` 오브젝트에 담긴다.  

## 7) HTTP 응답 돌려주기
`DispatcherServlet은` 공통적으로 진행해야 할 후처리 작업이 있는지 확인하고 이를 수행한다.  
수행이 끝나면 `HttpServletResponse`에 담긴 최종 결과를 서블릿 컨테이너에게 돌려준다.  
컨테이너는 이 정보를 HTTP 응답으로 만들어 사용자의 브라우저나 클라이언트에 전송하고 작업을 종료한다.  

---

# DispatcherServlet의 변경 가능한 전략
위에 나와있듯이, DispatcherServlet은 하나의 요청에 대해 상당히 많은 작업들을 거친다  
(하나의 요청에 HandlerMapping 거치고 HandlerAdapter 거치고 ViewResolver 거치고 등등..)  
이 작업들을 `전략`이라고 부른다. 기본적으로 default 전략들이 있고, 별다른 설정을 하지 않는다면 이 기본 전략들을 사용하여 요청을 수행하게 된다  
중요한것은, 스프링의 `DispatcherServlet`은 전략패턴이 잘 적용되어 있으므로 이 전략들을 아주 쉽게 확장(변경)할 수 있다는 점이다(!!)  
간단하게 확장하고 싶은 전략들을 빈으로 등록만 해두면, DispatcherServlet이 플로우를 수행하면서 해당 전략을 기본전략 대신해서 실행하게 된다(등록된 확장 전략들이 있는가 체크하고 있으면 수행, 없으면 기본 전략 수행)  
> `DispatcherServlet`은 스프링이 관리하는 오브젝트가 아니므로 직접 `DI` 하는 방식으로 전략이 확장되는 것은 아니다.  
`DispatcherServlet`은 기본적으로 **DispatcherServlet.properties** 파일을 통해 설정을 초기화하고,
내부적으로 가지고 있는 어플리케이션 컨텍스트를 통해 확장 가능한 전략이 있나 찾은 뒤, 이를 가져와 디폴트 전략을 대신해서 사용하는 방식이다.  

아래는 확장 가능한 전략들과, default로 등록된 전략들이다.  

## HandlerMapping
URL과 요청 정보를 기준으로 어떤 컨트롤러를 사용할지 결정한다.  
- default
    1. BeanNameUrlHandlerMapping
    2. DefaultAnnotaionHandlerMapping

## HandlerAdapter
핸들러 매핑으로 선택된 컨트롤러를 직접 호출한다.  
- default
    1. HttpReqeustHandlerAdapter
    2. SimpleControllerHandlerAdapter
    3. AnnotaionMethodHandlerAdapter

## HandlerExceptionResolver
예외가 발생했을 때 이를 처리하는 로직을 갖고 있다.  
예외 발생 시 에러페이지 출력, 로그 전송등의 작업은 개별 컨트롤러가 아닌 프론트 컨트롤러의 역할이다.  
`DispatcherServlet`은 등록된 `HandlerExceptionResolver`중에서 발생한 예외에 적합한 것을 찾아 예외처리를 위임한다.  
- default  
    1. AnnotaionMethodHandlerExceptionResolver
    2. ResponseStatusExceptionResolver
    3. DefaultHandlerExceptionResolver  

## ViewResolver
컨트롤러가 리턴한 논리적인 뷰 이름을 참고하여 적절한 뷰 오브젝트를 찾아준다.  
- default
    1. InternalResourceViewResolver
    2. UrlBasedViewResolver

## LocaleResolver
지역정보를 결정해주는 전략이다.  
디폴트는 헤더 정보를 보고 지역정보를 설정한다.  
파라미터나 쿠키, xml 설정등을 통하게 할 수 있다.  
- default
    1. AcceptHeaderLocaleResolver

## RequestViewNameTranslator
컨트롤러에서 뷰 오브젝트나 이름을 제공하지 않았을 경우 URL 요청정보를 참고해서 자동으로 뷰 이름을 생성해주는 전략이다.  
- default
    1. DefaultRequestToViewNameTraslator

> 전략의 변경을 위해 빈을 직접 등록하게 되면 해당 전략의 default 전략들이 모두 무시되므로, 유의해야 한다.  
> 또한 디폴트 전략에 추가 옵션을 주고 싶으면 해당 빈을 등록하면서 옵션을 줘야 한다.  

참고 : [이일민, 『토비의 스프링 3.1』, 에이콘출판(2012)](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788960773431&orderClick=LAG&Kc=)

<!-- more -->
