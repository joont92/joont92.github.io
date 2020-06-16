---
title: '[tdd] TDD로 화폐 개발하기(2)'
date: 2019-04-13 21:57:34
tags:
    - 테스트 주도 개발
---

이번 예제는 아래의 것들을 설명해주고 있다  
- 큰 단위의 테스트를 작게 접근하는 방법  
- 테스트를 기반으로 수월하게 리팩토링 하는 방법  
- 중복되는 두 클래스의 형태를 동일하게 바꾼 뒤 상위 클래스로 올리기(push up)  
- 해결법이 보이지 않을때 다시 되돌아가서 시도하는 방법  

# 큰 단위의 작업에 접근하기
이제 처음에 복잡해보여서 시도하지 못했던 `$5 + 10CHF = $10(환율이 2:1일 경우)` 을 작성해보자  
알다시피 이 요구사항의 경우 테스트 단위가 크다  
**우리는 이렇게 큰 테스트를 공략할 수 없으므로 진전을 나타낼 수 있는 작은 테스트를 만들어야 한다**  

일단 뭐 잘 모르겠고, 다시 보니 Dollar와 비슷하게 Franc이 필요해보이니 추가해보자  
```java
// FrancTest
@Test
public void testMultiplication() {
    Franc five = new Franc(5);

    assertThat(five.times(2)).isEqaulTo(new Franc(10));
    assertThat(five.times(3)).isEqaulTo(new Franc(15));
}

@Test
public void testEquality() {
    assertThat(new Franc(5)).isEqualTo(new Franc(5));
    assertThat(new Franc(5)).isNotEqualTo(new Franc(6));
}
```

알다시피 이 테스트를 빨리 초록막대에 도달하게 해야한다  
가장 빠른 방법은 Dollar를 복사히여 Franc을 만드는 것이다(뭐 테스트도 복사한 마당에..)  

복사 붙여넣기라니, 굉장히 이상해보일 수 있다  
하지만 알다시피 5단계(중복제거)에서 다 수정하면 된다  
**4단계까지는 설계보다 속도가 더 중요하고, 그것을 위해선 어떤 죄악도 저지를 수 있다**  

**물론 5단계에서 이 죄악을 수습하지 않고는 다음 단계로 나아갈 수 없다**  
여기서 적절한 설계를 하고, 돌아가게 만들고, 올바르게 만들어야 한다  

# 죄를 수습하자  
**중복을 제거하기 위한 방법은 뭐가 있을까? 두 클래스의 공통된 상위 클래스를 추출해보는건 어떨까?**  

먼저 Dollar 부터 적용시켜보자  
1. 먼저 Dollar와 Franc의 공통 클래스로 `Money`라는 애를 만들고, 이를 상속받게 한다  
    ```java
    class Money{}

    class Dollar extends Money {
        private int amount;
        // ...
    }
    ```
2. amount 변수를 Money로 올리고(protected), Dollar에서 amount 변수를 제거할 수 있다  
    ```java
    class Money{
        protected int amount;
    }

    class Dollar extends Money {
        // ...
    }
    ```
3. 이제 equals 부분을 Money로 올려볼 수 있다  
    - 우선 equals 부분을 Money에 맞춰 변경하고  
        ```java
        public boolean equals(Object object) {
            Money money = (Money) object;
            return amount == money.amount;
        }
        ```
    - Money 클래스로 올린다  
    - Dollar에서 equals는 삭제한다  

**각 과정을 진행할때마다 계속 테스트를 돌리면서 진행했고, 수월하게 리팩토링 할 수 있었다**  
이제 Franc도 똑같이 진행할 것이다  
아까 DollerTest를 복사해서 FranTest를 만들어줬기 때문에 Franc 또한 Dollar 처럼 수월하게 리팩토링이 가능하다  
> 만약 Franc에 대한 테스트를 추가하지 않았다면 라팩토링 전에 추가해주는 것이 좋다  
> **(이 외에도 있어야 할 것 같은 테스트가 있다면 작성해줘야 한다)**  
>
> 그렇게 하지 않으면 리팩토링 하다가 결국 뭔가 꺠트릴 것이고,  
> 리팩토링에 대해 안좋은 느낌을 갖게 되고,  
> 리팩토링을 덜 하게 되고,  
> 코드의 질이 떨어지게 되고,  
> 해고당한다(!!!!)  

# 또 한번 찾아온 불길한 느낌  
우리는 앞 단계에서 배웠다  
부작용에 대한 혐오감이 생기는 경우, 즉시 테스트를 추가해서 결과를 확인해봐야 한다  

```java
public void testEquality() {
    assertThat(new Dollar(5)).isNotEqualTo(new Franc(5));
}
```

테스트가 실패한다!  
그래도 불안감을 눈으로 확인해보니 마음이 한결 편해졌다  
이 부분은 객체의 클래스를 비교함으로써 간단하게 구현가능하다  

```java
public boolean equals(Object object) {
    Money money = (Money) object;
    return amount == money.amount
        && getClass() == money.getClass();
}
```

간단하게 테스트를 통과시켰다  
클래스 타입 비교로 통화를 비교한다는게 조금 그렇긴하지만 일단은 그냥 넘어간다  
> 더 많은 동기가 있기 전에는 더 많은 설계를 하지 않는것이 좋다  

# 하위클래스를 없애기 위한 시도 - 직접 참조 제거  
상위 클래스 추출이 성공했으니, 남아있는 `times()` 메서드의 리턴타입도 상위 클래스로 변경해도 괜찮겠다  
```java
class Dollar {
    Money times(int multiplier) {
        return new Dollar(amount * multiplier);
    }
}

class Franc {
    Money times(int multiplier) {
        return new Franc(amount * multiplier);
    }
}
```

이쯤되니 두 하위 클래스의 형태가 많이 비슷해졌다  
바로 제거하는 테크를 타고 싶긴하지만, 그렇게 한번에 큰 step을 밟는것은 좀 위험하고 TDD에도 맞지 않으니 단계적으로 진행하도록 한다  

**첫번째 단계는 `하위클래스에 대한 직접적인 참조들을 제거`하는 것이다**  
**`new Dollar(5)` 와 같은 강력한 직접 참조들을 먼저 제거해보자**  
직접 참조를 제거하는 방법으로 팩토리 메서드를 써보면 좋을 것 같다  

```java
// DollarTest
@Test
public void testEquality() {
    assertThat(Money.dollar(5)).isEqualTo(new Dollar(5));
    assertThat(Money.dollar(5)).isNotEqualTo(new Dollar(6));
}

public void testMultiplication() {
    Dollar dollar = Money.dollar(5);
    assertThat(Money.dollar(10)).isEqualTo(dollar.times(2));
    assertThat(Money.dollar(15)).isEqualTo(dollar.times(3));
}
```

팩토리 메서드는 아래와 같이 만든다  

```java
static Dollar dollar(int amount) {
    return new Dollar(amount);
}
```

`times()`의 직접 참조도 제거하고,  
```java
public Money times(Integer multiplier) {
    return Money.dollar(amount * multiplier);
}
```

Dollar에 대한 참조까지 제거해주자  
```java
public void testMultiplication() {
    Money money = Money.dollar(5);
    // ...
}
```

Money에 times가 없기 때문에 오류가 발생한다  
이를 위해 추상 클래스를 하나 추가해준다(더 먼저해야 했을수도 있었다)  

```java
abstract class Money {
    abstract Money times(int multiplier);
}
```

이제 완벽하게 new Dollar()를 팩토리 메서드로 대체할 수 있다  
> 하위 클래스의 존재를 테스트 메서드에서 분리(decoupling)헀으므로, 이제 테스트에 영향을 주지 않고 상속구조를 마음껏 변경할 수 있게 된다  
> 테스트 메서드뿐 아니라 모든 클라이언트 코드에서도 이런식으로 결합도를 낮추면 변화에 유연해진다  

이제 Franc에 대해서도 똑같이 작업해준다  

똑같이 변경해놓고 보니, `DollarTest`와 `FrancTest`의 테스트들의 형태가 매우 중복되어 보인다  
나는 이 시점에서 두 테스트를 합쳐서 `MoneyTest`로 만들었다  
> 이런식으로 하위클래스가 분리되다보면 몇몇 테스트가 불필요한 여분의 것이 된다  

# 하위클래스를 없애기 위한 시도 - 추상화  
`times()`의 모양을 같게 하려면 `Money.dollar`와 `Money.franc`을 같게 만들어야한다  
그러므로 둘을 추상화 할수있는 뭔가가 필요하다  
`통화`라는 개념을 도입하면 뭔가 하위 클래스 제거에 좀 더 가까워질 것 같다  

먼저 통화 개념에 대한 테스트를 작성해본다  
```java
public void testCurrency() {
    assertThat(Money.dollar(1).currency()).isEqualTo("USD");
    assertThat(Money.franc(1).currency()).isEqualTo("CHF");
}
```

통화를 인스턴스 변수에 저장하고 메서드에서 그걸 반환해주면 될 것 같다  
(좀 더 step by step으로 갈수도 있긴 하지만 바로 떠오르니깐 뭐)  

```java
// Dollar
private Strnig currency;

public Dollar(int amount) {
    this.amount = amount;
    currency = "USD";
}

public String currency() {
    return currency;
}

// Franc
private String currency;

public Franc(int amount) {
    this.amount = amount;
    currecny = "CHF";
}

public String currency() {
    return currency;
}
```

변수 선언과 currency() 메서드가 동일하므로, 이를 위로 올릴(push up) 수 있다(야호!)  
```java
// Money
protected String currency;

String currency() {
    return currency;
}
```

Dollar와 Franc의 생성자에서 currency 세팅하는 부분을 파라미터로 받게하면  
두 생성자의 형태가 똑같아질수 있을 것 같다  
```java
// Dollar
public Dollar(int amount, String currency) {
    this.amount = amount;
    this.currency = currency;
}
// Franc도 동일
```

이렇게 바꿨더니 아래의 메서드가 깨진다(ㅠㅠ)  
```java
public static Dollar dollar(Integer amount) {
    return new Dollar(amount);
}

public static Franc franc(Integer amount) {
    return new Franc(amount);
}
```

우린 하위클래스의 형태를 맞추고 push up을 진행하는 중이었는데, 구조상의 변경으로 인해 이런식의 상황이 종종 발생할떄가 있다  
이럴때는, 이걸 지금 고쳐야할까, 아니면 나중에 고쳐야할까?  
교리상은 하던일을 중단하지 않고 다 끝낸 다음에 고치는것이 맞지만, 이정도의 짧은 중단은 그냥 받아들이고 가도 괜찮다  
> 중요한것은 **하던 일을 중단하고 다른 일을 하는 상태에서 또 그 일을 중단하면 안된다는 것**이다(Jim Coplien)  

```java
// Money
public static Money dollar(int amount) {
    return new Dollar(amount, "USD");
}
// Franc도 동일
```

컴파일 에러를 해결했으니 위로 올리자.  
```java
// Money
public Money(int amount, String currency) {
    this.amount = amount;
    this.currency = currency;
}

// Dollar
public Dollar(int amount, String currency) {
    super(amount, currency);
}
// Franc
public Franc(int amount, String currency) {
    super(amount, currency);
}
```

생성자를 바로 제거하고 싶었으나 아직 Money가 추상 클래스라 불가능하다  
times를 먼저 손봐야한다  

> 이런식으로 단계적으로 밟아가는 과정이 답답할수도 있다  
> 중요한 것은 **이런식으로 일해야 한다는 것이 아니라, 이런식으로 일할수도 있어야 한다**는 것이다  
> 종종걸음 step이 답답하면 조금 보폭을 늘려도 되고, 성큼성큼 걷는것이 불안하면 조금 보폭을 줄이면 된다  
> **TDD란 조종해나가는 과정이다. 올바른 보폭이란 존재하지 않는다**  

## 다시 돌아가서 생각을..
`times()` 메서드를 바로 위로 올려보려고 했는데, 생김새가 좀 막막하다  
```java
// Dollar
public Money times(Integer multiplier) {
    return Money.dollar(amount * multiplier);
}

// Franc
public Money times(Integer multiplier) {
    return Money.franc(amount * multiplier);
}
```

잘 모르겠으니 팩토리 메서드를 다시 생성자로 돌려보자  
**(이렇게 방법이 없을땐 잠시 후퇴해서 보는것도 방법이다)**  
(+ 여기서 currency는 클래스에 있는 값을 바로 쓰도록 한다. 굳이 파라미터로 또 전달할 필요 없다)  

```java
// Dollar
public Money times(Integer multiplier) {
    return new Dollar(amount * multiplier, currency);
}

// Franc
public Money times(Integer multiplier) {
    return new Franc(amount * multiplier, currency);
}
```
이렇게 돌려놓으니 좀 보이는것 같다  
Dollar나 Franc으로 생성하느냐는 별로 중요한 것 같지가 않다. 어쩌피 currency가 있는데 굳이 뭐하러.  
저걸 Money로 바꿔보자.  

```java
// Dollar
public Money times(Integer multiplier) {
    return new Money(amount * multiplier, "CHF");
}
// Franc
```

Money를 구현 클래스로 만들어야 하므로, 추상 메서드를 제거한다  
```java
// Money
public Money times(Integer multiplier) {
    return null;
}
```

이렇게 하고 테스트를 돌렸더니, Money 클래스가 Dollar 클래스가 아니라는둥, Money 클래스가 Franc 클래스가 아니라는 둥의 결과가 출력된다  
이 말인 즉, 문제는 equals에 있었던 것이다. 비교해야 될 것은 클래스가 아니라 currency이다  

이를 위해 추가적인 테스트가 작성되어야하는데, 현재 빨간막대 상태이다  
빨간 막대 상태일때는 테스트를 추가하지 않는 것이 좋다  
좀 보수적이긴 하지만, 다시 Dollar와 Franc의 `new Money`를 `new Dollar, new Franc`으로 돌린다  

그리고 테스트를 작성한다  
```java
@Test
public void testDifference() {
    assertThat(new Money(5, "USD")).isEqualTo(new Dollar(5, "USD"));
}
```

이 테스트를 통과시키기 위해 equals를 변경한다  
```java
public boolean equals(Object obj) {
    Money money = (Money) obj;
    return amount.equals(money.amount)
        && currency().equals(money.currency());
}
```

테스트가 통과하니, 다시 times의 `new Dollar, new Franc`을 `new Money`로 바꾼다  
이 테스트 또한 잘 통과하고, 이제 times의 형태가 같아졌으니 push up 할 수 있다!!  

**뒤로 돌아가지 않았다면 이처럼 팩토리 메서드 사용을 제거할 수 있다는 사실을 알기 힘들었을 것이다**  

# 하위클래스를 없애기 위한 시도 - 나머지 부분 제거  
이제 두 클래스에 남은건 생성자밖에 없다  
단지 생성자 때문에 하위클래스를 남겨놓을수는 없으니, 제거하는 것이 좋다  
유일하게 남아있는 하위 클래스 직접 참조인 `정적 팩토리 메서드`를 수정하자  

```java
public static Money dollar(Integer amount) {
    return new Money(amount, "USD");
}

public static Money franc(Integer amount) {
    return new Money(amount, "CHF");
}
```

이제 완벽하게 제거하..려고 하는데, 생각해보니 아직 직접 참조가 한군데 더 남아있다  
```java
@Test
public void testDifference() {
    assertThat(new Money(5, "USD")).isEqualTo(new Dollar(5, "USD"));
}
```

보니까 이 테스트는 이미 다른 테스트에서 수행하고 있는 작업들이다. 제거하자.  
이로 인해 최종적으로 하위 클래스를 전부 제거할 수 있게된다!!  

<!-- more -->