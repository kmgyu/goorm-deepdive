준원님 발표

스프링 시큐리티2

AuthenticationProvider

사용자 자격증명 확인, 인증 과정 관리한다.
시스템에 액세스하기 위해 제공한 정보가 유효한지 검증과정 포함

성공적인 인증 후 Authentication 객체 반환, 신원 정보와 인증된 자격 증명을 포함
문제 발생 시 AuthenticationException과 같은 예외 발생

authentication()을 통해 사용자 유무 검증, 비밀번호 검증, 보안 강화 처리 등을 수행함.

사용방법?
일반 객체로 생성하는 것


빈으로 생성하는 것
빈을 한 개만 정의 시, DaoAuthenticationProvider를 자동으로 대체
두 개 이상 정의 시, 어...


UserDetailsService
사용자와 관련된 상세 데이터 로드
AuthenticationProvider에서 주로 사용.
시스템에 존재하는지 여부와 사용자데이터 검색, 인증 과정을 수행한다.

사용자 이름을 통해 사용자 데이터를 검색, 해당 데이터를 UserDetails 객체로 반환한다.

securityFilterChain에서 등록해서 사용 가능하다?

UserDetails
스프링 시큐리티에서 사용하는 사용자 타입
DB에서 끌고 온 사용자 정보인데, 객체 타입이 개발자, 프로젝트마다 다르니 시큐리티가 인식할 수 있게 인터페이스 형태로 제공해주는 것임.
구현체로서 User 클래스가 제공된다.
저장된 사용자 정보는 추후 인증 절차에서 사용되기 위해 Authentication 객체에 포함된다. -> ?


병목지점의 경우, Authentication vs Authorization 어느 단계에서 가장 클까?
- Authentication에서 DB 조회가 일어난다.

DB 접근만으로 병목현상

기존 ID 패스워드를 가져와서 검사한다? ok
근데 뭐 OTP나 외부 서비스 같은 것들, 검증하는 것들 워커 들이 있을 건데... 통신으로 인한 소비가 있을 것이다.
패스워드 해싱 후 검증과정. 여기서도 리소스가 필요하다.
이런 암호화를 역산(디코딩)하는 과정이 cpu를 많이 한다고 함.

