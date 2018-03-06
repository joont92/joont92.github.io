---
title: '@SessionAttribute, SessionStatus'
date: 2018-03-07 00:12:49
tags:
---

### 문제상황
아래는 기본적인 회원정보 수정 샘플 코드이다.  
```java
// 수정 폼
@RequestMapping(value="/user/{uid}", method=RequestMethod.GET)
public String modify(@PathVariable Long uid, ModelMap modelMap){
  User user = userService.getUser(uid);
  modelMap("user", user);

  return "user/modify"
}

// 수정
@RequestMapping(value="/user/{uid}", method=RequestMethod.GET)
public String modify(@PathVariable Long uid, @ModelAttribute User user){

  userService.modify(user);

  return "user/modifySuccess"
}
```
근데 위 코드는 기본적으로 문제가 있다.  
수정 폼을 출력하는 메서드에서 uid로 회원을 조회하고, 결과인 User 오브젝트를 모델로 내린다.  
여기까진 문제가 없다. 그러나 이후에 수정을 하는 부분에서 문제가 발생한다.  
전달받은 User 오브젝트에서 실제로 사용자가 수정할 수 있는 필드는 제한적이라는 것이다.  
그러면 회원 수정 폼에서는 수정이 필요한 부분만 input 태그에 넣어서 modify 메서드를 호출하게 되고,  
결과적으로 modify 메서드는 수정이 필요한 부분만 값이 찬, 불완전한 user 오브젝트를 받게 된다.  
이 상태에서 modify 메서드를 실행할 경우 값이 차지 않은 프로퍼티들로 인해  
실제 DB 데이터가 null이나 0으로 업데이트 되는 심각한 상황을 초래하게 된다.  

이 상황에서 생각할 수 있는 해결법은 대표적으로 3가지가 있다.  

#### 해결법1. hidden 필드
수정하면 안되는 데이터는 모두 hidden 필드에 넣어주는 것이다.  
이럴 경우 서버로 모든 값이 전달될 수 있기 때문에 위의 문제점은 해결된다.  
```html
<input type="hidden" name="id" value="joont" />
<input type="hidden" name="level" value="3" />
...
```
하지만 이는 더 큰 문제를 초래할 수 있다.  

첫째로 이 방식은 데이터 보안에 매우 심각한 문제를 초래한다.  
hidden 필드의 값은 매우 간단하게 조작될 수 있기 떄문이다.  
조작된 값은 서버에서 그대로 업데이트 되기 때문에 이는 매우 심각한 문제를 초래할 수 있다.  

둘째로 도메인 오브젝트가 변경될 때 마다 매번 hidden 필드도 관리해줘야 한다는 점이다.  
그리고 혹여나 실수로 hidden 필드를 하나라도 누락하게 된다면 또 데이터가 null이나 0으로 업데이트되는 상황이 발생할 것이다.  

이는 해결법이라고 보기에는 무리가 있어 권장되지 않는 방식이다.  

#### 해결법2. DB 조회
업데이트 전 DB에서 데이터를 한번 읽어와 도메인 오브젝트에 빈 프로퍼티가 없게 하는 방식이다.  
```java
@RequestMapping(value="/user/{uid}", method=RequestMethod.GET)
public String modify(@PathVariable Long uid, @ModelAttribute User formUser){
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

#### 해결법3. 계층간 강한 결합
각 계층의 코드를 특정 작업을 중심으로 긴밀하게 결합시키는 방법이다.  
간단하게 말해 기본정보 수정은 modifyDefault, 패스워드 수정은 modifyPassword, 프로필 사진 수정은 modifyProfile 과 같이 작성하여 필요한 부분만 업데이트 치게 하는 것이다.  
이게 당장은 편리할지 모르나, 애플리케이션의 규모가 조금만 커져도 단점이 드러난다.  
각 메서드는 각각 거의 하나의 화면에서만 사용되고, 각 메서드의 재사용성이 떨어진다.  
게다가 메서드들을 각 계층마다 써줘야 한다. 이러면 코드의 중복이 발생하고 수정에도 취약하다.  
이러다간 서비스 계층과 DAO 계층을 구분하는 것도 의미가 없어질지도 모른다.  

그러므로 이 방식은 권장되지 않는 방법이다.  
객체지향성이 떨어지고 데이터 중심으로 코드가 작성되기 때문이다.  
각 계층이 서로 독립적이여야 재사용성이 높고, 확장이나 변경에도 유연하게 대응이 가능하다.  

### @SessionAttribute
스프링에서 위의 상황을 해결하기 위해 제공해주는 방법
~~ 설명
위에서 언급하지 않았지만 위저드 폼 같은 경우에도 대응 가능함

<!-- more -->
