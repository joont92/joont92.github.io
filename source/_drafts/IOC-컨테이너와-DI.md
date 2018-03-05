---
title: IOC 컨테이너와 DI
tags:
---

# IOC 컨테이너와 DI

어플리케이션 컨텍스트 = IOC + DI + 여러 기능(for making enterprise application)  
(어플리케이션 컨텍스트는 실제 ApplicationContext 인터페이스를 구현한 클래스를 말한다.  
ApplicationContext는 BeanFactory를 구현한 인터페이스이다.)  

이 어플리케이션 컨텍스트가 본격적인 IOC 컨테이너로 동작하려면 아래의 2가지가 필요하다.
> POJO, 설정 메타정보

어플리케이션 컨텍스트 구현체, POJO, 설정 메타정보가 준비되었으면 이제 IOC 컨테이너로 등록하고 사용해야한다.  
웹은 싱글 어플리케이션 처럼 main 메서드가 있는 것이 아니므로, 아래와 같은 방식을 통해 등록한다.  
1. 컨테이너가 기동될 떄 미리 어플리케이션 컨텍스트를 만든다.(web.xml의 listener, servlet등을 이용)
2. 서블릿에서 요청이 올때마다 어플리케이션 컨텍스트에서 필요한 빈을 가져와 정해진 메서드를 실행한다.

그림으로 보면 아래와 같다.  
![image](https://user-images.githubusercontent.com/18513953/31042860-4ff60afc-a5ec-11e7-94da-0ac123c02e43.png)
메타정보에 관해서는 아래에서 설명할 것인데, 일단은 우리가 잘 알고 있는 **\*\*\*-context.xml** 파일이라고 생각하시면 됩니다.  
컨테이너가 기동될 때 POJO 클래스와 메타정보를 조합하여 각종 빈 들이 만들어지고, 그들을 관리하는 IOC 컨테이너가 생성되게 됩니다.(DispatcherServlet)  
그리고 각 HTTP 요청에 지정된 서블릿의 메서드가 실행될 때, 필요한 빈을 어플리케이션 컨텍스트에서 가져온 뒤 해당 메서드를 실행합니다.  


## 컨테이너 계층구조
스프링은 어플리케이션 컨텍스트를 계층구조로 가질 수 있다.

빈 탐색 : 자식 -> 부모  
부모에서 자식을 찾을 수 없으며, 형제 컨텍스트의 빈도 검색 불가능.  
자식과 부모에 중복된 이름의 빈이 있을 경우 자식의 빈을 사용(피해야 할 상황)  

하나의 컨텍스트에 정의된 AOP의 경우 다른 컨텍스트의 빈에는 영향을 미치지 않음  
상황에 따라 다른 설정을 사용해야 할 경우, 그런데 중요한 설정을 공유해야 할 경우 사용하면 효율적.  
그러나 제대로 알지 못하고 사용하면(계층구조, 탐색순서) 예기치 못한 에러를 만날 수 있음

### 웹 어플리케이션 컨테이너 계층구조
웹 어플리케이션 레벨에 등록하는 루트 컨테이너,
어플리케이션에 등록된 서블릿들이 각각 자신의 컨테이너를 가지는 구조가 대표적인 경우이다.  
일반적으로 스프링을 사용하여 웹 개발을 할 경우 프론트 컨트롤러 패턴을 사용하므로 대부분 아래와 같다.
```
서블릿 어플리케이션 컨텍스트 -> 루트 어플리케이션 컨텍스트
```
프론트 컨트롤러 패턴임에도 굳이 계층구조로 형성하는 이유는 아래와 같은 상황을 대비하기 위함이다 ㅎㅎ  
![image](https://user-images.githubusercontent.com/18513953/31043947-8ff5225a-a600-11e7-83ae-a7ddda60d0fb.png)
스프링의 유틸리티 메소드를 사용하면 간단하게 어플리케이션 컨텍스트를 얻어올 수 있다.
```java
WebApplicationContextUtils.getWebApplicationContext(ServletContext)
```
ServletContext는 웹 어플리케이션마다 하나씩 만들어지는것이므로  
HttpServletRequest나 HttpSession 오브젝트만 있으면 간단히 얻을 수 있다.

#### 루트 어플리케이션 컨텍스트 등록
서블릿 컨텍스트의 생성/소멸을 알려주는 web.xml의 listener를 사용하여 등록한다.
스프링이 제공해주는 리스너로는 ServletContextListener를 구현한 ContextLoaderListener가 있다.
```xml
<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
</context-param>
```

- contextConfigLocation
설정파일의 위치를 바꿔줄 때 사용
리소스 로더 접두어, ant식 경로표기법 사용가능

- contextClass
어플리케이션 컨텍스트 클래스를 바꿔줄떄 사능

지정해주지 않을 경우 default로 class는 XmlWebApplicationContext, xml 파일 위치는 /WEB-INF/applicationContext.xml 을 사용한다.

#### 서블릿 어플리케이션 컨텍스트 등록
스프링 웹 기능을 지원하는 프론트 컨트롤러 서블릿인 DispatcherServlet은
초기화시에 자신만의 컨텍스트를 생성하고 초기화하고, 
웹 어플리케이션 레벨에 등록된 어플리케이션 컨텍스트를 찾아서 자신의 부모 컨텍스트로 사용한다.

```xml
<servlet>
    <servlet-name>appServlet</servlet>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

이클립스에서 Sprig MVC 프로젝트를 만들었을 때 생성되는 기본 구조이다.
루트 어플리케이션 컨텍스트를 만들 때 처럼 설정파일의 위치나 클래스를 변경해 줄 수 있다.
(파라미터는 루트 어플리케이션 컨텍스트와 동일하고, init-param을 사용한다는 것만 다르다.)

지정해주지 않을 경우 default로 class는 XmlWebApplicationContext, xml 파일 위치는 /WEB_INF/{서블릿 네임}-context.xml 을 사용한다.

루트 어플리케이션 컨텍스트에 등록되는 빈 들은 각각 성격이 구분되므로 여러 설정파일로 나누고,
서블릿 어플리케이션 컨텍스트의 경우 성격이 비슷하므로 디폴트 설정파일 위치를 사용하는 것이 일반적이라고 한다(근데 위는 왜 저렇지..)


### IoC/DI를 위한 빈 설정 메타정보 작성

앞서 언급한 빈 설정 메타정보에 대해 알아보겠다.

IoC 컨테이너는 설정 메타정보를 참고하여 POJO 오브젝트들을 빈으로 등록한다.

설정 메타정보라고 하면 보통 우리에게 익숙한 *-context.xml 파일등이 떠오르게 되는데, 이는 사실 실제 설정 메타정보는 아니다.

아래의 그림을 보면 쉽게 이해할 수 있다.

![image](https://user-images.githubusercontent.com/18513953/31396515-8b47cc00-ae1e-11e7-85e4-3a756f5d71bf.png)

정확하게 말해 설정 메타정보란 **BeanDefinition**이라는 순수 오브젝트를 의미한다.

우리가 작성하는 xml파일, 애노테이션 등은 모두 적절한 리더기에 의해 BeanDefinition 오브젝트로 변환되는 것이다!

BeanDefinition에는 IoC 컨테이너가 빈을 만들 때 필요한 핵심정보가 담겨 있다.
그러나 필수항목 몇개를 제외하면 대부분 디폴트 값을 그대로 사용한다.

빈 등록법
1. \<bean\> 태그 : 가장 강력하면서도 심플한 방법

2. 네임스페이스 & 전용태그 : 1번의 문제점을 보완한 방식이다.
1번의 경우 모두 \<bean\> 태그로 통일하기 때문에 구분이 잘 안된다는 단점이 있다.
(스프링은 자신에게 필요한 설정정보에도 DI를 사용하기 때문에 이런 현상이 벌어지는 것이다.)
그래서 각각의 성격에 맞게 네임스페이스와 태그를 제공한다.
ex) \<aop:pointcut .. /\>
가독성, 필수 프로퍼티, 클래스 선택, 다중 클래스 등록 등 많은 이점이 있다.

3. 빈 스캐닝
빈 스캐너를 이용하여 인자로 전달한 패키지 아래의 클래스들을 검사한다.
@Component나 @Component를 메타 애노테이션으로 가지는 애들을 빈으로 등록한다.
이런 형태의 애노테이션을 스프링에선 **스테레오 타입**이라고 부른다
스캐닝으로 등록되는 빈은 이름을 등록하지 않으면 default로 클래스명에서 앞글자만 소문자로 사용한다.
애노테이션 디폴트값을 이용해 빈 이름을 지정할 수 있다. ex) @Component("test")
xml 작성 방법보다 간편하게 빈을 등록할순 있으나, 등록된 빈 들을 한눈에 볼 수 없다는 단점이 있다.
또한 xml 처럼 상세 메타정보 항목을 지정할 수 없고(한계가 있음), 클래스당 1개의 빈만 등록할 수 있다.

빈 스캐닝이 편리하긴 하지만 각각의 상황에 따라 xml과 빈 스캐닝을 적절히 선택하는 것이 좋다(세밀한 관리가 필요할 경우 등)

빈스캐너 등록법
xml : \<context:component-scan base-package="..."\>
컨텍스트 : 어플리케이션 컨텍스트 등록 시 contextClass로 AnnotationConfigWebApplicationContext 사용

* 스테레오 타입 종류
@Repository : DAO, repository
@Service : service
@Controller : mvc controller
> 특정계층으로 분류하기 힘든 경우는 @Component를 사용하는 것이 바람직

> 빈 스캐닝 시 컨테이너 계층 구조에서 유의할 점이 있다.  
> 부모 컨테이너와 자식 컨테이너의 빈 스캐닝 지점이 같은 곳을 바라보고 있을 경우 아래와 같은 문제점이 발생할 수 있다.  
> ![image](https://user-images.githubusercontent.com/18513953/32144077-bc69b2d0-bcf6-11e7-9544-f6726765c034.png)  
> 탐색의 순서가 자식 -> 부모의 순서이므로 위의 경우 AOP, TX가 적용된 UserService 빈을 사용하지 못하게 된다.

### java config
```java
@Configuration // <beans>
public class JavaConfig{
    @Bean // <bean>
    public Hello hello(){ // method name == bean name
        return new Hello();
    }
}
```
등록 방법 : AnnotationConfigWebApplictionContext 생성자 파라미터로 패키지 대신 위의 클래스를 넣어주면 된다.

@Configuration이 붙은 클래스도 빈으로 등록된다! (빈 이름 == 클래스명(첫글자 소문자))

- 일반적인 자바 코드와 다른점(유의사항)
@Bean 아래 메서드에서 new로 인스턴스를 생성했지만 실제로 빈을 가져다 사용해보면 계속해서 같은 오브젝트가 리턴된다(직접 실행해도 마찬가지이다)
> @Configuration내에 @Bean만 해당함

- 자바 설정파일을 사용할 때의 장점  
    1. 컴파일러나 IDE를 통한 검증이 가능하고, IDE의 기능을 최대한 이용할 수 있음.  
    XML은 텍스트이기 떄문에 한계가 있다.
    2. @Bean 자체가 자바 메서드이기 때문에 복잡한 빈 등록도 팩토리 빈 없이 쉽게 등록할 수 있다.

- 자주 사용하는 빈 등록 방법
1. XML 단독 사용  
    생성되는 모든 빈을 XML에서 확인할 수 있는 장점이 있는 반면, 빈의 개수가 많아지면 XML 파일을 관리하기 번거로울 수 있다.  
    설정을 분리하고 순수한 POJO 코드를 유지하고 싶을 떄 사용할 수 있다.  
    커스텀 태그나 전용태그를 사용할 경우 장점이 부각된다.  
    스프링이 제공하는 모든 빈 설정을 할 수 있는 유일한 방법이다.
2. XML + 빈 스캐닝  
    3계층 빈 클래스은 빈 스캔을 사용하고(복잡한 메타정보가 필요없으므로), 복잡한 설정 정보는 XML을 사용한다.  
3. XML 없이 단독 스캐닝 사용  
    자바 설정파일이 반드시 필요하다.   
    루트/서블릿 어플리케이션 컨텍스트를 전부 AnnotationConfigWebApplicationContext로 변경하고, @Configuration 클래스 파일을 빈 설정 대상에 포함시킨다.  
    스프링이 제공하는 전용 태그를 사용할 수 없다는 단점이 있다.