---
title: '@RequestMapping'
date: 2018-03-03 00:21:23
tags:
    - '@RequestMapping'
    - DefaultAnnotationHandlerMapping
    - AnnotationHandlerAdapter
    - 토비의 스프링
photo: 
    - https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTIwOTU2NzA4ODYmdHlwZT1sJno9MjAxOC8wNC8yMiAxMjo1Nw==
---

`@RequestMapping`은 `DefaultAnnotationHandlerMapping`에서 컨트롤러를 선택할 때 대표적으로 사용하는 애노테이션이다.  
url당 하나의 컨트롤러에 매핑되던 다른 핸들러 매핑과 달리 메서드 단위까지 세분화하여 적용할 수 있으며,  
url 뿐 아니라 파라미터, 헤더 등 더욱 넓은 범위를 적용할 수 있다.  

---

# 속성
`DefaultAnnotationHandlerMapping`은 클래스와 메서드에 붙은 `@RequestMapping` 애노테이션 정보를 결합해 최종 매핑정보를 생성한다.  
기본적인 결합 방법은 클래스 레벨의 `@RequestMapping`을 기준으로 삼고,  
메서드 레벨의 `@RequestMapping`으로 세분화하는 방식으로 사용된다.  
`@RequestMapping`에 사용할 수 있는 속성들은 아래와 같다.  

## String[] value
URL 패턴을 지정하는 속성이다.  
String 배열로 여러개를 지정할 수 있으며, ANT 스타일의 와일드카드를 사용할 수 있다.  
```java
@RequestMapping(value="/post")
@RequestMapping(value="/post.*")
@RequestMapping(value="/post/**/comment")
@RequestMapping(value={"/post", "/P"})
```

`{}`를 사용하는 URI 템플릿을 사용할 수도 있다.  
```java
@RequestMapping(value="/post/{postId}")
```
`{}`를 패스 변수라고 부르며 컨트롤러에서 파라미터로 전달받을 수 있다.  

참고로 URL 패턴은 디폴트 접미어 패턴이 적용되므로 아래 2개는 동일한 의미이다.  
```java
// 2개는 동일하다!
@RequestMapping(value="/post")
@RequestMapping(value={"/post", "/post/", "/post.*"})
```

## RequestMethod[] method
`RequestMethod`는 HTTP 메서드를 정의한 ENUM이다.  
`GET, POST, PUT, DELETE, OPTIONS, TRACE`로 총 7개의 HTTP 메서드가 정의되어 있다.  
`@RequestMapping`에 `method`를 명시하면 똑같은 URL이라도 다른 메서드로 매핑해줄 수 있다.  
```java
// url이 /post인 요청 중 GET 메서드인 경우 호출됨
@RequestMapping(value="/post", method=RequestMethod.GET)

// url이 /post인 요청 중 POST 메서드인 경우 호출됨
@RequestMapping(value="/post", method=RequestMethod.POST)
```
이로인해 컨트롤러에서 번거롭게 HTTP 메서드를 확인할 필요가 없다.  

## String[] params
요청 파라미터와 값으로도 구분할 수 있다.  
String 배열로 여러개를 지정할 수 있으며, 아래와 같이 사용 가능하다.  
```java
// /post?useYn=Y 일 경우 호출됨
@RequestMapping(value="/post", params="useYn=Y")

// not equal도 가능
@RequestMapping(value="/post", params="useYn!=Y")

// 값에 상관없이 파라미터에 useYn이 있을 경우 호출됨
@RequestMapping(value="/post", parmas="useYn")

// 파라미터에 useYn이 없어야 호출됨
@RequestMapping(value="/post", params="!useYn")
```
GET 파라미터만이 아닌 POST 파라미터 또한 비교 대상이다.  
폼 내에서 input 태그 등으로 넘겨도 위의 매핑을 적용할 수 있다.  

근데 만약 아래와 같은 상황이라면 어떻게 될까?  
```java
// 요청 : /post?useYn=Y
@RequestMapping(value="/post")
@RequestMapping(value="/post", params="useYn=Y")
```
요청이 2가지 매핑을 다 만족하지만 이럴 경우 구현이 상세한 쪽으로 선택된다.  

## String[] headers
헤더 값으로 구분할 수 있다.  
방식은 위의 params와 비슷하다.  
```java
@RequestMapping(value="/post", headers="content-type=text/*")
```

---

# 매핑
핸들러 매핑이란 원래 오브젝트를 결정하는 전략이다.  
`@RequestMapping`으로 메서드 레벨까지 세분화하여 작성하긴 했지만,  
일관성을 위해 `DefaultAnnotationHandlerMapping`은 오브젝트까지만 찾아주고,  
최종 실행할 메서드는 `AnnotationHandlerAdapter`가 결정한다.

## 타입 레벨 + 메서드 레벨 매핑
타입 레벨(클래스 레벨)에서 공통 조건을 지정하고, 메서드 레벨에서 이를 세분화한다.  
그리고 URL 요청 시 이 둘을 조합하여 최종 조건이 결정된다.  
```java
@RequestMapping(value="/post")
public class PostController{
    // /post/add 에 매핑
    @RequestMapping(value="/add") 

    // /post/modify 에 매핑
    @RequestMapping(value="/modify")
    
    // /post/remove 에 매핑
    @RequestMapping(value="/remove")
}
```

타입 레벨에 ANT 패턴을 사용했을 경우에도 메서드 레벨과 조합될 수 있다.  
```java
@RequestMapping(value="/post/**")
public class PostController{
    // /post/**/add 에 매핑
    @RequestMapping(value="/add")
}
```
타입 레벨에 url을 주고 메서드 레벨엔 다른 매핑조건을 줄수도 있다.  

```java
@RequestMapping(value="/post")
public class PostController{
    @RequestMapping(method=RequestMethod.GET)
    @RequestMapping(method=RequestMethod.POST)
}
```
타입 레벨에도 메서드 레벨과 같이 `method`나 `params`등을 줄 수 있다.  

```java
@RequestMapping(value="/post", params="useYn=Y")
public class PostController{
    // /post?useYn=Y&isForeign=Y 에 매핑됨
    @RequestMapping(params="isForeign=Y")
}
```
타입 레벨에서 공통 조건을 지정하고 메서드 레벨에서 세분화 해준다는 개념만 지키면 어떤식의 조합도 가능하다.  

## 메서드 레벨 단독 매핑
매핑 조건에 딱히 공통점이 없는 경우 메서드 레벨에서만 매핑정보를 지정할 수 있다.  
하지만 위에서 얘기했듯이 `DefaultAnnotationHandlerMapping`은 오브젝트까지만 찾아주므로,  
타입 레벨에 조건 없는 `@RequestMapping`을 선언해 매핑 대상으로 만들어야 한다.  
```java
@RequestMapping
public class HomeController{
    @RequestMapping(value="/post")
    @RequestMapping(value="/magazine")
    @RequestMapping(value="/notice")
}
```

근데 만약 컨트롤러 클래스에 `@Controller` 애노테이션을 붙여 빈 자동 스캔으로 등록되게 했다면 타입 레벨 `@RequestMapping`은 생략할 수 있다.  
`@Controller` 애노테이션을 보고 `@RequestMapping`을 사용한 클래스라고 판단하기 때문이다.  

## 타입 레벨 단독 매핑
핸들러 매핑과 핸들러 어댑터는 서로 독립적인 전략이므로 아래와 같이 조합될 수 있다.  
```java
@RequestMapping(value="/post")
public class PostController implements Controller{
    @Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        // ....
    }
}
```

---

# 상속
`@RequestMapping`이 적용된 클래스를 다른 클래스에서 상속하여 사용할 수 있다.  
상속 시 서브 클래스는 부모의 `@RequestMapping`을 상속받을 수 있게된다.  
근데 만약 서브클래스 내에서 `@RequestMapping`을 재정의 할 경우 부모의 정보는 무시된다.  

## 부모 @RequestMapping 상속
부모 클래스에 `@RequestMapping`이 정의되어있고,  
자식에서 `@RequestMapping`을 재정의 하지 않을 경우 `@RequestMapping`은 그대로 상속된다.  
```java
@RequestMapping(value="/post")
public class PostController{
    @RequestMapping(value="/list")
    public String list(){ 
        // ... 
    }
}

public class ChildPostController extends PostController{
    @Override
    public String list(){
        // ...
    }
}
```
`ChildPostController`의 `list` 메서드는 `/post/list`에 매핑된다.  

## 부모 @RequestMapping + 자식 @RequestMapping
상속으로 조합도 가능하다.  
```java
@RequestMapping(value="/post")
public class PostController{ 

}

public class ChildPostController extends PostController{
    @RequestMapping(value="/list")
    public String list(){
        // ...
    }
}
```
부모와 자식의 `@RequestMapping`이 조합되어 `/post/list`에 매핑된다.  

반대로도 가능하다.  
```java
public class PostController{
    @RequestMapping(value="/list")
    public String list(){
        // ...
    }
}

@RequestMapping(value="/post")
public class ChildPostController extends PostController{
    @Override
    public String list(){
        // ...
    }
}
```

## 자식 @RequestMapping 재정의
@RequestMapping을 재정의 할 경우 모든 조건이 다 재정의 된다는 점에 주의하여야 한다.  
```java
@RequestMapping(value="/post")
public class PostController{
    @RequestMapping(value="/list", params="useYn=Y")
    public String list(){
        // ...
    }
}

public class ChildPostController extends PostController{    
    @Override
    @RequestMapping(value="/contents")
    public String list(){
        // ...
    }
}
```
`value`만 재정의 한것 처럼 보이나 실제로는 부모 `list` 메서드의 `@RequestMapping` 자체를 모두 재정의 한것이 된다.  
`params` 속성은 사라지게 된다.  

## 상속과 제네릭스를 이용한 매핑 전략
이제 앞서 나열한 상속 성질들을 이용하면 공통 컨트롤러를 만들 수 있다.  
대상으로는 도메인 오브젝트의 CRUD가 적합하다.  
CRUD의 경우 대부분이 서비스를 호출하는 위임코드가 많다는 점을 이용하면 아래와 같이 코드를 대폭 줄일 수 있다.  
```java
public abstract class CrudController<T, K, S>{
    S s;
    @RequestMapping(value="/list")
    public List<T> list(ModelMap map){ 
        // ...
    }

    @RequestMapping(value="/detail")
    public T detail(K id){
        // ...
    }

    @RequestMapping(value="/add")
    public void add(T entity){
        // ...
    }

    @RequestMapping(value="/modify")
    public void modify(T entity){
        // ...
    }

    @RequestMapping(value="/remove")
    public void remove(K id){
        // ...
    }
}

@RequestMapping(value="/post")
public class PostController extends CrudController<Post, Integer, PostService>{
    @RequestMapping(value="/best") // 공통 외에 필요한것은 따로 정의하면 된다  
    public List<Post> best(){
        // ...
    }
}
```
상속받고 `@RequestMapping`만 정의해줬을 뿐인데 중복되는 코드를 대폭 줄일 수 있게 되었다!  
만약 기본 메서드에 추가할 작업이 있다면 직접 구현해서 새로 작성하면 된다.  
그리고 CRUD외에 추가할 메서드가 있으면 `best` 메서드 처럼 넣어주면 되므로  
코드가 매우 간결해지고 개발 생산성이 대폭 상승된다.  

<div style="text-align: right">
From <img src="https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwNDAxMDkmdHlwZT1sJno9MjAxOC8wOC8wNiAwOTozOA==#width30" style="display:inline-block;"/>
</div>

<!-- more -->