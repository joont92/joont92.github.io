---
title: 'dynamic programming'
tags:
---

# Momoization
이미 계산된 `f(5)`같은 결과를 배열 같은곳에 저장해둬서 중복 계산하지 않게 함.  
Memoization에서는 이 배열을 캐시라고 한다.  

# 동적 계획법(bottom-up)
단순히 아래에서 위로 올라간다는 개념이 아니라,  
순환식의 오른쪽에 등장하는 값이 항상 순환식의 왼쪽 값보다 먼저 계산되어 있을 경우 bottom-up 이라고 한다.  

## Memoization vs Dynamic Programming
순환식의 값을 계산하는 기법들  
Memoization은 top-down 방식이며 기본적으로 recursion이므로 overhead가 좀 있다.  
동적계획법은 bottom-up 방식이며 overhead가 없다.  

> 도표작성을 통해 문제를 해결하는 것은 'bottom-up' 방식에 해당한다.  
먼저 하부문제(bottom)를 풀어나가면서, 당신은 n차 배열의 앞부분부터 채워나갈 것이다.  
이 n차 배열의 결과값에 기반하여 최종 문제(top)의 계산을 해나가는 것이다.  

> 만약 당신이 메모이제이션에 기반하여 문제를 해결한다면 이미 모든 하부문제를 해결한 n차 배열을 가지고 있을 것이다.  
존재할 수 있는 최종 문제(top)까지 모든 계산을 마치고, 이미 완성되어있는 표에 기반해 모든 하부문제(bottom)을 해결한다.  
그렇기 때문에 이와 같은 문제 해결 방식은 'top-down' 방식이 될 것이다.

## 피보나치
피보나치 수열을 단순 recursion으로 해결했을 때 문제점

```java
// recursive
int fibonacci(int n){
    if(n==1 || n==2){
        return 1;
    } else{
        return fibonacci(n-2) + fibonacci(n-1);
    }
}
```

이 함수의 문제점은 많은 계산이 중복된다는 점이다.  
`f(8) = f(7) + f(6)  `
이를 풀어서 계산해보면
`f(8) = (f(6) + f(5)) + (f(5) + f(4))`
보다시피 이미 계산된 f(5)에 의해 중복 계산이 발생한다.  
어디 이뿐이겠는가, 위의 식을 또 풀면 계속해서 중복이 나오고 있음을 발견할 수 있을 것이다.  

```java
// memoization
int fibonacci(int n){
    if(n==1 || n==2){
        return 1;
    } else if(f[n] > -1){ // 배열 f가 전부 -1로 초기화 되었다고 가정
        return f[n];
    } else{
        f[n] = fibonacci(n-2) + fibonacci(n-1); // 중간결과 캐싱
        return f[n];
    }
}
```

```java
// bottom-up
int fibonacci(int n){
    f[1] = f[2] = 1;

    for(int i=3; i<=n; i++){
        // 이미 계산되어 있는 값을 가져와 계산
        // 바닥부터 하나씩 저장하며 올라오므로 bottom-up 이라고 함
        f[i] = f[i-1] + f[i-2]; 
    }

    return f[n];
}
```

## 이항 계수
nCk = n개 중에 k개를 선택하는 경우의 수  
기본 공식 : **nCk = n! / (n-k)! * k!**  

![이항계수](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI1MTE4ODIwMjUmdHlwZT1sJno9MjAxOC8wOC8xOSAwOTo0OQ==)  

```java
// recursive
int binomial(int n, int k){
    if(n==k || k==0){
        return 1;
    } else{
        return binomial(n-1, k) + binomial(n-1, k-1);
    }
}
```

피보나치와 마찬가지로 많은 계산이 중복된다.  

```java
// memoization
int binomial(int n, int k){
    if(n==k || k==0){
        return 1;
    } else if(binom[n][k] > -1){ // 배열 binom이 전부 -1로 초기화 되었다고 가정
        return binom[n][k];
    } else{
        binom[n][k] = binomial(n-1, k) + binomial(n-1, k-1); // 캐싱
        return binom[n][k];
    }
}
```

```java
// bottom-up
int  binomal(int n, int k){
    // j는 i보다 클 필요없으며, 구하고자 하는 k보다 클 이유도 없다.
    for(int i=0; j<=i && j<=k; j++){ 
        if(i==0 || i==j){
            binom[i][j] = 1;
        } else{
            binom[i][j] = binom[i-1][j-1] + binom[i-1][j];
        }
    }

    return binom[n][k];
}
```

## 행렬 경로 문제
정수들이 저장된 n*n 행렬의 좌상단에서 우하단으로 이동한다. 반대방향은 안됨.  
방문한 칸에 있는 정수들의 합이 최소화되도록 하라.  
![최단거리_그림](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI1MTE5NzI3OTUmdHlwZT1sJno9MTkvMDgvMjAxOCAxMDozNg==)

> 문제에 대해 순환식을 먼저 세우고(이게 개빡센데;;) -> 순환식을 계산  

(i,j)에 도달하기 위해선 (i, j-1) 또는 (i-1, j)를 거쳐야 한다.  
(i, j-1)까지 최선의 방법으로 와서 가거나, (i-1, j)까지 최선의 방법으로 와서 가야한다.  

![최단거리](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTI1MTE5NTQ4OTImdHlwZT1sJno9MTkvMDgvMjAxOCAxMDoyOA==)  


```java
// recursive
int mat(int i, int j){
    if(i==1 && j==1){
        return m[i][j];
    } else if(i == 1){ // 오른쪽으로 직진
        return mat(1, j-1) + m[i][j];
    } else if(j == 1){ // 아래로 직진
        return mat(i-1, 1) + m[i][j];
    } else{
        return Math.min(mat(i-1, j), mat(i, j-1)) + m[i][j];
    }
}
```

```java
// memoization
int mat(int i, int j){
    if(L[i][j] != -1){
        return L[i][j];
    } else if(i==1 && j==1){
        L[i][j] = m[i][j];
    } else if(i==1){
        L[i][j] = mat(1, j-1) + m[i][j];
    } else if(j==1){
        L[i][j] = mat(i-1, 1) + m[i][j];
    } else{
        L[i][j] = Math.min(mat(i,j-1), mat(i-1,j)) + m[i][j];
    }

    return L[i][j];
}
```

```java
// bottom-up
int mat(int n, int m){
    for(int i=1; i<=n; i++){
        for(int j=1; j<=m; j++){
            if(i==1 && j==1){
                L[i][j] = m[i][j];
            } else if(i==1){
                L[i][j] = L[i][j-1] + m[i][j];
            } else if(j==1){
                L[i][j] = L[i-1][j] + m[i][j];
            } else{
                L[i][j] = Math.min(mat(i,j-1), mat(i-1,j)) + m[i][j];
            }
        }
    }

    return L[n][m];
}

// 조금 트릭을 써서 간결하게 가능
// 원래라면 젤 아래식은 i, j가 1일 경우 0인덱스를 참조해서 참조 오류가 발생하는게 맞다.
// 하지만 실제 프로그래밍에서 배열은 0 인덱스 부터 시작한다.
// 그래서 그 인덱스의 값을 전부 무한대로 설정하면 아래의 공식을 성립시킬 수 있다
int mat(int n, int m){
    for(int i=1; i<=n; i++){
        for(int j=1; j<=m; j++){
            if(i==1 && j==1){
                L[i][j] = m[i][j];
            } else{
                L[i][j] = Math.min(mat(i,j-1), mat(i-1,j)) + m[i][j];
            }
        }
    }

    return L[n][m];
}
```

동적계획법은 순환식을 효율적으로 푸는 방법이다.  
어떤 수치적인 값을 구하는데 사용되는 테크닉이다.  
최적화문제(최소,최대값) 혹은 카운팅 문제에 적용됨

기본적으로 순환식을 먼저 세운 뒤, 그것을 memoization이나 bottom-up으로 푸는 것  
subproblem들을 풀고, 그것들을 조합해서 원래 문제를 푸는 방식 -> 분할정복법  
분할조합법 각각의 subproblem은 disjoint 하지만 동적계획법.  
위의 최단경로의 경우 분할정복법을 통해 문제를 풀었다.   

순환식을 세우는게 동적계획법에서 가장 중요하고 어려운 스킬이다.  
"최적해의 일부분이 그 부분에 대한 최적해인가?"  
s에서 u까지 최단경로가 v를 거쳐서 가는거라면, s에서 v까지의 경로도 