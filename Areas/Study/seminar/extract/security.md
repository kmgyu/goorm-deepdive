# Security

## Spring Security 기본 개념

### 인증과 인가
- **인증 (Authentication)**: "당신 누구야!" 하는 과정, 신원 확인
- **인가 (Authorization)**: 인증으로 당신 누구야! 하는 과정 후 권한 부여

### 인증 아키텍처
1. **Authentication**: 인증 프로세스 할 때 가장 먼저 인증 필터가 객체 만들어 사용자 인증 시 아이디 비밀번호를 저장해서 담는 토큰 객체
2. **보안 컨텍스트**: 인증 객체를 최종 성공한 인증 객체를 저장하고 인증 상태를 유지
3. **인증 관리자**: 인증 필터가 인증 시도 시 가장 먼저 인증처리를 맡김, 관리
4. **인증 제공자**: AuthenticationManager로부터 인증 처리를 다시 위임받음, 실행 총괄
5. **사용자 상세 서비스**: 이 클래스를 사용해 사용자 정보를 가져올 수 있음
6. **사용자 상세**: 유저 객체를 사용할 수 있음

### Authentication 객체
- 인증 정보를 저장하는 토큰 개념 객체로 활용됨
- SecurityContext를 통해 전역적으로 참조 가능
- SecurityContextHolder가 static으로 되어있어서 가능
- Principal을 상속받음
- 메서드들은 자격 증명, 인증 주체, 추가적인 세부사항 등의 getter
- 인증 상태 가져오는 것(isAuthenticated)과 인증 상태 설정하는 setter가 있음

### 인증 절차 흐름
AuthenticationFilter → Authentication → AuthenticationManager → Authentication

그리고 다시 역순으로 필터로 올라가서 시큐리티 컨텍스트 홀더가 가져감

### SecurityContext
- Authentication 저장
- ThreadLocal 저장소 사용
- 애플리케이션 전반에 걸친 접근성

### SecurityContextHolder
- 전략 패턴 사용
- SecurityContextHolderStrategy 인터페이스를 사용
- **MODE_THREADLOCAL**: 기본 전략
- **MODE_INHERITABLETHREADLOCAL**: 부모 스레드로부터 자식 스레드로 보안 컨텍스트 상속하며 스레드 간 분산 실행하는 경우 유용
- **MODE_GLOBAL**: 전역적으로 단일 보안 컨텍스트 사용, 서버 환경에서 부적합

### SecurityContext 참조 및 삭제
- **참조**: `SecurityContextHolder.getContextHolderStrategy().getContext()`
- **삭제**: `closeContext()`
- 각 앱마다 다른 전략을 줘서 사용하기 때문에 버전업할 때 getter로 전략을 가져오도록 변경
- 스레드 풀에서 운용되는 스레드일 경우 새로운 요청이더라도 기존 스레드 로컬이 재사용될 수 있기 때문에 클라이언트로 응답 직전에 항상 시큐리티 컨텍스트를 삭제해줌
- 각 스레드는 자신만의 독립적인 시큐리티컨텍스트를 가지게 되어 동시에 여러 요청이 들어와도 인증 정보가 섞이지 않게 됨

### AuthenticationManager
- Authentication 객체 받아서 인증 절차 시켜주고, 완전히 채워진 Authentication을 생성해 반환
- AuthenticationProvider 목록 중 인증 처리 요건에 맞는 Provider를 찾아 인증 처리 위임

### AuthenticationProvider
- 사용자 자격증명 확인, 인증 과정 관리
- 시스템에 액세스하기 위해 제공한 정보가 유효한지 검증과정 포함
- 성공적인 인증 후 Authentication 객체 반환, 신원 정보와 인증된 자격 증명을 포함
- 문제 발생 시 AuthenticationException과 같은 예외 발생
- `authenticate()`를 통해 사용자 유무 검증, 비밀번호 검증, 보안 강화 처리 등을 수행

#### 사용방법
- 일반 객체로 생성하는 것
- 빈으로 생성하는 것
- 빈을 한 개만 정의 시, DaoAuthenticationProvider를 자동으로 대체
- 두 개 이상 정의 시, 어...

### UserDetailsService
- 사용자와 관련된 상세 데이터 로드
- AuthenticationProvider에서 주로 사용
- 시스템에 존재하는지 여부와 사용자데이터 검색, 인증 과정을 수행
- 사용자 이름을 통해 사용자 데이터를 검색, 해당 데이터를 UserDetails 객체로 반환
- securityFilterChain에서 등록해서 사용 가능

### UserDetails
- 스프링 시큐리티에서 사용하는 사용자 타입
- DB에서 끌고 온 사용자 정보인데, 객체 타입이 개발자, 프로젝트마다 다르니 시큐리티가 인식할 수 있게 인터페이스 형태로 제공해주는 것
- 구현체로서 User 클래스가 제공됨
- 저장된 사용자 정보는 추후 인증 절차에서 사용되기 위해 Authentication 객체에 포함됨

## Spring Security 설정

### SecurityBuilder / SecurityConfigurer
- Spring Security는 인증, 인가 등 여러가지 함
- 위에 두개는 여러가지 객체 초기화, 설정들을 종합적으로 처리
- 이 두 개를 이해하면 전체적인 Spring Security를 알 수 있음

### SecurityBuilder
- 웹 보안 구성 빈 객체, 설정 클래스 생성 역할을 하는 빌더 클래스
- Builder 패턴 적용한 클래스들이라 빌더 클래스
- **WebSecurity**가 대표적인 구현체
- **HttpSecurity**라는 Http 요청에 대한 세부 보안 정책을 설정하는 것도 생성 (이거 가장 중요)
- 쉽게 보안 설정 만드는 설계도 역할

### SecurityConfigurer
- 세부 설정을 추가하는 역할
- Http 요청과 관련된 보안 처리 담당 필터들을 생성, 열 초기화 설정에 관여
- 스프링 시큐리티는 모든 인증 인가 요청을 필터를 통해 처리
- 그래서 필터 기반 보안 프레임워크라 해도 과언이 아닐 정도

### HttpSecurity
- HttpSecurityConfiguration에서 HttpSecurity를 생성하고 초기화를 진행
- HttpSecurity는 보안에 필요한 각 설정 클래스, 필터 생성
- 최종적으로 빌드를 통해 SecurityFilterChain 빈 생성
- 필터들이 체인 형식으로 연결되어 있음
- **DefaultSecurityFilterChain**이 기본 구현체
- requestMatcher와 filters를 가짐
- filters는 list 컬렉션

### SecurityFilterChain
- 필터 목록을 관리
- **matches()**: SecurityFilterChain에 의해 처리되어야 하는지 여부 결정
- **getFilters()**: Filter 객체의 리스트 반환
- 만약 matches에서 false 뜨면 다른 체인으로 넘어감

### WebSecurity
- WebSecurityConfiguration에서 생성, 초기화
- HttpSecurity에서 생성한 SecurityFilterChain 빈을 SecurityBuilder에 저장
- build() 실행 시 SecurityBuilder에서 SecurityFilterChain 꺼내 FilterChainProxy 생성자에 전달
- 최종적으로 FilterChainProxy를 만드는 작업

### DelegatingFilterProxy
- 스프링에서 사용되는 특별한 서블릿 필터
- 스프링 앱 컨텍스트 간의 연결고리 역할
- 서블릿 필터 기능 수행하는 동시에 스프링 DI 및 빈 관리 기능과 연동되도록 설계된 필터
- springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청을 위임
- 실제 보안 처리를 수행하지는 않음
- 서블릿 컨테이너(WAS), IOC 컨테이너 따로따로 있는데, DelegatingFilterProxy가 서블릿 컨테이너에서 스프링 IOC 컨테이너 쪽의 스프링 빈 springSecurityFilterChain에 요청을 위임

### FilterChainProxy
- springSecurityFilterChain의 이름으로 생성되는 필터 빈
- 내부적으로 하나 이상 시큐리티 필터 체인 객체 가짐
- HttpSecurity를 통해 API 추가 시 관련 필터 추가
- 요청 URL 정보 기준 적절한 시큐리티 필터 체인을 선택

## 인증 방식

### 폼 인증
- HTTP 기반 폼 로그인 인증 매커니즘 활성화 API
- **FormLoginConfigurer** 설정 클래스를 통해 여러 API들을 설정 가능
- 내부적으로 **UsernamePasswordAuthenticationFilter**가 생성되어 폼 방식 인증 처리 담당

#### FormLogin() API 설정
- loginPage, loginProcessingUrl, defaultSuccessURL, failureUrl, usernameParameter, passwordParameter, failureHandler, successHandler
- 마지막으로 permitAll

#### UsernamePasswordAuthenticationFilter
- AbstractAuthenticationProcessingFilter를 확장해서 요청 객체에서 제출된 사용자 이름과 비밀번호로부터 인증 수행
- 인증 프로세스 초기화 시 로그인, 로그아웃 페이지 생성 위한 필터가 초기화됨
- `attemptAuthentication`을 꼭 재정의해야 함 (이 두 개 초기화하는 메서드이기 때문)

### 기본 인증 (HTTP Basic)
- 가장 기본적인 인증 방식
- RFC 7235 표준, 인증 프로토콜은 http 인증 헤더에 기술됨

#### 동작 과정
1. 클라는 인증 정보 없이 서버 접속 시도
2. 서버는 인증 요구를 401, www-authenticate 헤더 기술해 realm(보안 영역)과 basic 인증방법 보냄
3. 클라가 서버로 base64로 username, password 인코딩, authorization 헤더에 담아 요청
4. 성공 시 200

#### 단점
- 디코딩 가능해서 보안 취약

#### HttpBasic() API
- 내부적으로 **BasicAuthenticationFilter**가 생성되어 기본 인증 방식의 인증 처리 담당
- 요청 헤더에 기록된 인증정보 유효성 체크, 인코딩된 유저네임 패스워드 추출
- 인증 이후 세션 사용, 미사용 경우에 따라 처리 흐름에 차이가 있음
- 세션 사용 시 매 요청마다 인증과정 거치지 않으나 세션 사용하지 않는 경우 매 요청마다 인증과정 거쳐야 함

### 기억하기 인증 (Remember Me)
- `rememberMe()` API 사용
- 사용자가 웹 사이트나 애플리케이션 로그인 시 자동으로 인증정보 기억
- UsernamePasswordAuthenticationFilter와 함께 사용
- 추상 클래스에서 훅을 통해 구현됨
- 인증 성공 시 RememberMeServices가 처리해줌
- 실패 시 쿠키 지움
- LogoutFilter와 연계해 로그아웃 시 쿠키 지움

#### Remember Me 토큰 생성
- 기본적으로 암호화되서 생성
- 브라우저에 쿠키 보내고, 향후 세션에서 이 쿠키 감지해 자동 로그인 이루어지는 방식으로 달성

#### RememberMeServices 구현체
- **TokenBasedRememberMeServices**

#### 인증 필터
- 클라이언트에서 요청하면 null 여부 체크함
- null일 때 autologin이 호출됨
- 클라이언트 쿠키와 서버의 쿠키를 서로 비교하고, 최종 인증 토큰을 생성하게 됨
- 실제로 쿠키를 만들고 전송해주는 건 usernameAuthenticationFilter에서 처리해줌

## OAuth2

### 정의
- Open Authorization
- 제3자 인증 제공자를 통해 프로필 정보, 이메일 등에 대한 접근 권한 위임 받을 수 있도록 함

### 주요 요소
- **Access Token**: API 요청하기 위한 인증키, 이걸 통해 SNS 로그인한 사용자 정보 조회 요청
- **인가 코드**: 액세스 토큰 발급받기 위한 키값, 화면 제어권이 제3자로 넘어가서 redirect 형식으로 사용자 정보 받아와야 함
- **스코프**: 제3자 서버로부터 받아올 수 있는 이메일, 프로필 정보 등 사용자 정보의 범위

### 구현 방식
1. **프론트 인가코드 받고 서버에서 액세스코드 발급**
   - 장점: 사용자가 서버로부터 JWT 토큰 받을 때 안전하게 바디를 통해 값 받기 가능, 직관적이고 문제 생긴 위치 알 수 있음
   - 단점: 보안

2. **서버에서 인가코드 및 액세스 코드 발급 방식**
   - 장점: 요청이 한꺼번에 진행되어 구현 간결, 서버측에서 관리하므로 안전하게 보관
   - 단점: 사용자에게 JWT 토큰 줄 때 리디렉션 방식 취할 수 밖에 없어 보안상 취약점 존재, 모든 절차가 라이브러리를 통해 통합되어 있고 자동화가 되어 있어 코드파악의 어려움과 디버깅의 어려움 존재

## JWT (JSON Web Token)

### 구조
- **Header**: 알고리즘, 토큰 타입
- **Payload**: 데이터 들어감
- **Signature**: 헤더와 서버가 가진 비밀키로 암호화
- 헤더와 서버는 `.`을 기준으로 분리됨
- 헤더, 페이로드, 시그니쳐 또한 `.`을 기준으로 분리됨

### 특징
- 별도 세션을 지정하지 않아도 되며, 무상태 인증 방식이라 확장에 유리
- 필요 정보를 담고 있어 인증 시 빠르게 처리할 수 있음

## 예외 처리

### 인증 안됐을 시 흐름
- 인증 절차 끝나는 곳에서 예외 발생 시 AccessDeniedException 발생
- ExceptionTranslationFilter가 이걸 캐치
- 그다음 엔트리 호출, 로그인 페이지 리디렉션

### 인증 성공, 실패 처리
- AuthenticationFailure와 SuccessHandler가 처리해서 예외를 던지지 않음
- GlobalExceptionHandler로 못 잡음

## 병목지점
- **Authentication에서 DB 조회가 일어남**
- DB 접근만으로 병목현상
- 기존 ID 패스워드를 가져와서 검사
- OTP나 외부 서비스 같은 것들, 검증하는 것들 워커들이 있을 건데 통신으로 인한 소비가 있음
- 패스워드 해싱 후 검증과정에서도 리소스가 필요
- 이런 암호화를 역산(디코딩)하는 과정이 CPU를 많이 함

## HTTP 요청 처리
- HTTP 던질 때 패킷으로 쪼개짐
- 네트워크 요청이 이 패킷들을 받아서 원래 HTTP 요청으로 조립
- 하나로 완성되면 그걸 인식해서 서블릿한테 보내주게 됨
- 그래서 한번만 HTTP 요청을 받음

## #todo
- Spring Security의 커스텀 인증 필터 구현
- Spring Security의 세션 관리 전략
- Spring Security의 CSRF 보호 메커니즘
- OAuth2의 세부적인 인증 플로우
- JWT 토큰의 보안 강화 방법
- Spring Security의 권한 기반 접근 제어
- Spring Security의 암호화 및 해싱 전략
- Spring Security의 로그아웃 처리 메커니즘
