---
title: Spring Batch
date: 2019-02-23 21:01:59
tags:
---

# 스프링 배치 설명  
<https://jojoldu.tistory.com/324?category=635883>  
<https://jojoldu.tistory.com/325?category=635883>  
<https://jojoldu.tistory.com/326?category=635883>  

배치서비스의 특징  
- 대용량 데이터
    - 배치 어플리케이션은 대량의 데이터를 가져오거나, 전달하거나, 계산하는 등의 처리를 할 수 ​​있어야 합니다.
- 자동화  
    - 배치 어플리케이션은 심각한 문제 해결을 제외하고는 사용자 개입 없이 실행되어야 합니다.
- 견고성  
    - 배치 어플리케이션은 잘못된 데이터를 충돌/중단 없이 처리할 수 있어야 합니다.
- 신뢰성  
    - 배치 어플리케이션은 무엇이 잘못되었는지를 추적할 수 있어야 합니다. (로깅, 알림)
- 성능  
    - 배치 어플리케이션은 지정한 시간 안에 처리를 완료하거나 동시에 실행되는 다른 어플리케이션을 방해하지 않도록 수행되어야합니다.  

Spring Batch는 Acceuture의 배치 프레임워크를 추상화 한것이다.  
Spring의 3대 요소인 DI, AOP, 서비스 추상화를 다 사용가능하다.  

Spring Quartz는 스케줄러이고, Spring Batch는 대용량 처리 배치이다.  
둘은 완전히 다르다.  
보통 Quartz + Batch를 조합해서 사용한다. Quartz가 Batch를 실행시키는 구조이다.  

- Spring Batch 를 사용하고 싶으면 스프링 부트 어플리케이션 루트에 `@EnableBatchProcessing` 어노테이션을 써줘야함  
- Job은 하나의 배치 작업 단위를 뜻함  
- Job 안에 여러 Step이 존재함  
    - Step이 실제 배치작업(비즈니스 로직)이 들어있는 곳이다  
    - 말 그대로 진짜 스텝이다. job이 실행해야 할 step을 작성하는 것이다.  
        - job 을 큰 작업, step 을 세부 작업이라고 생각하면 안된다  
        - 'job은 일괄 등록 요청, 요청 개수 만큼 step 생성' 은 잘못된 생각이다  
- Step 안에 Tasklet 또는 Reader & Processor & Writer 묶음이 존재함  
    - 진짜 비즈니스 로직을 작성하는 곳  
    - 둘은 같은 레벨이므로 하나만 실행가능  
    - Reader & Processing 후에 Tasklet 불가능  
- JOB은 @Configuration에 등록하고, 서비스가 뜨면 실행된다  
    ```java
    @Bean
    public Job simpleJob(){
        return jobBuilderFactory.get("simple-job")
                .start(simpleStep1(null))
                .next(simpleStep2(null))
                .build();
    }
    ```
- 등록한 Batch Job들을 관리하려면 Spring Batch에서 정의한 메타 테이블들을 사용해야 한다  
    - 배치와 관련된 전반적인 데이터를 관리한다  
    - spring batch와 같이 다운되는 `schema-XXX.sql` 파일에 스키마가 들어있다  

> 지정한 JOB만 실행되도록 하는 방법  
> `application.yml`에 `spring.batch.job.names: ${job.name:NONE}` 를 추가하고,  
> 외부 파라미터로 넘어오는 `job.name`의 값에 맞춰 배치를 실행한다. 전달되지 않으면 `NONE`에 의해 아무 배치도 실행하지 않는다.  
> parameter에 `--job.name=stepNextJob` 의 형태로 입력하면 된다  

## Spring Batch Metadata Table  
JOB은 파라미터에 따라 BATCH_JOB_INSTACNE에 저장됨  
JOB들의 실행을 관리하는 BATCH_JOB_EXECUTION  
JOB들의 실제 로직들인 STEP이 저장된 BATCH_STEP_EXECUTION  

1. BATCH_JOB_INSTANCE  
    - 등록한 JOB들에 대한 인스턴스들인데, 외부에서 전달한 파라미터에 따라 생성된다  
    - 파라미터가 같으면 중복해서 생성되지 않으며, 파라미터가 다를 때만 생성된다  
        - job이나 step에서 파라미터를 꼭 받아야 하는것은 아니다  
    - 이 파라미터들은 BATCH_JON_PARAMETERS에 저장된다. 근데 왜 BATCH_JOB_INSTANCE와 관계를 가지지 않고 BATCH_JOB_EXECUTION과 관계를 가질까?  
2. BATCH_JOB_EXECUTION  
    - JOB_INSTACNE의 실행 내역을 가짐(성공, 실패)  
    - 그러므로 JOB_INSTANCE와 1:N 관계임  
    - 똑같은 파라미터에 대해서는 2번 이상 실행하면 에러가 발생한다  
        - 정확히 말해 COMPLETED가 이미 있으면 에러가 발생하는 것이다  
        - FAILD가 있으면 다시 실행해도 에러가 발생하지 않는다  
3. BATCH_JOB_PARAMETERS  
    - EXECUTION에 사용한 파라미터들을 저장  
    - VALUE OBJECT COLLECTION Table 형태로 사용함  

# 스프링 배치 사용  
## step flow 제어  
Job 내에 Step 등록 시 호출 Flow를 제어 가능하다(순서, 조건별 분기 등등)  

```java
@Bean
public Job stepNextConditionalJob() {
    return jobBuilderFactory.get("stepNextConditionalJob")
            .start(conditionalJobStep1())
                .on("FAILED") // FAILED 일 경우
                .to(conditionalJobStep3()) // step3으로 이동한다.
                .on("*") // step3의 결과 관계 없이 
                .end() // step3으로 이동하면 Flow가 종료한다.
            .from(conditionalJobStep1()) // step1로부터
                .on("*") // FAILED 외에 모든 경우
                .to(conditionalJobStep2()) // step2로 이동한다.
                .next(conditionalJobStep3()) // step2가 정상 종료되면 step3으로 이동한다.
                .on("*") // step3의 결과 관계 없이 
                .end() // step3으로 이동하면 Flow가 종료한다.
            .end() // Job 종료
            .build();
}
```

- 처음 시작할떄는 무조건 start  
- ExitStatus를 catch하고 싶다면 on 사용하고, 뒤에 연결할 step에 to 사용  
- from은 이벤트 리스너. 인자로 받는 step의 이벤트를 listen함  
    - on + end 뒤에만 붙일 수 있는데, 왜 그런건지..  
    - step을 종료시키기 위해서 붙여야하는 패턴인건지?  

## Batch Status, Exit Status  


요청하는 일괄처리 개수만큼 job을 등록함  
step은 말 그대로 