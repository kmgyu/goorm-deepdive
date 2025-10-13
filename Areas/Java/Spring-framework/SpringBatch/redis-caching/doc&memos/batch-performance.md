배치 어플리케이션의 성능을 측정해보자.

부하테스트 도구를 겸사겸사 알아본다.

부하 테스트 및 소스코드
[포스트](https://jaeseo0519.tistory.com/400)
[깃헙](https://github.com/psychology50/spring-batch-performance)


# 측정 도구

## 선정 방식

Jmeter, locust, nGrinder 등이 있다.
차이점 비교~
https://wookkl.tistory.com/67

근데 이놈들은 HTTP 부하 테스트라 적절하지 않음

짜잔~
Micrometer, Spring Actuator, Prometheus 썼습니다~
마이크로미터랑 액추에이터는 같이 딸려오고, 프로메테우스는 도커 세팅

성능 모니터링 도구를 사용한다.

액추에이터 & 마이크로미터
마이크로미터가 성능 측정
액추에이터로 메트릭 전송
그 밖에도 기능 더있다.


프로메테우스

# 데이터 중심 어플리케이션 설계

## 응답 시간

### 응답 시간 고정
어떻게 하는 지 모름

### percentile

p50, p95, p99, p999

https://kscodebase.tistory.com/544
https://ayoung0073.tistory.com/entry/%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A4%91%EC%8B%AC-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%84%A4%EA%B3%84-%EC%9D%91%EB%8B%B5-%EC%8B%9C%EA%B0%84-p50-p95-p99


