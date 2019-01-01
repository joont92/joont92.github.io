---
title: HandlerExceptionResolver, LocaleResolve, MulitpartResolver
date: 2018-02-04 23:22:22
tags:
    - HandlerExceptionResolver
    - LocaleResolver
    - MultipartResolver
    - 토비의 스프링
---

`HandlerMapping`, `HandlerAdapter`, `ViewResolver` 등 외에도 `DispatcherServlet`에는 다양한 확장 가능한 전략들이 존재한다.  
아래는 남은 전략들 중 중요한 것들만을 나열한 것이다.  

# 핸들러 예외 리졸버
`HandlerExceptionResolver`는 컨트롤러 작업 중 발생한 예외를 어떻게 처리할 지 결정하는 전략이다.  
기본적으로 컨트롤러나 그 뒷 계층에서 발생한 예외는 일단 `DispatcherServlet`이 전달받은 다음 다시 서블릿으로 던져서 서블릿 컨테이너가 처리하게 된다.  
별다른 처리를 하지 않았다면 **500 Internal Server Error** 같은 메시지가 출력될 것이다.  
그런데 핸들러 예외 리졸버가 등록되어있다면 `DispatcherServlet`은 먼저 이 리졸버가 해당 예외를 처리할 수 있는지 확인한다.  
만약 해당 예외를 처리할 수 있으면 예외는 `DispatcherServlet` 밖으로 던져지지 않고 해당 핸들러 예외 리졸버가 처리한다.  

핸들러 예외 리졸버는 `HandlerExceptionResolver` 인터페이스를 구현해서 생성한다.  
```java
public interface HandlerExceptionResolver{
    ModelAndView resolveException(HttpServletRequest reqeust, HttpServletResponse response, Object handler, Exception ex);
}
```
구현 메서드인 `resolveException`의 리턴 타입은 `ModelAndView`이다.  
예외에 따라 사용할 뷰와 모델을 돌려주도록 되어있다.  
만약 처리 불가능한 예외라면 `null`을 리턴한다.  

스프링은 이미 4개의 `HandlerExceptionResolver` 구현 전략을 제공한다.  

## AnnotationMethodHandlerExceptionResolver
예외가 발생한 컨트롤러 내의 메서드 중에서 `@ExceptionHandler` 애노테이션이 붙은 메서드를 찾아 예외처리를 맡긴다.  
```java
@Controller
public class HelloController{
    @RequestMapping(value="/hello")
    public void hello(){
        // DataAccessException occured!
    }

    @ExceptionHandler(DataAccessException.class)
    public ModelAndView dataAccessExceptionHandler(DataAccessException ex){
        return new ModelAndView("error").addObject("msg", ex.getMessage());
    }
}
```
특정 컨트롤러의 예외만을 처리하고 싶을때 유용하다.  

## ResponseStatusExceptionResolver
예외를 특정 HTTP 응답 상태코드로 전환해준다.  
예외클래스에 `@ResponseStatus` 애노테이션을 붙이고 `value`(응답 상태 값)를 지정한 뒤 해당 예외를 발생시키면 `ResponseStatusExceptionResolver`가 HTTP 응답을 변환해준다.  
단순한 HTTP 500 에러 대신 의미있는 응답 상태를 클라이언트에 전달하고자 할 떄 유용하게 사용된다.  
```java
@ResponseStatus(value=HttpStatus.SERVICE_UNVAILABLE, reason="이유도 지정 가능")
public class ServiceException extends RuntimeException {

}
```
위와 같이 `@ResponseStatus`가 붙은 Exception을 정의하고,  
```java
public void helloService(){
    // ....
    throw new ServiceException(); // exception 발생
}
```
이렇게 해당 Exception을 발생시키면 `@ResponseStatus`에 지정된 형태로 Response를 받을 수 있다.  

만약 위처럼 정의할 수 없는 기존 예외가 발생했을 때는 위의 `@ExceptionHandler`를 사용하면 된다.  
```java
@ResposneStatus(value=HttpStatus.SERVICE_UNVAILABLE)
@ExceptionHandler(DataAccessException.class)
public ModelAndView dataAccessExceptionHandler(DataAccessException ex){
    return new ModelAndView("error").addObject("msg", ex.getMessage());
}
```

## DefaultHandlerExceptionResolver
위 두 가지 예외리졸버에서 처리하지 못한 예외를 다루는 예외 리졸버이다.  
스프링에서 발생하는 주요 예외를 처리하는 표준 예외처리 로직을 담고 있다.  
즉, 이 예외 리졸버는 신경쓰지 않아도 된다.  

## SimpleMappingExceptionResolver
`web.xml`의 `<error-page>`와 비슷하게 예외를 처리할 뷰를 지정할 수 있게 해준다.  
(뷰를 찾을때는 뷰리졸버가 사용된다.)  
```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="mappedHandlers">
        <props>
            <!-- 클래스 이름에 패키지를 쓰지 않아도 됨 -->
            <prop key="DataAccessException">error/dao</prop>
            <prop key="BusinessLoginException">error/business</prop>            
        </props>
    </property>
    <property name="defaultErrorView">error/default</property>
</bean>
```
사실상 실제로 활용하기에 가장 편리한 예외 리졸버이다.  
사용자에게 어려운 HTTP 상태코드를 보여주는 것보다는 문구가 있는 예외 페이지를 보여주는 편이 낫기 때문이다.  
또한 모든 컨트롤러에서 발생하는 예외에 일괄 적용할 수 있다.  

예외페이지를 보여주는 외에 로그를 남기거나 관리자에게 통보를 해야할 경우 `SimpleMappingExceptionResolver` 보다는 핸들러 인터셉터의 `afterCompletion()` 메서드를 사용하는 것이 좋다.  
에러페이지에 에러로그를 남기는 것은 바람직하지 않기 때문이다.  

---

# 지역정보 리졸버
`LocaleResolver`는 애플리케이션에서 사용하는 지역정보를 결정하는 전략이다.  
디폴트인 `AcceptHeaderLocaleResolver`는 `HTTP 헤더의 지역정보`를 그대로 사용한다.  
보통 HTTP 헤더의 지역정보는 브라우저의 기본 설정에 따라 보내진다.  
브라우저 설정을 따르지 않고 사용자가 직접 변경하게 하려면 `SessionLocaleResolver`나 `CookieLocaleResolver`를 사용하는 것이 편리하다.  
해당 리졸버를 사용하면 사용자가 국가 선택 시 쿠키나 세션의 `locale` 값을 변경하여 해당 지역의 리소스 파일이 사용되게 할 수 있다.  
다국어 서비스에 유용하게 활용할 수 있다.  
[LocaleResolver 사용 예(http://yookeun.github.io/java/2015/08/12/spring-i18n/)](http://yookeun.github.io/java/2015/08/12/spring-i18n/)  

---

# 멀티파트 리졸버
멀티파트 포맷의 요청정보를 처리하는 전략이다.  
멀티파트 리졸버 전략은 디폴트 전략이 없으므로 아래와 같이 빈을 등록해줘야 한다.  
```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="100000" />
</bean>
```
`DispatcherServlet`은 클라이언트로부터 멀티파트 요청을 받으면 멀티파트 리졸버에게 요청해서 `HttpServletRequest`의 확장 타입인 `MultipartHttpServletRequest`로 변환한다.  
`MultipartHttpServletRequest`에는 멀티파트를 디코딩한 내용과 이를 참조하거나 조작할 수 있는 기능이 추가되어 있다.  
이후 컨트롤러에서는 아래와 같이 `MultipartHttpServletRequest`로 캐스팅하여 사용할 수 있다.  
```java
public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response){
    MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
    MultipartFile multipartFile = multipartRequest.getFile("image");
    // ....
}
```

# RequestViewNameTranslator
컨트롤러에서 뷰 이름이나 뷰 오브젝트를 돌려주지 않을 경우 요청 URL 정보를 기준으로 해서 뷰 이름을 생성해준다.  
디폴트로 DefaultRequestViewNameTranslator가 등록되어 있다.  
잘 활용하면 매번 뷰 이름을 지정하는 수고를 덜어줄 수 있다.

<div style="text-align: right">
From <img src="https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwNDAxMDkmdHlwZT1sJno9MjAxOC8wOC8wNiAwOTozOA==#width30" style="display:inline-block;"/>
</div>

<!-- more -->
