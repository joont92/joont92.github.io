---
title: 'Scanner, BufferedReader, StringTokenizer'
date: 2018-03-10 16:11:47
tags:
    - 자바 입력
    - Scanner
    - BufferedReader
    - StringTokenizer
photo : 
  - https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTIwOTU2NzI2MjcmdHlwZT1sJno9MjIvMDQvMjAxOCAxMjo1OQ==
---

# Scanner VS BufferedReader

## Scanner를 통한 입력  
```java
Scanner sc = new Scanner(System.in);

String str = sc.nextLine(); // 1 2 3 4 5 6 7 8 9 10
String[] result = str.split(" ");
```

## BufferedReader를 통한 입력
```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
String[] result = br.readLine().split(" ");
```

문자열에 최적화 된 `BufferedReader`에 비해 `Scanner`는 다양한 기능을 지원하므로 속도가 조금 더 느리다.  


# split VS StringTokenizer

## split
```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
String[] result = br.readLine().split(" ");

// A B C D 입력
result[0]; // A
result[1]; // B
result[3]; // C
result[4]; // D
```

## StringTokenizer
```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
StringTokenizer st = new StringTokenizer(br.readLine());

st.nextToken(); // A
st.nextToken(); // B
st.nextToken(); // C
st.nextToken(); // D
```

문자열을 잘라 쓰는건 똑같은 맥락이나, `split`은 정규식을 기반으로 자르는 로직이므로 내부가 복잡하다.  
그에 반면 `StringTokenizer`의 경우 단순히 공백을 땡기는 것이므로  
정규식 처리가 딱히 필요한게 아닌 경우 `StringTokenizer`가 효율적이다.  

<!-- more -->