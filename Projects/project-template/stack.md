일단 오래오래 쓸 것이므로 레거시가 되면 안된다.
채-신 버전보다 조금 아래있는 버전을 이용해 호환성을 추가로 챙기면 좋지 않을?까???

무근본 스택 정하기

코어
자바 24까지 지원한다지만 호환성 안 맞는 패키지가 나올 것 같기 때문에 조금 연식이 쌓인 21을 선택했다.
스프링 부트는 3.5 이용하게 될 듯.

자바 21, Spring Boot 3.5

빌드
그래들&코틀린 dsl
~~도커~~

그래들로 작성하는 게 보기가 편했다. 작성도... 더 쉬웠음...
코틀린 dsl 이용 시 빌드에서 에러 띄운다고 한다.
인텔리제이에서 지원하는 것과, ide에서 빌드 오류 감지 등이 있으니 그루비보다 편하게 쓸 것 같았음..
https://jinhos-devlog.tistory.com/entry/Spring-Initializr%EC%97%90%EC%84%9C-Gradle-groovy%EC%99%80-Kotlin%EC%9D%98-%EC%B0%A8%EC%9D%B4

도커 이미지로 빌드 하는 것... 추후 작성 필요

DB 및 ORM
JPA & QueryDSL

- JPA(Hibernate 6.x) + QueryDSL. Boot가 호환 버전 관리하므로 버전은 가급적 생략. [Stack Overflow](https://stackoverflow.com/questions/74756871/spring-boot-3-with-querydsl?utm_source=chatgpt.com)
- 마이그레이션은 Flyway/Liquibase 중 하나 필수. prod에선 ddl-auto 끄기.
- 로깅은 p6spy 대신 datasource-proxy도 옵션. p6spy 쓰면 포맷 커스터마이징.
    

1. 보안
- Spring Security 6.x. WebSecurityConfigurerAdapter 없이 SecurityFilterChain 빈 구성. 3.x부터 문법 변경됨. [tothenew.com](https://www.tothenew.com/blog/migrating-to-spring-security-6/?utm_source=chatgpt.com)[Stack Overflow](https://stackoverflow.com/questions/74447778/spring-security-in-spring-boot-3?utm_source=chatgpt.com)
- 인증: stateless JWT. Nimbus JOSE JWT로 서명/검증. 리프레시 토큰은 DB/Redis에 바인딩(블랙리스트/토큰 로테이션).
- CORS/CSRF 정책을 API 성격에 맞춰 분리 프로필로 관리.
    

1. API 문서/테스팅
- springdoc-openapi v2.x + Swagger UI. Boot 3와의 호환 라인은 2.x. 최신 2.8.x BOM 제공. [OpenAPI 3 Library for spring-boot+1](https://springdoc.org/faq.html?utm_source=chatgpt.com)[Stack Overflow](https://stackoverflow.com/questions/74701738/spring-boot-3-springdoc-openapi-ui-doesnt-work?utm_source=chatgpt.com)
- 테스트: JUnit 5, Testcontainers로 DB 통합 테스트 표준화.

1. 관측성
- Micrometer + Prometheus + Grafana. Boot 3.3+는 Prometheus client 1.x 대응 변경점 있음(메트릭 이름 일부 변경 주의). [GitHub](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.3-Release-Notes?utm_source=chatgpt.com)
- OpenTelemetry 트레이싱(otlp-exporter)로 추적.
    
1. 품질/도구
- Spotless(포맷), Checkstyle/PMD 또는 ErrorProne(정적 분석).
- MapStruct(매퍼), Jakarta Validation, ProblemDetail(RFC7807) 기반 전역 예외 응답.