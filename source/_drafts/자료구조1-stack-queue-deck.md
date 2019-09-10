---
title: ''자료구조1(stack,queue,deck) ''
date: 2018-08-11 08:48:37
tags:
---

# 스택
후입선출(LIFO)  
![스택](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI0ODg4NTE0MTAmdHlwZT1sJno9MjAxOC8wOC8xMSAwODo1MA==)  

먼저 들어간것이 가장 나중에 나오고, 마지막에 들어간것이 가장 먼저 나옴  
브라우저의 `뒤로가기`도 스택으로 구현됨  

## 배열을 이용한 스택
배열의 크기를 입력받아 생성한다.  
스택내의 데이터 개수를 인덱스로 사용하여 push시에 size++, pop시에 --size해주면 된다.  

```java
public class ArrayBasedStack implements Stack{
    int[] stack;
    int currentSize;

    public ArrayBasedStack(int maxSize) {
        stack = new int[maxSize];
        currentSize = 0;
    }

    @Override
    public void push(int data) {
        stack[currentSize++] = data;
    }

    @Override
    public int pop() {
        return isEmpty() == 1 ? -1 : stack[--currentSize];
    }

    @Override
    public int getTop() {
        return isEmpty() == 1 ? -1 : stack[currentSize-1];
    }
}
```

배열의 크기를 지정해야한다는 단점이 있다.  

## 링크드리스트를 이용한 스택  
스택의 크기를 지정하지 않아도 된다는 것이 장점이다.  
하지만 그에 비해 구현은 나름 복잡한편.. 삽입삭제의 장점을 가진 링크드 리스트이지만 스택의 특성상 탐색 필요없이 마지막 인덱스만 컨트롤 하면 되므로 그닥 장점은 못느끼겠다  

```java
@Data
class Node { // Node가 필요함
    int data;
    Node nextNode;    

    public Node(int data){
        this.data = data;
        this.nextNode = null;
    }
}

public class LinkedListBasedStack implements Stack{
    Node bottom;
    Node top;
    int currentSize;

    @Override
    public void push(int data){
        Node newNode = new Node(data);

        // 스택이 비어있을 경우 추가
        if(bottom == null){
            bottom = newNode;

        // 마지막 노드의 다음 노드를 새로운 노드로 연결시킨다
        } else{
            top.setNextNode(newNode);
        }

        currentSize++;
        top = newNode;
    }

    @Override
    public int pop(){
        Node tempTop = top;

        if(top == null){
            return -1;
        } else if(top == bottom){
            bottom = null;
            top = null;
        } else{
            Node temp = bottom;

            // 최상위 노드의 바로 앞 까지 간다.
            // 그 위치에서 다음 노드의 연결을 끊어야 하기 때문이다.
            while(temp.getNextNode() != null && temp.getNextNode() != top){
                temp = temp.getNextNode();
            }

            temp.setNextNode(null);
            top = temp;
        }

        currentSize--;

        return tempTop.getData();
    }

    @Override
    public int getTop(){
        return top == null ? -1 : top.getData();
    }
}
```
<!-- more -->