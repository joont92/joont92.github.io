---
title: java I/O
date: 2018-11-27 00:25:52
tags:
---

자바 자료형은 선언하는 만큼 메모리에 할당된다.
즉, `long a = 1;` 같은 코드는 메모리를 낭비하는 코드이다.  

# char형에 대한 고찰
java에서 char형은 2byte의 크기를 가지며, 음수를 저장하지 않는 unsigned 여서 65,535 크기의 숫자를 저장가능하다.  
내부적으로 문자는 다 숫자로 저장하며, 출력할때는 문자로 보여준다.  
저장할때는 숫자, 문자, 유니코드형 모두 가능하다.  

```java
char a = '\u0061';
char b = 'a';
char c = '97';

/**
출력
a
a
a
**/
```

초기의 문자는 latin 문자 저장을 위한 1byte의 ASCII 코드였는데,  
전세계의 문자를 담기에는 128이라는 숫자가 턱없이 부족해서(...) 유니코드라는 문자셋이 나오게 되었다.  
전세계의 문자는 2byte 내로 전부 담을 수 있고, 이상의 문자들(emoji 등)은 3byte를 넘어간다.  
(유니코드라는 것이 기본적으로 2byte 까지는 말하는 것이 아닌듯? 규격일 뿐이니까?)  

문자열 String은 기본적으로 char 배열로 저장된다.  


표쥰 입출력
특정 프로그램에서 입/출력 대상을 지정하지 않았을 때 디폴트로 사용되는 곳?

자바 프로그램으로 들어오는 것을 인풋, 나가는 것을 아웃풋이라고 한다.  
이러한 인풋과 아웃풋에 관련된 모든 클래스들이 있는 곳이 IO 패키지이다.  

IO는 byte 단위(1byte) 입출력과 char 단위(2byte) 입출력으로 나뉘어짐
byte 단위 입출력 클래스는 모두 InputStream, OutputStream 추상 클래스를 상속 
char 단위는 Reader, Writer

InputStream, OutputStream의 read, write는 전부 추상 메서드이다.  
(구현체제 각각 읽고 쓰는 방법이 다르기 때문)

- 장식대상 클래스
직접적으로 IO를 수행하는 클래스.  
InputStream, OutputStream를 상속받고 추상메서드들을 구현한다
대표적으로 FileInputStream, ByteArrayInputStream 등이 있다.  

- 장식하는 클래스
장식대상 클래스들을 좀 더 편하게 받게 해주는 기능들만을 제공하는 클래스.  
FilterStream 클래스를 상속받은 클래스들을 말함  
(FilterStream 클래스도 InputStream을 상속받음. FilterStream은 생성자가 protected레 제공되어서 직접 생성 불가능)

> 자바 IO가 데코레이터 패턴으로 이루어져있기 때문이다.  

Input/OutputStream의 기본 상대경로는 프로젝트 경로이다.  

read 메서드는 한 바이트씩 읽는 메서드이며  
읽어들여진 int형 정수의 4바이트 중 가장 마지막 1바이트가 읽어들인 정수가 된다.  
굳이 4바이트 int형 정수로 표현한 것은, byte로 나타내면 EOF(end of file)을 표현하지 못하기 때문이다.  
그러므로 int형으로 표현해서 EOF를 표현할 수 있다.  
```java
int readData = -1;
while((readData = fis.read()) != -1){
    fos.write(readData);
}

// ...

finally{
    // 항상 닫아줘야 함
    fis.close();
    fos.close();
}
```

1바이트 대신 버퍼로 받는 방법도 있다  
```java
int readCount = -1;
int[] buffer = new buffer[512]; // 한번에 512씩
while((readCount = fis.read(buffer)) != -1){
    fos.write(buffer, 0, readCount);
}
```

읽어온 데이터는 buffer에 저장되고, 읽어온 바이트의 수 만큼 readCount에 저장된다.  

> 우리의 운영체제는 기본적으로 1바이트를 읽어오라고 해도 항상 대체적으로 512 바이트로 읽아온다.  
그러므로 1바이트만 읽어오라고 명령을 줘도 512바이트를 읽어온 뒤에 앞의 1바이트만 가지고, 뒤를 버리는 행위를 하게 된다.  
(그리고 다음 1바이트를 읽으면 또 512 바이트를 읽고, 1바이트 뒤를 다 버리는 행위를 반복해서 하게 된다)
>> 왜일까? 찾을수가 없다..

java7 부터 등장한 try-with-resource를 사용하면 stream close를 해 줄 필요가 없다.  
Filter Stream(장식하는 클래스) 예제  
```java
try(
    DataOutputStream outputStream = new DataOutputStream(new FileOutputStream("test.txt"));){
        out.writeInt(100);
        out.writeBoolean(true);
}
DataOutputStream outputStream = new DataOutputStream(new FileOutputStream("test.txt"));
```
 
writeInt, writeBoolean, writeDouble 등의 메서드를 통해 원하는 형태로 써 줄 수 있다(?)  

왜 Input/OutputStream에는 Filter가 있는데, Reader/Writer에는 없는건지?  
Reader/Writer는 InputStream, OutputStream을 사용하지 않는다

java Seriallize에 대해 좀 더 알아보기  
ObjectInput(FilterStream)  


--- 12월 1일
abstract class의 생성자가 있나?
protected 생성
native 키워드 사용법
byte, short가 더 좋은데 int를 쓰는이유?
유니코드가 21비트라는데?


<!-- more -->
