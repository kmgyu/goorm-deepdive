
https://innovation123.tistory.com/273#Spring%20Redis%20%EC%BA%90%EC%8B%B1%20%EA%B3%BC%EC%A0%95-1-3


# 기본 캐시 적용
---

레디스를 이용해 캐싱을 적용해준다.

그런데 레디스는 메서드에 @Cacheable 같은 것이 있어야만 적용할 수 있다.
그래서 추가로 찾아보니, Security를 위한 종속성이 따로 존재한다.

![[spring-redis-apply1.png]]
추가해주고, SessionConfig라는 것을 통해 세션 저장소를 설정해줄 수 있다.
나머지는 깃헙에서...


캐싱 / 캐싱 X 비교해보자.

캐싱X
![[spring-redis-apply2.png]]

캐싱
![[spring-redis-apply3.png]]

localhost에서 캐싱해주는 속도가 조금 빨라졌다.
대량의 트래픽이 발생할 때 더 효과적일 것이다.


> 사실 이건 틀렸다.
> 로그아웃해서 세션이 만료(...)되었기 때문에 비교하는 것이 잘못된 것.

한 개의 세션을 다른 곳에서 접속하는 식으로 시도해 보았다.
(이거에 대한 대응을 안해주었기 때문)

![[spring-redis-apply4.png]]





# Cache Unavailable
---


```java
@Bean  
public RedisSerializer<Object> springSessionDefaultRedisSerializer() {  
  return new GenericJackson2JsonRedisSerializer();  
}
```

직렬화를 위해 캐시를 JSON으로 바꿔주는 걸 사용했다.
이걸 이용할 때 어플리케이션 간의 호환성을 늘릴 수 있대서... 사용했는데..

```
Message Could not read JSON:Cannot construct instance of `org.springframework.security.authentication.UsernamePasswordAuthenticationToken` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
```

이런 문제가 생겼다.

그래서 아래와 같이 objectMapper를 통해 직렬화/역직렬화 가능하도록 해보았다.

```java
@Bean  
public RedisSerializer<Object> springSessionDefaultRedisSerializer() {  
  return new GenericJackson2JsonRedisSerializer(objectMapper());  
}  
  
private ObjectMapper objectMapper() {  
  ObjectMapper mapper = new ObjectMapper();  
  
  // 1. Spring Security가 사용하는 클래스(UsernamePasswordAuthenticationToken 등)를  
  //    직렬화/역직렬화할 수 있도록 Jackson 모듈을 등록합니다.  
  //    SecurityJackson2Modules.getModules(getClass().getClassLoader())로 모듈 목록을 가져옵니다.  
  mapper.registerModules(SecurityJackson2Modules.getModules(AbstractHttpSessionApplicationInitializer.class.getClassLoader()));  
  
  // 2. 다른 Java 8 날짜/시간 타입 등을 지원하기 위해 기본 모듈도 추가할 수 있습니다.  
  // mapper.registerModules(new JavaTimeModule());  
  return mapper;  
}
```

그랬더니 이번엔 아래같은 오류가 생겼다.
```
Could not read JSON:The class with batch.batchapplication.auth.domain.User and name of batch.batchapplication.auth.domain.User is not in the allowlist. If you believe this class is safe to deserialize, please provide an explicit mapping using Jackson annotations or by providing a Mixin. If the serialization is only done by a trusted source, you can also enable default typing. See https://github.com/spring-projects/spring-security/issues/4370 for details (through reference chain: org.springframework.security.core.context.SecurityContextImpl["authentication"])
```

https://mangkyu.tistory.com/402

찾아보니 클래스 정보가 들어가는 것이 문제인 듯 하다.
그래서 역직렬화, 그러니까 클래스 변환 시 오류가 생기는 것인가?

보안 취약점도 존재한다고 한다.


https://techblog.woowahan.com/22767/
더 정확하고 좋은 문제 확인과 해결을 보여준다.

```
private ObjectMapper objectMapper() {  
  ObjectMapper mapper = new ObjectMapper();  
  
  // 1. 필요한 모든 기본 Jackson 모듈 명시적 등록 (컬렉션, 파라미터 이름, Java 8 타입 처리)  
  //    -> "authorities"와 같은 setterless 컬렉션 필드 처리 능력 향상  
  mapper.registerModules(new ParameterNamesModule(), new Jdk8Module(), new JavaTimeModule());  
  
  // 2. Spring Security 모듈 명시적 등록 (UsernamePasswordAuthenticationToken 등의 생성자 문제 해결)  
  mapper.registerModules(SecurityJackson2Modules.getModules(AbstractHttpSessionApplicationInitializer.class.getClassLoader()));  
  
  // 2. 사용자 정의 클래스(Principal)의 Allowlist 문제를 해결하는 Default Typing 활성화  
  mapper.activateDefaultTyping(  
          mapper.getPolymorphicTypeValidator(),  
          ObjectMapper.DefaultTyping.NON_FINAL,  
          JsonTypeInfo.As.PROPERTY  
  );  
  return mapper;  
}
```

결국 제미니와 IQ 80 + IQ 80 최종 IQ 160의 해답을 만들었다.
먼저, 토큰으로 직렬화 할 시 `@Class` 형태로 클래스 정보를 레디스에 넘긴다.
그 다음 클래스 복원 시 생성자와 setter를 안 만들어준다.


```
Message Could not read JSON:Problem deserializing 'setterless' property ("authorities"): no way to handle typed deser with setterless yet
```

이제 이것에 대한 문제를 해결해야 한다.





일단 해결했다...

SecurityContext 안의 authentication.principal이 유저 엔티티 클래스인데, Spring Security의 JSON 역직렬화 허용 목록(allowlist)에 없어서 막힌 것.

해결법은 두가지
- ObjectMapper에 네 엔티티를 명시적으로 허용시키기
- principal을 엔티티 그대로 쓰지 말고, 허용된 타입(또는 DTO)로 바꿔 넣기



```
Could not read JSON:The class with batch.batchapplication.auth.domain.User and name of batch.batchapplication.auth.domain.User is not in the allowlist. If you believe this class is safe to deserialize, please provide an explicit mapping using Jackson annotations or by providing a Mixin. If the serialization is only done by a trusted source, you can also enable default typing. See https://github.com/spring-projects/spring-security/issues/4370 for details (through reference chain: org.springframework.security.core.context.SecurityContextImpl["authentication"])
```




캐시의 JSON 직렬화와 그에 따른 오류, 원인과 해결. 대처법에 대해 알아본 문서
[[cache-json-serialize-issue]]


캐시를 레디스에 넣은 모습. 댖지다
![[spring-redis-apply5.png]]


@Class를 넣어서 그런지 아주 댖지가 따로 없다.
이걸 절약할 수 있는 방법이 없을까?

1. @Class 제거하고 JSessionID만 넣어주기
	- 이 경우, SecurityContext 관련 정보가 사라져서 보안도 취약할 뿐더러 불안정해진다.
	- 권한은 어떻게 확인함? 사용자는 어떻게 확인함? 그 외에도 여러가지 정보가 있을 건데 나머지 context들은 어떻게 처리할 거임? 필터에서 어떻게 해줄 거임?
2. 경량 DTO 이용해서 세션 적재 용량 줄이기
	- 위의 것이 조금 개선된 것.
	- 하는 법은 나도 모른다.
	- 프로세스가 하나 더 추가되었다. DTO로 세션 저장소에 넣어주고 해당 DTO를 UserDetails로 변환하는 작업.
	- 얼마나 정보를 저장하냐에 따라 저장소 효율이 달라지는 TradeOff 관계이다.
		- 너무 간단하면 DB와 Network I/O 발생
		- 너무 무거우면 Redis의 속도 감소

좌충우돌 세션 적용기인데, 알려진 정보가 하나도 없어서 어떻게 적용해야할 지 모르겠다.
그냥 시간 효율을 포기해야 하나?
UserDetails를 백엔드서버에서 만드는 방법은 분명 안전할 것이다... 어떻게 해야 할 지... 점점 수렁으로 굴러가는 기분이다...

