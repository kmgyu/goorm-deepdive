# ExceptionHandling

## 예외 처리 유형

AuthenticationException
1. SecurityContext에서 인증 정보 삭제
2. AuthenticationEntryPoint 호출
3. 인증 프로세스 요청 정보를 저장하고 검색

AccessDeniedException
- AccessDeniedHandler 호출


> 언제 발생하는 Exception임?


exceptionHandling() API
기본적으로 제공된다.
설정 안해주면 http 설정에 따라 기본 제공 클래스에 맞게끔 실행시켜줌.
AuthenticationEntryPoint가 각각 따로따로 제공됨.
AuthenticationFilter에 따라 다른 EntryPoint가 사용됨!

커스텀해서 사용해줄 수도 있다.

ExceptionalTranslationFilter

줄여서 ETF
순서상으로는 ETF 먼저거치고 AuthorizationFilter를 거치게 된다.
흐름 상으로는 AuthorizationFilter 밑에 그려진다?
근데 이거 다운 플로우? 라고 해서 순서는 ETF가 먼저라고 함.
> 잘못들은 건가? Authorization

인가 필터이기 때문에 인증은 완료되었으므로 인증된 사용자에게 AccessDeniedException을 던짐
익명인지 기억하기 사용자인지 판별하고 다시 로그인하라고 하거나 인증은 되었는데 권한이 없는 경우, AcessDeniedHanlder를 통해 리디렉션 시켜줌.

필터도 보면 doFilter로 던지기만 하고 아무것도 안한다.

중간에 인터셉터가 들어올 수 있나?
5분 이후 들어오세요, 다음에 시도하세요 같은 것이라던가 부적절한 사용자라던가?
그것은 인증에서 하게 된다.
provider에서 검증 처리를 하게 되는데 