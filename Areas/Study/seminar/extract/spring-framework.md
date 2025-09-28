# Spring Framework

## 빈 관리 및 생명주기

### 빈 스코프
- **싱글톤**: 기본 스코프, 하나의 인스턴스만 생성
- **프로토타입**: 요청할 때마다 새로운 인스턴스 생성
- **웹 스코프**: request, session, application, websocket

### 빈 생명주기 콜백
- 초기화 콜백: 객체나 시스템이 초기화 과정을 완료한 후 실행되는 함수나 메서드
- 객체 초기화: 객체가 생성되고 필요한 속성들이 설정된 후, 추가적인 설정이나 검증 작업이 필요한 경우
- 시스템 초기화: 시스템이나 애플리케이션이 시작될 때, 필요한 설정 파일 로딩, 데이터베이스 연결, 초기 데이터 로드 등의 작업

### 빈 등록 방식
- **@Component 계열**: 컴포넌트 스캔을 통해서 자동으로 스프링 빈으로 등록
- **@Bean**: 명시적으로 빈을 등록하는 방법, 보통 @Configuration이 붙은 설정 클래스 안에서 사용

### 의존성 주입
- **@Autowired**: 의존성 있는 빈들끼리 연결
- **자동-자동**: 충돌 발생
- **수동-자동**: 수동이 자동을 오버라이딩

### 빈 스코프 처리 방식
- **Dependency Lookup**: getBean()을 통한 빈 조회
- **ObjectProvider, ObjectFactory**: 간단한 DL 인터페이스
- **Provider**: JSR-330 표준, lazy 생성을 가능하게 함

### Request Scope 처리
- HTTP 요청 처리 시, DispatcherServlet이 요청을 받아 ThreadLocal 같은 요청 전용 저장소에 정보를 넣음
- @Scope 빈은 내부적으로 RequestContextHolder라는 객체에 의존
- ThreadLocal은 쓰레드별 다른 저장 공간을 가짐

## Spring Boot

### 애플리케이션 시작 과정
- **register-shutdown-hook = true**: 가비지 컬렉팅하고 종료
- **configureHeadlessProperty()**: GUI 없이 서버 모드 실행
- **prepareContext**: 모든 세팅을 준비하고 완료하는 단계
  - 순환참조 체크, 허용할지 덫어쓸지 등
- **refreshContext**: 빈 생성, DI, 톰캣 실행 등

### 핫 리로드
- 처음 env 세팅할 때 소위 말하는 .env와 동일
- 수정된 파일 로드해서 서버 껐다 키는 것

## Spring MVC

### MVC 패턴
- 어댑터 패턴 활용
- 110V, 220V 같은 호환 안되는 것들을 연결해주는 패턴
- RestController나 media type, 다양한 프로토콜 사용 시 진가 발휘

### DispatcherServlet
- 프론트 컨트롤러 역할
- 핸들러 매핑, 핸들러 어댑터 등을 통해 요청 처리

## Spring Data JPA

### Repository 인터페이스
- Spring Data JPA가 런타임에 프록시 기반 실제 구현 클래스를 생성해주고, 인터페이스에 주입
- @Repository 생략 가능 (JpaRepository 때문)
- 컴포넌트 스캔을 스프링 데이터 JPA가 자동으로 처리
- JPA 예외를 스프링 예외로 변환하는 과정도 자동 처리

### Repository 계층 구조
- **Repository**: 최상위 부모
- **CrudRepository**: CRUD 메소드 정의
- **PagingAndSortingRepository**: 페이징이랑 정렬 기능 지원
- **JpaRepository**: 더 특화된 메소드 제공 (findAll, saveAll 등)

### 지원하는 쿼리 메소드 기능
1. **메소드 이름으로 쿼리 생성**: JPQL 같은 거 생각할 필요 없이 이름대로 써주면 알아서 실행
2. **메소드 이름으로 JPA NamedQuery 호출**: @NamedQuery 어노테이션 사용
3. **@Query 어노테이션**: 리포지토리 메소드에 쿼리 직접 정의

### 엔티티 관리
- **getter**: 열어둘 일이 많아서 상관없음
- **setter**: 데이터 변하기 때문에 열어두면 왜 변했는지 추적 감지가 힘듦
- 변경 지점이 명확하도록 비즈니스 메소드를 만들어야 함

### 준영속 컨텍스트
- 더이상 관리하지 않는 상태
- 관리하는 방법: 변경감지와 병합
- 트랜잭션 커밋 시점에 변경되도록 업데이트를 날림

### OSIV (Open Session In View)
- **켜져있는 게 기본 동작**: 요청 시점부터 렌더링 종료시점까지 영속성 컨텍스트 살아있음
- 인터셉터-컨트롤러-뷰-인터셉터 단은 수정 불가능
- 서비스-리포지토리 단은 수정 가능
- **꺼져있으면**: 컨트롤러단 준영속 상태, 트랜잭션 범위에서 영속 상태
- 모든 지연로딩을 트랜잭션 안에서 처리해야 하는 단점

## Spring Cloud

### 클라이언트 사이드 로드 밸런싱
- 클라이언트가 직접 서브 인스턴스 중 하나를 선택해 요청을 보내는 방식
- Netflix Ribbon 같은 것

### 서킷 브레이커
- 서비스 호출 실패가 일정 횟수 이상 발생 시 열림
- 일정 시간 후 일부 요청만 테스트 - 정상일 시 닫힘
- 일부 요청 실패 시 서킷 반 열림

### 대표적 구현체
- **Spring Cloud Netflix**: Netflix OSS를 스프링 클라우드 프로젝트와 통합
- **Eureka**: 서비스 등록, 검색 가능
- **Ribbon**: 클라이언트 사이드 로드밸런서
- **Hystrix**: 서킷 브레이커
- **Spring Cloud Config**: 설정 관리
- **Spring Cloud Stream**: 메시지 브로커를 사용하여 MSA 비동기통신

### API Gateway
- **보안**: 서버 결합성 떨어트려서 추상화 시키는 게 좋음
- **로드 밸런싱**: 가능
- **리버스 프록시 역할**: 서버 대신 해줘서 리버스 프록시가 됨
- **요청 검증**: URL에 대한 검증도 해줌

## Filter, Interceptor, AOP

### Filter
- 서블릿 컨테이너 레벨에서 작동
- HTTP 요청, 응답을 가로채 전처리/후처리
- 스프링 없이도 사용 가능
- DispatcherServlet 도달 전 시행
- **afterCompletion**: 뷰 렌더링이 끝난 후 시행

### Interceptor
- 스프링에 종속적
- DispatcherServlet, Controller 사이에서 동작
- 핸들러매핑에 의해 결정된 특정 컨트롤러에만 적용가능
- DI 활용 가능
- **preHandle**: 호출 전 체크, return true로 해야 컨트롤러 호출을 계속
- **postHandle**: 뷰 렌더링 후 실행

### AOP
- 반복되는 부가기능을 한 곳에 모아 핵심코드와 분리해주는 프로그래밍 기법
- 공통 관심사 분리가 목적인 패러다임
- **Aspect**: 공통 관심사를 가진 모듈
- **JoinPoint**: 관심사가 실행되는 영역
- **PointCut**: 어떤 위치의 코드에서 실행할 지 정의
- **Advice**: 언제 어떤 부가기능 실행할 지 정의
- **Weaving**: aspect를 실제 코드에 합치는 과정

### Advice 어노테이션
- @Before
- @After
- @AfterReturning
- @AfterThrowing
- @Around (메소드 실행 전/후, 가장 강력함)

## GraphQL

### 사용법
- Controller에서 QueryMapping, MutationMapping 등이 가능
- JS에서 fetch를 하는데, 기본적으로 `~/graphql`이라는 url이 자동으로 등록
- 쿼리를 작성하고 바디에 넣어줌
- 뮤테이션이 생성, 수정, 삭제 기능을 갖고있음

## ResponseEntity
- 리턴 데이터 전부 커스텀할 수 있게 해주는 것
- Controller vs RestController 할 때 DTO와는 다른 개념

## String Intern Pool
- "a" == "a", "a".equals(new String("b"))
- 리터럴 쓰면 힙메모리에 저장해서 관리시켜주는 것
- "a" + "b" 해주면 immutable이라 재생성해줌
- String은 불변 객체 (String pool 때문)
- Wrapper Class는 모두 불변 객체

## DNS
- 사실 오른쪽부터 도메인을 읽음
- 루트 도메인부터 3~4단계 도메인까지
- **FQDN**: 전체 주소 도메인 네임
- **Local Name Server**: ISP 같은 곳에 먼저 물어봄
- **Root Name Server**: 루트 도메인을 담당하는 서버
- **TLD Name Server**: TLD 도메인을 담당하는 서버
- **Authoritative Name Server**: 로컬 네임 서버가 마지막으로 질문하는 서버

### DNS 자원 레코드
- 이름, 값, TTL, 레코드 타입
- 레코드 유형: A, AAAA, CNAME, NS, MX 등

## Swagger
- OpenAPI Specification OAS
- @Tag, @Operation 같은 걸로 API docs 세팅 가능
- DTO에 스키마도 붙여주면 기본값이나 필드 정보 같은 것 넣어줄 수 있음
- JWT 인증 필요 시 Authorize 같은 거 가능
- 커스터마이징이나 환경 별 분리도 가능

## Ngrok
- 로컬에서 실행 중인 서버를 인터넷에 임시로 뿌려줌
- 터널링 도구
- TLS 터널링 사용
- 고급 옵션으로 서브도메인도 지정 가능

## 가상 스레드
- 멀티 패러다임 적용 시 복잡해짐
- 웹플럭스, 코루틴 같은 것들
- CPU 바운드 같은 IO 작업이 많이 없는 작업의 경우, 기존 스레드와 동일한 작업이 됨
- 기존에 스택 포인터를 이용해서 PCB로 바꾸고 복원하고 컨텍스트 스위칭으로 메모리를 자꾸 바꿔주는 건데 메모리 안에서 그냥 빨리빨리 바꾸다 보니 더 빨라짐
- ThreadLocal을 사용하게 될 때 값이 공유될 수 있어서 제대로 지정해주지 않으면 문제가 발생할 수 있음

## ForkJoin
- 분할 정복에 대한 것과 Work Stealing을 지원하기 때문에 노는 스레드를 줄일 수 있음
- Runnable이나 Callable의 작업 쪼개기 자체는 개발자가 직접 해줄 수 있으나 프레임워크로 이런 멀티 쓰레딩 환경을 지원해주는 것이 핵심

## #todo
- Spring Security의 세부적인 인증/인가 흐름
- Spring Cloud의 고급 기능들
- Spring Boot의 자동 설정 메커니즘
- Spring AOP의 프록시 생성 방식
- Spring Data JPA의 커스텀 리포지토리 구현
