---
title: '@SessionAttributes, SessionStatus'
date: 2018-03-07 00:12:49
tags:
  - Session
  - '@SessionAttributes'
  - SessionStatus
  - 토비의 스프링
---

# 문제상황
아래는 기본적인 회원정보 수정 샘플 코드이다.  
```java
@Controller
public class UserController{
  // 수정 폼
  @RequestMapping(value="/user/{uid}", method=RequestMethod.GET)
  public String modifyForm(@PathVariable Long uid,
                          ModelMap modelMap){

    User user = userService.getUser(uid);
    modelMap.addAttribute("user", user);

    return "user/modify"
  }

  // 수정
  @RequestMapping(value="/user/{uid}", method=RequestMethod.POST)
  public String modify(@PathVariable Long uid,
                      @ModelAttribute User user){

    userService.modify(user);

    return "user/modifySuccess"
  }
}
```
위 코드는 기본적으로 문제가 있다.  
수정 폼을 출력하는 메서드에서 `uid`로 회원을 조회하고, 결과인 `User` 오브젝트를 모델로 내린다.  
여기까진 문제가 없다. 그러나 이후에 수정을 하는 부분에서 문제가 발생한다.  
전달받은 `User` 오브젝트에서 실제로 사용자가 수정할 수 있는 필드는 제한적이라는 것이다.  

회원 수정 폼에서는 수정이 필요한 부분만 input 태그로 입력을 받게되고,  
이로 인해 최종적으로 `modify` 메서드를 호출 시 `@ModelAttribute`는 수정이 필요한 부분만 값이 찬, 불완전한 `User` 오브젝트를 받게 된다.  
이 상태에서 `modify` 메서드를 실행할 경우 값이 차지 않은 프로퍼티들로 인해  
실제 DB 데이터가 `null`이나 `0`으로 업데이트 되는 심각한 상황을 초래하게 된다.  

이 상황에서 생각할 수 있는 해결법은 대표적으로 3가지가 있다.  

## 해결법1 - hidden 필드
수정하면 안되는 데이터는 모두 `hidden` 필드에 넣어주는 것이다.  
이럴 경우 서버로 모든 값이 전달될 수 있기 때문에 위의 문제점은 해결된다.  
```html
<input type="hidden" name="id" value="joont" />
<input type="hidden" name="level" value="3" />
...
```
하지만 이는 더 큰 문제를 초래할 수 있다.  

첫째로 이 방식은 데이터 보안에 매우 심각한 문제를 초래한다.  
`hidden` 필드의 값은 매우 간단하게 조작될 수 있기 떄문이다.  
조작된 값은 서버에서 그대로 업데이트 되기 때문에 이는 매우 심각한 문제를 초래할 수 있다.  

둘째로 도메인 오브젝트가 변경될 때 마다 매번 `hidden` 필드도 관리해줘야 한다는 점이다.  
그리고 혹여나 실수로 `hidden` 필드를 하나라도 누락하게 된다면 또 데이터가 `null`이나 `0`으로 업데이트되는 상황이 발생할 것이다.  

이는 해결법이라고 보기에는 무리가 있어 권장되지 않는 방식이다.  

## 해결법2 - DB 조회
업데이트 전 DB에서 데이터를 한번 읽어와 도메인 오브젝트에 빈 프로퍼티가 없게 하는 방식이다.  
```java
@RequestMapping(value="/user/{uid}", method=RequestMethod.POST)
public String modify(@PathVariable Long uid,
                    @ModelAttribute User formUser){

  User user = userService.getUser(uid);
  // 수정할 오브젝트만 반영
  user.setNickName(formUser.getNickName());
  user.setEmail(formUser.getEmail());

  userService.modify(originUser);

  return "user/modifySuccess"
}
```
기능상으로만 보면 완벽하나, 매 submit 마다 DB에서 다시 데이터를 읽어와야 하는 부담이 있다.  
또한 수정할 오브젝트를 일일히 세팅해줘야 하는데, 번거롭고 실수할 가능성이 있다.  

## 해결법3 - 계층간 강한 결합
각 계층의 코드를 특정 작업을 중심으로 긴밀하게 결합시키는 방법이다.  
간단하게 말해 기본정보 수정은 `modifyDefault`, 패스워드 수정은 `modifyPassword`, 프로필 사진 수정은 `modifyProfile` 과 같이 작성하여 필요한 부분만 업데이트 치게 하는 것이다.  
이게 당장은 편리할지 모르나, 애플리케이션의 규모가 조금만 커져도 단점이 드러난다.  
각 메서드는 각각 거의 하나의 화면에서만 사용되고, 각 메서드의 재사용성이 떨어진다.  
게다가 메서드들을 각 계층마다 써줘야 한다. 이러면 코드의 중복이 발생하고 수정에도 취약하다.  
이러다간 서비스 계층과 DAO 계층을 구분하는 것도 의미가 없어질지도 모른다.  

그러므로 이 방식은 권장되지 않는 방법이다.  
객체지향성이 떨어지고 데이터 중심으로 코드가 작성되기 때문이다.  
각 계층이 서로 독립적이여야 재사용성이 높고, 확장이나 변경에도 유연하게 대응이 가능하다.  

---

# @SessionAttributes
스프링에선 위와 같이 수정 폼을 다루는 상황에서 깔끔한 해결법을 제공해주는데, 바로 세션을 이용하는 것이다.  

## 수정폼 처리
```java
@Controller
@SessionAttributes("user")
public class UserController{
  // 수정 폼
  @RequestMapping(value="/user/{uid}", method=RequestMethod.GET)
  public String modifyForm(@PathVariable Long uid,
                          ModelMap modelMap){

    User user = userService.getUser(uid);
    modelMap.addAttribute("user", user);

    return "user/form"
  }

  // 수정
  @RequestMapping(value="/user/{uid}", method=RequestMethod.POST)
  public String modify(@PathVariable Long uid,
                      @ModelAttribute User user){

    userService.modify(user);

    return "user/modifySuccess"
  }
}
```
추가된건 달랑 `@SessionAttributes` 애노테이션 하나이다.  
그러나 이 애노테이션으로 인해 앞의 모든 문제점이 해결되었다. 어떻게 동작하길래 그럴까?  

`@SessionAttributes`가 제공해주는 기능은 2가지가 있다.  

첫째로 컨트롤러의 메서드가 생성하는 모델정보 중에서 `@SessionAttributes`에 지정한 이름에 동일한 것이 있으면 이를 세션에 저장한다.  
위의 상황에서는 수정폼(`modifyForm`)이 호출될 때 `user` 오브젝트가 모델에 저장되는 동시에 세션에도 저장된다. 지정한 이름이 동일하기 때문이다.  

둘째로 파라미터에 `@ModelAttribute` 애노테이션이 있을 때 여기 전달할 오브젝트를 세션에서 가져오는 것이다.  
원래는 파라미터에 `@ModelAttribute`가 있으면 새 오브젝트를 생성한 뒤 HTTP 요청 파라미터를 바인딩한다.  
하지만 `@SessionAttributes`를 사용했을 경우는 다르다.  
새 오브젝트를 생성하기 전 `@SessionAttributes`에 지정된 이름과 `@ModelAttribute` 파라미터의 이름을 비교하여 동일할 경우 오브젝트를 새로 생성하지 않고 세션에 있는 오브젝트를 사용하게 된다.  
(참고로 비교하는 `@ModelAttribute` 파라미터의 이름은 타입의 이름이다. `User` 오브젝트일 경우 이름은 `user` 이다. 충돌이 발생하지 않도록 유의해야 한다!)  

즉, 수정폼을 거친 뒤 수정을 하게되면 비어있는 프로퍼티가 하나도 없는 상태로 수정을 진행할 수 있게 되는 것이다!  
이로인해 앞서 고민했던 문제점이 깔끔하게 해결되었다. 역시 대단하다.  
아래는 `@SessionAttributes`를 사용했을때의 흐름을 그림으로 나타낸 것이다.  
![세션을 이용한 폼 모델 저장/복구](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQ5NzAzMTQmdHlwZT1sJno9MDYvMDgvMjAxOCAxODozNA==#width80)  
`@SessionAttributes`를 지정한 클래스내의 모든 메서드의 모델에 적용되며, 하나 이상의 모델을 세션에 저장할 수 있다.  

## 등록폼 처리
`@SessionAttributes`은 수정폼외에 등록폼에서도 유용하게 사용할 수 있다.  
보통 수정폼의 경우 DB에서 조회한 결과를 폼에 보여줘야 하므로 도메인 오브젝트를 유지해야 하지만,  
등록폼의 경우 그럴 필요가 없다.  
그래서 대부분, 사용자가 폼을 submit 할 때 도메인 오브젝트를 새로 만들게 한다.  
```java
@RequestMapping(value="/user", method=RequestMethod.POST)
public String save(@ModelAttribute User user){ // User 오브젝트가 새로 생성된다

  userService.save(user);

  return "user/saveSuccess"
}
```
하지만 이럴 경우 모델을 사용하지 않는 등록폼과, 모델을 사용하는 수정폼으로 폼을 총 2개를 만들어줘야 한다.  
게다가 등록폼의 경우 모델 정보를 사용하지 않기 때문에 검증 로직에서 오류가 발생했을 경우 등록과 수정을 왔다갔다 거려야하는 꽤나.. 아니 매우 번거로운 상황이 발생한다.  
이럴바에는 처음부터 아예 모델을 사용하게 하고, 등록폼에는 빈(empty) 오브젝트라도 출력을 시켜주는 편이 낫다.  
이렇게 하면 폼을 굳이 두개로 분리할 필요도 없어진다.  
그리고 여기다가 `@SessionAttributes`를 이용하여 매번 폼이 새로 생성되지 않도록 세션에 저장해두는 것이 좋다.  
```java
@Controller
@SessionAttributes("user")
public class UserController{
  @RequestMapping(value="/user/add", method=RequestMethod.GET)
  public String saveForm(ModelMap modelMap){

    User user = new User();
    // 이런식으로 기본값이 필요한 경우 지정해줄 수 있다
    user.setJoinDt(new Date());

    modelMap.addAttribute("user", user);

    return "user/form"; // modifyForm과 동일한 폼 사용
  }
}
```

## SessionStatus
`@SessionAttributes`를 사용해 세션에 저장한 모델이 더 이상 필요없어질 경우 세션에서 제거해줘야 한다.  
세션의 제거 시점은 스프링이 알수없으므로 이를 제거하는 책임은 컨트롤러에게 있다.  
세션의 경우 서버의 메모리를 사용하므로 필요없는 시점에 제거해주지 않으면 메모리 누수가 발생할 수 있기때문에 항상 빼먹지 않고 제거해줘야 한다.  
세션의 제거는 `SessionStatus` 오브젝트의 `setComplete` 메서드로 제거할 수 있다.  
```java
@RequestMapping(value="/user/{uid}", method=RequestMethod.POST)
public String modify(@PathVariable Long uid,
                    @ModelAttribute User user,
                    SessionStatus sessionStatus){

  userService.modify(user);

  // 현재 컨트롤러 세션에 저장된 모든 정보를 제거해준다. 개별적으로 제거할 순 없다
  sessionStatus.setComplete();

  return "user/modifySuccess"
}
```

여기까지가 `@SessionAttributes`를 이용한 스프링의 전형적인 폼 처리 방식이다.  
겉으로 보여주는 것 없이 애노테이션 하나로만 처리하기 때문에 동작방식을 정확히 이해하고 있어야 한다.  

<div style="text-align: right">
From <img src="https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwNDAxMDkmdHlwZT1sJno9MjAxOC8wOC8wNiAwOTozOA==#width30" style="display:inline-block;"/>
</div>

<!-- more -->
