---
title: Java 입출력
date: 2018-01-06 16:10:31
tags: 
    - Java 입출력
    - InputStream
    - OutputStream
    - Reader
    - Writer
---

# 입/출력
컴퓨터 내/외부 장치와 프로그램간에 데이터를 주고받는 것을 말한다.  
여기서 입출력을 위해 두 대상을 연결하여 데이터를 주고받는 통로를 `스트림`이라고 한다.  
스트림은 `FIFO` 구조를 가진다.  

[java io package](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0ODQyNjQ1MDUmdHlwZT1sJno9MjAxOC8wOC8wOSAxNjowMw==)


## 바이트 기반 스트림  
바이트 단위로 데이터를 전송하는 스트림이다.  

### InputStream, OutputStream
모든 바이트기반 스트림의 최상위 클래스이며 둘 다 추상클래스이다.  
`InputStream`은 `read()`, `OutputStream`은 `write()`를 추상메서드로 제공한다.  
입출력 대상에 따라 읽고 쓰는 방법이 다르기 때문에 추상메서드로 제공하는 것이다.  

### ByteArrayInputStream, ByteArrayOutputStream
메모리, 바이트배열에 데이터를 입출력하는데 사용되는 스트림이다.  

### FileInputStream, FileOutputStream
파일 입출력용 스트림  


## 바이트 기반 보조 스트림
스트림의 기능을 향상시키도록 보조해주는 역할을 수행한다.  
직접 데이터를 입출력 할수있는 기능은 없다.  

```java
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("test.txt"));
bis.read();
```

### FilterInputStream, FilterOutputStream
`InputStream`, `OutputStream`의 자손이면서 모든 보조스트림의 조상격이다.  
생성자의 접근제어자가 `protected`이므로 직접 생성이 불가능하고 상속을 통해 오버라이드 되어야 한다.  
아래는 전부 `FilterInputStream`, `FilterOutputStream`을 상속받아 보조기능을 추가한 자식 클래스들이다.  

### BufferedInputStream, BufferedOutputStream
스트림 입출력의 효율을 높이기 위해 버퍼를 사용하는 보조스트림이다. 자주 사용된다.  
`BufferedInputStream`의 경우 `read()` 메서드가 호출되면 입력소스로부터 버퍼 크기만큼의 데이터를 읽어들인다.  
`BufferedOutputStream`의 경우 `write()` 메서드가 호출되면 데이터가 출력 버퍼에 저장되고, 이 버퍼가 가득차면 출력소스로 내용이 출력된다.  
버퍼가 가득차야만 출력되므로 모든 출력작업을 마친 후에는 `flush()`나 `close()`를 호출해서 마지막 버퍼의 내용을 출력하도록 해야한다.  

### DataInputStream, DataOutputStream
데이터를 읽고 쓰는데 있어 byte단위가 아닌 8가지 기본 자료형의 단위로 읽고 쓸 수 있다. (`readInt()`, `readFloat()`, `readBoolean()`...)  

### SequenceInputStream

### PrintStream  


## 문자 기반 스트림
char 배열을 이용한 문자기반 스트림 제공. Text 읽기/쓰기에 특화되어 있다.  
(인코딩에 따라 한번에 읽는 크기가 달라질 수 있다.)  

### Reader, Writer
문자 입출력 스트림의 최상위 클래스이며, 추상클래스이다.  
`Reader`의 경우 `InputStream`과 동일하게 `read()` 대표 메서드이며, 바이트 단위가 아닌 char 단위로 입력된다는 점이 다르다.  
`Writer`의 경우 `OutputStream`과 동일하게 `write()`가 대표 메서드이며, char 단위 출력과 String 출력 둘 다 가능하다.  

### FileReader, FileWriter
파일로부터 텍스트를 읽고 쓰는데 사용된다.  
```java
Reader reader = new FileReader("text.txt");

int data = reader.read();
while(data != -1){
    char dataChar = (char)data;
    data = reader.read();
}
```

### PipedReader, PipedWriter
쓰레드간에 데이터를 주고받을 떄 사용  
`PipedReader`는 반드시 `PipedWriter`에 연결되어야 한다.  

### StringReader, StringWriter
입출력 대상이 메모리인 스트림이다..   
문자열 데이터를 `Reader` 형태로 변환해준다.  

### BufferedReader, BufferedWriter
버퍼를 이용해서 입출력의 효율을 높여준다.  
버퍼 사이즈를 지정 가능하다.  

### InputStreamReader, OutputStreamWriter
바이트기반 스트림을 문자기반 스트림으로 연결시켜준다.  

```java
Reader inputStreamReader = new InputStreamReader(new FileInputStream("test.txt"));
```


# 표준 입출력
JVM이 시작할 때 초기화되믈 직접 초기화 할 필요 없다.  
1. System.in  
콘솔 키보드 인풋에 연결된 `InputStream`이다.  
2. System.out  
콘솔로 데이터를 내보내는 역할을 한다.  
3. System.err  
System.out과 비슷하지만 주로 에러를 내보낼 때 사용한다.  

> **표준 입출력 대상 변경**  
`setIn()`, `setOut()`, `setErr()`로 표준 입출력을 변경할 수 있다.  
```java
OutputStream output = new FileOutputStream("test.txt");
PrintStream printOut = new PrintStream(output);

System.setOut(printOut);
```  

<!-- more -->