---
title: spring batch AsyncTaskExecutor 등록
date: 2019-03-15 14:53:00
tags:
---

Job들을 비동기로 실행시키고 싶을 경우 사용한다.  

아래와 같이 빈으로 등록한다
```java
@RequiredArgsConstructor
@Configuration
class JobConfiguration {
    private final JobRepository jobRepository;

    @Bean
    public JobLauncher asyncJobLauncher(){ // 메서드명이 빈 이름이 된다
        SimpleJobLauncher simpleJobLauncher = new SimpleJobLauncher();
        simpleJobLauncher.setJobRepository(jobRepository);
        simpleJobLauncher.setTaskExecutor(asyncTestExecutor());

        return simpleJobLauncher;
    }

    @Bean
    public TaskExecutor asyncTestExecutor(){
        SimpleAsyncTaskExecutor asyncTaskExecutor = new SimpleAsyncTaskExecutor();
        asyncTaskExecutor.setConcurrencyLimit(5);
        return asyncTaskExecutor;
    }
}
```

메서드의 이름이 bean의 이름이 된다.  

사용하는 곳은 아래와 같다  

```java
@Service
class SomeService {
    private final JobLauncher asyncJobLauncher; // 스프링은 타입 scan -> 이름 scan을 한다  
    private final Job someJob;

    public void doSomething(){
        jobLauncher.launch(someJob);
    }
}
```

스프링은 타입 injection을 먼저 수행하고, 똑같은 타입이 있으면 이름으로 injection을 수행한다.  
만약 JobLauncher가 asyncJobLauncher 외에 더 등록되어 있다면 injection에서 오류가 발생할테니 위처럼 이름을 지정해줘야 한다.  

<!-- more -->