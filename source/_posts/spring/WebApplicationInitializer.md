---
title: WebApplicationInitializer
date: 2018-02-06 00:05:50
tags:
---
서블릿은 3.0 이후부터 web.xml 없이도 서블릿 컨테이너 초기화 작업이 가능해졌다.  
프레임워크 레벨에서 직접 초기화할 수 있게 도와주는 ServletContainerInitializer API를 제공하기 때문이다.  
> **서블릿 컨텍스트 초기화**  
web.xml에서 했던 서블릿 등록/매핑, 리스너 등록, 필터 등록 같은 작업들을 말한다.  

스프링은 웹 모듈내에 이미 ServletContainerInitializer를 구현한 클래스가 포함되어 있고,  
이는 WebApplicationInitializer 인터페이스를 구현한 클래스를 찾아 초기화 작업을 위임하는 역할을 수행한다.  
```java
public interface WebApplicationInitializer{
    void onStartup(ServletContext servletContext) throws ServletException;
}
```
이 인터페이스를 구현한 클래스를 만들어두면 웹 어플리케이션이 시작할 때 자동으로 onStartup() 메서드가 실행된다.  
여기서 초기화 작업을 수행하면 된다.  

### 루트 웹 컨텍스트 등록
기존에는 아래와 같이 web.xml에 리스너 형태로 등록했다.  
```xml
<listener>
    <!-- /WEB-INF/applicationContext.xml 생성-->
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>    
</listener>
```
> **리스너의 역할**  
// 서블릿 컨텍스트? 서블릿 컨테이너?
리스너는 서블릿 컨텍스트가 생성하는 이벤트를 전달받는 역할을 한다.  
서블릿 컨텍스트가 만드는 이벤트는 컨텍스트 초기화 이벤트와 종료 이벤트이다.  
즉 웹 어플리케이션이 시작되고 종료되는 시점에 이벤트가 발생하고, 리스너를 등록해두면 이를 받을 수 있는 것이다.  

스프링은 왜 루트 애플리케이션 컨텍스트를 서블릿 리스너 레벨에서 등록하게 했을까?  
이는 위의 리스너의 역할에 보이듯이 루트 애플리케이션 컨텍스트의 생명주기가 서블릿 컨테이너와 일치하기 때문이다.  

아래와 같이 리스너를 등록해준다.  
```java
ServletContextListener listener = new ContextLoaderListener();
servletContext.addListener(listener);
```
> WebApplicationInitializer의 onStartup은 서블릿 컨텍스트 초기화 시점에 실행된다고 했으니 리스너를 등록하지 않아도 된다고 생각할 수 있으나,  
초기화 시점만 잡을 수 있지 종료 시점을 잡을수 없으므로 위와 같이 Listener는 등록해줘야 한다.  
종료 시점을 잡아주지 않으면 애플리케이션이 종료되어도 리소스가 반환되지 못하므로 메모리 누수가 일어날 수 있다.  

디폴트 루트 컨텍스트 클래스(XmlWebApplicationContext)와 디폴트 XML 설정파일(/WEB-INF/applicationContext.xml)을 바꾸고 싶을때 web.xml에선 아래와 같이 했었다.  
```xml
<!-- 애플리케이션 컨텍스트 클래스 변경 -->
<context-param>
    <param-name>contextClass</param-name>
    <param-value>
    org.springframework.web.context.support.AnnotationWebApplicationContext
    </param-value>
</context-param>

<!-- 설정파일 위치 변경 -->
<context-param>
    <param-name>contextConfigLoaction</param-name>
    <param-value>/WEB-INF/root-context.xml</param-value>
</context-param>
```
이는 서블릿 오브젝트의 setInitParameter() 메서드를 통해 지정할 수 있다.  
```java
// 애플리케이션 컨텍스트 클래스 변경 
servletContext.setInitParameter("contextClass", "org.springframework.web.context.support.AnnotationWebApplicationContext");
// 설정파일 위치 변경
servletContext.setInitParameter("contextConfigLoaction", "/WEB-INF/root-context.xml");
```

만약 Java Config를 사용한다면 더 깔끔하게 사용할 수 있다.  
```java
AnnotationConfigWebApplicationContext rootContext = new AnnotationConfigWebApplicationContext();
rootContext.register(RootConfig.class); // RootConfig == root-context java config

servletContext.addListener(new ContextLoaderListener(rootContext));
```
register() 대신 scan() 메서드를 사용하여 패키지를 통째로 스캔할수도 있다.  

### 서블릿 컨텍스트 등록
서블릿 컨텍스트는 서블릿 안에서 초기화되고 서블릿이 종료될 떄 같이 종료된다.  
이때 사용되는 서블릿이 DispatcherServlet이다.  
기존의 DispatcherServlet 등록은 아래와 같이 작성했었다.  
```xml
<servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <sevlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
이를 자바코드로 바꾸면
```java
ServletRegistration.Dynamic dispatcher = servletContext.addServlet("appServlet", new DispatcherServlet());
dispatcher.setInitParameter("contextConfigLocation", "/WEB-INF/spring/appServlet/servlet-context.xml");
dispatcher.setLoadOnStartUp(1);
dispatcher.addMapping("/");
```
이 또한 루트 애플리케이션 컨텍스트 처럼 Java Config를 사용할 수 있다.  
```java
AnnotationConfigWebApplicationContext sac = new AnnotationConfigWebApplicationContext();
sac.register(WebConfig.class);

ServletRegistration.Dynamic dispatcher = servletContext.addServlet("appServlet", new DispatcherServlet(sac));
dispatcher.setLoadOnStartUp(1);
dispatcher.addMapping("/");
```

여기까지가 루트 컨텍스트와 서블릿 컨텍스트를 자바 코드로 분리한 과정이다.  
필터나 리스너들도 이런식으로 모두 등록할 수 있고, 그 이후엔 web.xml을 아예 제거할 수도 있다.  

그 전까진 WebApplicationInitializer와 web.xml을 같이 사용할 수 있다.  
대신 아래와 같이 web.xml에 version을 명시해줘야 한다.  
```xml
<web-app version="3.0">
    <!-- 
    ....
    -->
</web-app>
```