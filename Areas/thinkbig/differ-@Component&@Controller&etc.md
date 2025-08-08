`@Controller`, `@Repository`, `@Service`는 실제로 기능적으로는 전부 **`@Component`를 메타 어노테이션(meta-annotation)으로 가진 특수화 어노테이션**이다.

코드 상으로 보면 이런 구조예요:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component  // 여기서 @Component를 포함
public @interface Controller {
    @AliasFor(annotation = Component.class)
    String value() default "";
}
```

그래서 **빈 스캐닝 관점**에서 보면,  
이 세 가지 어노테이션 모두 `@Component`와 똑같이 동작해요.  
`@ComponentScan`이 클래스패스에서 이들을 찾아 스프링 빈으로 등록합니다.

---

## 1. **기능적인 차이**

스프링이 `@Component`를 기반으로 하지만, 어노테이션마다 **추가적인 의미나 부가 기능**이 붙습니다.

|어노테이션|실제 역할|@Component와 차이|
|---|---|---|
|`@Controller`|MVC의 컨트롤러 역할 표시|DispatcherServlet이 핸들러 매핑 시 `@Controller` 빈을 우선적으로 스캔하여 요청 매핑(`@RequestMapping`) 대상 등록|
|`@Service`|서비스 계층 표시|특별한 기능은 없지만, **AOP 적용 시 서비스 빈에 대한 의미 부여** (예: 트랜잭션 로깅, 비즈니스 로직 레이어 구분)|
|`@Repository`|DAO(데이터 접근 계층) 표시|**PersistenceExceptionTranslationPostProcessor**가 동작하여, JDBC/ORM 예외를 `DataAccessException`으로 변환|
|`@Component`|범용 빈 등록|특별한 부가 기능 없음|

---

## 2. **스프링 서블릿 컨테이너에서 등록 과정**

1. **Context 초기화**
    
    - `AnnotationConfigApplicationContext` 또는 `AnnotationConfigServletWebServerApplicationContext`가 시작됨
        
2. **ComponentScan 실행**
    
    - `@ComponentScan`이 지정된 패키지 경로를 스캔
        
    - 모든 클래스 중에서 `@Component` 또는 메타 어노테이션(`@Controller`, `@Service`, `@Repository`)이 붙은 클래스를 찾아 **BeanDefinition** 생성
        
3. **BeanDefinition 등록**
    
    - BeanDefinitionRegistry에 각 클래스 정보 등록
        
4. **빈 생성**
    
    - ApplicationContext가 BeanDefinition을 기반으로 객체 생성, 의존성 주입, 초기화 실행
        

---

## 3. **구별 방식**

- **빈 등록 과정에서는 차이가 없음** → 전부 `@Component`의 자식격이라 스캔 규칙이 동일
    
- **빈 후처리기(BeanPostProcessor)** 단계에서 차이가 발생
    
    - `@Repository` → `PersistenceExceptionTranslationPostProcessor`가 감지하여 예외 변환
        
    - `@Controller` → `RequestMappingHandlerMapping`이 감지하여 URL 매핑 등록
        
    - `@Service` → 특별 기능은 없지만 AOP 포인트컷에서 서비스 레이어를 명확히 식별 가능
        

---

정리하면,  
**빈 등록 메커니즘**에서는 전부 `@Component`의 확장판이라 동일하게 동작하지만,  
**빈 후처리기나 MVC/AOP 인프라**에서 이 특수화 어노테이션을 구분해 추가 동작을 붙입니다.



# 스프링 컨테이너는 각 빈을 어떻게 구분할까?
핵심만 딱 정리할게요.  
스프링은 “빈 등록” 단계에서는 @Component 계열을 전부 똑같이 취급하지만, “웹(서블릿) 인프라 초기화”나 “빈 후처리” 단계에서 각 애노테이션을 직접 검사해서 구별합니다. 즉, 구별은 등록이 아니라 소비(초기화/후처리) 단계에서 이루어져요.

# 어디서 어떻게 구별하나

1. **@Controller**
    

- 등록: @ComponentScan이 ScannedGenericBeanDefinition로 등록(= @Component와 동일)
    
- 구별 시점: DispatcherServlet 초기화 시 HandlerMapping들이 컨텍스트의 모든 빈을 훑으면서 “핸들러인지” 판별
    
- 판별 방식: RequestMappingHandlerMapping.isHandler(beanType)에서 타입에 @Controller 또는 타입/메서드에 @RequestMapping이 있는지 직접 검사
    
- 결과: 매핑된 핸들러 메서드들을 URL 매핑 테이블에 등록
    

핵심 포인트: Controller 구별은 HandlerMapping이 “@Controller 자체”를 본다. 단순히 “@Component의 하위”라는 사실로 구별하지 않음.

2. **@Repository**
    

- 등록: 스캔 및 빈 등록은 @Component와 동일
    
- 구별 시점: 컨텍스트 리프레시 중 BeanPostProcessor 동작
    
- 판별 방식: PersistenceExceptionTranslationPostProcessor가 빈의 타입에 @Repository가 붙었는지 검사
    
- 결과: 해당 빈에 예외 변환 어드바이스(ExceptionTranslationAdvisor) 적용 → JDBC/ORM 예외를 DataAccessException 계열로 변환
    

핵심 포인트: 예외 변환은 “@Repository가 붙었는지”를 직접 체크해서 프록시/어드바이스를 적용한다.

3. **@Service**
    

- 등록: @Component와 동일
    
- 구별 시점/방식: 코어 프레임워크가 특별 취급하진 않음. 다만 AOP 포인트컷(예: within(@org.springframework.stereotype.Service *))이나 운영 가이드(레이어 식별)에 활용
    
- 결과: 의미론적 레이어 표식. 트랜잭션(@Transactional) 같은 건 @Service 때문이 아니라 별도의 설정/포인트컷으로 적용됨
    

# 정리: “등록”은 동일, “소비 단계”에서 애노테이션을 직접 읽어 구별

- 스캔/등록: @Controller/@Service/@Repository 모두 @Component 메타라서 동일 루트로 BeanDefinition 등록
    
- 구별 트리거
    
    - 웹 레이어: RequestMappingHandlerMapping이 @Controller 또는 @RequestMapping을 직접 검사해 핸들러로 채택
        
    - 영속 레이어: PersistenceExceptionTranslationPostProcessor가 @Repository를 직접 검사해 예외 변환 어드바이스 적용
        
    - 서비스 레이어: 프레임워크 기본 특수 처리 없음(보통 AOP 포인트컷 규칙으로 구분)
        

# 참고가 될 짧은 흐름도

1. Context 생성 → ComponentScan → BeanDefinition 등록(세 애노테이션 모두 동일 경로)
    
2. Context refresh
    
    - RequestMappingHandlerMapping 초기화 → 모든 빈 타입 조사 → @Controller/@RequestMapping 있으면 핸들러 등록
        
    - PersistenceExceptionTranslationPostProcessor 실행 → @Repository 있으면 프록시로 감싸 예외 변환 적용
        
    - 그 외 AOP/트랜잭션 포인트컷 규칙에 따라 @Service 등 레이어 기반 규칙이 있으면 해당 빈에 프록시 적용