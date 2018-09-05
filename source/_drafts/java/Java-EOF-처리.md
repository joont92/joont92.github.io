---
title: Java EOF 처리
date: 2018-02-01 16:27:04
tags:
    - Java EOF
    - Scanner EOF
    - BufferedReader EOF
photo : 
    - https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTIwOTU2NzI2MjcmdHlwZT1sJno9MjIvMDQvMjAxOCAxMjo1OQ==
---

`EOF`란 `End Of File`의 약자로 데이터 소스로부터 더 이상 읽을 데이터가 없음을 나타낸다.  

# Scanner에서 EOF 체크
```java
Scanner sc = new Scanner(System.in);

while(sc.hasNextLine()){
    sc.nextLine();
}
```

# BufferedReader에서 EOF 체크
```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

String input = "";
while((input = br.readLine() != null)){
    // input..
}
```

<!-- more -->