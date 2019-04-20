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
- Job 안에 여러 Step이 존재함  
    - Step이 실제 배치작업(비즈니스 로직)이 들어있는 곳이다  
    - 말 그대로 진짜 스텝이다. job이 실행해야 할 step을 작성하는 것이다.  
        - job 을 큰 작업, step 을 세부 작업이라고 생각하면 안된다  
        - 'job은 일괄 등록 요청, 요청 개수 만큼 step 생성' 은 잘못된 생각이다  
- Step 안에 Tasklet 또는 Reader & Processor & Writer 묶음이 존재함  
    - 진짜 비즈니스 로직을 작성하는 곳  
    - 둘은 같은 레벨이므로 하나만 실행가능  
    - Reader & Processing 후에 Tasklet 불가능  
- Job, Step, Tasklet(ItemReader, ItemWriter 등) 은 전부 스프링 빈이다  

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

> BatchStatus, ExitStatus  
> JOB_EXECUTION, STEP_EXECUTION 테이블을 보면 status, exit_code라는 것이 있다.  
> status는 job이나 step의 실행 결과를 기록할 때 사용하는 Enum이고,  
> exit_code는 job이나 step의 실행 후 상태를 나타내는 일반 string이다.  
> 기본적으로 exit_code는 status와 같도록 설정되어 있으나, 커스텀한 exit_code가 있으면 추가할 수 있는 구조이다.  

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
    - 위의 상황에선 step1의 이벤트를 이미 캐치하고 있으므로, 추가로 이벤트를 캐치하려면 from을 사용해야했음  
    - on + end 뒤에만 붙일 수 있는데, 왜 그런건지..  
    - step을 종료시키기 위해서 붙여야하는 패턴인건지?  
- step에서 contribution.setExitStatus 를 세팅해줘야만 job에서 catch 가능  
- custom ExitStatus를 세팅하려면 StepExecutionListenerSupport 를 상속받은 클래스를 추가로 등록해야 한다  

## decide
step들의 flow 속에서 분기만 담당하는 애들  
ExitStatus 세팅하는 부분을 분리할 수 있다  

## Scope, jobParameter
@JobScope, @StepScope 를 선언하면 빈 생성시점이 해당 scope가 실행되는 시점까지 지연된다  
Scope를 이렇게 뒤로 미루면서 얻는 장점이 여러가지가 있다  
- 비즈니스 로직 처리단계에서 Job Parameter 할당받을 수 있음  
- 병렬처리 가능(step당 각자 tasklet을 가지므로)  

## jobParameter
@JobScope, @StepScope 에서 파라미터(외부/내부)를 받을 수 있다  
step에 @JobScope를 선언하고 parameter를 매개변수로 받는 방법과,  
tasklet, itemReader 등에 @StepScope를 선언하고 parameter를 매개변수로 받는 방법이 있다.  
> jobParameter가 필요한 곳에서 사용하도록 해야할듯  
> 예를들어 step 단위에서 parameter가 필요하다면 @JobScope에서 파라미터를 받아야 할 것이고, 
> tasklet 단위에서 parameter가 필요하다면 @StepScope에서 파라미터를 받아야 할 것이다  

jobParameter는 @JobScope(step), @StepScope(chunk) 빈을 생성할때만 사용할수있으며,  
step과 chunk의 최상위에는 job이 있다.  
즉, job을 생성할때 던진 파라미터를 이용해서 scope bean에서 job parameter로 사용하는 것이다(아마도)  
> 이 scope 빈을 @XXXScope로 생성하지 않고 일반 싱글톤으로 생성하면 job Parameter를 찾을 수 없다  
> jobParameter는 job으로 부터 오는것이니까  
> job을 생성(실행)할 때 던진 parameter를 가지고 @JobScope, @StepScope 빈을 생성하며 parameter를 주는 것이다  
> 그러므로 job 코드에는 parameter로 null이 들어가게 된다(... 아마도?)  

```java
JobParameters jobParameters = new JobParametersBuilder()
                        .addString("input.file.name", fileName)
                        .addLong("time", System.currentTimeMillis())
                        .toJobParameters();
jobLauncher.run(job, jobParameters);
```

jobLauncher.run을 따라가보면(SimpleJobLauncher 기준)  
jobExecution이 있으면 오류가 발생하고, jobExecution이 없으면 jobExecution을 생성한다  
그리고 해당 jobExecution은 async로 실행시킨다(아마도)  

# chunk
chunk 지향 처리란 itemReader, itemProcerssor, itemWriter로 이어지는 형태를 말함
tasklet은 내부에 모든 로직이 다 있다  
reader에서 job을 읽어오고, process에서 처리하고 writer로 쓴다  
reader, process는 1건씩 처리되고, chunkSize만큼 processing이 완료되면 writer로 한방에 쓴다?  

chunk는 배치에서 한번에 트랜잭션으로 처리할 단위이다  
pageSize와는 다르다. pageSize는 한번에 조회하는 단위이다.  
pageSize와 chunkSize를 같게해야 성능상 좋다  

## reader  
reader로 읽을 수 있는 데이터는 데이터베이스만이 아니라 입력 데이터, 파일, jms 등등 여러가지이다  
cursor, paging  



---

# 장점
1. spring batch에 사용할 단위들을 bean으로 생성할 수 있고, scope가 Job, Step scope 단위라는 점  
그러므로 파라미터를 원하는 타이밍에 자유자재로 받을 수 있고, 병렬처리에도 좋다  

2. chunk 