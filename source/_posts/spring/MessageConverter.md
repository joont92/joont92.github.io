---
title: MessageConverter
date: 2018-05-17 23:44:44
tags:
    - MessageConverter
    - '@RequestBody'
    - '@ResponseBody'
    - 토비의 스프링
---

앞서 우리가 HTTP 요청을 모델에 바인딩하고 클라이언트에 보낼 HTTP 응답을 만들기 위해 뷰를 사용했던 방식과는 달리,  
HTTP 요청 본문과 HTTP 응답 본문을 통째로 메세지로 다루는 방식이다.  
주로 `XML`이나 `JSON`을 이용한 AJAX 기능이나 웹 서비스를 개발할 때 사용된다.  

아래와 같이 스프링의 `@RequestBody`와 `@ResponseBody`를 통해 구현할 수 있다.
```java
@ResponseBody // 응답
@RequestMapping(value= "/hello", method=RequestMethod.POST)
public String hello(@RequestBody String param){ // 요청
    return "result";
}
```
위와 같은 애노테이션을 명시해두게 되면 스프링은 `메세지 컨버터`라는 것을 사용하여 HTTP 요청이나 응답을 메세지로 변환하게 된다.  
즉 위처럼 파라미터 부분에 `@RequestBody`를 입력할 경우, 파라미터 타입에 맞는 메세지 컨버터를 선택한 뒤 HTTP 요청 본문을 통째로 메세지로 변환하여 파라미터에 바인딩하는 것이다.  
메서드의 상단에 `@ResponseBody`를 입력할 경우 또한 마찬가지로 리턴 타입에 맞는 메세지 컨버터를 선택한 뒤 리턴 값을 통째로 메세지로 변환한 뒤 리턴해주는 것이다.  
> 참고로 GET 방식의 요청일 경우 HTTP 요청 본문이 없으므로 `@RequestBody`를 사용할 수 없다. `@RequestParam`이나 `@ModelAttribute`를 사용해야 한다.  

# 메세지 컨버터의 종류
이렇게 사용되는 메세지 컨버터는 `AnnotationMethodHandlerAdapter`를 통해 등록할 수 있고, 이미 디폴트로 4가지의 메세지 컨버터가 등록되어 있다.  

아래는 디폴트 메세지 컨버터들이다.  

## ByteArrayHttpMessageConverter
지원하는 오브젝트 타입은 `byte[]`이고, 미디어타입은 모든 것을 다 지원한다.  
즉 파라미터에 `@RequestBody byte[] param`과 같이 작성하면 모든 요청을 다 `byte`배열로 받을 수 있다는 말이다.  
그리고 리턴타입을 `byte[]`로 했을 경우 `Content-Type`이 `applcation/octet-stream`으로 설정되어 전달된다.  
바이너리 정보를 주고받을 경우가 아니라면 그닥 유용해 보이진 않는다.  

## StringHttpMessageConverter
지원하는 오브젝트 타입은 `String`이고, 미디어타입은 모든 것을 다 지원한다.  
파라미터에 사용할 경우 HTTP 본문을 그대로 `String`으로 가져올 수 있게되고,  
리턴에 사용할 경우 단순 문자열을 그대로 전달해줄 수 있다.  
`Content-Type`은 `text/plain`으로 전달된다.  

## FormHttpMessageConverter
지원하는 오브젝트 타입은 `MultiValueMap<String, String>`이고, 미디어타입은 `application/x-www-form-urlencoded`만 지원한다.  
즉 정의된 폼 데이터를 주고받을 때 사용할 수 있다는 말인데,  
폼 데이터는 `@ModelAttribute`를 사용하는 것이 훨씬 유용하므로 이것 또한 자주 사용할 일은 없다.  

## SourceHttpMessageConvreter
지원하는 오브젝트 타입은 `DomSource`, `SAXSource`, `StreamSource`이고, 미디어타입은 `application/xml`, `application/*+xml`, `text/xml` 세가지를 지원한다.  
XML 문서를 Source 타입의 오브젝트로 변환하고 싶을 떄 사용할 수 있다.  
하지만 요즘은 OXM 기술이 많이 발달되었으므로 이 또한 잘 쓰이지 않는다.  

<br/><br/>

이제 아래는 디폴트가 아닌 메세지 컨버터들이다. 실제로 이 컨버터들이 더 유용하다.  
여기서 필요한게 있다면 직접 `AnnotationMethodHanlderAdapter`의 `messageConverters`에 등록하고 사용해야 한다.  

```xml
<bean class="org.springframework...AnnotationMethodHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.MappintJacksonHttpMessageConverter" />
            <bean class="org.springframework.http.converter.xml.Jaxb2RootElementHttpMessageConverter" />
        </list>
    </property>
</bean>
```

다른 전략과 마찬가지로 위와 같이 등록 시 디폴트 전략이 모두 무시된다는 점에 주의해야 한다.  

## Jaxb2RootElementHttpMessageConverter
JAXB의 `@XmlRootElement`와 `@XmlType`이 붙은 클래스를 이용해 XML과 오브젝트 사이의 메세지 변환을 지원한다.  
지원하는 미디어 타입은 `SourceHttpMessageConvreter`와 동일하다.  

## MashallingHttpMessageConverter
스프링 OXM 추상화의 `Mashaller`와 `Unmarshaller`를 이용해서 XML과 오브젝트 사이의 변환을 지원한다.  
이 컨버터를 등록할 때는 `marshaller`와 `unmarshaller`를 설정해줘야 한다.  
지원하는 미디어 타입은 `SourceHttpMessageConvreter`와 동일하다.  

## MappingJacksonHttpMessageConverter
`Jackson`의 `ObjectMapper`를 이용해서 JSON과 오브젝트 사이의 변환을 지원한다.  
지원하는 미디어 타입은 `application/json`이다.  
변환하는 오브젝트 타입의 제한은 없지만 프로퍼티를 가진 자바빈 스타일이나 HashMap을 이용해야 정확한 변환 결과를 얻을 수 있다.  

<div style="text-align: right">
From <img src="https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0NzQwNDAxMDkmdHlwZT1sJno9MjAxOC8wOC8wNiAwOTozOA==#width30" style="display:inline-block;"/>
</div>

<!-- more -->