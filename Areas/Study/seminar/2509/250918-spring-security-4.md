준원님 발표
스프링 시큐리티 - 인증 프로세스

저번 질문 답변!

HTTP 던질 때 패킷으로 쪼개진다.
네트워크 요청이 이 패킷들을 받아서 원래 HTTP 요청으로 조립한다.
하나로 완성되면 그걸 인식해서 서블릿한테 보내주게 된다.
그래서 한번만 HTTP 요청을 받는다고 한다.

> http 요청을 별개로 인식하게 되면 다 다르게 인증해주겟구민

저번 시간 리마인드

---


필터
서블릿 필터는 웹 앱에서 클라 요청과 서버의 응답을 가공하거나 검사 시 사용되는 구성 요소
서블릿 필터는 클라 요청이 서블릿 도달 전이나 서블릿 응답을 클라에게 본기 전에 특정 작업을 수행 가능
서블릿 필터는 서블릿 컨테이너(WAS)에서 생성, 실행, 종료

DelegatingFilterProxy
스프링에서 사용되는 특별한 서블릿 필터


FilterChainProxy
springSecurityFilterChain의 이름으로 생성되는 필터 빈
내부적으로 하나 이상 시큐리티 필터 체인 객체 가짐
HttpSecurity를 통해 API 추가 시 관련 필터 추가

요청 URL 정보 기준 적절한 시큐리티 필터 체인을 선택한다..?

> 필터 체인은 순서대로 만들어지는 것 아닌가?
> 필터 체인이 여러개가 될 수 있나?
> 어떻게 만들어지는 것인가?


# 사용자 정의 보안 기능 구현

SecurityConfig
@EnableWebSecurity 해주기
이거 해주면 자동 설정으로 인해 안의 클래스들이 초기화, 설정들이 이루어짐.

SecurityBean? 정의, HttpSecurity를 주입받는다고 함.

2가지 일 가능

인증 API, 인가 API

> 필터 체인은 함수형 프로그래밍과 어떤 연관이 있는가?

사용자 추가 설정

스프링 시큐리티는 실행 전 기본으로 사용자 정보를 설정해줄 수 있음.

인증 프로세스
폼, 기본, 기억하기, 익명, 로그아웃, 요청 캐시

폼 인증
http 기반 폼 로그인 인증 매커니즘 활성화 api


만약 인증 안됐을 시 흐름
인증 절차 끝나는 곳에서 예외 발생 시 AccessDeniedException 발생?

어떤 필터가 이걸 캐치해서 ExceptionTranslationFilter?

그다음 엔트리 호출, 로그인 페이지 리디렉션


FormLogin() api
FormLoginConfigurer 설정 클래스를 통해 여러 API들을 설정 가능
내부적으로 UsernamePasswordAuthenticationFilter가 생성되어 폼 방식 인증 처리 담당
이게 인증 필터

시큐리티에서는 인증 인가 설정을 위해 http 시큐리티 객체 사용함.
formLogin에서 람다식으로 httpSecurityFormLoginConfigurer 생성
loginPage, loginProcessingUrl, defaultSuccessURL, failureUrl, usernameParameter, passwordParameter, failureHandler, successHandler
마지막으로 permitAll

> 만약 아이디 비밀번호 인증서 로그인 방식이면 이걸 추가로 받아주는 건 어떻게 하나?
> 이건 커스텀해줘야 하는 것인지?
> 아니면 JSON으로 받아주나?

> 폼 로그인 말고 다른 것도 소개 가능한지?

폼 인증 필터
UsernamePasswordAuthenticationFilter

AbstractAuthenticationProcessingFilter : 사용자 자격증명 인증하는 기본 필터
위 필터를 확장해서 요청 객체에서 제출된 사용자 이름과 비밀번호로부터 인증 수행

인증 프로세스 초기화 시 로그인, 로그아웃 페이지 생성 위한 필터가 초기화 된다.

그래서 attemptAuthentication을 꼭 재정의해야 한다고 함.
이 두 개 초기화하는 메서드이기 때문

RequestMatcher로 필터가 처리하는 흐름도
아주 흥미롭ㅂ따!

인증 성공, 실패는 AuthenticationFailure와 SuccessHandler가 처리해서 예외를 던지지 않음.
> GlobalExceptionHandler로 못 잡는다고 봐야 한다?

---

인증 방식

기본 인증 - HTTP Basic
가장 기본적인 인증 방식
RFC 7235 표준, 인증 프로토콜은 http 인증 헤더에 기술됨.

1. 클라는 인증 정보 없이 서버 접속 시도
2. 서버는 인증 요구를 401, www-authenticate 헤더 기술해 realm(보안 영역)과 basic 인증방법 보냄
3. 클라가 서버로 base64로 username, password 인코딩. authorization 헤더에 담아 요청
4. 성공 시 200

대신 디코딩 가능해서 보안 취약



HttpBasic() API

내부적으로 BasicAuthenticationFilter가 생성되어 기본 인증 방식의 인증 처리 담당

엔트리포인트? 내부적으로 리디렉션해주는 게 있는데 여기선 다르게 처리한다?


BasicAuthenticationFilter를 기본 인증 필터로 사용
기본 인증 서비스 제공 시 사용

요청 헤더에 기록된 인증정보 유효성 체크, 인코딩된 유저네임 패스워드 추출

인증 이후 세션 사용, 미사용 경우에 따라 처리 흐름에 차이가 있다.
세션 사용 시 매 요청마다 인증과정 거치지 않으나 세션 사용하지 않는 경우 매 요청마다 인증과정 거쳐야함.

---

기억하기 인증

rememberMe()

사용자가 웹 사이트나 애플리케이션 로그인 시 자동으로 인증정보 기억
UsernamePasswordAuthenticationFilter와 함께 사용
추상 클래스 그 긴거에서 훅을 통해 구현됨.
인증 성공 시 RememberMeServices가 처리해줌
실패 시 쿠키 지움
LogoutFilter와 연계해 로그아웃 시 쿠키 지움


remember me 토큰 생성
기본적으로 암호화되서 생성
브라우저에 쿠키 보내고, 향후 세션에서 이 쿠키 감지해 자동 로그인 이루어지는 방식으로 달성

> #todo 그럼 토큰 탈취 시 대응은 어떻게 함?
> 세션보다 보안이 취약한 것 같음. 어떻게 감지를 하는지?

RememberMeServices 구현체
TokenBasedRememberMeServices

rememberMe() API

인증 필터
클라이언트에서 요청하면 null 여부 체크한다.
null일 때 autologin이 호출된다.
클라이언트 쿠키와 서버의 쿠키를 서로 비교하고, 최종 인증 토큰을 생성하게 된다.

실제로 쿠키를 만들고 전송해주는건 usernameAuthenticatinoFilter에서 처리해준다.

---

질문 답변 타임

민성님
세션같은 경우 개념 상으로 관리해주어야 하는데, rememberMe는 메모리 상인지? 아니면 데이터 상인지?

폼인증은 HttpSession, Http Basic은 선택 사항?

TokenBasedRememberMeServices
쿠키 기반 토큰 보안 위해 해싱 사용.

그렇다면 정책 상 블랙리스트를 써야하는 경우가 있다. 이 경우 DB 써야하나? 넹 써야 함.


> 그럼 여기서 만들어지는 사용자는 UserDetails로 정의되나?
> 아니면 이걸 상속받는 클래스의 이름으로 임의로 세팅 해줄 수 있나?
> DB까지도 올라올 수 있는지?
> 인증과정에서 생성되지 않을까용.
> DB에는 유저 정보가 없다. 시스템에서 미리 설정해두는 것이니, UserDetail이 미리 생기는 건지는 잘 몰루?


인증서 기반 로그인은 사이트 자체랑 네트워크 안정성을 보장하기 위해 사용함.
아이디/패스워드 기반이 아니라 커넥션 자체의 안정성이라고 봐야 함.

로그인 방식 자체가 되게 다양함.
OAuth2 같은 경우도 있다.
인증서도 검증하는 과정인데... 아예 다르지 않을까염
OAuth2랑 비슷할 것 같긴 함.

GPT삣쀠의 답변
공동인증서(구 공인인증서) 로그인은 말씀하신 **formLogin**이나 **OAuth2** 같은 일반적인 방식과는 달라요.

### 1. 어떤 로그인 방식인가?

- 공동인증서는 기본적으로 **PKI(Public Key Infrastructure, 공개키 기반 구조)** 인증을 사용합니다.
    
- 사용자가 로그인 시 인증서를 선택하고, 인증서의 개인키(Private Key)로 특정 데이터를 전자서명합니다.
    
- 서버(또는 인증 모듈)는 사용자의 공개키(Public Key)와 루트 CA를 통해 이 전자서명이 유효한지 확인합니다.
    
- 즉, **ID/PW 대신 "전자서명 검증"으로 본인임을 증명**하는 구조입니다.
    

### 2. Form Login, OAuth2와의 차이

- **Form Login**: 단순히 사용자가 입력한 아이디/비밀번호를 DB와 비교.
    
- **OAuth2**: 외부 인증 제공자(구글, 카카오, 네이버 등)의 토큰을 받아 인증.
    
- **공동인증서 로그인**: 비밀번호 대신 **개인키로 생성한 서명값을 서버가 검증** → PKI 기반 신뢰 체인에 의존.
    

즉, 기존 웹 로그인과는 완전히 다른 보안 모델이고, 금융/정부 서비스에서 법적 효력을 갖기 때문에 많이 사용됩니다.

### 3. 구현 방식

- Spring Security 같은 프레임워크가 바로 제공하지는 않아요.
    
- 보통은 다음과 같은 흐름으로 **커스텀 인증 필터/Provider**를 만듭니다:
    
    1. 클라이언트에서 인증서 선택 후 Challenge(난수 등)에 개인키로 서명
        
    2. 서명값과 인증서 체인을 서버로 전달
        
    3. 서버는 공개키/루트 CA 체인을 통해 서명 검증
        
    4. 성공 시 인증 객체(SecurityContextHolder)에 사용자 정보를 넣음
        

즉, **커스텀 AuthenticationProvider**를 작성해서 FormLogin 자리에 붙이는 식으로 확장하는 경우가 많습니다.

---

정리하면:  
공동인증서 로그인은 **PKI 기반 전자서명 인증**이고, Form Login이나 OAuth2와는 다른 방식이에요. Spring Security에서는 따로 제공하지 않으므로 보통 **커스텀 인증 처리**를 구현해서 붙입니다.

혹시 원하시면, Spring Security에 "공동인증서 로그인 Provider"를 붙이는 예시 구조를 그려드릴까요?