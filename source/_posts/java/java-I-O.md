---
title: [java] java I/O
date: 2017-11-27 00:25:52
tags:
    - java 입출력
    - InputSream
    - OutputStream
    - FiliterInputStream
    - FilterOutputStream
    - Reader
    - Writer
---

I/O는 Input/Output의 약자로, 입력/출력을 말한다.  
입출력은 컴퓨터 내/외부 장치와 프로그램간의 데이터를 주고받는 것을 말한다.  

# 스트림
데이터를 주고받으려면 두 대상을 연결하고 데이터를 전송할 수 있는 연결통로 같은것이 필요한데, 이를 스트림이라고 한다.  
스트림은 단방향으로만 가능하므로 하나의 스트림으로 입,출력을 동시에 진행할 수 없다.  
그러므로 입력 스트림, 출력 스트림이 따로 존재하고, 입력과 출력을 동시에 수행하려면 2개의 스트림이 필요하다.  
스트림은 FIFO 구조로 되어있다.  

자바에서 제공하는 스트림은 크게 2가지가 있다.  
- 바이트 단위로 데이터를 전송하는 `바이트기반 스트림`과 해당 스트림의 기능을 보완하기 위한 필터 스트림
- 문자 단위로 데이터를 전송하는 `문자기반 스트림`과 해당 스트림의 기능을 보완하기 위한 필터 스트림  

![java i/o package](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI5MzA1MTcxNjQmdHlwZT1sJno9MjAxOC8xMi8xMiAxNTowMw==)  

# 바이트 기반 스트림
바이트 단위로 데이터를 전송하는 스트림이다.  
최상위 클래스는 `InputStream`과 `OutputStream`이고,  
입출력 대상에 따라 이를 상속한 `FileInputStream`, `ByteArrayInputStream`, `PipedInputStream` 등이 있다.  

## InputStream, OutputStream  
- [InputStream](https://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html)  
입출력 대상에 따라 입력, 출력의 방식이 다르므로, abstract 메서드로 `read()`를 제공하고 자식쪽에서 이를 구현하도록 했다.  
이 같은 추상화로 인해 입출력의 방식이 달라져도 일관된 방법으로 입출력이 가능하다.  
`read(byte[] b)`, `read(byte[] b, int off, int len)`도 결국 내부적으로 `read()`를 사용하므로 `read()`는 필수적으로 구현해야 한다.  

`read()`는 입력받은 바이트를 반환하는데, 바이트를 받음에도 불구하고 byte가 아닌 int를 사용하고 있음을 볼 수 있다.  
이는 입력소스(파일 등)으로 부터 1byte(0~255)와 더 이상 읽을 데이터가 없음(-1(EOF))을 받기로는 byte 자료형으로 부족하기 때문이다.  
그러므로 무조건 byte보다는 큰 자료형을 사용해야 하고, 다소 크긴 하지만 가장 효율적인 int형을 사용한다.  
[int대신 short를 사용하는 이유](https://stackoverflow.com/questions/21062744/why-does-inputstream-read-return-an-int-and-not-a-short)  

> **read(byte[] b), read(byte[] b, int off, int len)**  
> 1 byte씩 데이터를 전송하면 너무 느리므로, byte 배열에 데이터를 담아서 전송함으로써 효율을 높인다.  
> 하나씩 전달하는 것 보다 바구니에 담아서 전달하는 것이 더 빠른것을 생각하면 된다.  
> 반환되는 int 값은 읽은 바이트의 개수이다.  
> - `read(byte[] b)`: 배열 b의 크기만큼 데이터를 읽어와서 b에 저장한다.  
> - `read(byte[] b, int off, int len)` : len의 크기만큼 데이터를 읽어와서 배열 b의 off 위치부터 저장한다.  

- [OutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html)  
abstract 메서드로 `write(int b)` 를 제공한다. 방식은 `InputStream`과 동일하다.  
(`flush`의 경우 버퍼가 있는 출력 스트림의 경우에만 의미가 있다.)  

> **write(byte[] b), write(byte[] b, int off, int len)**  
> - `write(byte[] b)` : 배열 b에 저장된 모든 내용을 출력소스에 쓴다  
> - `write(byte[] b, int off, int len)` : 배열 b에 저장된 내용을 off 위치부터 len개 만큼 출력소스에 쓴다.  

## FileInputStream, FileOutputStream
파일에 입출력을 하기 위한 스트림으로, 실제 프로그래밍에서 많이 사용된다.  
- [FileInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/FileInputStream.html)  
- [FileOutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/FileOutputStream.html)  

아래는 file 복사의 간단한 예제이다.  

```java
@Test
public void fileCopy(){
    try(
        FileInputStream fileInputStream = new FileInputStream("/Users/home/test.mov");
        FileOutputStream fileOutputStream = new FileOutputStream("/Users/home/test_copy.mov")
    ){
        byte[] temp = new byte[8192];

        long start = System.currentTimeMillis();
        while(fileInputStream.read(temp) > 0){
            fileOutputStream.write(temp);
        }
        long end = System.currentTimeMillis();

        System.out.println((end - start) / 1000.0);
    } catch(IOException e){
        e.printStackTrace();
    }
}
```

## ByteArrayInputStream, ByteArrayOutputStream
바이트 배열에 입출력 하기 위한 스트림이다.  
입출력 대상이 메모리(배열은 java heap에 저장되므로) 이므로 close를 하지 않아줘도 된다는 특징이 있다.  
자주 사용되진 않지만 `read(byte[] b)` 사용 시 발생할 수 있는 실수를 알아보기 위해 첨부하였음.  

```java
@Test
public void readFromByte() {
    byte[] src = {1,2,3,4,5,6,7,8,9,10};
    byte[] dst;

    byte[] temp = new byte[4];

    ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(src);
    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();

    try{
        while(byteArrayInputStream.read(temp) > 0){
            byteArrayOutputStream.write(temp);
        }
    } catch (IOException e){
        e.printStackTrace();
    }

    dst = byteArrayOutputStream.toByteArray();
    
    System.out.println(Arrays.toString(dst));
}
```

`[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 7, 8]` 이 출력된다.  
미자막 루프에서 `9, 10` 2byte만 읽었으므로 temp의 앞 2byte만 교체되는데,  
`write(temp)`에서 temp의 모든 내용을 출력하도록 했기 때문이다.  

```java
int len;
try{
    while((len = byteArrayInputStream.read(temp)) > 0){
        byteArrayOutputStream.write(temp, 0, len);
    }
} catch (IOException e){
    e.printStackTrace();
}
```

이처럼 변경해줘야 한다.  

# 바이트 기반 보조 스트림
바이트 기반 스트림의 기능을 보완(성능향상 및 기능추가)하기 위함.  
자체적인 입출력 기능은 없다. 기반 스트림에 입출력 기능을 위임한다.  
그러므로 생성자에서 항상 기반 스트림을 받는다.  
조상 클래스는 `FilterInputStream`, `FilterOutputStream`이고,  
자식으로는 `BufferedInputStream/BufferedOutputStream`, `DataInputStream/DataOutputStream`, `SequenceInputStream`, `PrintStream` 등이 있다.  
보조 스트림을 close 하면 기반 스트림도 같이 close 된다.  

## FilterInputStream, FilterOutputStream
- [FilterInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/FilterInputStream.html)
- [FilterOutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/FilterOutputStream.html)  

위 두 클래스는 생성자가 `protected`이므로 직접 생성이 불가능하고, 상속을 통해 오버라이딩 되어야 한다.  

데코레이터 패턴을 기반으로 디자인 된 클래스 구조이기 때문에, 여러 Filter 클래스들을 겹쳐서 사용할 수 있다!  
```java
DataInputStream dis = new DataInputStream(new BufferedInputStream(System.in));
```

## BufferedInputStream, BufferedOutputStream
- [BufferedInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedInputStream.html)
- [BufferedOutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedOutputStream.html)  

한 바이트씩 입출력 하는 것 보다는 버퍼를 이용해 한번에 여러 바이트를 입출력하는 것이 빠르므로 자주 사용된다.  
`read(byte[] b)`는 모아뒀다가 한번에 전송하는것이고, 이건 들고올 때 부터 지정한 크기만큼 들고온다. 둘은 다르다.(조금 더 알아봐야함. 이중 버퍼 같은건가?)  
<https://stackoverflow.com/questions/53735627/difference-between-fileinputstream-with-buffer-and-bufferedinputstream>
(답글이 달릴까?)  
그러므로 보통 입력소스로부터 한번에 가져올 수 있는 데이터의 크기로 지정한다.  

`BufferedOutputStream`은 버퍼의 내용이 가득차면 출력소스로 출력한다.  
버퍼가 가득 차지 못해서 출력되지 못할 수도 있으니 항상 마지막에 `flush()`나 `close()`를 호출해서 버퍼를 비우도록 해야한다.  

# 문자 기반 스트림
java는 UTF16 인코딩을 사용하기 때문에 문자형을 2byte로 처리한다.  
그러므로 바이트 기반 스트림으로 문자를 처리하기에는 어려움이 많다.  
그래서 탄생한 것이 `Reader`, `Writer`이다. 이 또한 가장 최상위 클래스이다.  
자식은 바이트 기반 스트림 자식에서 이름만 `InputStream -> Reader`, `OutputStream -> Writer`로 바꿔주면 된다.(`FileReader/FileWriter`, `PipedReader/PipedWriter` 등)  

## Reader, Writer
- [Reader](https://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)
- [Writer](https://docs.oracle.com/javase/8/docs/api/java/io/Writer.html)  

byte배열 대신 char배열을 사용한다는 것과, 추상메서드가 `read(byte[] b, inf off, int len)` 으로 달라졌다는게 차이점이다.  
프로그래밍 관점에서 이 추상메서드를 사용하는게 좀 더 바람직하기 때문이다.  

## FileReader, FileWriter
File로 부터 문자열을 입출력할 때 사용한다.  
- [FileReader](https://docs.oracle.com/javase/8/docs/api/java/io/FileReader.html)
- [FileWriter](https://docs.oracle.com/javase/8/docs/api/java/io/FileWriter.html)  

```java
@Test
public void InputStream와Reader의문자처리() {
    try(
        FileInputStream fileInputStream = new FileInputStream("/Users/home/Desktop/대충 정리.md");
        FileReader fileReader = new FileReader("/Users/home/Desktop/대충 정리.md")
    ){
        int a;
        while((a = fileInputStream.read()) != -1){
            System.out.print((char)a);
        }

        System.out.println();

        int b;
        while((b = fileReader.read()) != -1){
            System.out.print((char)b);
        }

    } catch(Exception e){
        e.printStackTrace();
    }
}
```

`InputStream`으로 읽었을 경우, latin 문자가 아니면 문자가 깨진다.  

# 문자 기반 보조 스트림
바이트 기반 보조 스트림과 용도는 동일하다.  
`FilterInputStream/FilterOutputStream` 처럼 기반이 되는 클래스는 따로 없다.  

## BufferedReader, BufferedWriter
- [BufferedReader](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html)
- [BufferedWriter](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedWriter.html)  

버퍼를 이용해 입출력의 성능을 높여준다.  
`BufferedReader`는 라인 단위로 읽어올 수 있는 `readLine()` 메서드를 제공하고,  
`BufferedWriter`는 개행을 출력할 수 있는 `newLine()` 메서드를 제공한다.  

## InputStreamReader, OutputStreamWriter
- [InputStreamReader](https://docs.oracle.com/javase/8/docs/api/java/io/InputStreamReader.html)
- [OutputStreamWriter](https://docs.oracle.com/javase/8/docs/api/java/io/OutputStreamWriter.html)  

이름에서 알 수 있듯이 바이트 기반 스트림을 문자 기반 스트림으로 변환해주는 역할을 한다.  
인코딩을 직접 지정할 수도 있다. 인코딩을 지정하지 않으면 OS에서 기본으로 사용하는 인코딩을 사용할 것이다.  

콘솔 입력을 받을 때 주로 이용한다.(이 외에도 많겠지?)  
```java
InputStreamReader in = new InputStreamReader(System.in);
BufferedReader br = new BufferedReader(in);

// do something...
br.readLine();
```

<!-- more -->
