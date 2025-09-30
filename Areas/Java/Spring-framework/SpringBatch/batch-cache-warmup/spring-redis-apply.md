
https://innovation123.tistory.com/273#Spring%20Redis%20%EC%BA%90%EC%8B%B1%20%EA%B3%BC%EC%A0%95-1-3


# 기본 캐시 적용
---

레디스를 이용해 캐싱을 적용해준다.

그런데 레디스는 메서드에 @Cacheable 같은 것이 있어야만 적용할 수 있다.
그래서 추가로 찾아보니, Security를 위한 종속성이 따로 존재한다.

![[Pasted image 20250930232603.png]]
추가해주고, SessionConfig라는 것을 통해 세션 저장소를 설정해줄 수 있다.
나머지는 깃헙에서...


캐싱 / 캐싱 X 비교해보자.

캐싱X
![[Pasted image 20250930232332.png]]

캐싱
![[Pasted image 20250930232304.png]]

localhost에서 캐싱해주는 속도가 조금 빨라졌다.
대량의 트래픽이 발생할 때 더 효과적일 것이다.

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


Message Could not read JSON:Cannot construct instance of `org.springframework.security.authentication.UsernamePasswordAuthenticationToken` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)


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



해결 못했다.



일단 