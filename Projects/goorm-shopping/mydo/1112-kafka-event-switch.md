카프카를 이용해서 스케줄러 제어한다.
공부 공부 공부 또 공부 뇌가 녹는다

그냥 카프카로는 안된다.
지연 시간 큐를 쓰기 때문에.

---
### 아키텍처 흐름

1. **API (Producer):** `CouponDelayProducer`가 `coupon-delay-events` 토픽으로 이벤트 발행 (기존과 동일)
    
2. **Stream Processor (K-Streams):**
    
    - `CouponDelayStreamProcessor`가 이벤트를 즉시 수신합니다.
        
    - **`process()`:** 이벤트의 `processAt` (처리 시간)을 계산합니다.
        
    - **`StateStore` (상태 저장소):** 이벤트를 `processAt_couponIssueId`를 Key로 하여 내부 RocksDB(`StateStore`)에 저장합니다.
        
    - **`Punctuator` (스케줄러):** 1분(또는 10초)마다 주기적으로 실행됩니다.
        
    - **`punctuate()`:** `StateStore`를 스캔하여 현재 시간(`now`)보다 `Key`가 이른(즉, 처리 시간이 된) 모든 이벤트를 찾습니다.
        
    - 찾은 이벤트를 `coupon-delay-processed` 토픽으로 `forward` (전달)합니다.
        
    - `StateStore`에서 해당 이벤트를 삭제합니다.
        
3. **최종 처리 (Consumer):** `CouponDelayConsumer`가 `coupon-delay-processed` 토픽을 구독하여 실제 알림 로직 수행 (기존과 동일)
---

대략적인 흐름이다.

