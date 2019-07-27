---
title: '[tdd] TDD 도구들'
date: 2019-03-23 17:58:17
tags:
    - 테스트 주도 개발로 배우는 객체지향 설계와 실천
    - JUnit
    - assertThat
    - hamcrest
    - assertj
    - jmock
    - mockito
---

# JUnit
가장 많이 쓰이는 java 테스트 프레임워크이다.  
그 중에서도 JUnit4 버전이 가장 많이 사용된다.(현재는 JUnit5 버전까지 나와있다)  
> JUnit은 기본적으로 리플렉션을 통해 클래스 구조를 파악한 후, 해당 클래스에서 테스트를 나타내는 것을 모두 실행한다.  

## 테스트 케이스  
JUnit은 `@Test`라는 어노테이션이 지정된 메서드는 모두 테스트 케이스로 취급한다.  
테스트 메서드는 값을 반환하거나 매개변수를 받아서는 안 된다.  

```java
public class SomeTest {
    private Some some = new Some();

    @Test
    public void testSomething1() {
        // do something
    }

    @Test
    public void testSomething2() {
        // do something
    }
}
```

하나 특별한점은, JUnit은 각 테스트를 실행할 때 마다 해당 테스트 클래스의 새 인스턴스를 생성하여 호출한다는 점이다.  
> 위의 테스트들을 전부 실행하면 2개의 `SomeTest` 인스턴스가 생성되고, 각자 `@Test` 메서드들을 실행하게 된다.  

이런식으로 매번 새 인스턴스를 생성하면 각 테스트간의 격리성을 확보할 수 있다. 테스트 객체의 필드가 각 테스트에 앞서 대체되기 때문이다.  
이는 테스트에서 테스트 객체 필드의 내용을 맘껏 바꿀 수 있다는 의미이다.  
> `testSomething1`에서 `some`의 값을 변경해도 `testSomething2`에는 영향을 주지 않는다  

## assertion  
기본적으로 테스트들은, 각 테스트를 수행하고 그 결과를 assertion(단정)하는 식으로 작성된다.  
여기서 JUnit에서 제공하는 assertion 메서드들을 사용할 수 있다.  

```java
public class StoreTest {
    private Store store = new Store();

    @Test
    public class 결제수단을_체크한다() {
        assertTrue(store.canPay(PaymentMethod.Money));
        assertFalse(store.canPay(PaymeneMethod.CreditCard));
    }

    @Test
    public class 특정_상품을_찾는다() {
        assertNotNull(store.find("치킨"));
        assertNull(store.find("가죽자켓"));
    }
}
```

보다시피 canPay의 결과가 true/false가 나올것이다 라고 단정(assertion) 했고,  
find의 결과가 null이 아니라고 단정(assertion) 했다.  

## 예외 예상하기  
`@Test` 어노테이션은 선택적 매개변수로 `expected` 라는 것을 지원한다.  
이 매개변수는 테스트 케이스에서 던져질 예외를 선언한다. 선언한 예외가 발생하면 테스트가 성공한다.  

```java
@Test(expected = IllegalArgumentException.class)
public void 검색어를_입력하지_않는다() {
    store.find(null);
}
```

find 메서드는 검색어로 null을 받으면 IllegalArgument 예외를 리턴하게 작성되어 있다.  
그러므로 위의 메서드는 성공하게 된다.  

## 테스트 픽스쳐  
테스트가 시작할 때 `존재하는 고정된 상태`를 의미한다.  
테스트를 수행하기 전에 필요한 특정 상태들을 의미한다. 특정 협력객체나, 특정 데이터들이 있을 수 있다.  

JUnit은 각 테스트마다 인트턴스를 새로 생성하므로   
간단히 인스턴스 변수 선언과 동시에 초기화하거나 생성자 같은것을 써서 픽스쳐를 초기화하면 되긴하지만,  
JUnit에서 제공하는 특정 어노테이션들을 사용하면 좀 더 명시적으로 픽스쳐 초기화가 가능하다.  

```java
private Store store = new Store();

@Before
public void setUp() {
    store.addProduct("chicken");
    store.addProduct("pizza");
    store.addProduct("beer");
}

@After
public void tearDown() {
    // 별로 할게 없음..
}
```

`@Before` 메서드가 모든 테스트 실행전에 실행되므로, 모든 테스트는 3가지 상품이 들어간 상태에서(동일한 상태에서) 테스트를 수행할 수 있게 된다.  
이런식으로 모든 테스트 메서드에서 필요한 상태를 초기화하는데 사용하면 유용하다.  
`@After` 메서드는 테스트 메서드가 끝난 후 수행되는데, 사실상 여기서 수행할 작업이 많지는 않다.  
생성된 픽스쳐를 정리하는 작업 같은것도 전부 JVM 가비지 컬렉터에서 잘 수행해주기 때문이다.  

## 테스트 러너
JUnit이 클래스를 대상으로 리플렉션을 수행해 테스트를 찾아 해당 테스트를 실행하는 방식은 `테스트 러너(test runner)` 에서 제어한다.  
테스트 클래스에서 사용하는 러너는 `@RunWith` 어노테이션으로 설정할 수 있다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
public class SomeTest {

}
```

> JUnit4의 현재 default runner는 `BlockJUnit4ClassRunner` 라고 한다.  

# 햄크레스트 매처와 assertThat
햄크레스트는 매칭 조건을 선언적으로 작성하는 프레임워크이다.  
`matches`라는 boolean을 반환하는 메서드를 가진 `Matcher` 인터페이스를 구현하는 많은 클래스들을 제공한다.  
이 클래스들을 사용하면 기존의 단순한 assertion 구문들을 좀 더 다양하게 사용 가능하다.  

```java
Matcher<String> containsBananas = new StringContains("bananas");

assertTrue(containsBananas.matches(str));
```

햄크레스트는 코드의 가독성을 높이고자 Matcher를 생성하는 부분을 static factory 메서드로 제공한다.  

```java
import static org.hamcrest.CoreMatchers.*;

@Test
public void banana_match() {
    assertTrue(containsString("bananas").matches(str));
}
```

하지만 실제로는 위와 같은 방법보단, self-describing 특성을 가진 `assertThat`을 주로 사용한다.  

```java
assertThat(str, containsString("bananas"));
assertThat(str, not(containsString("bananas")));
```

`assertThat`은 Matcher를 직접 인자로 받을 수 있다.  
보다시피 `str은 "bananas" 문자열을 포함하고 있다` 의 형태로 작성됨으로써 테스트 코드가 더 잘 읽힌다.  
(참고로 햄크레스트는 위에서 사용된 `not` Matcher 처럼 다른 매처를 조합할 수 있는 유용한 기능을 제공한다)  

이러한 Matcher는 몇가지 조건만 만족하면 사용자가 쉽게 직접 정의해서 사용할 수 있다.  

# AssertJ
assertThat과 햄크레스트를 적절히 조합하여 테스트를 작성하는 것만해도 충분하지만, 좀 더 풍부한 구문을 제공하는 `AssertJ` 라는 라이브러리도 있다.  

```java
import static org.assertj.core.api.Assertions.assertThat;

public void 오브젝트() {
    assertThat(object).isNotNull();

    assertThat(object).isSameAs(otherObject);
    assertThat(object).isEqualTo(otherObject);
}

public void 컬렉션() {
    assertThat(list).isSorted();
    assertThat(list).hasSize(4);
}
```

보다시피 좀 더 직관적이고, 풍부한 메서드들을 제공한다.  
게다가 assertion과 햄크레스트를 직접 static import 하지 않아도 되는 편리함도 있으니, 사용해보는 것도 나쁘지 않다.  

# JMock2
JMock을 사용하면 mock 객체를 JUnit 같은 테스트 프레임워크에 붙여서 사용할 수 있도록 해준다.  
JMock의 핵심 개념은 모조 객체와 목 객체, 예상 구문이다.  
아래는 JMock을 이용한 행위 검증의 예제이다.  

```java
@RunWith(JMock.class)
public class AuctionMessageTranslatorTest {
    private final Mockery context = new JUnit4Mockery(); // 1
    private final AuctionEventListener listener = context.mock(AuctionEventListener.class);
    private final AuctionMessageTranslator translator = new AuctionMessageTranslator(listener);
    
    @Test public void notifiesAuctionCloseWhenCloseMessageReceived() {
        Message message = new Message();
        message.setBody("SOLVersion : 1.1; Event: Close;");
        
        context.checking(new Expectations() {{
            oneOf(listener).auctionClosed();
        }});
        
        translator.processMessage(UNUSED_CHAT, message);
    }
}
```

여기서 `모조 객체`라는 개념이 나오는데, 현재 이해하기로는 `목 객체들을 담는 그릇` 정도로 이해된다.. JMock runner가 읽는 대상들?  

1. JUnit이 JMock 테스트 러너를 사용하게 된다. 이 러너는 테스트가 끝나는 시점에 모든 모조 객체를 자동으로 호출해 모든 목 객체가 예상대로 호출되었는지 검사한다.  
2. 모조 객체를 생성한다. JMock 러너가 검사할 목 객체들이 담긴다.  
3. AuctionEventListener의 mock 객체를 생성한다. 자동으로 context에 담긴다.  
4. SUT에 mock 객체를 주입한다. SUT는 그 사실을 모르고, 알 필요도 없다.  
5. 모조객체에서 검사할 내용(행위 검증)을 작성한다. listener 목객체가 auctionClosed 메서드를 정확히 1번 호출할 것을 예상하고 있다.  
6. SUT가 행위를 수행한다.  
7. 테스트가 끝나면 JMock runner는 모조객체(현재는 context)에 명시된 행위들을 검증한다  

개인적 느낌인데, 작성하기가 조금 어려운 부분이 있는 것 같다.  

# mockito
JMock보다 좀 더 강력한 기능을 제공하는 mockito 라는 라이브러리가 있다.  
아래는 사용법에 대해 작성된 글이다.  

<https://github.com/mockito/mockito/wiki/Mockito-features-in-Korean>  

간단히 내가 느끼는 mockito의 장점은 아래 정도이다.  
- mockito는 JMock 처럼 모조객체, 목 객체에 대한 구분이 없다.  
- JMock에도 있는지 모르겠으나(당연히 있곘지..) stub을 매우 간단하게 지원한다.  
- 행위 검증도 매우 간단하게 지원한다.  

나는 개인적으로 이 라이브러리가 좀 더 작성하기 편리하고, 명시적인 것 같다.  

참고로 mockito 보다 더 강력한 기능을 제공하는 PowerMock 이라는 애도 있다.(private 메서드 테스트, static 메서드 주입 등 까지 제공한다)  

<!-- more -->