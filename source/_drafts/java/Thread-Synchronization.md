---
title: 'Thread, Synchronization'
date: 2018-06-10 13:05:24
tags:
---

# 쓰레드
프로그램에 메모리가 할당되면 이것을 수행중인 프로그램, 프로세스라고 한다.  
이때까지 항상 프로세스는 하나의 실행흐름만을 형성해왔는데(main 함수),  
쓰레드를 이용하면 하나의 프로세스 내에서 여러개의 실행흐름을 형성할 수 있다.  

## 쓰레드 구현
자바는 아래처럼 2가지 방법으로 쓰레드를 구현할 수 있는 방식을 지원한다.  
첫째는 Thread 클래스를 상속하는 것이다.  

```java
@Test
public void testThread() {
    ThreadTest t1 = new ThreadTest("Thread1");
    ThreadTest t2 = new ThreadTest("Thread2");

    t1.start();
    t2.start();
    System.out.println("Both thread was over");
}

class ThreadTest extends Thread{
    String threadName;

    public ThreadTest(String threadName) {
        this.threadName = threadName;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(threadName + " : " + i);
        }
    }
}
```

둘쨰는 Runnable 인터페이스를 구현하고 Thread 클래스의 생성자에 주입하는 방식이다.  
다중상속이 되지 않는 자바에서는 이 방법이 조금 더 낫다고 할 수 있다.  

```java
@Test
public void testThread() {
    Thread t1 = new Thread(new ThreadTest("Thread1"));
    Thread t2 = new Thread(new ThreadTest("Thread2"));

    t1.start();
    t2.start();
    System.out.println("Both thread was over");
}

class ThreadTest implements Runnable{
    String threadName;

    public ThreadTest(String threadName) {
        this.threadName = threadName;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(threadName + " : " + i);
        }
    }
}
```

run()은 쓰레드의 메인 함수라고 보면 되고, start()는 JVM이 쓰레드를 생성하고 실행하기 위해 메모리 공간 할당, CPU 사용을 위한 정보 등록 등을 수행하는 메서드이다.  

어찌됐든, 구현했다. 그리고 실행해보자.  
실행해보면.. 정말 제각각으로 실행됨을 알 수 있다.  
어떤때는 Thread1, Thread2, main 순서로 실행이 잘 될때도 있고  
어떤때는 Thread1, Thread2, main 실행 순서가 섞여 있을때도 있고  
어떤때는 하나의 쓰레드만 실행 또는 쓰레드가 아예 실행 안되기도 한다.  

이는 쓰레드를 생성하는 행위 자체가 별도의 실행흐름을 생성하는 과정이기 때문이다.  
즉 main 메서드라는 하나의 실행흐름 안에서 t1, t2라는 별도의 실행흐름을 생성된 것이다. 그리고 그 실행 흐름들은 각각 독립적으로 실행된다.(main 메서드 또한 하나의 쓰레드가 실행한다)  
그래서 위와 같은 결과가 출력되는 것이다.  
아다리가 잘 맞아서 한번에 예쁘게 나오기도 하고, 중간중간 서로 끼어들기도 하고, 실행흐름을 생성하기도 전에 메인함수가 종료되어버려서 실행되지 못하기도 한다.  
각각 독립적인 실행흐름이므로 메인함수가 종료되기 전에 쓰레드가 실행되면 메인함수의 종료여부와 상관없이 동작한다.  

join 함수  
해당 쓰레드가 종료되기 전까지 기다리게 할 수 있다.  

## 쓰레드 우선순위
프로그램의 실행은 결국 CPU가 하는데, CPU는 한번에 한개의 작업밖에 처리하지 못한다.  
이러한 CPU의 자원을 JVM이 수많은 쓰레드들에게 랜덤으로 할당해줄리는 없고, 우선순위라는 것을 부여하여 순차적으로 할당해준다.  
우선순위는 쓰레드가 생성될 때 부여된다. Thread 클래스의 setPriority() 메서드를 통해 지정해줄 수 있다. 10 ~ 1 까지 역순으로 부여되며, 이는 운영체제마다 값이 다르므로 Thread.MAX_PRIORITY, Thread.MIN_PRIORITY 와 같은 상수값을 사용하는 것을 권장한다.  
우선순위를 지정하게 되면 우선순위대로 CPU가 할당되므로 실행 순서가 지켜지는 것을 볼 수 있다.  
그리고 만약 쓰레드 내 run 함수에서 sleep과 같이 CPU가 필요하지 않은 함수를 사용할 경우 쓰레드는 CPU를 우선순위가 더 낮은 애들한테 내준다. 그러므로 우선순위가 낮다고 무조건 실행되지 않는 것이 아니다.  
※ 우선순위는 OS 특성을 타는것 같다.. 우분투에서는 우선수위가 전혀 먹지 않는다.  

## 쓰레드 라이프사이클
new : 쓰레드가 처음 생성되었을 때. 이떄는 쓰레드라 부르긴 좀 그런데 자바에서는 쓰레드라 부른다고 함  
runnable : start를 호출했을 때. 바로 실행되는 것이 아니라 스케줄러에 의해 선택되었을 떄 실행된다!   
blocked : sleep, join 등에 의해 쓰레드가 멈춰있는 상태. blocked의 조건이 해제되면 다시 runnable이 되어 쓰레드가 실행된다.  
dead : run 함수가 실행을 끝마치고 종료된 상태.  

## 쓰레드 메모리 구성
쓰레드는 각각 스택 메모리만을 독립적으로 가지며, 힙과 메서드 영역은 공유한다!!  
그러므로 힙에 들어가있는 인스턴스 변수를 여러개의 쓰레드에서 공유할 경우 원하지 않을 결과가 나올 가능성이 있다.  

# Synchronization(동기화)
위의 인스턴스 변수 공유 얘기를 좀 더 깊게 해보자.  
하나의 인스턴스 변수를 공유하는 2개의 쓰레드가 인스턴스 변수의 멤버 변수의 값을 변경한다고 가정해보자.  
(예를 들어 SomeItem이란 클래스에 있는 num값을 양쪽 스레드에서 1씩 증가시킨다고 해보겠다. 편의상 num++ 이라고 말하겠음)  

근데 num++ 를 하는 과정이 단순히 변수 num에 저장된 숫자에 1을 더하는게 아니다.  
쓰레드가 먼저 num에 저장된 값을 가져오고, cpu에 연산을 요청하고, 증가된 값을 받아 num 변수에 저장하는 과정으로 나뉘어진다.  

이 과정에서 충분히 아래와 같은 일이 일어날 수 있다.  
첫번째 쓰레드가 0값인 num을 cpu에 연산하여 1로 만들고 num 변수에 저장하기 전에,  
두번째 쓰레드가 아직 변경되지 않은 num 값(0)을 가져와서 연산하고, num 변수에 저장하는 행위. 이후 첫번째 쓰레드가 연산된 값을 num에 저장하는 행위.  

즉, 두 쓰레드가 num값에 대해 각각 증가연산을 요청하였지만, 결과적으로 num의 값이 1이 되는 현상이 발생한다.  

아래의 소스코드 정도의 연산량이면 실감해볼 수 있다.  

```java
@Test
public void testThread() {
    SomeItem someItem = new SomeItem();
    ThreadTest t1 = new ThreadTest(someItem);
    ThreadTest t2 = new ThreadTest(someItem);
    ThreadTest t3 = new ThreadTest(someItem);

    t1.start();
    t2.start();
    t3.start();

    try{
        t1.join();
        t2.join();
        t3.join();
    } catch (Exception e){
    }

    System.out.println(someItem.num);
}

class ThreadTest{
    SomeItem someItem;

    public ThreadTest(SomeItem someItem){
        this.someItem = someItem;
    }

    @Override
    public void run() {
        someItem.increaseNum();
    }
}

class SomeItem{
    int num = 0;

    public void increaseNum(){
        for (int i = 1; i <= 10000; i++) {
            for (int j = 0; j < 10000; j++) {
                num++;
            }
        }
    }
}
```

이는 여러개의 쓰레드가 인스턴스 변수를 공유했기 때문에 발생한 일이다.  
이를 막기 위해선 하나의 쓰레드가 increaseNum 메서드에 접근했을 때 다른 쓰레드가 접근하지 못하도록 락을 걸어야 할 것이다. 이런 기법을 동기화 라고 한다.  
자바에서는 간단한 키워드 하나만으로 동기화를 지원해준다.  
increaseNum 메서드를 아래와 같이 변경해줄 수 있다.  

```java
public synchronized void increaseNum(){
    for (int i = 1; i <= 10000; i++) {
        for (int j = 0; j < 10000; j++) {
            num++;
        }
    }
}
```

몇번을 수행해도 똑같은 결과값이 나온다.  
그러면 혹시, 아래와 같은 상황에서는 어떨까?  

```java
class Calculator{
    int callCnt = 0;

    public synchronized int plus(int a, int b){
        callCnt++;
        return a + b;
    }

    public synchronized int minus(int a, int b){
        callCnt++;
        return a - b;
    }

    public int getCallCnt(){
        return callCnt;
    }
}
```

하나의 쓰레드는 plus를 실행하고, 하나의 쓰레드는 minus를 실행한다면 여전히 문제되지 않을까?  
하지만 다행스럽게도(?) 자바 동기화는 락의 범위가 메서드가 아니라 인스턴스이다.  
즉 어떤 쓰레드에서 synchronized 키워드가 적용된 메서드를 호출하고 있다면 나머지 synchronized 메서드들도 전부 락이 걸린다는 뜻이다.  

듣기에는 아름답게 들리나.. 문제가 있다.  
속도가 매우 느려진다는 점이다. 메서드 하나 호출하면 나머지 synchronized 까지 전부 호출이 안된다니... (그리고 솔직히, 이렇게 동기화 할꺼면 멀티쓰레드를 사용할 이유도 없다...)  

그래서 이에 대한 방안으로 자바에서는 블록 단위 동기화를 지원한다.  
지금 plus, minus 메서드 전체에 synchronized가 걸려있지만 솔직히 동기화를 걸어야 되는 부분은 고작 callCnt++ 부분이지 않은가? (plus, minus의 로직이 존내이 복잡하다고 가정하자..)  

굳이 메서드 전체에 synchornized를 걸어서 성능을 저하시킬 필요가 없다.  
아래와 같이 변경 가능하다.  

```java
class Calculator{
    int callCnt = 0;

    public int plus(int a, int b){
        // 졸라 복잡한 로직
        
        synchronized(this){ // 여기만 걸자!
            callCnt++;
        }
        
        // 졸라 복잡한 로직

        return a + b;
    }

    public synchronized int minus(int a, int b){
        // 졸라 복잡한 로직

        synchronized(this){
            callCnt++;
        }

        // 졸라 복잡한 로직

        return a - b;
    }

    public int getCallCnt(){
        return callCnt;
    }
}
```

위와 같이 변경해주면 동기화의 범위를 최소화 함으로써 성능 향상을 도모할 수 있다.  
근데 저기서 사용한 this는 뭘까?  

자바의 모든 인스턴스는 열쇠를 가지고 있다. 전문용어로 lock 또는 monitor라고 하는데, 열쇠라고 이해해도 된단다.  
그리고 synchornized 키워드를 선언한 메서드는 자물쇠를 가지는 것으로 표현할 수 있다.  


<!-- more -->