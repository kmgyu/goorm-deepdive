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



chunk 단위 작업의 위험성?
Item Read 시 Connection Timeout 제대로 체크 안하면 읽다가 연결 끊길 수 있다.

chunk의 작업 수행 시간이 wait_timeout 시간보다 짧아야 한다.

작업 시간 영향 미치는 것
read 지연
process 지연
write 지연

read 시 slow query와 같은 이슈가 발생할 수 있다고 함.
- DBMS가 Client로 요청을 받아서 Query를 실행할경우 일정시간안에 수행되지 못한 Query


https://jaehun2841.github.io/2020/08/08/2020-08-08-spring-batch-db-connection-issue/#wait-timeout-%EC%8B%9C%EA%B0%84%EC%9D%B4-%EC%A7%80%EB%82%98%EA%B3%A0-session-%EC%A0%9C%EA%B1%B0
