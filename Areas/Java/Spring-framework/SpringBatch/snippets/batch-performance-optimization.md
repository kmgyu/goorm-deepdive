연관 문서
[[batch-performance]]

레퍼런스
https://tech.kakaopay.com/post/spring-batch-performance/#processor%EC%97%90%EC%84%9C-network-io-%EC%84%B1%EB%8A%A5-%EC%A0%80%ED%95%98


# 성능 저하 원인
---

## Processor

스프링 배치의 ItemProcessor는 단건으로 Item을 처리하도록 디자인 되어있다.
주문 API를 단건으로 조회하기 때문에 청크 사이즈만큼 HTTP 통신을 진행하게 되어 이로 인한 성능 저하가 이뤄질 수 있다.

## Writer

JpaItemWriter의 Drity Checking을 통해 업데이트를 진행. 단건으로 처리
또한 영속성 컨텍스트 기반이기 때문에 I/O 횟수 뿐만 아니라 엔티티 변경 감지 비용 발생.

대량 처리가 목적이라면 Projection을 통해서 영속성 컨텍스트를 사용하지 않는 것이 성능 향상에 좋다

# 개선 방법

## 병렬 처리를 통한 HTTP 대기 시간 최소화

주문 API에서 대량 유저 조회로 변경하는 것
레퍼런스 (카카오 기술문서)에서는 변경이 불가능하다고 가정, 클라이언트 측면에서 병렬처리를 활용해 최적화하였다고 함.
RX Kotlin을 사용

병렬 처리를 위해 데이터를 여러 개의 레일로 분할.
150ms 대기 시간 동안 레일 수만큼 처리되므로 성능이 향상된다고 한다.
병렬로 처리된 데이터를 병합, 병합된 데이터 기반 업데이트 수행.

다음은 업데이트 최적화.


## In Update Query를 통한 DB I/O 최소화


청크 사이즈가 100이면 100만큼의 IO를 소비한다.

따라서 In 쿼리를 이용해 하나의 쿼리에서 청크를 처리한다.
이러면 한번의 쿼리에서 찾게된다. 따라서 네트워크 I/O가 발생하지 않음.
-> 비동기 최적화.
이 경우 더티 체킹보다 속도가 더 빠르게 측정된다.
거의 90% 차이남...

## JDBC Execute Batch

만약 In Update를 사용하지 못하는 경우에 대해 생각해본다.

> 예시 시나리오
> 등급과 등급 포인트를 변경하는 상황. 등급 포인트는 각 유저마다 다르기 때문에 In Query를 사용하지 못한다.

이거 사용 시 쿼리를 한 번에 묶어서 DB로 전송 가능

Exposed라는 Kotlin 기반 ORM 프레임워크를 JPA와 함께 사용해서 데이터 처리와 관련된 여러 문제를 해결 가능하다.
한 번에 쿼리를 여러 개 보내서 커넥션 타임아웃 발생 가능성 존재.
따라서 여기서도 작업을 청크 크기로 나눠야 한다.

> In Query는 뭐가 다름...?

update 성능 측정
1. Exposed Bulk Update (압도적)
2. Exposed Update
3. JPA None Persistence Context
4. JPA Dirty Checking Update

```java
// JPA Dirty Checking Update
@Transactional
fun update(writers: List<Writer>) {
    for (writer in writers) {
        writer.name = "update"
    }
}

// JPA None Persistence Context
@Transactional
fun update(ids: List<Long>) {
    for (id in ids) {
        update(qWriter)
            .set(qWriter.name, "update")
            .where(qWriter.id.eq(id))
            .execute()
    }
}

// Exposed Update
fun update(ids: List<Long>) {
    transaction {
        for (id in ids) {
            Writers.update({ Writers.id eq id })
            {
                it[email] = "update"
            }
        }
    }
}

// Exposed Batch Update
fun batchUpdate2(ids: List<Long>) {
    transaction {
        BatchUpdateStatement(Writers).apply {
            ids.forEach {
                addBatch(EntityID(it, Writers))
                this[Writers.email] = "update"
            }
        }.execute(this)
    }
}
```

|개선 이전 방법|개선 방법|
|---|---|
|Processor 기반 단일 I/O 처리 시|Processor 제거 이후 Writer에서 병렬 처리|
|Dirty Checking 기반 단일 Update|데이터 그룹화 및 In Update 활용으로 Database I/O 최소화|
|In Update 불가능한 경우|Execute Batch를 사용하여 여러 작업을 묶어서 Database I/O를 최소화|


마치며?

기존 구조에서는 반복적인 I/O 작업이 개별적으로 처리되고 있습니다. 이를 그룹화하여 처리하는 방식으로 구조를 변경하지 않고도 성능을 개선할 수 있습니다. **이런 성능 최적화는 Spring Batch 외에도 일반적으로 적용 가능하며, 구조를 변경하지 않고도 성능을 향상시킬 수 있는 점을 강조 드리고 싶습니다.**

성능을 개선하기 위해서는 다른 측면에서 어떤 것들이 희생되는지 고려해야 합니다. 병렬처리를 사용하는 경우 테스트, 디버깅 및 트러블 슈팅과 같은 어려움이 발생할 수 있습니다. 따라서 이미 충분히 만족스러운 성능을 갖고 있을 경우 **섣부른 최적화를 피하는 것을 권장합니다. 결국 모든 결정은 트레이드오프이며, 성능을 개선하기 위해서는 다른 측면에서 희생되는 부분이 반드시 발생하게 됩니다.**