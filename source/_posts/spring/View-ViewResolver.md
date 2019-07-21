---
title: [spring] View, ViewResolver
date: 2018-02-04 18:03:37
tags:
    - View
    - ViewResolver
    - 토비의 스프링
---

뷰는 모델이 가진 정보를 어떻게 표현해야 하는지에 대한 로직을 갖고 있는 컴포넌트이다.  
일반적인 뷰의 결과물은 브라우저에서 볼 수 있는 HTML이다.  
이 외에도 엑셀, PDF, RSS 등 다양한 결과를 생성할 수 있는 뷰 오브젝트들이 있다.  

# 뷰
뷰는 `View` 인터페이스를 구현해서 생성한다.  
```java
public interface View{
    void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse resposne) throws Exception;
}
```
스프링에서 제공하는 뷰 목록은 아래와 같다.  

## InternalResourceView
`RequestDispatcher`의 `forward()`, `include()`를 이용하는 뷰다.  
```java
RequestDispatcher dispatcher = request.getRequestDispatcher("/WEB-INF/view/hello.jsp");
request.setAttribute("message", "This is message");
dispatcher.forward(request, response);
```
위는 `RequestDispatcher`를 사용했던 때의 방식이다.  
`InternalResourceView`의 동작 방식도 이와 동일하다고 보면 된다.  
아래는 사용 예제의 일부분이다.  
```java
View view = new InternalResourceView("/WEB-INF/view/hello.jsp");

return new ModelAndView(view, model);
```
> **JstlView**  
`InternalResourceView`의 서브 클래스이다.  
`JSP`를 뷰 템플릿으로 사용할 때 `JstlView`를 사용하면 여러가지 추가 기능을 더 활용할 수 있다.(지역화 메세지 등)  

## RedirectView
`HttpServletResponse`의 `sendRedirect()`를 호출해주는 기능을 가진 뷰다.  
실제 뷰를 생성하진 않고 URL만 만들어 다른 페이지로 리다이렉트 해준다.  
모델정보가 있다면 URL 뒤에 파라미터로 추가된다.  
```java
// 뷰 사용시
return new ModelAndView(new RedirectView("/main"));
// 뷰 리졸버 사용시
return new ModelAndView("redirect:/main");
```
보다시피 `redirect:` 접두어를 사용하여 뷰 리졸버가 인식하게 할 수 있다.  
redirect 경로는 절대경로(컨텍스트 패스, 서블릿 패스 포함)이어야 하는데,  
`contextRelative` 옵션을 `true`로 주면 컨텍스트 패스는 생략하고 작성할 수 있다.  

## VelocityView, FreeMarkerView
`JSP` 대신 `Velocity`, `FreeMarker` 템플릿 엔진을 뷰로 사용할 수 있게 해준다.  
둘은 `JSP`보다 훨씬 문법이 강력하고 속도도 빠른 장점이 있다.  
개인적으로 둘은 써보지 않았고 `Thymeleaf`라는 템플릿 엔진을 써봤었는데,  
이것도 추천합니다..  

## MarshallingView, MappingJacksonJsonView
모델의 정보를 `xml`, `json`으로 변환시킬 수 있는 뷰다.  
`MashallingView`를 사용할 경우 `xml`, `MappingJacksonJsonView`를 사용할 경우 `json`으로 변환된다.  
메세지 컨버터를 이용해서도 동일한 작업을 할 수 있다.  

## AbstractExcelView, AbstractJExcelView, AbstractPdfView
엑셀과 PDF문서를 만들어주는 뷰다.  
`Abstract`가 붙어있으니 상속을 해서 구현해줘야 한다. 구현하는 부분은 문서를 생성하는 부분이다.  
구현한 뷰 클래스는 빈으로 등록하여 컨트롤러에 `DI`해줘도 되고, 직접 생성해도 된다.  
> 뷰 오브젝트는 멀티스레드 환경에서 공유해도 안전하다.  
즉 싱글톤 빈으로 생성해도 문제되지 않는다.

## AbstractAtomFeedView, AbstractRssFeedView
`application/atom+xml`과 `application/rss+xml` 타입의 피드 문서를 생성해주는 뷰다.  
컨트롤러에서 사용하는 방법은 위의 `AbstractExcelView` 등과 동일하다.  

# 뷰 리졸버
뷰를 선택하는 것도 컨트롤러의 역할이다.  
하지만 위처럼 컨트롤러에서 매번 뷰를 생성하는 것은 비효율적이므로, 스프링에서는 이 작업을 적절히 분리하였다.  
컨트롤러는 뷰의 논리적인 이름만을 리턴한 뒤 역할을 종료하고, 이를 `DispatcherServlet`의 `뷰 리졸버`가 받아 사용할 뷰 오브젝트를 찾고 생성하는 작업을 진행해준다.  
게다가 뷰 리졸버는 보통 뷰 오브젝트를 캐싱하므로 같은 URL의 뷰가 반복적으로 만들어지지 않는 장점도 있다.  
뷰 리졸버도 하나 이상 등록해서 사용할 수 있는데, 이때는 핸들러 매핑처럼 `order` 프로퍼티를 이용해 적용 순서를 적용해주는 것이 좋다.  
뷰 리졸버는 `ViewResolver` 인터페이스를 구현해서 생성한다.  
```java
public interface ViewResolver{
    View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

## InternalResourceViewResolver
주로 `JSP`를 사용할 때 쓰이는 뷰 리졸버이다.  
뷰를 생성할 필요없이 논리적인 이름만을 리턴해주면 되는데, 그대로 사용할 경우 풀 패스를 써줘야 하므로 그대로 사용하는 것은 피해야 한다.  
`prefix`, `suffix` 프로퍼티를 이용하면 앞뒤에 붙는 내용을 생략할 수 있다.  
```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/view" />
    <property name="suffix" value=".jsp" />
</bean>
```
컨트롤러에서는 hello 만을 리턴해주면 된다. 이는 나중에 변경에도 용이하다.  
> JSTL 라이브러리가 클래스패스에 존재할 경우 `JstlView`를 사용하고, 존재하지 않으면 `InternalResourceView`를 사용한다.  

## VelocityViewResolver, FreeMarkerViewResolver
`Velocity`와 `FreeMarker`를 사용하게 해주는 뷰 리졸버이다.  
`InternalResourceViewResolver`와 같이 `prefix`, `suffix`를 사용가능하다.  

## ResourceBundleViewResolver
컨트롤러가 아닌 외부에서 뷰를 결정할 때, 한가지 뷰 만이 아니라 컨트롤러마다 뷰가 달라질 수 있을 때 사용하면 괜찮은 방식이다.  
`ResourceBundleViewResolver`를 사용하면 클래스패스의 `views.properties` 파일에 논리적 이름과 뷰 정보를 정의하여 작성하고, 이를 사용하여 뷰를 선택하게 할 수 있다.  
아래는 `views.properties` 파일 예시이다.  
```
hello.(class)=org.springframework.web.servlet.view.JstlView
hello.url=/WEB-INF/view/hello.jsp

bye.(class)=org.springframework.web.servlet.view.velocity.VelocityView
bye.url=bye.vm
```
독립된 파일을 통해 뷰를 자유롭게 매핑할 수 있지만, 모든 뷰를 일일히 매핑해줘야 하는 불편도 뒤따른다.  
그래서 단독으로 사용하는 것은 추천되지 않고, 다른 뷰 리졸버와 함께하면 유용하게 사용될 수 있다.  
```xml
<bean class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="order" value="0" />
</bean>

<!-- order 기본값이 Integer.MAX 이므로 굳이 order 프로퍼티를 안줘도 됨 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" />
```
이렇게 사용하면 `view.properties`에 뷰가 없을 경우 아래 `InternalResourceViewResolver`를 사용하게 된다.  
이로써 특별한 타입의 뷰가 필요할 때만 `view.properties`에 작성해주며 사용할 수 있다.  

## XmlViewResolver
`ResourceBundleViewResolver`와 용도는 동일하고, `views.properties` 대신 `/WEB-INF/views.xml`을 사용한다.  
추가로 이 파일은 서블릿 컨텍스트를 부모로 가지므로 `DI`가 가능하다는 장점이 있다.

## BeanNameViewResolver
뷰 이름과 동일한 이름을 가진 빈을 찾아서 뷰로 이용하게 해준다.  

<div style="text-align: right">
From <img src="https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwNDAxMDkmdHlwZT1sJno9MjAxOC8wOC8wNiAwOTozOA==#width30" style="display:inline-block;"/>
</div>

<!-- more -->
