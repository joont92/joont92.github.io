---
title: process concept
date: 2018-12-02 21:34:29
tags:
---

복잡한 컴퓨터 시스템에서 가장 중요한것 == abstraction, decompositon  
descomposition : 복잡한 문제를 단순한 여러개의 문제로 나누는 방법론  

process == program in execution(수행중인 프로그램)  

OS == 정부. 시민들을 관리한다(법을 만들어 시민들을 통제하는 등)  
프로세스 == 시민, 수행의 주체, 자원할당의 주체  

프로세스는 OS위에서 프로그램을 실행시키는 주체  
OS의 입장에서 가장 중요한 단위  

decomposition의 한 유닛이 프로세스이다.  
각각의 쪼개진 조각들을 하나하나 실행할 수 있다면 편리하다.  
decomposition 유닛들이 궁극적으로 수행의 단위까지 된 것 == 프로세스  

# Program과 Process의 차이점
- Program  
스토리지만 점유한다.  
수동적인 존재이다.  

- Process  
CPU, memory, IO device 등등을 점유한다.  
능동적인 존재이다.  

# Process State  
프로그램이 수행되는데 필요한 정보, 수행의 결과로 영향을 받을 수 있는 정보들  
1. Memory Context  
- code segment : 어셈블리어  
- data segment : 프로그램의 전역변수들  
- stack segment : 프로그램의 지역변수, 매개변수들  
- heap segment : 동적할당  

2. Hardware Context
- cpu register
- I/O register  

3. System Context
OS가 여러개의 process들을 관리해야 하다보니 각 프로세스들의 정보들을 저장해둔다.  
- process table
- open file table
- page table


# Executon Stream
프로세스가 수행한 모든 명령어들의 순서(sequence)  

# Multi programming, Multi Processing
1. Multi Programming
메모리의 입장이다.  
메인 메모리에 액티브한 프로세스가 여러개 올라와있는 상태를 말한다.  
> 옛날에는 메모리의 용량이 작았기때문에 메인 메모리에 현재 실행되는 프로세스만 올라가고,  
> 사용하지 않는 프로세스는 다른 저장 장치로 내보내는 식으로 멀티 프로그래밍을 구현하였다.  
> 이를 `swapping`이라고 한다.  
> 요즘은 메모리의 용량이 크므로 swapping 하지는 않는다.  

2. Multi Processing  
Multi Programming을 CPU의 관점에서 바라본 것이다.  
CPU는 하나의 명령만 수행가능하므로 계속 스위칭 하면서 여러 프로세스들을 실행한다.  

# Sw system 개발
1. 설계  
> 요구사항 명세서 -> 설계(decomposition) -> tasks  
2. 구현  
> tasks -> 구현 -> program  

이 program이 OS에 의해 process가 된다.  

<!-- more -->