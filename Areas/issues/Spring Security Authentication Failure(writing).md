Spring Security Filter Chain은 스프링 시큐리티가 로그인에 대한 책임을 가짐으로써 간편하게 구현된다.
그렇다면 로그인 시 로깅이나 에러 메시지는 어떻게 반환해야 할까?
필터체인에서 책임을 가져가서 글로벌 익셉션 핸들러로도 잡아줄 수 없다.
binding result도 사용할 수 없으므로 무용지물이 된다.

이럴 때 사용하는 것이 AuthenticationFailureHandler 
커스텀하여 이를 사용하게 된다.
구현체 종류


---

현재 이슈
- 스프링 시큐리티를 통한 로그인 구현 중 로그인이 되지 않음.
- 로깅할 수 없어 로그인 도중 에러를 확인할 수 없음.

발생 지점
리포지토리 : goorm-spring2
커밋 : d06e18917448f2f1731cd740eaa2d3daaa9de967
> 해당 커밋 이전부터 발생하였으나 해당 커밋 이후를 기준으로 해결 시도 중

로깅이 작동하지 않는 문제 해결
```
""2025-09-15 13:56:11.496 [http-nio-8080-exec-3] WARN  g.m.a.security.LoginFailureHandler - 로그인 실패: email=test@test.com, code=unknown, ex=ProviderNotFoundException
""2025-09-15 13:56:11.823 [http-nio-8080-exec-5] ERROR g.m.handler.GlobalExceptionHandler - Unexpected error occurred
```

레퍼런스
[[Spring Boot] Security ProviderNotFoundException에러 해결](https://studio108.tistory.com/22)

전혀 다른 이슈였지만 예상치 못하게 해결했다.

Provider가 없다는 것은 인증할 수단이 존재하지 않는다는 것인데, 사용자를 찾는 책임은 UserDetailsService로 넘어간다.

이전에 도메인을 추가하면서 MyBatis와 Jpa 프로필을 분리했다.
이 과정에서 구현체를 다르게 구현하는 코드를 작성하였는데 여기서 문제가 발생했다.

MyBatis 프로필을 사용하는 구현체가 UserDetailsService 인터페이스를 구현하지 않은 것인데, 이 때문에 빈 등록과정에서 스프링 시큐리티가 인식하지 못하는 문제가 생긴 것이다.

```
public class MyBatisUserServiceImpl implements UserService, UserDetailsService {
```

여기서 UserDetailsService를 상속받지 않는 문제가 존재하였음. 빈으로 가져오는 과정에서 해당 문제로 인해 UserDetailsService가 인식하지 못한 듯.

그럼에도 문제가 해결되지 않는다.

```
"2025-09-15 14:04:41.217 [http-nio-8080-exec-5] WARN  o.s.w.s.m.m.a.ExceptionHandlerExceptionResolver - Resolved [org.springframework.web.servlet.resource.NoResourceFoundException: No static resource favicon.ico.]
""2025-09-15 14:04:44.501 [http-nio-8080-exec-3] WARN  g.m.a.security.LoginFailureHandler - 로그인 실패: email=test@test.com, code=unknown, ex=ProviderNotFoundException
""2025-09-15 14:04:44.761 [http-nio-8080-exec-2] ERROR g.m.handler.GlobalExceptionHandler - Unexpected error occurred
```

추가적으로 다음과 같은 로그를 확인함.
```
""2025-09-15 14:04:28.174 [restartedMain] WARN  o.s.s.c.a.a.c.InitializeUserDetailsBeanManagerConfigurer$InitializeUserDetailsManagerConfigurer - Found 2 UserDetailsService beans, with names [userServiceMyBatis, userServiceJpa]. Global Authentication Manager will not use a UserDetailsService for username/password login. Consider publishing a single UserDetailsService bean.
""2025-09-15 14:04:28.861 [restartedMain] DEBUG o.m.s.b.a.MybatisAutoConfiguration - Not found configuration for registering mapper bean using @MapperScan, MapperFactoryBean and MapperScannerConfigurer.
```

드디어 문제점을 찾았다.
빈 등록 우선순위를 잊어두고 있었다.

처음에는 @Primary를 사용해봤는데 어노테이션을 다시 살펴보니 jpa 구현체에 프로필 지정을 안해두었다.
두 빈을 모두 구현하게 되기 때문에 충돌이 나는 단순한 문제였다...

@Profile("jpa")를 추가하여 해결했다.
![[Pasted image 20250915141805.png]]