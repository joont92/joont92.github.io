---
title: '@Controller'
date: 2018-03-04 21:37:23
tags:
    - '@RequestMapping param'
    - '@RequestMapping return'
    - '@ModelAttribute'
    - 토비의 스프링
photo: 
    - https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTIwOTU2NzA4ODYmdHlwZT1sJno9MjAxOC8wNC8yMiAxMjo1Nw==
---

여기서 말하는 `@Controller`란 빈 자동 스캔시 사용되는 스테레오 타입 애노테이션이 아니라,  
**애노테이션을 이용해 컨트롤러를 개발하는 방법**을 말한다.  
즉, `AnnotationMethodHandlerAdapter`가 실행하는 각 메서드들을 의미한다.  

---

# 파라미터
개발자가 명시한 애노테이션과 파라미터 타입 등에 따라 `AnnotationMethodHandlerAdapter`가 적절히 변환하여 제공해줌

## HttpServletRequest, HttpServletResponse, ServletRequest, ServletResponse
대게는 좀 더 상세한 파라미터 타입을 사용하면 되지만,   
원한다면 직접 `HttpServletRequest`, `HttpServletResponse` 타입을 받을 수 있다.  
`ServletRequest`, `ServletResponse` 타입도 가능하다.  

## HttpSession
`HttpServletRequest`에서 얻을 수 있는 `HttpSession`을 바로 받을 수 있다.  
`HttpSession`은 멀티스레드 환경에서 안전성이 보장되지 않으므로  
핸들러 어댑터의 `synchronizeOnSession` 프로퍼티를 `true`로 줘야한다.  

## Locale
`java.util.Locale` 타입으로 `DispatcherServlet`의 Locale Resolver가 결정한 `Locale` 오브젝트를 받을 수 있다.  

## InputStream, Reader
`HttpServletRequest`의 `getInputStream()`를 통해 받을 수 있는 `InputStream`과,  
`getReader()`를 통해 받을 수 있는 `Reader`를 바로 받을 수 있다.  

## OutputStream, Writer
`HttpServletResponse`의 `getOutputStream()`를 통해 받을 수 있는 `OutputStream`과,  
`getWriter()`를 통해 받을 수 있는 `Writer`를 바로 받을 수 있다.  

## @PathVariable
`@RequestMapping` url에 `{}`로 들어가는 패스 변수를 받는다.  
```java
@RequestMapping(value="/post/{postNo}")
public String detail(@PathVariable("postNo") Integer postNo){
    // ...
}
```
속성으로 받을 패스 변수의 이름을 지정할 수 있으며,  
전달받은 패스 변수는 선언한 파라미터 타입으로 형변환 된다.  
즉, `/post/10` 이라고 요청하게 되면 `postNo` 변수에 `Integer` 타입으로 형 변환되어 담기게 된다.  
만약 `/post/notNumber` 과 같은 형태로 요청하여 형변환이 불가능 할 경우  
`400 Bad Request` 에러가 발생한다.  

## @RequestParam
`HttpServletRequest`의 `getParameter()`로 받을 수 있는 파라미터를 바로 받을 수 있다.  
전달받은 파라미터는 선언한 파라미터 타입으로 자동 형 변환된다.  
또한 필수여부, 디폴트 값 등을 설정할 수 있다.  
```java
// page라는 이름으로 전달된 파라미터를 받아 Integer 타입으로 변환한다
public String list(@RequestParam("page") Integer page)

// 필수 여부와 디폴트 값을 줄 수 있다. 필수 여부는 default가 true이다.  
public String list(@RequestParam(value="page", required=false, defaultValue="1") Integer page)
```

파라미터 타입을 `Map`으로 선언하면 모든 파라미터를 맵으로 받을 수 있다.  
```java
public String list(@RequestParam Map<String, String> params)
```

## @CookieValue
쿠키값을 받아올 수 있다. 속성으로 쿠키의 이름을 지정해주면 된다.  
```java
// 쿠키 name이 auth인 것을 가져온다
public String list(@CookieValue("auth") String auth)

// @RequestParam과 마찬가지로 필수 여부와 디폴트 값을 줄 수 있다.
// 필수 여부 default는 true이다.
public String list(@CookieValue(value="auth", required=false, defaultValue="NONE") String auth)
```

## @RequestHeader
헤더값을 받아올 수 있다. 속성으로 헤더의 이름을 지정해주면 된다.  
`@RequestParam`, `@CookieValue`와 마찬가지로 `required`, `defaultValue`를 설정해 줄 수 있다.  
```java
public String list(@RequestHeader("Host") String host)
```

## Model, ModelMap, Map
모델 정보를 담을 수 있는 `Model`과 `ModelMap` 객체를 파라미터 레벨에서 바로 받을 수 있다.  
`Map`도 앞에 특별한 애노테이션이 없다면 모델 정보를 담는데 사용할 수 있다. 하지만 갠적으로 좀 헷갈린다.. 안써야지  
```java
public String list(ModelMap model){
    model.addAttribute("key", "value");

    // collection에 담긴 모든 오브젝트를 모델에 추가할 수 있다(자동 이름 생성 방식을 통해)
    model.addAllAttribute(collection);
}
```

## @ModelAttribute
이름에 `Model`이 들어가 있긴 하지만 우리가 일반적으로 사용하는 모델과는 조금 의미가 다르다.

컨트롤러가 받는 요청정보 중에서, 하나 이상의 값을 가진 오브젝트 형태로 만들 수 있는 정보를 `@ModelAttribute` 모델이라고 부른다.  
`@ModelAttribute`라고 별다를 건 없다.
기존과 똑같이 파라미터를 받는데,  
그걸 메서드에서 1:1로 받으면 `@RequestParam`인거고  
도메인 오브젝트나 DTO에 바인딩해서 받으면 `@ModelAttribute` 인 것이다.  

사용자가 리스트에서 검색할 떄 사용하는 파라미터를 한번 생각해 보자.  
기본적으로 전달될 파라미터는 검색 키워드겠고, 그 외에도 검색 타입, 페이지 번호 등이 전달 될 수 있다.  
이를 기존의 @RequestParam으로 표현하면 아래와 같이 된다.  
```java
public String search(
    @RequestParam("q") String q,
    @RequestParam("type") String type,
    @RequestParam(value="page", required=false, defaultValue="1") Integer page){

    service.search(q, type, page);
}
```

일단 서비스 메서드 부터 문제가 있다.. 저런식으로 파라미터를 나열할 경우 변경에 매우 취약하게 되며,  
같은 타입의 파라미터가 여러개면 실수할 가능성이 매우 높아진다.  
이럴 경우에는 아래와 같이 오브젝트를 하나 만들어 전달하는 편이 낫다.  
```java
public class Search{
    private String q;
    private String type;
    private Integer page;

    // getter, setter
}
```

서비스 메서드는 이 오브젝트를 사용하며 해결이 가능한데, 오브젝트를 매번 초기화 해줘야 한다는 귀찮음이 따른다.  
이럴떄 사용할 수 있는것이 `@ModelAttribute`이다!  
```java
public String search(@ModelAttribute Search search){

    service.search(search);
}
```

이제 요청 파라미터들은 자동으로 `Search` 오브젝트에 바인딩 되어 들어오게 된다(타입 변환도 자동으로 된다).  
코드가 매우 깔끔해지고 위에서 언급한 문제점 또한 단번에 해결할 수 있다.  

`@ModelAttribute`는 위와 같이 사용할 수도 있지만 보통은 폼의 데이터를 받을 때 훨씬 유용하게 사용할 수 있다.  
게다가 `@ModelAttribute`의 기능중에 하나가 전달받음과 동시에 컨트롤러가 리턴하는 모델에 자동으로 추가해준다는 점이다.  
이로인해 사용자가 입력을 잘못했을 경우에도 입력한 모델을 다시 출력해주며  
잘못 입력한 정보에 대해 재입력을 요구하는 기능을 쉽게 구현할 수 있게 된다.  

## Errors, BindingResult
`@ModelAttribute`와 같이 사용하는 파라미터 들이다.  
기본적으로 `@ModelAttribute`는 파라미터를 처리할 떄 `@RequestParam`과는 달리 검증 작업이 추가적으로 진행된다.  
검증작업이란 기본적으로 진행되는 타입 변환 외에도 필수 정보 입력 여부, 길이 제한, 값 허용 범위 등 다양한 기준이 적용될 수 있다.  

`BidingResult`와 `Errors`는 이러한 검증작업의 결과가 담겨지는 곳이다.  
컨트롤러에서는 이 두 오브젝트의 결과를 통해 사용자에게 적절한 조치를 취할 수 있게 되는 것이다(검증 에러가 난 부분에 대해 재입력 요구 등).  
이러한 특성 때문에 `@ModelAttribute`는 바인딩에서 타입 변환이 실패하는 오류가 발생해도 400 에러를 발생시키지 않는다.  
타입 변환 실패 또한 검증의 한 단계로 보고 `BindingResult`에 그에 해당하는 결과만을 담을 뿐이다.  
`BindingResult`의 검증결과에서 오류가 없다고 나오면 그제서야 로직을 통과시키고, 오류가 있을 경우 사용자에게 계속 수정을 요구해야 한다.  
```java
public String add(@ModelAttribute User user, BindingResult result){
    if(result.hasError()){
        // 재입력 요구
    } else{
        userService.add(user);
    }
}
```
`BindingResult`는 반드시 `@ModelAttribute` 뒤에 나와야 한다.  
현재 위의 메서드로는 기본적인 검증인 타입 변환 검증만을 수행하는 상태이다.  
[모델 바인딩과 검증 바로가기](/spring/모델-바인딩과-검증)  

## SessionStatus
`@SessionAttributes`를 통해 저장된 현재 세션을 다룰 수 있는 오브젝트이다.  
[@SessionAttributes, SessionStatus 바로가기](/spring/@SessionAttributes,-SessionStatus)  

## @RequestBody
이 애노테이션이 붙은 파라미터에는 HTTP 요청의 본문 부분이 그대로 전달된다.  
XML이나 JSON 기반으로 요청하는 경우 매우 유용하게 사용된다.  
`AnnotationMethodHandlerAdapter`에는 `HttpMessageConverter`타입의 메세지 변환기가 여러개 등록되어 있다.  
`@RequestBody`가 붙은 파라미터가 있으면 요청의 미디어 타입을 먼저 확인한 후,  
메세지 변환기들 중에서 이를 처리할수 있는것이 있다면 HTTP 요청 본문 부분을 통째로 변환하여 파라미터로 전달해준다.  
```java
public String test(@RequestBody String body){		
	System.out.println(body); // http body가 그대로 출력

	return "test";
}
```
`StringHttpMessageConverter`가 http body를 그대로 받고 있다.  

## @Value
시스템 프로퍼티나 다른 빈의 프로퍼티 값, `SpEL`등을 이용하는데 사용된다.  
파라미터 변수 뿐 아니라 필드 변수에도 사용할 수 있다.  
```java
// 필드로 받기
@Value("#{'systemProperties['os.name']'}") String osName;
// 파라미터로 받기
public String hello(@Value("#{systemProperties['os.name']}") String osName){
  // ...
}
```
상황에 따라 적절히 선택해서 사용하면 된다.  

## @Valid
@ModelAttribute 검증에 사용되는 애노테이션이다.  
[모델 바인딩과 검증 바로가기](/spring/모델-바인딩과-검증)  

---

# 리턴
파라미터 뿐만 아니라 리턴 타입도 다양하게 사용할 수 있다.  
각 리턴 타입에 대해 다양한 결과를 얻어낼 수 있다.
참고로 어떤 방식으로 리턴하든 마지막에는 `ModelAndView`로 만들어져 `DispatcherServlet`에 전달된다.  

## 모델에 자동으로 추가되는 오브젝트
리턴 타입을 알아보기 전에 굳이 명시하지 않아도 모델에 자동으로 추가되는 오브젝트들 부터 살펴보자.  

1. @ModelAttribute 파라미터
파리미터에서 `@ModelAttribute`로 받은 오브젝트는 자동으로 모델에 추가된다.  
모델 오브젝트의 이름은 기본적으로 파라미터 타입 이름을 따른다.  
이름을 직접 지정하고 싶으면 `@ModelAttribute("모델이름")` 의 형태로 지정해주면 된다.  

2. Map, Model, ModelMap
파라미터에 `Map`, `Model`, `ModelMap` 타입의 오브젝트를 사용하면 미리 생성된 모델 맵 오브젝트를 전달받을 수 있다.  
이후 추가하고 싶은 모델 오브젝트가 있으면 여기에 추가하면 된다.  

3. @ModelAttribute 메서드
파라미터를 오브젝트로 받는 `@ModelAttribute`의 기능보단 공통적으로 사용되는 모델 오브젝트를 정의하기 위해 유용하게 사용되는 방식이다.  
```java
@ModelAttribute("countries")
public List<Country> countries(){
  return commonService.getCountries();
}
```
이런식으로 클래스 내에 별도로 정의해놓으면 클래스 내의 다른 메서드들의 모델에 자동으로 추가된다.  
같은 클래스 내의 메서드들의 모델에는 항상 `"countries"` 이름의 `List<Country>` 오브젝트가 추가되어있는 것이다.  
`<select>` 태그를 써서 선택 가능한 목록을 보여주는 경우가 대표적이다.  

4. BidingResult
`@ModelAttribute`와 같이 사용하는 `BindingResult`도 모델에 자동으로 추가된다.  
모델 맵에 추가될때의 키는 `'org.springframework.validation.BindingResult.모델이름'` 이다.  

## ModelAndView
컨트롤러가 리턴해야 할 정보를 담고 있는 가장 대표적인 클래스이다.  
하지만 이것보다 편한 방법이 훨씬 많으므로 자주 사용되진 않는다.  
```java
public ModelAndView test(){
  ModelAndView mv = new ModelAndView();
  mv.addObject("key", "value");
  mv.setViewName("test");

  return mv;
}
```
참고로 `ModelAndView`를 리턴하더라도 `Map`, `Model`, `ModelMap` 파라미터는 모델에 자동 추가된다.  

## String
문자열을 리턴하면 이는 뷰 이름으로 사용된다.  
모델은 `Model`, `ModelMap` 파라미터를 이용한다.  
```java
public String test(ModelMap modelMap){
  modelMap.addAttribute("key", "value");

  return "test";
}
```

## void
아예 아무것도 리턴하지 않을경우 `RequestToViewNameResolver`에 의해 자동으로 뷰 이름이 생성된다.  
뷰 이름을 일관되게 통일해야 하므로 규칙을 잘 정해야 한다.  

## 모델 오브젝트
`RequestToViewNameResolver`를 사용해서 뷰 이름을 자동생성하고,  
모델에 추가할 오브젝트가 하나뿐이라면 모델 오브젝트를 바로 반환해도 된다.  
스프링은 리턴 타입이 단순 오브젝트이면 이를 모델 오브젝트로 인식해서 모델에 자동으로 추가해준다.  
모델명은 모델 오브젝트의 타입 이름을 따른다.  
```java
public List<Post> getPostList(){

  return postService.getPostList(); // return List<Post>
}
```

## Map/Model/ModelMap
메서드에서 직접 `Map`/`Model`/`ModelMap` 오브젝트를 생성해서 리턴하면 모두 모델로 사용된다.  
하지만 모델은 파라미터로 받을 수 있기 때문에 이 방식은 잘 사용되지 않는다.  

여기서 한가지 주의해야 할 점이 있다. 바로 `Map` 오브젝트이다.  
서비스 메서드에서 결과로 `Map`을 내려주는 경우가 있는데, 이를 모델 오브젝트라고 생각하고 바로 리턴했다가는 원치않은 결과를 얻게 된다.  
`Map`은 모델 오브젝트가 아닌 모델 맵으로 인식되기 때문에, `Map`의 모든 속성이 모델에 추가되는 상황이 발생한다.  
```java
// 잘못된 코드!!
public Map getUser(){
  Map user = userService.getUser();

  return user; // user의 모든 속성이 모델에 추가되어 리턴된다.
}
```

## View
뷰 이름대신 직접 View 오브젝트를 넘겨도 된다.  
```java
public View getPostListByExcel(ModelMap modelMap){
  // model add..

  return excelView; // excel view
}
```

## @ResponseBody
`@ReuqestBody`와 비슷하게 동작한다.  
메서드 레벨에 이 애노테이션이 붙어있으면, 리턴하는 오브젝트가 뷰를 통해 결과를 만들어내는데 사용되지 않고 메세지 컨터버를 통해 바로 HTTP 응답으로 반환된다.  
```java
@ResponseBody
public String test(){			
	return "succeed"; // 문자열 그대로 반환
}
```
`@ResponseBody`에 의해 `succeed`가 `viewName`으로 사용되지 않고 HTTP 응답으로 반환된다.  

<div style="text-align: right">
From <img src="https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwNDAxMDkmdHlwZT1sJno9MjAxOC8wOC8wNiAwOTozOA==#width30" style="display:inline-block;"/>
</div>

<!-- more -->
