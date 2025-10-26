Spring batch 자습서 - 소개


---

# 시작에 앞서...

## 배치 처리란

배치 처리(Batch Processing)는 컴퓨터가 주기적으로 대량의 반복적인 데이터 작업을 처리하는 방식을 말한다. 특정 데이터 처리 태스크는 개별적으로 실시간 실행하기에는 비효율적일 수 있다. 예를 들어, 월말 정산, 대용량 로그 분석, 일일 데이터 백업 등이 이에 해당한다.

이러한 작업을 밤늦은 시간처럼 시스템 사용량이 적은 시간대에 한꺼번에 처리하면, 컴퓨팅 자원의 효율을 극대화하고 시스템 부하를 줄일 수 있다. 또는 사람의 개입을 최소화하면서 반복적인 작업을 자동화하여 운영 효율성을 높일 수도 있다.

---

### **핵심 사용 사례**

- **금융 서비스**
  일일 정산, 위험 관리, 사기 감시 등 대규모 거래를 정확하고 효율적으로 처리
    
- **미디어 및 엔터테인먼트**
  고화질 영상 변환, 그래픽 처리와 같은 대용량 미디어 데이터 자동 처리
    
- **의학 연구**
  게놈 시퀀싱, 임상 데이터 분석 등 대량의 데이터 분석, 연구 속도 향상
    
- **데이터 파이프라인**
  ETL(추출·변환·적재) 작업이나 머신러닝 모델의 피처(feature) 전처리 등 대량의 데이터를 이동하고 정제하는 데 사용
    

---

### **배치 처리 vs 스트림 처리**

배치 처리와 자주 비교되는 개념으로 스트림 처리(Stream Processing)가 존재함.

| 구분     | 배치 처리 (Batch Processing)     | 스트림 처리 (Stream Processing)          |
| ------ | ---------------------------- | ----------------------------------- |
| 처리 방식  | 대량의 데이터를 한 묶음(Batch)으로 모아 처리 | 실시간으로 지속적으로 들어오는 데이터를 처리            |
| 데이터 크기 | 유한하고, 미리 정해진 데이터셋            | 무한하고, 끊임없이 들어오는 데이터 스트림             |
| 주요 사용처 | 정산, 리포트 생성, 데이터 마이그레이션       | 로그 모니터링, 사물 인터넷(IoT) 데이터 분석, 실시간 추천 |

배치 처리는 정해진 데이터셋을 안정적으로 처리하는 데 강점이 있음.
스트림 처리는 실시간으로 변화하는 데이터에 즉각 대응하는 데 적합함.

두 방식은 서로 보완적인 관계이며, 많은 기업이 두 방식을 함께 활용한다.

---

# Spring Batch Introduction

## Spring Batch가 탄생한 이유와 개발 철학

Spring Batch는 대규모 배치 작업을 개발하고 운영하는 데 필요한 모든 공통적인 문제 해결 방법을 추상화하여 제공한다.

개발자들은 반복적으로 마주치는 문제들, 예를 들어 실패 시 재시작, 트랜잭션 관리, 로깅, 그리고 성능 최적화 등을 매번 처음부터 구현해야 했으나, 이는 개발 생산성을 떨어뜨리고, 오류 발생 가능성을 높이는 문제를 유발했다.

Spring Batch는 이러한 문제를 해결하기 위해 탄생했으며, 그 핵심 개발 철학은 다음과 같다.

1. **개발 생산성 향상**: 반복적인 보일러플레이트 코드를 줄이고, 개발자가 비즈니스 로직에 집중할 수 있도록 한다.
    
2. **견고하고 안정적인 애플리케이션**: 실패에 강하고, 데이터 무결성을 보장하며, 예측 가능한 방식으로 동작한다.
    
3. **확장성**: 대량의 데이터를 효율적으로 처리할 수 있도록 병렬 처리, 파티셔닝 등 다양한 확장 기법을 지원한다.
    

---

### **Spring Batch의 핵심 기능

Spring Batch의 강력한 기능들은 대부분 `JobRepository`를 중심으로 구현된다. `JobRepository`는 작업의 실행 상태와 메타데이터를 데이터베이스에 저장하는 역할을 한다.

1. **실패 지점부터 재시작**: 작업이 실패하면 `JobRepository`에 저장된 메타데이터를 기반으로 마지막으로 성공한 지점부터 다시 시작할 수 있다. 예를 들어, 100만 건의 데이터 중 50만 건 처리 후 실패해도, 다음번 실행 시 50만 1번째 데이터부터 시작한다.
    
2. **중복 실행 방지**: 동일한 `Job Parameters`로 성공적으로 완료된 작업은 다시 실행되지 않도록 `JobInstanceAlreadyCompleteException` 예외를 발생시킨다. 이는 이중 정산과 같은 치명적인 오류를 방지하는 중요한 안전장치이다.
    
3. **청크(Chunk) 기반 트랜잭션**: Spring Batch의 가장 일반적인 모델은 청크 처리 모델이다.
    
    - `ItemReader`: 정해진 개수의 아이템을 읽어온다.
    - `ItemProcessor`: 아이템을 처리한다.
    - `ItemWriter`: 처리된 아이템들을 한꺼번에 DB에 저장한다.
        
    이 모든 과정은 하나의 트랜잭션으로 묶여있어, 중간에 오류가 발생하면 청크 전체가 롤백되어 데이터의 무결성을 보장한다.
    

---

### **성능 최적화 및 확장성**

Spring Batch는 대용량 데이터를 효율적으로 처리하기 위한 다양한 기법을 제공한다.

- **커서 기반 페이징(Cursor) vs 오프셋 기반 페이징(Paging)**: 데이터베이스에서 데이터를 읽어오는 두 가지 주요 방식
    - **커서 방식**: 한 번에 소량의 데이터만 메모리에 올리고 커서로 다음 데이터를 가리킨다. 메모리 효율이 높아 매우 큰 데이터셋에 적합합니다.
        
    - **오프셋 방식**: 청크 단위(페이지)로 데이터를 한꺼번에 가져온다. 일반적으로 커서보다 빠르나 더 많은 메모리를 사용한다.
        
- **파티셔닝(Partitioning)**: 하나의 거대한 작업을 여러 개의 작은 작업으로 분할하여 병렬로 실행할 수 있다. 이를 통해 여러 스레드 또는 다른 서버에서 동시에 작업을 처리 가능하게 하여 처리 속도를 향상시킨다.
    

---

### **Spring Batch vs 스케줄러**

Spring Batch는 스케줄러가 아니다.

- **배치 프로그램**: 대량 데이터 처리를 위한 프로그램
    
- **스케줄러**: 정해진 시간에 프로그램을 자동으로 실행시키는 도구
    

Spring Batch는 배치 프로그램을 만드는 데 필요한 것을 제공한다.
하지만 그 프로그램을 언제, 어떻게 실행할지에 대한 기능(스케줄링)은 제공하지 않는다. 따라서 Spring Batch는 Quartz, Jenkins, OS 스케줄러(Cron) 등 외부 스케줄러와 함께 연동하여 사용해야 한다.

# **Spring Batch 용어**

![[spring-batch-basic.png]]

## **Job**
---
### **Job**

![[spring-batch-job.png]]

- 배치 계층 구조 최상위 개념으로, 하나의 배치 작업 그 자체
- Job Configuration을 통해 생성되는 객체 단위로서, 어떻게 구성하고 실행할 것인지 전체적으로 설정
- 배치 Job을 구성하기 위한 최상위 인터페이스, 스프링 배치가 기본 구현체를 제공
- 여러 Step을 포함하는 컨테이너로, 반드시 한 개 이상의 Step으로 구성됨.

**기본 구현체**
- SimpleJob
	- 모든 Job에서 사용 가능한 표준 기능 존재
	- Step을 순차적으로 실행시켜 작업을 진행함.
- FlowJob
	- 특정 조건과 흐름에 따라 Step을 구성해 실행시키는 Job
	- Flow 객체를 실행시켜 작업을 진행함.

 > Flow 개념은 Step을 순차적으로 구성하는 것이 아닌 특정 상태에 따라 흐름을 전환하도록 구성해야 할 때 사용한다.

### **JobInstance**

- Job이 실행될 때 생성
- Job의 논리적 실행 단위 객체, 고유하게 식별 가능한 작업 실행을 나타낸다.
	- Job의 설정, 구성은 동일하나 실행 시점에 처리되는 내용이 달라 구분할 필요가 있음.

**JobInstance 생성 및 실행 과정**
1. 처음 시작하는 Job 및 JobParameter의 경우, 새 JobInstance 생성
2. 이전과 동일한 Job과 JobParameter의 경우 이미 존재하는 JobInstance 리턴
	- 내부적으로 JobName + JobKey(jobParameters의 해시값)를 통해 JobInstance 객체를 얻음
	- 이를 통해 배치 멱등성 보장
3. Job과 1:N 관계

### **JobParameters**

- Job 실행 시 함께 포함되어 사용되는 파라미터를 가진 도메인 객체
	- JobInstance를 구분하기 위함
- JobParameters와 JobInstance는 1:1 관계

**생성 및 바인딩**
- 어플리케이션 실행 시 주입
```bash
java -jar LogBatch.jar requestDate=20210101
```

- 코드로 생성
```
JobParameterBuilder, DefaultJobParametersConverter
```

- SpEL(Spring expression language) 이용
```
@Value(“#{jobParameter[requestDate]}”), @JobScope, @StepScope 선언
```


JobExecution과 1:N 관계

String, Double, Long, Date 4가지 형식만을 지원

### JobExecution

- JobInstance에 대한 실행 시도를 의미하는 객체
	- Job 실행 중 발생 정보를 저장
	- 시작 및 종료 시간, 실행 상태(시작, 완료, 실패 등), 실행 결과(알 수 없음, 실행 중, 완료 등)의 속성을 가짐

**실행 결과 상태에 따른 JobExecution의 동작**

**COMPLETED**
- JobInstance 실행이 완료된 것으로 간주
- 재실행 불가
**FAILED**
- JobInstance 실행이 완료되지 않은 것으로 간주
- 재실행 가능
	- JobParameter가 동일한 값으로 Job 실행 시에도 계속 실행 가능

따라서, JobExecution의 실행 상태 결과가 COMPLETED가 될 때 까지 하나의 JobInstance 내에서 여러 번의 시도가 생길 수 있음.

JobInstance와 1:N의 관계로, 성공/실패 내역을 가짐.

## Step
---

### **Step**

- Batch job을 구성하는 독립적인 하나의 단계
- 실제 배치 처리를 정의, 컨트롤하는 데 필요한 모든 정보를 가지는 도메인 객체
- 단순 단일 태스크 뿐 아니라 입출력 및 처리에 관련된 복잡한 비즈니스 로직을 포함하는 모든 설정을 담는다.
- 배치 작업을 어떻게 구성하고 처리할 것인지 Job의 세부 작업을 Task 기반으로 설정 및 명세한 객체
- 모든 job은 하나 이상의 step으로 구성

**기본 구현체**
- **TaskletStep**
	- 가장 기본 구현체, Tasklet 타입 구현체들을 제어
```java
public Step taskletStep() { 
	return this.stepBuilderFactory.get("step") 
	.tasklet(myTasklet()) 
	.build();
}
```

- **PartitionStep**
	- 멀티 스레드 방식으로 Step을 여러 개로 분리하여 실행
	- ChunkOrientedTasklet을 실행
```java
public Step taskletStep() { 
	return this.stepBuilderFactory.get("step") 
		.chunk(100) .reader(reader()) 
		.writer(writer()) 
		.build();
}
```

- **JobStep**
	- Step 내에서 Job을 실행
```java
public Step jobStep() { 
	return this.stepBuilderFactory.get("step") 
		.job(job()) 
		.launcher(jobLauncher) 
		.parametersExtractor(jobParametersExtractor()) 
		.build();
}
```

- **FlowStep**
	- Step 내에서 Flow를 실행
```java
public Step flowStep() { 
	return this.stepBuilderFactory.get("step") 
		.flow(myFlow()) 
		.build();
}
```

> JobStep과 FlowStep은 Job, Flow를 재사용할 때 사용한다.

### **StepExecution**

- Step에 대한 한 번의 시도를 의미하는 객체
	- Step 실행 중 발생한 정보를 저장
	- Step이 매번 시도될 때마다 생성
	- 각 Step 별로 생성
- Job 재시작 시에도 이미 성공한 Step은 재실행 되지 않고, 실패한 Step만 재실행
- 이전 단계 Step이 실패하고 현재 Step 미실행한 경우, StepExecution 미생성
	- 실제 Step 시작 시에만 생성

**JobExecution과 StepExecution**
- Step이 모두 정상적으로 완료될 경우, JobExecution이 완료
- StepExecution이 하나라도 실패 시, JobExecution은 실패

JobExecution에 저장되는 정보 외에 read 수, write 수, commit 수, skip 수 등의 정보들도 저장

### StepContribution

![[spring-batch-stepcontribution.png]]

- 청크 프로세스의 변경 사항을 버퍼링 후 StepExecution 상태를 업데이트하는 도메인 객체
- 청크 커밋 직전 StepExecution의 apply 메서드를 호출하여 상태를 업데이트
- ExitStatus의 기본 종료 코드 외에 사용자 정의 종료코드를 생성해 적용 가능

> Chunk는 하나의 작업을 n개로 더 작게 나눈 단위
> Tasklet 지향 혹은 Chunk 지향 프로세스가 존재.

## Execution
---
### **ExecutionContext**

![[spring-batch-executionContext.png]]

- 프레임워크에서 유지 및 관리 하는 키/값으로 된 컬렉션
	- StepExecution 또는 JobExecution 객체의 상태 저장
- DB에 직렬화한 값으로 저장
	- 예시 : `{ "key": "context" }`
- Job 재시작 시 이미 처리한 Row 데이터를 건너뛰고 이후 수행하도록 할 때 상태 정보 활용

**공유 범위**
- step - 각 Step의 StepExecution에 저장, Step 간 공유 불가
- job - 각 Job의 JobExecution에 저장, Job 간 공유 불가. 해당 Job의 Step 간 공유 가능

### **JobRepository**

![[spring-batch-jobRepository.png]]

- 배치 작업 중의 정보를 저장
- 배치 작업 수행과 관련된 모든 메타 데이터를 저장
	- Job 수행 및 종료 시기, 실행 횟수 및 실행 결과 등의 정보
	- JobLauncher, Job, Step 구현체 내부에서 CRUD 기능을 처리

> 보통은 영구 DB에 데이터를 저장하며, 인-메모리도 가능
> 스키마를 생성해 과거 및 현재의 실행 정보, 성공 여부 등을 관리
> 수동으로 스키마 생성하는 것도 가능


**설정 방식**
- `@EnableBatchProcessing` 어노테이션 선언 시 자동으로 빈 생성
- `BatchConfigurer` 인터페이스를 구현하거나 `BasicBatchConfigurer` 를 상속하여 JobRepository 설정 커스터마이징 가능

> Spring Batch 5 이상에서는 더 이상 명시하지 않아도 JobRepository가 자동으로 등록되니 참고
> 오히려 자동 생성이 비활성화 된다.

1. **JDBC 방식 설정 - JobRepositoryFactoryBean**
	- 내부적으로 AOP 기술을 통해 트랜잭션 처리
	- 트랜잭션 isolation 기본 값 : SERIALIZEBLE (최고 수준), READ_COMMITED, REPEATABLE_READ로 지정 가능
	- 메타테이블 Table Prefix 변경 가능, 기본 값은 BATCH_

*예시 코드*
```java
@Override 
protected JobRepository createJobRepository() throws Exception {
	JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean(); 
	factory.setDataSource(dataSource); 
	factory.setTransactionManager(transactionManager); 
	factory.setIsolationLevelForCreate("ISOLATION_SERIALIZABLE"); // isolation 수준, 기본값은 “ISOLATION_SERIALIZABLE” 
	factory.setTablePrefix("SYSTEM_"); // 테이블 Prefix, 기본값은 “BATCH_”, BATCH_JOB_EXECUTION 가 SYSTEM_JOB_EXECUTION 으로 변경됨 
	factory.setMaxVarCharLength(1000); // varchar 최대 길이(기본값 2500) 
	return factory.getObject(); // Proxy 객체가 생성됨 (트랜잭션 Advice 적용 등을 위해 AOP 기술 적용) 
}
```


2. IN Memory 방식 설정 - MapJobRepositoryFactoryBean
	- 성능 등의 이유로 도메인 오브젝트를 DB에 저장하지 않을 경우 사용
	- 보통 Test나 프로토타입의 빠른 개발 필요 시 사용

*예시 코드*
```java
@Override 
protected JobRepository createJobRepository() throws Exception { 
	MapJobRepositoryFactoryBean factory = new MapJobRepositoryFactoryBean(); 
	factory.setTransactionManager(transactionManager); // ResourcelessTransactionManager 사용 
	return factory.getObject(); 
}
```

## **JobLauncher**

![[spring-batch-jobLauncher.png]]

- 배치 Job을 실행시키는 역할
- Job과 JobParameters를 인자로 받아 요청된 배치 작업 수행 후 최종 client에게 JobExecution을 반환함
- 스프링 부트 배치 구동 시 JobLauncher 빈 자동 생성

**Job 실행**
- `JobLauncher.run(Job, JobParameters)`를 통해 실행함
- 스프링 부트 배치에서는 JobLauncherApplicationRunner 가 자동적으로 JobLauncher를 실행
- 동기적 실행
	- taskExecutor를 SyncTaskExecutor로 설정한 경우 (기본값)
	- JobExecution을 획득하고 배치 처리를 최종 완료한 이후 client에게 JobExecution 반환
	- 스케줄러에 의한 배치 처리에 적합 (배치 처리 시간이 길어도 상관 없는 경우)
- 비동기적 실행
	- taskExecutor가 SimpleAsyncTaskExecutor로 설정한 경우
	- JobExecution을 획득한 후 Client에게 바로 JobExecution 반환, 배치 처리 완료
	- JobExecutionListener 등을 통해 콜백 로직을 구현함.
	- HTTP 요청에 의한 배치처리에 적합.

## **ItemReader**

- Step에서 Item을 읽어오는 인터페이스
- ItemStream 구현 시 open/update/close로 오프셋/커서 포지션을 기록하여 재시작과 상태를 관리함.
- Reader 단계에서 스키마 미스매치, 필수 컬럼 누락 등 조기 실패 처리를 통해 1차적인 검증 및 필터링을 담당

오프셋 방식과 커서 방식에 따라 구현체와 전략이 구분됨.
- 페이징: 트랜잭션 경계 명확, 커넥션을 오래 붙잡지 않음, 대규모 처리에 안전
- 커서: 큰 테이블 순차 스캔에 효율적이나 커넥션 장시간 점유 가능

**일관성/격리수준**
- 장시간 배치에서는 스냅샷 일관성이 깨질 수 있음.
- 대량 읽기 시 fetchSize 설정을 통해 네트워크 왕복 최소화

**구현체**
- JDBC: JdbcPagingItemReader(페이지 기반), JdbcCursorItemReader(커서/스트리밍)
- JPA: JpaPagingItemReader
- 파일: FlatFileItemReader(CSV/TSV), JsonItemReader, StaxEventItemReader(XML)
- 컬렉션/메시지: ListItemReader, Kafka/AMQP 기반 커스텀 Reader


## **ItemWriter**
- ItemWriter는 처리 된 Data를 Write 할 때 사용
- Writer는 처리 결과물에 따라 Insert가 될 수도 Update가 될 수도 있음.
- Queue를 사용한다면 Send가 될 수도 있다. 
- Writer 또한 Read와 동일하게 다양한 인터페이스가 존재한다. 
- Writer는 기본적으로 Item을 Chunk로 묶어 처리한다.

**구현체**
- JDBC: JdbcBatchItemWriter(벌크 insert/update)
- JPA: JpaItemWriter(영속성 컨텍스트 flush/clear 전략 필요)
- 파일/오브젝트 스토리지: FlatFileItemWriter, S3/GCS 등의 커스텀 가능
- 메시지: KafkaItemWriter, AmqpItemWriter 등의 커스텀 가능
        
**청크 커밋 단위**
- writer는 기본적으로 청크 단위로 묶어 기록하고 트랜잭션 커밋
- 청크 크기는 성능/롤백 비용을 동시에 좌우
	
**벌크 연산 최적화**
- 배치 사이즈, 배치 SQL 사용, 인덱스/트리거 영향 고려
- JPA 사용 시 flush/clear 주기 제어로 메모리 폭주 방지

## **ItemProcessor**

- Item Processor는 Reader에서 읽어온 Item을 가공/검증/정제등을 통해 데이터를 처리하는 역할을 하고 있다. 
- Processor는 배치를 처리하는데 필수 요소는 아니며 Reader, Writer, Processor 처리를 분리하여 각각의 역할을 명확하게 구분한다.

- 전형적 책임
    - 검증/정합성 체크(필수값, 범위, 포맷)
    - 변환/정규화(타입 변환, 코드 매핑, 단위 환산)
    - 보강(외부 조회·캐시로 속성 채우기)
    - 필터링(반환값을 null로 하면 해당 Item은 쓰기 단계에서 제외)
- 구성 패턴
    - CompositeItemProcessor: 여러 Processor 파이프라인 연결
    - ClassifierCompositeItemProcessor: 조건별로 다른 Processor 라우팅

- 외부 I/O 주의
    - API 호출, DB 추가 조회 등은 캐싱/배치 조회로 병목 최소화
    - 순수 함수 형태 유지 시 테스트/재시작 용이
- 예외/품질 관리
    - 가벼운 오류는 Skip(건너뛰기)로 누적 처리, 심각한 오류는 Fail Fast

## **JobExecutionListener**

- Job 실행 전후로 수행할 작업을 정의하는 인터페이스

핵심 메서드로 beforeJob()과 afterJob() 존재, 아래와 같은 작업 수행 가능

`beforeJob()`
- 파라미터 유효성 검증(날짜 범위, 필수 인자)
- 리소스 준비(임시 디렉터리 생성, 캐시 프리로드, 외부 토큰 획득)
- 실행 메타 로깅(트레이스 ID, 배포 버전, 파라미터 스냅샷)
	
`afterJob()`
- 결과 집계/지표 전송(건수, 소요시간, skip/retry 통계 → Prometheus/CloudWatch 등)
- 산출물 후처리(파일 업로드/아카이브, 머티리얼라이즈드 뷰 리프레시 트리거)
- 알림/콜백(Slack/메일/웹훅), 실패 시 재시작 가이드 포함
- ExecutionContext 승격/기록(다음 Job/Step에서 사용할 키 값 저장)
	
실패/성공 분기
- JobExecution.getStatus()로 COMPLETED/FAILED 분기 처리
- 재시작 가능성 판단 기준과 알림 수위 차등화

# 예제 살펴보기
---

## 프로젝트 설정

스프링 배치는 `@EnableBatchProcessing`를 통해 구성해야 했으나, 스프링 배치 5 이후에는 명시적 선언이 아니더라도 자동으로 활성화된다.

사용자 정의 설정이 필요할 경우, 직접 `@Configuration`이 선언된 클래스에 `@EnableBatchProcessing`를 선언하고 필요한 빈들을 오버라이드 할 수도 있다.

```java
@Configuration
@RequiredArgsConstructor
public class MyBatchJobConfig {

    // JobBuilderFactory, StepBuilderFactory 자동 주입
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job myJob() {
        return jobBuilderFactory.get("myJob")
                .start(myStep())
                .build();
    }

    @Bean
    public Step myStep() {
        return stepBuilderFactory.get("myStep")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("Hello, Spring Batch!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```
사용자는 빈으로 Job과 Step을 등록해주기만 하면 사용할 수 있다.

서버 실행 시 자동으로 트리거되어 등록된 Job을 실행시킬 수 있다.

끄고 싶다면 `spring.batch.job.enabled=false` 속성을 properties에 추가해주면 된다.

## Step 구성하기

### 단일 Step 구성
```java
@Slf4j
@Configuration
@EnableBatchProcessing
public class ExampleJobConfig {

    @Autowired public JobBuilderFactory jobBuilderFactory;
    @Autowired public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job ExampleJob(){

        Job exampleJob = jobBuilderFactory.get("exampleJob")
                .start(Step())
                .build();

        return exampleJob;
    }

    @Bean
    public Step Step() {
        return stepBuilderFactory.get("step")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }


}
```

### 다중 Step 구성
```java
@Slf4j
@Configuration
@EnableBatchProcessing
public class ExampleJobConfig {

    @Autowired public JobBuilderFactory jobBuilderFactory;
    @Autowired public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job ExampleJob(){

        Job exampleJob = jobBuilderFactory.get("exampleJob")
                .start(startStep())
                .next(nextStep())
                .next(lastStep())
                .build();

        return exampleJob;
    }

    @Bean
    public Step startStep() {
        return stepBuilderFactory.get("startStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Start Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step nextStep(){
        return stepBuilderFactory.get("nextStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Next Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step lastStep(){
        return stepBuilderFactory.get("lastStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Last Step!!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

}
```


### Flow를 통한 Step 구성

```java
@Slf4j
@Configuration
@EnableBatchProcessing
public class ExampleJobConfig {

    @Autowired public JobBuilderFactory jobBuilderFactory;
    @Autowired public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job ExampleJob(){

        Job exampleJob = jobBuilderFactory.get("exampleJob")
                .start(startStep())
                    .on("FAILED") //startStep의 ExitStatus가 FAILED일 경우
                    .to(failOverStep()) //failOver Step을 실행 시킨다.
                    .on("*") //failOver Step의 결과와 상관없이
                    .to(writeStep()) //write Step을 실행 시킨다.
                    .on("*") //write Step의 결과와 상관없 이
                    .end() //Flow를 종료시킨다.

                .from(startStep()) //startStep이 FAILED가 아니고
                    .on("COMPLETED") //COMPLETED일 경우
                    .to(processStep()) //process Step을 실행 시킨다
                    .on("*") //process Step의 결과와 상관없이
                    .to(writeStep()) // write Step을 실행 시킨다.
                    .on("*") //wrtie Step의 결과와 상관없이
                    .end() //Flow를 종료 시킨다.

                .from(startStep()) //startStep의 결과가 FAILED, COMPLETED가 아닌
                    .on("*") //모든 경우
                    .to(writeStep()) //write Step을 실행시킨다.
                    .on("*") //write Step의 결과와 상관없이
                    .end() //Flow를 종료시킨다.
                .end()
                .build();

        return exampleJob;
    }

    @Bean
    public Step startStep() {
        return stepBuilderFactory.get("startStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Start Step!");

                    String result = "COMPLETED";
                    //String result = "FAIL";
                    //String result = "UNKNOWN";

                    //Flow에서 on은 RepeatStatus가 아닌 ExitStatus를 바라본다.
                    if(result.equals("COMPLETED")) 
                        contribution.setExitStatus(ExitStatus.COMPLETED);
                    else if(result.equals("FAIL"))  
                        contribution.setExitStatus(ExitStatus.FAILED);
                    else if(result.equals("UNKNOWN"))  
                        contribution.setExitStatus(ExitStatus.UNKNOWN);

                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step failOverStep(){
        return stepBuilderFactory.get("nextStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("FailOver Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step processStep(){
        return stepBuilderFactory.get("processStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Process Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }


    @Bean
    public Step writeStep(){
        return stepBuilderFactory.get("writeStep")
                .tasklet((contribution, chunkContext) -> {
                    log.info("Write Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }


}
```

## Step 설정하기

### startlimit
```java
    @Bean
    @JobScope
    public Step Step() throws Exception {
        return stepBuilderFactory.get("Step")
                .startLimit(3)
                .<Member,Member>chunk(10)
                .reader(reader(null))
                .processor(processor(null))
                .writer(writer(null))
                .build();
    }
```


### skip
```java
    @Bean
    @JobScope
    public Step Step() throws Exception {
        return stepBuilderFactory.get("Step")
                .<Member,Member>chunk(10)
                .reader(reader(null))
                .processor(processor(null))
                .writer(writer(null))
                .faultTolerant()
                .skipLimit(1) // skip 허용 횟수, 해당 횟수 초과시 Error 발생, Skip 사용시 필수 설정
                .skip(NullPointerException.class)// NullPointerException에 대해선 Skip
                .noSkip(SQLException.class) // SQLException에 대해선 noSkip
                //.skipPolicy(new CustomSkipPolilcy) // 사용자가 커스텀하며 Skip Policy 설정 가능
                .build();
    }
```

### retry
```java
    @Bean
    @JobScope
    public Step Step() throws Exception {
        return stepBuilderFactory.get("Step")
                .<Member,Member>chunk(10)
                .reader(reader(null))
                .processor(processor(null))
                .writer(writer(null))
                .faultTolerant()
                .retryLimit(1) //retry 횟수, retry 사용시 필수 설정, 해당 Retry 이후 Exception시 Fail 처리
                .retry(SQLException.class) // SQLException에 대해선 Retry 수행
                .noRetry(NullPointerException.class) // NullPointerException에 no Retry
                //.retryPolicy(new CustomRetryPolilcy) // 사용자가 커스텀하며 Retry Policy 설정 가능
                .build();
    }
```

### noRollback

```java
    @Bean
    @JobScope
    public Step Step() throws Exception {
        return stepBuilderFactory.get("Step")
                .<Member,Member>chunk(10)
                .reader(reader(null))
                .processor(processor(null))
                .writer(writer(null))
                .faultTolerant()
                .noRollback(NullPointerException.class) // NullPointerException 발생  rollback이 되지 않게 설정
                .build();
    }
```



# Reference

https://khj93.tistory.com/entry/Spring-Batch%EB%9E%80-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0

https://jiwondev.tistory.com/295

https://jojoldu.tistory.com/324

https://adjh54.tistory.com/169

https://godekdls.github.io/Spring%20Batch/contents/

https://tech.kakaopay.com/post/ifkakao2022-batch-performance-read/

https://aws.amazon.com/ko/what-is/batch-processing/


