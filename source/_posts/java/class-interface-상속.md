---
title: 'class, interface 상속'
date: 2018-01-21 21:40:24
tags:
    - class extends
    - interface implements
    - interface extends
---

# class
1. 기본적인 오버라이딩
    ```java
    class Parent{
        Integer test(){
            return 0;
        }
    }
    class Child extends Parent{
        Integer test(){
            return 1;
        }
    }

    // child.test() == return 1;
    ```

2. java는 기본적으로 이름이 같은데 리턴타입이 다른 메서드가 있을 수 없음. 뭘 호출하는지 알 수 없기 때문에.  
    ```java
    class Parent{
        Integer test(){
            return 0;
        }
    }
    class Child extends Parent{
        String test(){ // error
            return "1";
        }
    }
    ```
3. 필드의 경우 그냥 같은 이름은 무시됨  
    ```java
    class Parent{
        Integer test = 0;
    }
    class Child extends Parent{
        String test = "1";
    }

    // child.test == 1;
    ```

# interface
상수랑 추상메서드만 올 수 있음. 키워드는 자동으로 생략됨.  
```java
interface Team{
    // public static final int memberNumber 와 동일
    // public static final은 생략해도 자동으로 붙음
    int memberNumber = 11;

    // public abstract String printMembers() 와 동일
    String printMembers();
}
```

1. 상속은 기본적인 클래스 상속과 동일함. 특이점은 다중 상속이 된다는 것임.  
    ```java
    interface A{
        int a();
    }
    interface B{
        int b();
    }

    // 2개 구현
    class Impl implements A, B{
        @Override
        public int a(){
            return 10;
        }

        @Override
        public int b(){
            return 20;
        }
    }
    ```

    만약 메서드명이 겹치면 하나만 선언해도 됨  
    ```java
    interface A{
        int a();
    }
    interface B{
        int a();
    }

    // 1개 구현
    class Impl implements A, B{
        @Override
        public int a(){
            return 10;
        }
    }
    ```

    같은 이름이지만 리턴타입이 다른 메서드를 가진 인터페이스를 다중 상속할 수 없음.  
    언급했듯이 자바는 기본적으로 동일한 이름에 다른 리턴타입의 메서드를 허용하지 않음.  
    ```java
    interface A{
        int a();
    }
    interface B{
        String a();
    }

    class Impl implements A, B{ // error
    }
    ```

2. 인터페이스 끼리 상속도 됨
    ```java
    interface A{
        int a();
    }
    interface B extends A{
        int b();
    }

    // 2개를 구현해야함
    class Impl implements B{
        @Override
        public int a(){
            return 10;
        }

        @Override
        public int b(){
            return 20;
        }
    }
    ```

    인터페이스도 오버라이딩 되는데, default 메서드를 사용할때나 의미있지 평소에는 오버라이딩이 의미없음.
    ```java
    interface A{
        default int a(){
            return 10;
        }
    }
    interface B extends A{
        default int a(){
            return 20;
        }
    }

    // impl.a() == 20
    ```

    인터페이스 상속은 다중 상속을 지원함  
    ```java
    interface A{
        int a();
    }
    interface B{
        int b();
    }
    interface C extends A, B{
        int c();
    }

    // 3개를 구현해야함
    class Impl implements C{
        @Override
        public int a(){
            return 10;
        }

        @Override
        public int b(){
            return 20;
        }

        @Override
        public int c(){
            return 30;
        }
    }
    ```

3. 필드는 구현체를 따라가지 않음. 기본적으로 static 이기 때문.  
    ```java
    interface A{
        int a = 10;
    }
    interface B extends A{
        int a = 20;
    }
    class Impl implements B{
    }

    public class Main{
        @Test
        public void test(){
            A a = new Impl();
            System.out.println(a.a); // 10
            B b = new Impl();
            System.out.println(b.a); // 20
        }
    }
    ```

이만하면 되지 않았을까..

<!-- more -->