---
title: 자료형, 형변환
date: 2018-03-17 22:55:49
tags:
  - Java 자료형
  - Java 형 변환
photo : 
  - https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTIwOTU2NzI2MjcmdHlwZT1sJno9MjIvMDQvMjAxOCAxMjo1OQ==
---

# 기본형(Primitive Type)
자료형 | 표현 데이터 | 크기 | 범위
- | - | - | -
boolean | 참/거짓 | 1byte | true, false
char | 문자 | 2byte | 모든 유니코드 문자    
byte | 정수 | 1byte | -128 ~ 127
short | 정수 | 2byte | -32768 ~ 32767
int | 정수 | 4byte | -2147483648 ~ 2147483647
long | 정수 | 8byte | -9223372036854775808 ~ 9223372036854775807
float | 실수 | 4byte | 1.4E-45 ~ 3.4028235E38
double | 실수 | 8byte | 4.9E-324 ~ 1.7976931348623157E308  

---

# 리터럴
정수값은 기본이 int 리터럴이다.  
실수값은 기본이 double 리터럴이다.  
```java
int i = 10; // int형
double d = 10.10; // double형
```
long, float을 사용하고 싶으면 추가적인 문자를 사용해야 한다.  
```java
long l = 3000000000L; // 30억은 int 범위를 넘어가므로 long 리터럴을 사용하지 않으면 표현할 수 없다.
float f = 10.10F;
```
short, byte는 리터럴이 없다. 그냥 정수값을 넣으면 자동으로 변환된다.  
```java
byte b = 10; // byte형이 됨
short s = 10; // short형이 됨
```

---

# 형변환
## 명시적 형 변환
캐스팅 연산자를 써서 명시적으로 형을 변환하는 방법이다.  
```java
double d = (double)10; // int를 double로 형 변환
float f = (float)10.10; // double을 float으로 형 변환
```

## 크기에 따른 자동 형 변환
`byte < short < int < long < float < double`  
왼쪽으로 오른쪽 순서로 타입의 크기이다.  
큰 자료형에 작은 자료형을 저장할 경우 자동으로 형 변환이 일어난다.  
```java
long l = 10; // long형이 됨
float f = 10L; // float이 더 크므로 long을 저장 가능하다
```

## 산술연산에 의한 자동 형 변환
연산을 하는 숫자들 중 가장 큰 자료형으로 나머지 숫자의 자료형이 결정된다.  
```java
int i1 = 10L + 10; // long형으로 변환되므로 컴파일 에러
int i2 = 3 + 3.5; // double형으로 변환되므로 컴파일 에러

double d1 = 3 / 2; // 둘다 int 형이므로 소수점 이하가 잘린 1이 저장된다.
double d2 = (double)3 / 2; // 3을 double로 강제 형 변환 하였으므로 연산의 결과는 double이 된다.
```

byte형이나 short형은 연산 시 모두 int형으로 변환된다.  
```java
short s1 = 1;
short s2 = 2;
short s3 = s1 + s2; // int로 변환되었기 때문에 컴파일 에러
short s4 = (short)s1 + s2; // short로 변환해야 함
```

char형은 문자형이지만 연산할 수 있다.  
```java
char c = 'a';
int i = 100;
System.out.println(c + i); // 문자 a는 아스키코드값(97)로 변환되어 197이 반환된다.
```

<!-- more -->
