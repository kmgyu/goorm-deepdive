
핵심 최소 구성

1. UserRepository
    - JpaRepository<User, Long>
    - findByUsername(String username) 필수
        
2. CustomUserDetails
    - org.springframework.security.core.userdetails.UserDetails 구현
    - 내부에 User 보관, getUsername()/getPassword() 매핑
    - 권한은 당장 없으면 빈 컬렉션 혹은 기본 ROLE_USER 반환
        
3. CustomUserDetailsService
    - org.springframework.security.core.userdetails.UserDetailsService 구현
    - loadUserByUsername에서 UserRepository로 조회 → CustomUserDetails 리턴
        
4. SecurityConfig
    - SecurityFilterChain 빈
    - authorizeHttpRequests, formLogin, logout, csrf, sessionManagement 설정
    - PasswordEncoder 빈(BCryptPasswordEncoder)
    - 필요 시 AuthenticationManager 설정
        
5. LoginView(선택)
    - 커스텀 로그인 화면 쓸 거면 templates/login.html 정도
    - 아니면 스프링 기본 /login 사용
        
6. 데이터 초기화(선택)
    - CommandLineRunner 등으로 테스트 유저 1명 생성(비밀번호는 BCrypt로 인코딩)
        

옵션 구성(상황 따라 채택)  
A) 권한/롤을 DB로 관리하고 싶다면
- User 엔티티는 안 건드리면서 별도 테이블로 가능
- Role(id, name), UserRole(user_id, role_id) 엔티티 추가
- CustomUserDetails에서 UserRole 조회해 GrantedAuthority 매핑
- 나중에 중간 관리자 권한도 여기서 확장
    

B) Spring Session으로 세션 외부화
- 메모리 세션 대신 Redis/JDBC 사용하려면 Spring Session 의존성
- Redis면 @EnableRedisHttpSession, JDBC면 @EnableJdbcHttpSession
- JDBC는 SPRING_SESSION*, Redis는 key-space 자동 관리
    

C) 핸들러 커스터마이징
- AuthenticationSuccessHandler, AuthenticationFailureHandler
- LogoutSuccessHandler
- AccessDeniedHandler, AuthenticationEntryPoint
    

D) CSRF 토큰 전략
- 서버 렌더링 폼이면 기본 동작으로 충분
- SPA/AJAX 섞이면 CookieCsrfTokenRepository 등 설정