---
title: TDD로 화폐 개발하기(3)
date: 2019-04-20 20:08:19
tags:
    - 테스트 주도 개발
---

# 다시 한번 작은 테스트로 시작
이제 더하기를 구현해야하는데, 아직까진 `$5 + 10CHF = $10`에 대한 테스트를 작성하기가 어렵다  
그래서 좀 더 작은 단위(`$5 + $5 = $10`)로 줄여서 시작해본다  

```java
public void testSimpleAddiction() {
    Money sum = Money.dollar(5).plus(Money.dollar(5));
    assertThat(sum).isEqualTo(Money.dollar(10));
}
```

어떻게 구현해야할지 명확하므로 바로 작성해본다  
```java
// Money
Money plus(Money addend) {
    return new Money(amount + addened.amount, currency);
}
```
> TDD를 하면서 이런식의 단계조절을 계속해서 배워야한다  
> 지금처럼 구현이 명백히 떠오를때는 조금 성큼성큼 나가도 되고, 사려깊게 고민해야할때는 천천히 나가는 것이 좋다  

**우리는 다중 통화 사용에 대한 내용을 시스템의 나머지 코드에게 숨겨야하는데(설계상 가장 어려운 제약), 현재의 Money 객체로는 그 행위가 불가능하다**  
이처럼 사용하는 객체가 우리가 원하는 방식으로 동작하지 않을 경우엔, 그 객체와 외부 프로토콜이 같으면서 내부 구현은 다른 새로운 객체(imposter)를 만들 수 있다  
> TDD는 적절한때에 번뜩이는 통찰을 보장하지는 못한다(우리가 다 생각해야함)  
> 그렇지만 확신을 주는 테스트와 조심스럽게 정리된 코드를 통해, 통찰에 대한 준비와 함께 통찰이 번뜩일때 그걸 적용할 준비를 할 수 있다  

우리는 Money와 비슷하게 동작하지만 사실은 두 Money의 합을 나타내는 imposter를 만들것이다  
imposter가 될 수 있는 후보로 생각해본것들은 아래와 같다  
1. `지갑` 같은 객체. 여러 화폐들이 들어갈 수 있다.
2. `(2 + 3) x 5` 같은 `수식` 객체.  
    2$와 같은 `Money`가 수식의 가장 작은 단위가 되고, 수식들을 연산한 결과도 수식이 나온다  
    최종적으로 수식에 환율을 적용하여 단일통화를 얻게된다  
    (어떻게 이런 생각을 도출해냈는지 잘 떠오르지를 않는다..)  

2번을 택하기로 하고, 테스트를 작성해본다  
```java
public void testSimpleAddiction() {
    Money reduced = // ...
    assertThat(reduced).isEqualTo(Money.dollar(10));
}
```

reduced는 수식에 환율을 적용하여 나온 단일통화(Money)가 된다  
reduced를 얻는 과정을 좀 더 작성하면 아래와 같다  
```java
public void testSimpleAddiction() {
    Money five = Money.dollar(5);

    Expression sum = five.plus(five);
    Money reduced = bank.reduce(sum, "USD");

    assertThat(reduced).isEqualTo(Money.dollar(10));
}
```

덧셈의 과정으로 `수식(Expression)`이 나오게되고, 여기에 환율을 적용하여 단일통화를 얻게끔 했다  
> 사실상 현재과정에서 `은행` 없이 `수식`에서 `reduce`를 구현할수도 있지만 그렇게 하지 않은 이유는,  
> - Expression이 우리가 하려는 일의 핵심이기 때문에, 다른 부분(환율 적용)에 대해서는 최대한 모르게 하기 위함이다  
>   - 그렇게 해야 핵심 객체가 가능한 오래 유지되고, 테스트하기 쉽고, 재활용하기 쉬운 상태로 남을 수 있게된다  
> - 환율적용 외에도 Expression과 관련있는 오퍼레이션이 많을 수 있기 때문이다  
>   - 그때마다 모든 오퍼레이션을 Expression에만 추가하면다면 Expression은 무한히 커질 것이다  

이제 컴파일 에러를 잡아야한다  
먼저 `plus` 메서드가 Expression을 반환해야 한다  
```java
// Money
Expression plus(Money addend) {
    return new Money(amount + addened.amount, currency);
}
```

클래스로 만들 수 있지만 더 가벼운 인터페이스를 선택한다  
```java
interface Expression {
}

class Money implements Expression {
    // ...
}
```

`Bank` stub을 가볍게 작성해서 테스트를 통과시킨다  
```java
class Bank {
    Money reduce(Expression source, String to) {
        return Money.dollar(10);
    }
}
```

메타포를 선택하고 빠르게 테스트 작성하고, 그를 통과시키는 과정?  

# 순방향 진행
이제 `bank.reduce()`에 작성한 가짜 구현을 제거해줘야하는데, 이번 경우는 어떻게 (거꾸로)작업해야 할지가 명확하지가 않다  
그래서 이번에는 순방향(?)으로 작업해보기로 한다  

먼저 현재 `bank.reduce()` 메서드는 인자로 넘기는 source와 반환하는 Money의 값이 중복이다.  
source에 넘겨주는 값과 리턴하는 Money의 값이 사실상 동일한 값이기 때문이다(삼각측량을 이용해서 가짜구현을 제거하더라도 동일하다)  
이 시점에서 우리가 Expression을 만들때 생각했던, 구현체인 Sum을 등장시켜보자  
`Money.plus()`가 Money가 아닌 Expression(Sum)을 반환하도록 변경해주도록 하자  
테스트 먼저 작성해본다  
```java
@Test
public void testPlusReturnsSum() {
    Money five = Money.dollar(5);
    Expression result = five.plus(five);
    Sum sum = (Sum) result;

    assertThat(sum.augend).isEqualTo(five);
    assertThat(sum.addend).isEqualTo(five);
}
```
> 이 테스트는 너무 구현 종속적이라 오래가지 못할것이다)  

이제 정확한 expected/actual 형태가 나오도록 수정해야한다  
Money.plus에서 Sum을 반환하도록 수정하고, `Sum` 클래스를 만들어야한다  
```java
// Money
Expression plus(Money addend) {
    return new Sum(this, addend);
}

public class Sum implements Expression {
    public Money augend;
    public Money addend;

    public Sum(Money augend, Money addend) {
        this.augend = augend;
        this.addend = addend;
    }
}
```
> 좀 빠른감이 있지만 구현이 명백하게 떠오르니 바로바로 진행한다  

Sum을 작성하고 나니 추가적인 테스트가 바로 떠오른다  
Sum에 전달한 Money 통화가 모두 동일하고, reduce를 통해 얻어내고자 하는 통화 역시 같다면 결과는 Sum 내의 amount를 합친 값을 갖는 Money 객체여야한다  
```java
public void testReduceSum() {
    Expression sum = new Sum(Money.dollar(3), Money.dollar(4));
    Bank bank = new Bank();
    Money result = bank.reduce(sum, "USD");
    assertThat(result).isEqualTo(Money.dollar(7));
}
```

테스트를 통과시킨다  
```java
public Money reduce(Expression source, String to) {
    Sum sum = (Sum) source;
    int amount = sum.augend.amount + sum.addend.amount;
    return new Money(amount, to);
}
```

이 코드는 현재 2가지 이유로 지저분하다  
1. 형변환. `reduce()`는 모든 Expression에 대해 동작해야 한다  
2. `Sum`의 `public` 필드와 `sum.augend.amount` 같이 2단계에 걸친 레퍼런스  

2번 문제는 간단히 고칠 수 있다. 메서드 일부를 Sum 클래스 내부로 옮겨버리면 된다  
```java
// Sum
public Money reduce(String to) {
    int amount = augend.amount + addend.amount;
    return new Money(amount, to);
}

// Bank
public Money reduce(Expression source, String to) {
    Sum sum = (Sum) source;
    return sum.reduce(to);
}
```

덧셈은 됐으니 환율 적용에 대해 생각해보자  
그냥 `Money`가 인자로 왔을 경우 환율을 적용시킨 Money를 내보내야 한다  
근데 우린 지금 `Money` 부터 받을수가 없어서, 이를 먼저 통과시켜야 한다  
테스트를 바로 작성해보자  
```java
@Test
public void testReduceMoney() {
    Bank bank = new Bank();
    Money result = bank.reduce(Money.dollar(1), "USD");
    assertThat(result).isEqualTo(Money.dollar(1));
}
```

```java
// Bank
public Money reduce(Expression source, String to) {
    if(source instanceof Money) {
        return (Money) source;
    }
    Sum sum = (Sum) source;
    return sum.reduce(to);
}
```

코드가 너무 지저분해졌다  
**다른 환율에 대한 테스트를 작성하기 전에, 지저분한 코드들부터 정리하고 가는것이 좋겠다**  
이런식으로 클래스를 명시적으로 검사하는 코드가 있을떄는 항상 `다형성`을 적용해주는 것이 좋다  

Money에도 `reduce()`를 구현해준다  
```java
@Override
public Money reduce(String to) {
    return this;
}
```

이제 `Expression`을 구현하는 `Money`, `Sum`에 `reduce()` 메서드가 있으니 인터페이스에도 선언할 수 있다  
```java
public interface Expression {
    Money reduce(String to);
}
```

이로써 불필요한 캐스팅 코드를 모두 제거할 수 있다  
```java
public Money reduce(Expression source, String to) {
    return source.reduce(to);
}
```

# 환율 적용!
이제 다른 통화간 환율을 적용하는 테스트를 작성해본다  
```java
@Test
public void testReduceMoneyDifferentCurrency() {
    Bank bank = new Bank();
    bank.addRate("CHF", "USD", 2);
    Money result = bank.reduce(Money.franc(2), "USD");
    assertThat(result).isEqualTo(Money.dollar(1));
}
```

Money에서 직접 환율을 관장할수도 있지만, 별로 좋은 방식이 아니다  
환율에 관한건 `Bank`가 처리하게 해야한다  
`reduce()` 하기전에 `Bank`에 환율 관련된 부분을 물어보게끔 처리하면 될 것 같다  
`Bank`를 인자로 전달하게끔 파라미터를 변경하자  
```java
public interface Expression {
    Money reduce(Bank bank, String to);
}

// Money, Sum 적용
```

환율을 물어볼 메서드를 작성한다  
```java
// Bank
public int rate(String from, String to) {
    if(from.equals("CHF") && to.equals("USD")) {
        return 2;
    }
    return 1;
}
```

Money에서 `rate()`에 환율을 물어본다  
```java
@Override
public Money reduce(Bank bank, String to) {
    int rate = bank.rate(currency, to);
    return new Money(amount / rate, to);
}
```

보다시피 아직 좋은 방법이 아니다. 게다가 `addRate()`로 환율 추가하는 메서드까지 만들어놓고 전혀 활용하지 않고 있다.  
`addRate()`로 해시테이블 같은 곳에 환율을 추가하고(환율표), 필요할 때 매번 찾아보게 하면 될 것 같다  

해시테이블에서 바로 찾기 위해 환율의 from과 to를 위한 객체를 따로 만든다  
그리고 이 `Pair` 클래스는 `키`로 사용될 것이므로 `equals`와 `hashCode`를 구현해준다  
(**현재는 리팩토링 과정중이므로 따로 테스트를 작성하지 않는다. 리팩토링이 끝난 후 모든 테스트가 잘 통과한다면 리팩토링이 잘 되었다고 판단할 수 있기 때문이다.**)  
```java
class Pair {
    String from;
    String to;

    public Pair(String from, String to) {
        this.from = from;
        this.to = to;
    }

    @Override
    public boolean equals(Object obj) {
        Pair pair = (Pair) obj;
        return from.equals(pair.from) && to.equals(pair.to);
    }

    @Override
    public int hashCode() {
        return 0;
    }
}
```
> 0은 최악의 해시코드지만, 지금은 빠르게 달려야하니까 그냥 저렇게 작성한다  
> 나중에 많은 통화를 다루게 될 경우 추가적으로 수정한다  

이제 이 환율표를 사용하도록 Bank를 수정한다  
```java
// Bank
Map<Pair, Integer> rateTable = new Hashtable<>();

public void addRate(String from, String to, int rate) {
    rateTable.put(new Pair(from, to), rate);
}

public int rate(String from, String to) {
    return rateTable.get(new Pair(from, to));
}
```

잘 동작할 줄 알았는데 테스트가 실패한다!  
살펴보니 같은 통화일떄가 문제였다. 이렇게 뜻밖지 못하게 발견한 일의 경우 테스트를 추가해서 다른 사람들이 알게끔 해줘야 한다  
```java
@Test
public void testIdentityRate() {
    Bank bank = new Bank();
    assertThat(bank.rate("USD", "USD")).isEqualTo(1);
}
```
> 이렇게 리팩토링하다가 실수한 경우 이 문제를 분리하기 위해 또 다른 테스트를 작성하고, 전진해나간다  

이제 `rate()` 를 수정하자  
```java
public int rate(String from, String to) {
    if(from.equals(to)) {
        return 1;
    }

    return rateTable.get(new Pair(from, to));
}
```

# 다른 통화간 더하기  
드디어 `5$ + 10CHF = 10$` 를 테스트 해볼 떄가 왔다  
아래가 우리가 최종적으로 원하는 테스트의 모습이다  
```java
@Test
public void testMixedAddition() {
    Expression dollar = Money.dollar(5);
    Expression franc = Money.franc(10);

    Bank bank = new Bank();
    bank.addRate("CHF", "USD", 2);

    Money result = bank.reduce(dollar.plus(franc), "USD");
    assertThat(result).isEqualTo(Money.dollar(10));
}
```

하지만 안타깝게도 컴파일 에러가 난다  
좀 더 천천히 진행해보기로 하고(모든 에러를 컴파일러가 잡아줄것이라는 기대?), 한 단계만 뒤로 물러나보자  

먼저 `testMixedAddition()` 상단의 `Expression`을 `Money`로 바꿔서 컴파일 에러를 제거하고, 테스트를 돌려보자  
```java
@Test
public void testMixedAddition() {
    Money dollar = Money.dollar(5);
    Money franc = Money.franc(10);
    // ...
}
```

테스트가 실패한다. 10$ 대신 15$가 나오는 것이 축약을 하지 않는 것 처럼 보인다.  
```java
@Override
public Money reduce(Bank bank, String to) {
    int amount = augend.reduce(bank, to).amount
        + addend.reduce(bank, to).amount;
    return new Money(amount, to);
}
```

테스트가 통과했으니, 처음 컴파일 오류에서 봤던 내용을 다시 생각해보자  
**사실상 모든 Money는 Expression이어야 한다. 이제 이를 조금씩 없애도록 하자.**  
**파급효과를 피하기 위해 가장자리부터 작업해 나가기 시작해서 테스트 케이스까지 거슬러 올라가도록 한다**  

먼저 Sum 부터 고친다  
```java
public class Sum implements Expression {
    // 1
    public Expression augend;
    public Expression addend;

    // 2
    public Sum(Expression augend, Expression addend) {
        this.augend = augend;
        this.addend = addend;
    }

    // ...
}
```
> 인스턴스 변수 타입을 고치고, 파라미터 타입도 바꾼다  
> 이제 `Sum`을 사용하는 곳에서는 `Expression`을 받을 수 있다  

`Money.plus()`의 파라미터를 Expression으로 바꾼다.  
바꾸는 김에 `times()`의 반환 타입도 바꾼다  
```java
public Expression plus(Expression addend) {
    return new Sum(this, addend);
}

public Expression times(int multiplier) {
    return new Money(amount * multiplier, currency);
}
```

이제 다시 `testMixedAddition()`의 참조변수들을 바꾼다  
```java
@Test
public void testMixedAddition() {
    Expression dollar = Money.dollar(5);
    Expression franc = Money.franc(10);
    // ...
}
```

컴파일러가 `Expression`에 `plus()`를 구현해야 한다고 알려주고 있다  
컴파일러의 지시대로 따라가자  
```java
// Expression
public interface Expression {
    // ...
    Expression plus(Expression addend);
}

// Money
// 이미 구현되어 있음

// Sum
@Override
public Expression plus(Expression addend) {
    return null; // stub
}
```

# 추상화
`Expression.plus()`를 끝마치려면 `Sum.plus`를 구현해야 한다  
테스트를 작성한다  
```java
@Test
public void testSumPlusMoney() {
    Expression dollar = Money.dollar(5);
    Expression franc = Money.franc(10);

    Bank bank = new Bank();
    bank.addRate("CHF", "USD", 2);

    Expression sum = new Sum(dollar, franc).plus(dollar);
    Money result = bank.reduce(sum, "USD");

    assertThat(result).isEqualTo(Money.dollar(15));
}
```

테스트가 통과하게끔 작성한다  
```java
@Override
public Expression plus(Expression addend) {
    return new Sum(this, addend);
}
```
> Money와 형태가 똑같아져서, 추상클래스로 분리할 수 있을 것 같다  

이제 `Expression.times`를 작성해야 한다  
`Sum.times`를 작성한다면 `Expression.times`를 선언하는 일은 어렵지 않을 것 같다  
`Sum.times`에 대한 테스트를 작성한다  
```java
@Test
public void testSumTimes() {
    Expression dollar = Money.dollar(5);
    Expression franc = Money.franc(10);

    Bank bank = new Bank();
    bank.addRate("CHF", "USD", 2);

    Expression sum = new Sum(dollar, franc).times(2);
    Money result = bank.reduce(sum, "USD");

    assertThat(result).isEqualTo(Money.dollar(20));
}
```

Expression에 times 메서드를 선언하고, Sum에도 times를 작성한다  
```java
interface Expression {
    // ...
    Expression times(int multiplier);
}

@Override
public Expression times(int multiplier) {
    return new Sum(augend.times(multiplier), addend.times(multiplier));
}
```

난 사실 이 장이 잘 이해되지 않는다..

<!-- more -->