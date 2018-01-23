---
title: 스프링 웹 기술과 스프링 MVC
tags:
---

## 스프링의 웹 프레젠테이션 계층 기술
presentation layer =
스프링 웹 기술을 사용하는 프레젠테이션 계층 + 스프링 외의 웹 기술을 사용하는 프레젠테이션 계층

컨텍스트
root contetxt : 웹 기술에서 독립적인 비즈니스 계층, 데이터 엑세스 계층
servlet application context : 스프링 웹 기술을 기반. 웹 관련 빈
\> 통째로 다른 기술로 교체하기 용이하기 위해서

스프링 서블릿은 mvc 아키텍쳐를 따르는 웹 프레임워크를 구축할 수 있는 유연한 기반을 제공
@MVC와 같이 스프링이 확장시켜 만든 프레임워크도 있다.

> JSP/Servelt에서 스프링 빈 사용하려면 WebApplicationContextUtils의 스태틱 메서드를 사용하면 된다.
> JSP에서 사용할 수 있는 HttpServletRequest 타입의 오브젝트를 이용하여 ServletContext를 가져올 수 있다.
```java
ApplicationContext context =
WebApplicationContextUtils.getWebApplicationContext(request.getSession().getServletContext());
HelloBean helloBean = context.getBean(HelloBean.class);
```
스프링 : 유연성, 확장성, 다양성에 무게를 둠
집착에 가까울 정도로 모든 기능을 확장 가능하도록 설계되어 있다.
스프링을 잘 사용하는 법은 스프링 프레임워크를 완성된 고정적인 프레임워크로 보지말고,
스프링이 가진 기존 기능을 확장해서 프로젝트에 맞는 프레임워크로 확장하는 것이다.

### DispatcherServlet, MVC architecture
스프링 웹 기술은 MVC 아키텍쳐를 기반으로 하고있고, 핵심 기술은 DispatcherServlet이다.
M(Model) : 프레젠테이션 계층의 구성요소 정보를 담음
V(View) : 화면 출력 로직
C(Controller) : 제어 로직

프론트 컨트롤러 패턴 :
중앙 집중형 컨트롤러(e.g. DispatcherServlet)을 계층의 젤 앞에 둬서 요청을 가장 먼저 받아 처리하게 만드는 패턴
선행작업, 뷰 선택, 예외처리 등을 행한다.

![DispatcherServlet Flow](https://user-images.githubusercontent.com/18513953/30509764-e17c5fc6-9af2-11e7-99be-b7c65e956fb0.png)

#### DispatcherServlet의 requset -> response과정
1. web.xml등에 따라 요청을 dispatcherServlet이 받고, 공통적으로 진행해야 하는 전처리 작업 등을 한다.
e.g 보안, 파라미터 조작, 디코딩 등
2. url, parameter 정보 등을 사용하여 어떤 컨트롤러에게 작업을 위임할 지 결정(핸들러 매핑 전략)
개발자는 어떤 오브젝트가 처리할지를 매핑하는 전략을 만들어 DI가능하도록 제공해주기만 하면 된다.
이후 요청이 들어오면, DispatcherServlet은 컨트롤러를 선택하고, 해당 메서드를 실행하여 실제 작업을 처리한다.
여기서 중요한 것은 컨트롤러가 특정 인터페이스를 구현하는 식의 규약을 따르지 않았다는 것이다.
그러한데도 DispatcherServlet이 컨트롤러의 메서드를 호출할 수 있는 이유는 **어댑터** 라는 것을 이용하기 때문이다.

> DispatcherServlet -> HandlerAdapter -> Controller 의 형태이다.
> DispatcherServlet은 모든 웹 요청정보가 담긴 HttpServletRequest 오브젝트를 전달해주고,
> 이를 어댑터가 컨트롤러 메서드가 받을 수 있도록 파라미터로 변환하여 전달한다. (HttpServletResponse도 전달해준다)
3. 컨트롤러는 사용자의 요청을 해석하여 로직을 수행한 뒤 결과에 따라 모델을 생성한다. 모델은 key, value로 이뤄진 쌍이라고 보면 된다.
이후 뷰를 선택한다. 뷰도 하나의 오브젝트이다. 컨트롤러가 직접 선택할 수도 있지만 보통은 논리적인 이름을 리턴해주고,
DispatcherServlet의 뷰 리졸버가 이를 이용해 뷰 오브젝트를 생성해준다.
이후 준비된 모델과 뷰를 DispatcherServlet에 돌려줌으로써 컨트롤러의 역할은 끝난다.
(ModelAndView, Model 등의 오브젝트를 사용하여 전달할 수 있다.)
4. DispatcherServlet은 뷰 오브젝트에 모델을 전달하고, 결과물을 생성해달라고 요청한다.
(JstlView의 경우 컨트롤러가 리턴한 논리적 이름의 jsp 템플릿을 가져다가 HTML을 생성한다.)
HTML이 일반적이고, 엑셀, PDF등과 같이 파일 형태로 만드는 뷰도 있다.
모델이 같아도 어떤 뷰를 선택하느냐에 따라 결과가 달라질수 있다는 의미이다.
최종 결과는 HttpServletReponse에 담긴다.
5. DispatcherServlet은 마지막으로 후처리기가 있는가 확인 후,
있으면 후처리를 진행한 뒤 최종 HttpServletResponse를 서블릿 컨테이너에 돌려준다.
6. 서블릿 컨테이너는 이를 HTTP 응답으로 만들어 클라이언트에 전송하고 작업이 종료된다.

### DispatcherServlet의 확장 가능한 전략(using DI)
- HandlerMapping
URL과 요청정보를 기준으로 컨트롤러를 결정한다. HandlerMapping 인터페이스를 구현해서 만들 수 있다.
- HandlerAdapter
HandlerMapping으로 선택한 컨트롤러를 호출할 때 사용하는 어댑터다.
컨트롤러 호출 방법은 타입에 따라 다르기 때문에 컨트롤러를 결정했다고 해도 호출방법을 DispatcherServlet이 알 수가 없으므로, 어댑터가 필요하다.
- HandlerExceptionResolver
비즈니스로직이 아닌 초기 요청단계에서의 Exception은 프론트 컨트롤러인 DispatcherServlet에 의해 처리되어야 한다.
DispatcherServlet은 등록된 HandlerExceptionResolver중에 적합한 것을 찾아 예외처리를 위임한다.
- ViewResolver
컨트롤러가 리턴한 논리적 뷰 이름을 통해 뷰 오브젝트를 찾아주는 오브젝트다.
디폴트는 InternalResourceViewResolver이며, JSP같은 리소스를 사용할 수 있게 해주는 뷰 리졸버이다.
- LocaleResolver
지역정보를 결정해주는 전략. default인 AcceptHeaderLocaleResolve는 HTTP header정보를 보고 locale정보를 설정한다.
이 전략을 바꾸면 파라미터, 쿠키, xml등을 이용할 수도 있다.
- RequestToViewNameTranslator
컨트롤러에서 뷰 정보를 제공하지 않았을 경우 URL등을 이용해서 자동으로 뷰 이름을 생성해주는 전략

> DispatcherServlet은 스프링 컨테이너가 아닌 서블릿 컨텍스트가 관리하는 오브젝트이기 
때문에 직접 DI가 불가능하다.

> DispatcherServlet은 각 전략의 default 설정을 DispatcherServlet.properties라는 파일로부터 가져와서 초기화하는데, 

> 이전에 개발자가 추가하거나 수정한 전략이 있나 찾아보고 있으면 대체, 없으면 default 전략을 사용한다.

