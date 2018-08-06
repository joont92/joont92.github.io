---
title: DispatcherServlet의 Controller 전략
date: 2018-01-26 01:05:16
tags:
	- Controller
	- HandlerAdapter
	- HandlerMapping
	- HandlerInterceptor
	- 토비의 스프링
photo: 
    - https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTIwOTU2NzA4ODYmdHlwZT1sJno9MjAxOC8wNC8yMiAxMjo1Nw==
---

컨트롤러는 MVC의 세가지 컴포넌트 중 가장 많은 책임을 지고있다.  
사용자 요청 파악, 요청 검증, 비즈니스 로직 수행, 뷰 선택, 뷰에 출력할 모델 생성, 세션 관리...  
기본적인 것만 나열해도 이렇게 역할이 많다.  

애플리케이션 성격상 컨트롤러의 역할이 크다면 책임의 성격, 특징, 변경 사유 등을 기준으로 세분화 해 줄 필요가 있다.  
스프링은 이를 각각의 전략으로 세분화하였고, 확장 또한 쉽게 가능한 형태로 제공하고 있다.  

---

# 핸들러 매핑
HTTP 요청정보를 이용해서 컨트롤러를 찾아주는 기능을 수행한다.  
스프링이 제공하는 핸들러 매핑 전략은 총 5가지이다.  
핸들러 매핑은 `HandlerMapping` 인터페이스를 구현해서 생성한다.  
```java
public interface HandlerMapping{
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```

## BeanNameUrlHandlerMapping
HTTP 요청 URL과 빈의 이름을 비교하여 일치하는 빈을 찾아준다.  
빈 이름에는 ANT패턴이라고 불리는 `*, **, ?` 를 이용한 패턴을 넣을 수 있다.  
```xml
<!-- hello로 시작하면 모두 여기 매핑된다 -->
<bean name="/hello*" class="HelloController" />

<!-- **는 하나 이상의 경로를 매핑 할 수 있다 -->
<bean name="/root/**/sub" class="SubController" />
```
하지만 컨트롤러의 개수가 많아지면 URL정보가 XML이나 애노테이션에 분산되어 파악하기 어려우므로, 복잡한 애플리케이션에서는 잘 사용하지 않는다.  

## ControllerBeanNameHandlerMapping
`BeanNameUrlHandlerMapping`과 유사하지만 위처럼 빈 이름을 URL 형태로 짓지 않아도 된다는 것이 차이점이다.  
빈 이름 앞에 자동으로 `/`이 붙여져 URL에 매핑된다.  
```xml
<bean name="hello" class="HelloController" /> <!-- /hello에 매핑 -->
```

## ControllerClassNameHandlerMapping
빈의 클래스 이름을 URL에 매핑해주는 매핑 클래스이다.  
기본적으로는 클래스 이름을 모두 사용하지만 클래스 이름이 `Controller` 로 끝날 경우 `Controller`를 뺀 나머지 이름을 URL에 매핑해준다.  
```java
public class HelloController implements Controller{ // /hello에 매핑
	// ...
}
```

## SimpleUrlHandlerMapping
URL과 컨트롤러 매핑정보를 한곳에 모아놓을 수 있는 전략이다.  
매핑정보는 `SimpleUrlHandlerMapping` 빈의 프로퍼티에 넣어준다.  
```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="mappings"> <!-- Properties 타입으로 URL과 빈 이름을 넣어준다 -->
		<props>
			<prop key="/hello">helloController</prop>
			<prop key="/root/**/sub">subController</prop> <!-- ANT 패턴 사용 가능 -->
		</props>
	</property>
</bean>

<!-- 빈 이름이 위의 value 와 매핑 -->
<bean id="helloController" />
<bean id="subController" />
```
`mappings`는 `Properties` 타입이므로 프로퍼티 타입 포맷을 이용하여 더 간단히 작성할 수 있다.  
```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="mappings">
		<value>
			/hello=helloController
			/root/**/sub=subController
		</value>
	</property>
</bean>
```
매핑정보가 한군데 모여있어 URL을 관리하기 편리하여 대규모 프로젝트에서 선호하기도 한다.  
하지만 매핑정보를 직접 작성하므로 오타가 발생할 수도 있다는 단점이 있다.  

## DefaultAnnotationHandlerMapping
`@RequestMapping`이라는 애노테이션을 이용해 매핑하는 전략이다.  
`@RequestMapping`은 클래스는 물론 메서드 단위로도 URL을 매핑할 수 있다.  
또한 URL외에도 method, parameter, header 등의 정보도 애노테이션을 이용해 매핑에 활용할 수 있다.  
굉장히 강력하고 편리한 방법이지만 매핑 애노테이션의 사용 정책, 작성 기준을 잘 마련해놓지 않으면 매핑정보가 금방 지저분해지므로 주의해야 한다.  
[@RequestMapping 바로가기](/spring/@RequestMapping/)

## HandlerMapping 공통 설정정보
아래는 핸들러 매핑에서 공통적으로 사용되는 주요 프로퍼티이다.  

1. order  
핸들러 매핑은 1개 이상을 동시에 사용할 수 있다.  
1개 매핑으로 통일하는것이 가장 이상적이긴하나, 그렇지 않을 상황이 종종 있다.  
2개 이상의 핸들러 매핑이 등록되었는데 URL이 중복되었을 경우, `order` 프로퍼티를 통해 매핑 우선순위를 지정할 수 있다.  

2. defaultHandler  
URL을 매핑할 빈을 찾지 못할 경우 자동으로 디폴트 핸들러(컨트롤러)를 선택하게 한다.  
원래 URL을 찾지 못하면 HTTP 404 error가 발생하는데, 이를 디폴트 핸들러로 넘겨 적절한 에러처리를 할 수 있다.  
```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
	<property name="defaultHandler" ref="defaultController" />
</bean>
```

3. alwaysUseFullPath  
URL 매핑은 기본적으로 애플리케이션 컨텍스트 패스, 서블릿 패스를 제외한 나머지만 가지고 비교한다.  
즉 애플리케이션이 `/test`에 배포되고, `DispatcherServlet` `URL mapping`이 `/app/*`일 경우 전체 URL은 `/test/app/hello` 와 같은 형태지만, 핸들러 매핑은 `/hello`만을 대상으로 삼는다는 의미이다.  
이는 애플리케이션이나 서블릿이 변경되어도 애플리케이션이 영향을 받지 않게 하기 위해서이다.  
하지만 alwaysUseFullPath 옵션을 true로 주면 이를 해제하고 모든 URL을 대상으로 변경할 수 있다.  

4. detectHandlersInAncestorContexts
기본적으로 애플리케이션 컨텍스트는 계층형 구조를 가지므로, 자식 컨텍스트는 부모 컨텍스트를 참조할 수 있고, 그 반대는 안된다.  
즉 `루트 컨텍스트 -> 서블릿 컨텍스트` 의 `부모-자식` 형태의 경우  
서블릿 컨텍스트에 선언된 빈은 루트 컨텍스트에 선언된 빈을 DI할 수 있지만,  
루트 컨텍스트에 선언된 빈은 서블릿 컨텍스트에 선언된 빈을 DI할 수 없다.  
그런데 핸들러 매핑의 경우 이와 좀 다르다.  
핸들러 매핑 클래스는 매핑할 클래스를 현재 컨텍스트, 즉 서블릿 컨텍스트 내에서만 찾는다.  
컨트롤러는 서블릿 컨텍스트에만 두는 것이 바람직하기 때문이다.  
`detectHandlersInAcestorContexts` 옵션을 `true`로 주면서 이 방식을 바꿔줄 수 있긴한데,  
이 옵션은 절 대 사용하지 말자. 그냥 스프링의 극단적인 유연성을 보여주기 위한 옵션일 뿐이다.  

---

# 핸들러 어댑터
`HandlerMapping`을 통해 찾은 컨트롤러를 `DispatchserServlet`이 연결할 때 사용하는 핸들러 어댑터이다.  
스프링 MVC가 지원하는 컨트롤러는 총 4개이므로, 핸들러 어댑터도 4개이다.  
핸들러 어댑터는 `HandlerAdapter` 인터페이스를 구현해서 생성한다.  
```java
public interface HandlerAdapter{
	boolean supports(Object handler);

	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	long getLastModified(HttpServletRequest request, Object handler);
}
```

## SimpleServletHandlerAdapter(Servlet interface)
표준 서블릿 인터페이스인 `javax.servlet.Servlet`을 구현한 클래스를 컨트롤러로 사용할 때 사용되는 어댑터이다.  
이 방식의 장점은 서블릿 클래스 코드를 그대로 유지하면서 스프링 빈으로 등록할 수 있다는 점인데, 이는 서블릿 코드를 점진적으로 스프링 어플리케이션으로 포팅할 떄 유용하게 사용된다.  
이 컨트롤러의 어댑터로는 `SimpleServletHandlerAdapter`가 사용된다.  
> **참고사항**
1. 서블릿 라이프사이클 메서드인 `init`, `destroy`는 실행되지 않는다. 스프링 빈의 `init-method`나 `@PostConstruct`를 이용해야 한다.  
2. 이 컨트롤러는 모델과 뷰를 리턴하지 않는다. 서블릿은 원래 `Response`에 결과를 넣어주는 방식이기도 하고, `ModelAndView`의 개념을 모른다.  
※ `DispatcherServlet`은 `ModelAndView` 리턴 대신 `null`을 리턴할 경우 뷰를 호출하는 작업을 생략한다.  

## HttpRequestHandlerAdapter(HttpRequestHandler interface)
아래의 `HttpRequestHandler` 인터페이스를 구현한 클래스를 컨트롤러로 사용할 때 사용되는 어댑터이다.  
```java
public interface HttpRequestHandler{
    void handleRequest(HttpServletRequest request, HttpServletResponse resposne) throws ServletException, IOException;
}
```
서블릿 인터페이스와 생김새가 유사하다.  
서블릿 스펙을 준수할 필요없이 HTTP 프로토콜을 기반으로 한 전용 서비스를 만들려고 할 때 사용한단다.. 이정도만 알고 넘어가도 될듯.  

## SimpleControllerHandlerAdapter(Controller interface)
아래의 `Controller` 인터페이스를 구현한 클래스를 컨트롤러로 사용할 때 사용되는 어댑터이다.  
```java
public interface Controller{
    ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```
스프링의 대표적인 컨트롤러 타입이다.(3.0 전까지)  
`DispatcherServlet`과 주고받는 정보를 그대로 파라미터와 리턴값으로 갖고 있다.  
하지만 이 인터페이스를 직접 구현해 컨트롤러를 만드는 것은 권장되지 않으며, 필수 기능이 구현된 `AbstractController`를 사용하거나 직접 확장한 `Controller` 클래스를 사용하는 것을 권장한다.  
아래는 직접 확장의 간단한 예제이다.  
```java
// 기반 컨트롤러
public abstract class SimpleController implements Controller{
	private String[] requiredParams; // 필수 파라미터
	private String viewName;

	public void setRequiredParams(String[] requiredParams) {
		this.requiredParams = requiredParams;
	}

	public void setViewName(String viewName){
		this.viewName = viewName;
	}

	@Override
	final public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		if(viewName == null){
			throw new IllegalStateException();
		}

		Map<String, String> params = new HashMap<String, String>();
		for(String param : requiredParams){
			String value = request.getParameter(param);
			if(value == null){
				throw new IllegalStateException();
			}

			params.put(param, value);
		}

		Map<String, Object> model = new HashMap<String, Object>();

		this.control(params, model); // 개별 컨트롤러가 구현할 메서드

		return new ModelAndView(this.viewName, model);
	}

	public abstract void control(Map<String, String> params, Map<String, Object> model) throws Exception;
}

// 개별 컨트롤러
public class TestController extends SimpleController{
	public TestController(){
		this.setRequiredParams(new String[]{"name", "age"});
		this.setViewName("/WEB-INF/view/test.jsp");
	}

	@Override
	public void control(Map<String, String> params, Map<String, Object> model) throws Exception {
		// make model using params...
	}
}
```
기계적으로 `Controller` 인터페이스의 `handleRequest` 메서드를 사용하지 말고 위와 같이 클래스를 확장하여 사용하는 것이 좋다.  

## AnnotationMethodHandlerAdapter
앞의 핸들러 어댑터들과 달리 호출하는 컨트롤러의 타입이 정해져 있지 않다.  
클래스와 메서드에 붙은 애노테이션, 메서드 이름, 파타미터, 리턴타입에 대한 규칙 등을 조합하고 분석해서 컨트롤러를 선별한다.  
또한 다른 컨트롤러와 다르게 컨트롤러 하나가 여러 URL에 매핑될 수 있다.(이때까지는 1개의 URL에 1개의 컨트롤러가 매핑되었다)  
이는 메서드 단위로 URL매핑이 가능하기 때문이다.  
이 `AnnotationMethodHandlerAdapter`는 다른 핸들러 어댑터와는 다르게 `DefaultAnnotationHandlerMapping` 핸들러 매핑과 같이 사용해야 한다.  
두 가지 모두 동일한 애노테이션을 사용하기 때문이다.  
이 방식은 매우 강력하고 작성이 간결한 대신 꽤나 많은 규칙이 존재하고, 이를 잘 숙지하고 사용해야 한다.  
[@RequestMapping 바로가기](/spring/@RequestMapping/)

---

# 핸들러 인터셉터
핸들러 매핑은 요청정보를 통해 컨트롤러를 찾아주는 기능 외에 핸들러 인터셉터를 적용해주는 기능 또한 제공한다.  
핸뜰러 인터셉터는 `DispatcherServlet`이 컨트롤러를 호출하기 전과 후에 요청, 응답을 가공할 수 있는 일종의 필터이다.  
핸들러 매핑은 `DispatcherServlet`으로 부터 매핑 작업을 요청받으면 등록된 핸들러 인터셉터들을 순서대로 수행하고 컨트롤러를 호출한다.  
등록된 핸들러 인터셉터가 없다면 컨트롤러를 바로 호출한다.  

## 구현
핸들러 인터셉터를 만들고자 할 때는 아래의 `HandlerInterceptor` 인터페이스를 구현해야 한다.  
```java
public interface HandlerInterceptor {
	boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	void postHandle(
			HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception;

	void afterCompletion(
			HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception;
}
```

1. preHandle  
컨트롤러가 호출되기 전에 실행된다. `handler` 파라미터는 컨트롤러 빈 오브젝트이다.  
리턴값이 `true`이면 다음 인터셉터로 진행되고, `false`일 경우 다음 인터셉터들을 실행되지 못한다.    

2. postHandle
컨트롤러를 호출한 후에 실행된다. `ModelAndView`가 제공되므로 작업결과를 참조하거나 조작할 수 있다.  

3. afterCompletion
모든 작업(뷰 생성까지)이 완료된 후 실행된다.  

## 적용
적용할 핸들러 인터셉터들을 핸들러 매핑 클래스의 속성으로 지정해줘야 하므로, 핸들러 매핑 클래스를 빈으로 등록해줘야 한다.  
이후 `interceptors` 프로퍼티를 이용하여 인터셉터를 등록한다.  
```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
	<property name="interceptors">
		<list>
			<ref bean="firstInterceptor" />
			<ref bean="secondInterceptor" />
		</list>
	</property>
</bean>

<bean name="firstInterceptor" class="~~~FirstInterceptor" />
<bean name="secondInterceptor" class="~~~SecondInterceptor" />
```
보다시피 인터셉터들은 핸들러 매핑 단위로 등록된다.  
즉 작성한 인터셉터들을 여러개의 핸들러 매핑에 적용시키고 싶으면 핸들러 매핑 빈 마다 반복적으로 다 등록해줘야 한다.  
물론 인터셉터 빈은 한개만 등록하면 된다.  

## 핸들러 인터셉터 vs 서블릿 필터
보다시피 핸들러 인터셉터는 서블릿 필터와 기능이 유사하지만 조금 차이가 있으니 선택에 주의를 기울여야 한다.  

서블릿 필터는 `web.xml`에 등록하면서 웹 애플리케이션으로 들어오는 모든 요청에 적용할 수 있다는 장점이 있다.  
반면에 스프링의 빈이 아니라는 점과, 핸들러 인터셉터보다 정교한 컨트롤이 어렵다는 단점이 있다.  

핸들러 인터셉터는 특정 핸들러 매핑에 제한된다는 단점이 있지만,  
인터셉터를 스프링 빈으로 등록할 수 있고 `ModelAndView`를 컨트롤 하는 등 더욱 정교한 컨트롤이 가능하다.  

프로젝트의 상황에 따라 적절한 선택을 하는것이 좋다.  

<div style="text-align: right">
From <img src="https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwNDAxMDkmdHlwZT1sJno9MjAxOC8wOC8wNiAwOTozOA==#width30" style="display:inline-block;"/>
</div>

<!-- more -->
