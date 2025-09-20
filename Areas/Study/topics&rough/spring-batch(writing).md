https://khj93.tistory.com/entry/Spring-Batch%EB%9E%80-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0

https://jiwondev.tistory.com/295

https://jojoldu.tistory.com/324

https://adjh54.tistory.com/169

https://godekdls.github.io/Spring%20Batch/contents/

https://tech.kakaopay.com/post/ifkakao2022-batch-performance-read/


쓰다 만 것
https://www.notion.so/Spring-batch-27493fd2f5ac80deba93f0455f67259f
노션에서 작성 vs 여기서 작성

---
# **Spring Batch**

Spring Batch는 로깅/추적, 트랜잭션 관리, 작업 처리 통계, 작업 재시작, 건너뛰기, 리소스 관리 등 대용량 레코드 처리에 필수적인 기능을 제공한다.
최적화 및 파티셔닝 기술을 통해 대용량 및 고성능 배치 작업을 가능하게 하는 고급 기술 서비스 및 기능 또한 제공한다.

Spring Batch에서 배치가 실패하여 작업 재시작을 하게 된다면 실패한 지점부터 실행을 하게 된다.

또한 중복 실행을 막기 위해 성공한 이력이 있는 Batch는 동일한 Parameters로 실행 시 Exception이 발생한다.

## 용도

대량 배치 처리 전반
- ETL(추출·변환·적재)
- 집계/리포트
- 정산/청구 배치
- 로그/이벤트 일괄 처리
- 데이터 정제·익명화 
- 보관/아카이브
- 재처리·백필(backfill)
- 캠페인 발송 일괄 처리


운영 자동화
- 주기적 통계 갱신
- 캐시 프리힛(preheat)
예시로, 레디스는 preheat 기능이 없다. 따라서 스프링 배치를 이용해서 미리 캐시를 적재시켜줄 수 있다.

- 지표 스냅샷 생성
    
머신러닝 오프라인 파이프라인
- 피처 추출·전처리의 배치 러닝 단계

이외 용도
데이터 마이그레이션/백필(back fill)
단위/통합 테스트 지원


## 핵심 기능

- **실패 지점부터 재시작:** Spring Batch는 **JobRepository**를 사용하여 작업 상태에 대한 메타데이터를 데이터베이스에 저장함으로써 이 기능을 구현합니다. 작업이 재시작되면, 프레임워크는 이 메타데이터를 참조하여 마지막으로 성공한 스텝을 확인하고 다음 스텝부터 실행을 시작합니다.
    
- **중복 실행 방지**: 성공적으로 완료된 배치 작업은 동일한 Job Parameters(작업 파라미터)로 다시 실행하면 `JobInstanceAlreadyCompleteException` 예외가 발생합니다. 이는 매우 중요한 안전장치입니다. Job Instance(작업 인스턴스) 는 고유한 작업 파라미터 집합에 의해 정의됩니다. 동일한 파라미터로 작업이 성공적으로 완료되면, 의도치 않은 이중 처리(예: 이중 정산)를 방지하기 위해 재실행을 막습니다. 다만, 타임스탬프나 고유 ID를 추가하는 등 다른 파라미터를 사용하면 새로운 작업 인스턴스를 생성하여 다시 실행할 수 있습니다.

- **트랜잭션 관리:** 청크(chunk) 수준에서 트랜잭션을 처리하여 데이터 무결성을 보장합니다. 이는 여러 아이템을 하나의 원자적 단위로 묶어 처리하며, 만약 그중 하나라도 실패하면 전체 청크를 롤백할 수 있습니다.

- **로깅 및 추적:** 각 작업 실행에 대한 메타데이터(시작 시간, 종료 시간, 상태, 실행 시간 등)를 기록합니다. 이는 모니터링 및 디버깅에 필수적입니다.

## Spring Batch 사용 이유

아래와 같은 장점이 존재한다.

- **재시작 가능성:** 이는 배치 작업을 실패에 강한(resilient) 애플리케이션으로 만들어주는 큰 장점입니다.
    
- **청크 트랜잭션:** Spring Batch의 가장 일반적인 처리 모델은 **청크 모델**입니다. 이는 지정된 수의 아이템(청크)을 읽고, 처리한 후, 한 번에 모두 기록하는 방식입니다. 이 모델은 효율적이고 트랜잭션적으로 동작하여 데이터베이스 왕복 횟수를 줄여줍니다.
    
- **건너뛰기/재시도:** 이는 잘못된 데이터에도 중단되지 않고 견고하며 오류를 허용하는 배치 애플리케이션을 구축할 수 있게 해줍니다.
    
- **대량 처리 성능 튜닝(페이징/커서):** Spring Batch는 데이터베이스에서 데이터를 읽는 다양한 방법을 제공합니다.
    
    - **커서 기반(cursor-based)** 리더는 한 번에 소수의 아이템을 가져오고 결과 집합의 다음 아이템을 가리키는 커서를 유지합니다. 이는 매우 큰 데이터셋에 대해 메모리 효율적입니다.
        
    - **페이징 기반(paging-based)** 리더는 청크(페이지) 단위로 아이템을 가져와 메모리에 저장합니다. 일반적으로 커서보다 성능이 뛰어나지만 더 많은 메모리를 소비할 수 있습니다.
        
- **모니터링 메타데이터:** **JobRepository**는 `JobExecution` 및 `StepExecution` 객체를 포함하여 각 작업에 대한 상세 정보를 저장합니다. 이 메타데이터는 작업 상태 모니터링, 성능 분석 및 실패 디버깅에 매우 중요합니다.
    
- **파티셔닝·멀티스레드/원격 청킹:** Spring Batch는 확장성을 고려하여 설계되었습니다.
    
    - **파티셔닝(Partitioning)**은 하나의 큰 스텝을 여러 개의 작은 독립적인 스텝으로 분할하여 여러 스레드나 심지어 다른 머신에서 병렬로 실행할 수 있게 하는 기술입니다. 이는 방대한 데이터셋을 처리하는 일반적인 전략입니다.
        
    - **멀티스레딩** 및 **원격 청킹**은 각각 여러 스레드나 분산 시스템에 워크로드를 분산하여 고성능 처리를 가능하게 하는 다른 기술들입니다.

> 추가적으로, 비동기 프로세싱을 사용하게 되면 fork-join을 사용할 수 있어 

## **Spring Batch vs Scheduler?**

*그렇다면 배치 작업을 자동화해서 일정 시간마다 실행하는 것도 가능한가?*

> **Spring Batch는 Scheduler가 아니다.**

배치 프로그램
- 일괄 처리를 위한 프로그램
- 사용자 명령이 있을 때 실행

스케줄러
- 정해진 시간에 자동으로 실행
- 주기적으로 실행되는 작업 설정

> **스케줄링이란?**
> 일정한 시간 간격으로 반복적 작업을 수행하는 도구

> **배치 스케줄링이란?**
> 대량 데이터 처리 작업을 자동화하는 도구

Spring Batch는 Batch Job을 관리하지만 Job을 구동하거나 실행시키는, 자동화 기능은 지원하지 않는다.
Spring에서 Batch Job을 실행시키기 위해서는 Quartz, Scheduler, Jenkins등 전용 Scheduler를 사용하여야 한다.

배치 프로그램과 스케줄러는 실행 방법과 목적에 큰 차이가 있다.

> 자바에서는 다양한 스케줄링 기능을 제공하는 클래스들이 존재함.
> Timer, ScheduledExeccutorService 등이 일반적
> 스프링에서는 Spring Quartz라는 라이브러리를 제공한다.

여기서는 배치 프로그램에 대해 설명한다.
스케줄러에 대한 것은 여기에서 마무리할 것.



# **Spring Batch 용어**

## **Job**

Job은 배치처리 과정을 하나의 단위로 만들어 놓은 객체입니다. 또한 배치처리 과정에 있어 전체 계층 최상단에 위치하고 있습니다.

## **JobInstance**

JobInstance는 Job의 실행의 단위를 나타냅니다. Job을 실행시키게 되면 하나의 JobInstance가 생성되게 됩니다. 예를들어 1월 1일 실행, 1월 2일 실행을 하게 되면 각각의 JobInstance가 생성되며 1월 1일 실행한 JobInstance가 실패하여 다시 실행을 시키더라도 이 JobInstance는 1월 1일에 대한 데이터만 처리하게 됩니다.

## **JobParameters**

JobInstance는 Job의 실행 단위라고 했습니다. 그렇다면 JonInstance는 어떻게구별 할까요? 이는 바로 JobParameters 객체로 구분하게 됩니다. JobParameters는 JobInstance 구별 외에도 개발자 JobInstacne에 전달되는 매개변수 역할도 하고 있습니다.

또한 JobParameters는 String, Double, Long, Date 4가지 형식만을 지원하고 있습니다.

## **JobExecution**

JobExecution은 JobInstance에 대한 실행 시도에 대한 객체입니다. 1월 1일에 실행한 JobInstacne가 실패하여 재실행을 하여도 동일한 JobInstance를 실행시키지만 이 2번에 실행에 대한 JobExecution은 개별로 생기게 됩니다. JobExecution는 이러한 JobInstance 실행에 대한 상태,시작시간, 종료시간, 생성시간 등의 정보를 담고 있습니다.

## **JobExecutionListener**

Job 실행 전후로 수행할 작업을 정의하는 인터페이스를 의미합니다.

## **Step**

Step은 Job의 배치처리를 정의하고 순차적인 단계를 캡슐화 합니다. Job은 최소한 1개 이상의 Step을 가져야 하며 Job의 실제 일괄 처리를 제어하는 모든 정보가 들어있습니다.

## **StepExecution**

StepExecution은 JobExecution과 동일하게 Step 실행 시도에 대한 객체를 나타냅니다. 하지만 Job이 여러개의 Step으로 구성되어 있을 경우 이전 단계의 Step이 실패하게 되면 다음 단계가 실행되지 않음으로 실패 이후 StepExecution은 생성되지 않습니다. StepExecution 또한 JobExecution과 동일하게 실제 시작이 될 때만 생성됩니다. StepExecution에는 JobExecution에 저장되는 정보 외에 read 수, write 수, commit 수, skip 수 등의 정보들도 저장이 됩니다.

## **ExecutionContext**

ExecutionContext란 Job에서 데이터를 공유 할 수 있는 데이터 저장소입니다. Spring Batch에서 제공하느 ExecutionContext는 JobExecutionContext, StepExecutionContext 2가지 종류가 있으나 이 두가지는 지정되는 범위가 다릅니다. JobExecutionContext의 경우 Commit 시점에 저장되는 반면 StepExecutionContext는 실행 사이에 저장이 되게 됩니다. ExecutionContext를 통해 Step간 Data 공유가 가능하며 Job 실패시 ExecutionContext를 통한 마지막 실행 값을 재구성 할 수 있습니다.

## **JobRepository**

JobRepository는 위에서 말한 모든 배치 처리 정보를 담고있는 매커니즘입니다. Job이 실행되게 되면 JobRepository에 JobExecution과 StepExecution을 생성하게 되며 JobRepository에서 Execution 정보들을 저장하고 조회하며 사용하게 됩니다.

## **JobLauncher**

JobLauncher는 Job과 JobParameters를 사용하여 Job을 실행하는 객체입니다.

## **ItemReader**

ItemReader는 Step에서 Item을 읽어오는 인터페이스입니다. ItemReader에 대한 다양한 인터페이스가 존재하며 다양한 방법으로 Item을 읽어 올 수 있습니다.

## **ItemWriter**

ItemWriter는 처리 된 Data를 Writer 할 때 사용한다. Writer는 처리 결과물에 따라 Insert가 될 수도 Update가 될 수도 Queue를 사용한다면 Send가 될 수도 있다. Writer 또한 Read와 동일하게 다양한 인터페이스가 존재한다. Writer는 기본적으로 Item을 Chunk로 묶어 처리하고 있습니다.

## **ItemProcessor**

Item Processor는 Reader에서 읽어온 Item을 데이터를 처리하는 역할을 하고 있다. Processor는 배치를 처리하는데 필수 요소는 아니며 Reader, Writer, Processor 처리를 분리하여 각각의 역할을 명확하게 구분하고 있습니다.


---

궁금한거

Spring Batch가 대량으로 쿼리 만들 때 사용할 수 있는데, Flyway로도 해줄 수 있지 않나? 어떤 점에서 차이점이 발생하나?

JobLauncher는 단순히 Job을 JobExecution으로 만들어주기만 하는 역할인가? Tasklet과 Item에 따라 실행이 달라지나?

Task와 Chunk 지향 기반은 개발자 경험(DX)에서 차이가 나는가?
성능적으로 차이 나는 점이 있는지?


파티셔닝은 Task, Chunk 같은 단위로 분리하는가?
배치 인스턴스는 어떻게 구분하는가?

입력 소스별로 세가지로 분류 가능
DB 기반
파일 기반
메시지 기반
그렇다면 세 개 모두 트리거는 어떻게 되는 것인지?
DB의 경우 row 입력 시 이벤트 발생?
파일은 입력/수정이 일어날 경우 이벤트 발생?
메시지는 어플리케이션에서 메시지를 수신할 시 발생?

N+1 문제와 같은 Nested Loop가 발생할 수도 있을까?

Spring Quartz는 스케줄러 기능을 제공하며, Spring Integration의 경우 메시징 기반으로 여러 어플리케이션을 관리함. 그렇다면 Quartz 방식과 Integration으로 구현하는 것의 차이점은?

스프링 배치는 동시성 문제에 대해 어떻게 대처하나?

메시징을 통해 관리할 때 Spring Intergration과 Kafka를 사용하는 것 둘 사이에 차이점이 존재하나?

---
## 1. 아키텍처 딥다이브

- **JobRepository 내부 구조**  
    메타데이터가 어떻게 저장되는지, 트랜잭션 롤백/커밋 시 어떤 방식으로 동기화되는지.
    
- **Step 실행 모델**  
    Task 기반 vs Chunk 기반 처리의 차이와 각각의 장단점.
    
- **ExecutionContext의 저장 메커니즘**  
    Commit 단위, Job/Step 단위에서 언제 기록되고 어떻게 복구되는지.
    

---

## 2. 성능 최적화 & 대량 데이터 처리

- **ItemReader 전략 비교**  
    Cursor 기반과 Paging 기반의 DB 접근 차이, 각각의 성능 특성.
    
- **쿼리 최적화**  
    DB 인덱스 전략, batch size 조정, fetch size 설정 같은 실전적인 DB 튜닝 포인트.
    
- **I/O 최적화**  
    파일 읽기/쓰기 시 Buffering, Streaming API 적용, 멀티 리더 패턴.
    

---

## 3. 동시성 처리 & 확장성

- **멀티스레드 Step**  
    단일 Step 내에서 멀티스레드 실행 시 주의점 (동시성 문제, 트랜잭션 경합).
    
- **Partitioning**  
    마스터/슬레이브 Step 구조, 데이터 파티션 기준 설계 (ID 범위, 날짜 범위 등).
    
- **Remote Chunking & Remote Partitioning**  
    메시지 큐(Kafka, RabbitMQ)를 활용한 분산 처리 모델.
    

---

## 4. 실패 처리 전략

- **Retry / Skip 정책**  
    Checked/Unchecked Exception 구분, `RetryTemplate`, `SkipPolicy`의 동작 방식.
    
- **재시작 전략**  
    실패 시 ExecutionContext를 활용해 이어 실행하는 방식.
    
- **보상 트랜잭션 (Compensation Transaction)**  
    중간 실패 시 비즈니스적으로 데이터를 어떻게 복구할지.
    

---

## 5. 모니터링 & 운영

- **Job Explorer / Job Operator**  
    배치 실행 관리 및 조회 API.
    
- **모니터링 지표**  
    StepExecution에 기록되는 read/write/skip/rollback 수치 활용.
    
- **Spring Boot Actuator 통합**  
    배치 실행 상태를 메트릭/엔드포인트로 노출하는 방법.
    

---

## 6. 배치 vs 스트리밍 비교

- **Spring Batch와 Spring Cloud Data Flow**  
    오프라인 배치와 실시간 스트리밍 파이프라인의 경계.
    
- **Kafka Consumer와 배치 처리 비교**  
    배치의 강점(트랜잭션·재시작) vs 스트리밍의 강점(실시간성).