---
title: Controller
date: 2018-01-26 01:05:16
tags:
---

컨트롤러는 MVC의 세가지 컴포넌트 중 가장 많은 책임을 지고있다.  
사용자 요청 파악, 요청 검증, 비즈니스 로직 수행, 뷰 선택, 뷰에 출력할 모델 생성, 세션 관리...  
기본적인 부분만 나열해도 이렇게 역할이 많다.  
이 작업들을 단순히 컨트롤러 메서드에 모두 담는것은 매우 비효율적이다.  

애플리케이션 성격상 컨트롤러의 역할이 크다면 책임의 성격, 특징, 변경 사유 등을 기준으로 세분화 해 줄 필요가 있고, 스프링은 이를 잘 세분화하여 제공한다.  
필요에 따라 확장도 가능하다.  

---

### 컨트롤러의 종류와 핸들러 어댑터
스프링 MVC가 지원하는 컨트롤러는 4개이다.  
그리고 각 컨트롤러마다 DispatcherServlet과 연결하기 위한 핸들러 어댑터가 필요하므로, 핸들러 어댑터도 4개이다.  

#### Servlet과 SimpleServletHandlerAdapter
표준 서블릿 인터페이스인 javax.servlet.Servlet을 구현한 클래스를 컨트롤러로 사용할 수 있다.  
이 방식의 장점은 서블릿 클래스 코드를 그대로 유지하면서 스프링 빈으로 등록할 수 있다는 점인데, 이는 서블릿 코드를 점진적으로 스프링 어플리케이션으로 포팅할 떄 유용하게 사용된다.  
이 컨트롤러의 어댑터로는 SimpleServletHandlerAdapter가 사용되는데, 이는 디폴트 어댑터가 아니므로 스프링 빈 으로 등록해줘야 한다.  
> **참고사항**
1. 서블릿 라이프사이클 메서드인 init, destroy는 실행되지 않는다. 스프링 빈의 init-method나 @PostConstruct를 이용해야 한다.  
2. 이 컨트롤러는 모델과 뷰를 리턴하지 않는다. 서블릿은 원래 Response에 결과를 넣어주는 방식이기도 하고, ModelAndView의 개념을 모른다.  
※ DispatcherServlet은 ModelAndView 리턴 대신 null을 리턴할 경우 뷰를 호출하는 작업을 생략한다.  

#### HttpRequestHandler와 HttpRequestHandlerAdapter
아래는 HttpRequestHandler 인터페이스이다. 이를 구현해서 컨트롤러를 만든다.  
```java
public interface HttpRequestHandler{
    void handleRequest(HttpServletRequest request, HttpServletResponse resposne) throws ServletException, IOException;
}
```
서블릿 인터페이스와 생김새가 유사하다.  
서블릿 스펙을 준수할 필요없이 HTTP 프로토콜을 기반으로 한 전용 서비스를 만들려고 할 때 사용한단다.. 이정도만 알고 넘어가도 될듯.  
어댑터는 디폴트이니 따로 빈으로 등록할 필요없다.  

#### Controller와 SimpleControllerHandlerAdapter
아래는 Controller 인터페이스이다. 이를 구현해서 컨트롤러를 만든다.  
```java
public interface Controller{
    ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```
스프링의 대표적인 컨트롤러 타입이다.(3.0 전까지)  
DispatcherServlet과 주고받는 정보를 그대로 파라미터와 리턴값으로 갖고 있다.  
Controller 타입 컨트롤러는 스프링 MVC를 확장해서 애플리케이션에 최적화된 전용 컨트롤러를 설계할 때 유용하다.  
아래는 간단한 확장의 예제이다.  
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
기계적으로 Controller 인터페이스의 handleRequest 메서드를 사용하지 말고 위와 같이 확장 포인트를 활용하는 것이 좋다.  

#### AnnotationMethodHandlerAdapter
앞의 핸들러 어댑터들과 달리 컨트롤러의 타입이 정해져 있지 않다.  
클래스와 메서드에 붙은 어노테이션, 메서드 이름, 파타미터, 리턴타입에 대한 규칙 등을 조합하고 분석해서 컨트롤러를 선별한다.  
또한 다른 컨트롤러와 다르게 컨트롤러 하나가 여러 URL에 매핑될 수 있다.(이때까지는 1개의 URL에 1개의 컨트롤러가 매핑되었다)  
이는 메서드 단위로 URL매핑이 가능하기 때문이다.  
이 AnnotationMethodHandlerAdapter는 다른 핸들러 어댑터와는 다르게 DefaultAnnotationHandlerMapping 핸들러 매핑과 같이 사용해야 한다.  
두 가지 모두 동일한 어노테이션을 사용하기 때문이다.  
이 방식은 매우 강력하고 작성이 간결한 대신 꽤나 많은 규칙이 존재하고, 이를 잘 숙지하고 사용해야 한다.  
> 게시글 바로가기..

