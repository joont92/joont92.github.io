---
title: '[tdd] TDD로 화폐 개발하기(1)'
date: 2019-04-06 22:57:35
tags:
    - 테스트 주도 개발
    - TDD 주기
---

이번 예제는 아래의 것들을 설명해주고 있다  
- 처음 테스트를 작성하고, 주기를 완성하는 방법  
- 불길한 예감을 추가해서 테스트를 단단하게 하는 방법  
- 프로덕션 구현을 어떻게 해야할지 모르겠을 때 삼각측량을 이용하는 방법  
- 코드나 테스트가 리팩토링되면서 감춰도 되는 변수를 private 으로 감추는 방법  

# TDD 주기 만들기  
**요구사항(작업해야 할 목록)을 나열한다**  
> - 통화가 다른 두 금액을 더한 금액(주어진 환율에 맞게)을 결과로 얻을 수 있어야한다  
> - 어떤 금액을 어떤 수에 곱한 금액을 결과로 얻을 수 있어야한다  

요구사항을 보고 할일 목록을 작성한다  
> - $5 + 10CHF = $10(환율이 2:1일 경우)
> - $5 x 2 = $10
> - Dollar 부작용(side effect?)  
> - 등등등

**이중 간단한 것 부터 시작한다**  
- 복잡한 것은 작게 나눠서 시작하던지, 아예 손을 대지 않는것이 좋다  
- 여기서는 `$5 x 2 = $10` 부터 시작한다  

필요할 테스트를 생각해보고, 작성한다  
- 이때 **테스트할 메서드의 완벽한 인터페이스(형태)에 대해 상상해보는 것**이 좋다  
- 가능한 최선의 API에서 시작해서 거꾸로 작업하는 것이 애초부터 일을 복잡하고 보기 흉하며 `현실적`이게 하는 것보다 낫다  

```java
@Test
public void testMultiplication() {
    Dollar five = new Dollar(5);
    five.times(2);
    assertThat(five.amount).isEqaulTo(10);
}
```

Dollar 클래스, 내부 메서드들이 구현되지 않았기 때문에 컴파일부터 실패하므로, 이를 해결한다  
- Dollar 클래스 생성  
- 생성자 생성  
- times(int) 생성  
- amount 필드 생성 
    ```java
    int amount;
    ```
> 컴파일만 될 수 있게 최소한의 구현만 해서, **expected/actual 을 반환하는 형태가 되도록 빨리 만든다**  

현재 상태에서는 `exptected : 10, actual : 0` 을 반환하며 테스트가 실패하지만, 이것도 진척이다(정확히 만족시켜야 할 상황을 알게되기 때문이다)  
**이제 스텁구현(끔찍한 죄악!)을 통해 테스트를 만족시킨다**  
```java
int amount = 10;
```

테스트가 통과하니, 이제 리팩토링(중복 제거)해야한다  
10이라는 숫자는 사실 초기값과 곱하고자 하는 수가 같이 들어가있는, 중복 데이터이다  
이를 분리한다  
```java
int amount = 5 * 2;
```

그리고 뭐.. 이렇게 저렇게해서 아래와 같이 진행한다  

```java
// constructor
Dollar(int amount) {
    this.amount = amount;
}

void times(int multiplier) {
    amount *= multiplier;
}
```

> 단계들이 굉장히 작다고 느껴질 수 있는데,  
> **TDD의 핵심은 작은 단계를 밟아야 한다는 것이 아니라, 이런 작은 단계를 밟을 능력을 갖추어야 한다는 것이다**  
> 작은 단계로 작업하는 방법을 배우면, 저절로 적절한 크기의 단계로 작업할 수 있게 된다  
> **하지만 큰 단계로만 작업했다면, 더 작은 단계가 적절한 경우에 대해 결코 알지 못하게 된다**  

위 과정에서 볼 수 있는 일반적인 TDD의 주기는 아래와 같다  
1. 테스트를 작성한다  
    - 메서드 형태가 어떤식으로 나타나길 원하는지 생각해본다  
2. 실행 가능하게 만든다  
    - 다른 무엇보다도 중요한 것은 빨리 초록 막대를 보는 것이다  
    - 여기서 어떠한 죄악을 저질러도 상관없다  
        - 만약 깔끔하고 단순한 해법이 명백히 보인다면 그것을 입력한다  
        - 굳이 돌아갈 필요는 없다  
3. 올바르게 만든다  
    - 이제 시스템이 작동하므로(초록 막대!) 그 전에 저질렀던 죄악을 수습해야 한다  
    - 죄악을 수습하는 과정은 대부분 중복 제거이다  

**우리의 목적은 동작하는 깔끔한 코드를 얻는 것이다**  
하지만 이는 최고의 프로그래머들도 도달하기 힘든 목표이고, 우리같은 일반적인 사람들은 거의 불가능한 일이다  
그래서 분할하여 정복(Divide and Conquer)를 사용하는 것이다  
일단 `작동하는` 부분을 먼저 해결하고, `깔끔한 코드` 부분을 해결하는 것이다  
> 아키텍쳐 주도 개발과 정 반대다  

# 뭔가 이상한 것 같은데?
**뭔가 기존 코드에 사이드 이펙트가 있는 것 같으니, 빠르게 테스트를 추가해서 확인해보자!!**  
```java
@Test
public void testMultiplication() {
    Dollar five = new Dollar(5);
    five.times(2);
    assertThat(five.amount).isEqaulTo(10);
    five.times(3);
    assertThat(five.amount).isEqaulTo(15);
}
```

테스트가 실패한다. 알다시피 times 수행때마다 내부 amount가 변경되기 떄문이다  
times() 메서드에서 매번 새로운 객체를 반환하게 하면 이 문제를 해결가능할 것 같다  
> 근데 이렇게 변경하려니, 프로덕션 코드와 테스트 코드가 둘 다 변경되어야 한다  
> 뭔가 죄를 저지른듯한 기분이지만, 괜찮다    
> **어떤 구현이 올바른지에 대한 우리 추측이 완벽하지 못한 것과 마찬가지로, 올바른 인터페이스에 대한 추측 역시 절대 완벽하지 못하기 때문이다. 괜찮다.**  

테스트를 변경하자  
```java
@Test
public void testMultiplication() {
    Dollar five = new Dollar(5);

    Dollar product = five.times(2);
    assertThat(product.amount).isEqaulTo(10);
    product = five.times(3);
    assertThat(product.amount).isEqaulTo(15);
}
```

그리고 올바르다고 생각되는 코드를 넣고, 테스트를 돌린다  
(지금은 운이 좋아 명백히 올바른 코드가 떠올랐지만, 바로 떠오르지 않을떄는 스텁으로 구현한다)  
```java
Dollar times(int multiplier) {
    return new Dollar(amount * multiplier);
}
```

이처럼 **느낌(부작용에 대한 혐오감)을 테스트로 변환하는 것은 TDD의 일반적인 주제**이다  
**이런 작업을 오래 할수록 느낌(미적 판단)을 테스트로 담아내는 것에 점점 익숙해지게 된다**  

# 삼각측량을 이용한 VO 구현  
구현해놓고 보니, `new Dollar(5)`와 `new Dollar(5)`가 같아야 할 것 같다  
테스트를 작성한다  
```java
@Test
public void testEquality() {
    assertThat(new Dollar(5)).isEqualTo(new Dollar(5));
}
```

빠르게 통과시켜보자  
```java
@Override
public boolean equals(Object obj) {
    return true;
}
```

알다시피 이는 끔찍한 죄악이다  
얼른 수정해야하는데, 어떻게 구현해야할지 잘 모르겠다(사실은 여기는 너무 간단해서 잘 알지만, 뭔가 복잡한 코드라고 생각해보자)  
여기서 삼각측량을 이용해서 답을 도출해낼 수 있다  
> 삼각측량은 어떤 한 점의 좌표와 거리를 삼각형의 성질을 이용하여 알아내는 방법이다  
> 간단하게 말해 **2개를 통해 나머지 1개를 알아내는 것** 이다  

테스트에 삼각측량을 도입해본다  
```java
@Test
public void testEquality() {
    assertThat(new Dollar(5)).isEqualTo(new Dollar(5));
    assertThat(new Dollar(5)).isNotEqualTo(new Dollar(6));
}
```

amount 5를 가진 Dollar와 amount 6을 가진 Dollar는 같아서는 안된다  
amount를 대상으로 비교하면 될 것 같다  
```java
@Override
public boolean equals(Object obj) {
    Dollar dollar = (Dollar) obj;
    return amount == dollar.amount;
}
// null이나 다른 객체에 대한 검증도 필요하지만  
// 당장은 필요하지 않으므로, 간단히 어디에 메모만 해두고 넘어간다  
```

보다시피 **2개의 테스트를 이용해 올바른 프로덕션 코드를 알아냈다**  
> 사실상 위의 equals 처럼 시작부터 일반적인 해법이 보일 경우 그냥 그 방법대로 구현하면 된다  
> 이 방법은 **설계를 어떻게 할지 떠오르지 않을떄 사용해보면 조금 다른 방향에서 생각해볼 기회를 제공해주는 역할을 한다**  

# 테스트 리팩토링
**Dollar의 동치성을 구현하고 기존 테스트를 보니, 테스트가 그것을 정확히 얘기해주고 있지 않는것 같다**  
바꿔주자  
```java
@Test
public void testMultiplication() {
    Dollar five = new Dollar(5);

    Dollar product = five.times(2);
    assertThat(product).isEqaulTo(new Dollar(10));
    product = five.times(3);
    assertThat(product).isEqaulTo(new Dollar(15));
}
```

불필요한 임시변수(product)를 인라인 시키는 리팩토링을 실시한다  
```java
@Test
public void testMultiplication() {
    Dollar five = new Dollar(5);

    assertThat(five.times(2)).isEqaulTo(new Dollar(10));
    assertThat(five.times(3)).isEqaulTo(new Dollar(15));
}
```

이제 amount 변수는 Dollar 에서 밖에 사용하지 않으니 private으로 선언한다  
```java
private int amount;
```

위 과정에서 우리는 `위험한 상황을 만들었다`라는 점을 인지해야 한다  
**Dollar에 대한 동치성 테스트(testEquality)가 실패하면 곱하기 테스트(testMultiplication) 역시 실패하게 된다는 점이다(테스트간 의존)**  
이것은 TDD를 하면서 적극적으로 관리해야할 위험 요소이다  
> 근데 뭐 어떻게 하라는지는 딱히 알려주고 있진 않네..  

<!-- more -->