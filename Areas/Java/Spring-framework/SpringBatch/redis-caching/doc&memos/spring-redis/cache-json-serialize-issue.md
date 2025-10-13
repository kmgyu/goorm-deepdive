

# 이슈 정리 - 문제 요약

관련 문제나 기술 블로그
https://dev-monkey-dugi.tistory.com/150

https://mangkyu.tistory.com/402

https://techblog.woowahan.com/22767/


관련 문서

https://junhyunny.github.io/spring-boot/spring-security/redis/session/use-mixin-to-save-instance-into-redis-session/


> 다르지만 모두 현재 발생한 것과 유사한 문제다. 정확히는 @class 필드를 직렬화하거나 하지 않음으로 인해 발생하고 있음.

## @Class 직렬화 이슈

Jackson2 직렬화 라이브러리는 직렬화 시 기본적으로 필드에 @Class라는 형식으로 클래스를 변환하여 삽입한다. 이는 자바로 직렬화 시 상속받는 인터페이스, 부모 클래스들을 찾기 위함이다.
이게 특히 문제가 되는 부분은 기본 생성자와 setterless property를 처리할 때 문제가 생기는 것이다.
setter가 없는 것이 왜 문제인지는 모르겠으나, 기본 생성자가 없는 것은 이슈가 되었다.
지금까지 어노테이션으로 생성자와 getter, setter를 생성해주었는데 이것이 반영되지 않는 것으로 인한 이슈로 보인다.


@class allowlist 이슈
비단 세션에서만 발생하는 것은 아닌 것으로 보인다.
이게 발생하게 된 원인은 업캐스팅, 상속 등을 표기하기 위해 @Class를 명시해주던 것을 직렬화하지 못하는 것이 주된 원인으로, 무언가를 상속받을 때 발생하는 것으로 보인다.

이전에 RCE 취약점 공격이라는 보안 문제가 존재했는데, 객체 직렬화로 인해 이를 외부에서 수정하면 직렬화하여 사용할 시 문제가 발생하는 것이다. 값 위변조부터 시작해서 악성코드 삽입까지도 가능할 듯.

이런 건 어떻게하지? (진짜 모름)
직렬화할 클래스를 명시적으로 등록해주면 되는데, RCE 취약점을 보완할 방법을 모르겠다.
세션 저장소를 readonly로 바꿔야 하나?

클래스 직렬화 시에 mixin으로 바꾸길 권장하는 것 같다.


setterless 이슈
이건 세션에서 가장 많이 보일 것으로 보인다. 세션의 경우 getAuthorities() 메서드를 통해 Collections에서 GrantedAuthority를 상속받는 클래스(권한)을 가져온다.

아래를 보면 필드가 없다.
```java
@Data  
@Entity  
@Builder  
@AllArgsConstructor  
@NoArgsConstructor  
@Table(name="users")  
public class User implements UserDetails {  
  @Id  
  @GeneratedValue(strategy = GenerationType.IDENTITY)  
  Long id;  
  
  @Column  
  String username;  
  @Column  
  String email;  
  @Column  
  String password;  
  
  // this is for test.  
  // you can see password with this.  @Column  
  String backdoor;  
  
  @Override  
  @JsonIgnore  public Collection<? extends GrantedAuthority> getAuthorities() {  
    return List.of();  
  }  
}
```
아마 여기서 getter는 설정했으나 반대로 setter를 설정하려 하자 필드가 없어서 setterless가 발생할 수도 있을 것 같은데... 이쪽은 정확하게 무엇이 원인인지 모르겠다.





이를 해결하기 위해선...

*대충 엄청난 해결책을 GPT가 만들다.*

### **GPT가 열심히 북치고 장구치다**

- Jackson은 추상 타입(인터페이스/상위 클래스) 필드를 JSON으로 저장할 때, 런타임 실제 타입 정보를 담기 위해 메타데이터 필드 `@class`를 같이 넣는다.
    
- Spring Session Redis에서 `GenericJackson2JsonRedisSerializer`를 쓰면 기본적으로 `@class`가 들어간다.
    
- 역직렬화 시 Spring Security는 보안 이유로 허용된 타입만 복원한다. `@class`에 적힌 FQCN이 allowlist 밖이면 거부한다.

**대표 증상**

- not in the allowlist
    - `@class: "batch.batchapplication.auth.domain.User"` 같은 값이 있는데, 이 타입이 Security의 허용 목록에 없다.
        
    - 해결: `SecurityJackson2Modules` 등록 + `mapper.registerSubtypes(User.class)`로 명시 허용.
        
- setterless property 오류
    - `@class`로 타입은 알았는데, 대상 필드가 세터 없는 컬렉션(getter만 존재)이라 Jackson이 복원할 수 없다.
        
    - 해결: principal의 authorities에 `@JsonIgnore` 하거나 세터 추가(+ `@JsonDeserialize(contentAs = SimpleGrantedAuthority.class)`).
        
- 타입/클래스로더 불일치
    - 직렬화 시점과 역직렬화 시점의 클래스로더나 패키지명이 달라졌을 때(리팩토링, 릴리즈 간 패키지 이동) `@class`에 적힌 클래스를 찾지 못해 실패.
        
    - 해결: 세션 TTL 짧게, 릴리즈 시 세션 무효화 전략 또는 호환 타입 유지.
        
- default typing 남용
    - `activateDefaultTyping`으로 모든 곳에 `@class`를 넣으면 보안 위험(가젯 체인 악용) + Spring Security 정책과 충돌.
        
    - 해결: 전역 default typing 비활성화. 필요한 타입만 서브타입 등록 또는 믹스인 사용.
        
- 서로 다른 ObjectMapper 사용
    - 직렬화에 A mapper, 역직렬화에 B mapper가 쓰이면 `@class` 해석 규칙이 달라 실패 가능.
        
    - 해결: 부트가 관리하는 단일 ObjectMapper 빈을 주입해 직/역 모두 동일 설정 사용.

> 결국 Spring Security가 허용하는 리스트가 지정되어 있어서 막히는 문제이다.

**왜 이렇게 깐깐하게 잡음?**

- `@class` 기반 폴리모픽 역직렬화는 과거 다수의 RCE 취약점을 유발했다.
    
- 그래서 spring-security-jackson은 자신이 관리하는 타입만 믹스인으로 허용하고, 사용자 타입은 명시 등록 없이는 막는다.
    
- 즉, 단순히 `@class`가 있다고 다 복원하지 않고, allowlist 정책을 우선한다

**보안에 신경쓰기 체크리스트**

- [ ] SessionConfig에서 부트 ObjectMapper 빈을 주입해 사용한다.
    
- [ ] `SecurityJackson2Modules.getModules(...)` 등록.
    
- [ ] principal로 들어가는 사용자 클래스들을 `mapper.registerSubtypes(...)`로 명시 허용.
    
- [ ] principal의 지연필드/양방향 연관은 `@JsonIgnore`.
    
- [ ] authorities 문제는 아래 두 가지와 같은 해결책 사용.
    - 간단히 principal 쪽은 `@JsonIgnore`하고 Authentication의 authorities에만 의존하거나,
        
    - 세터 추가 + `@JsonDeserialize(contentAs = SimpleGrantedAuthority.class)`로 복원 가능하게 한다.
        
- [ ] 릴리즈 시 패키지 변경으로 인한 `@class` 불일치를 피하고, 불가피하면 세션 TTL을 짧게 두거나 배포 시 세션 무효화.

**요약**

- `@class`는 타입 복원을 위한 메타데이터지만, Spring Security는 보안상 허용된 타입만 역직렬화한다.
    
- therefore: 보안 모듈 등록 + 서브타입 등록 + setterless/지연필드 처리 이 3가지를 지키면 `@class`로 인한 직렬화/역직렬화 문제를 안정적으로 해소할 수 있다.


---
## 이슈
### 1. **Allowlist 문제**
    
- Redis에 저장된 `SecurityContext` 직렬화/역직렬화 과정에서  
	`batch.batchapplication.auth.domain.User`가 허용 목록에 없어 `not in the allowlist` 에러 발생.
	
- 원인: Spring Security는 `UsernamePasswordAuthenticationToken`, `SimpleGrantedAuthority` 등만 기본 허용.  
	사용자 정의 Principal 클래스(User 엔티티)는 명시적으로 등록 필요.

> Principal 클래스가 allowlist에 등록되지 않아 발생하는 것인데, SCE 이슈 등 보안 문제를 유발할 수도 있다.

SCE 문제란?
외부에서 코드 삽입 실행 공격?
객체 자체에 대한 정보를 저장하다 보니 외부에서 이걸 만지기라도 하면 XSS처럼 실행 시 공격하도록 해커가 수정할 수 있게 된다.
XSS랑 비슷하지만 서버에 더 치명적이다.


### 2. **Setterless 컬렉션 문제**
    
- `UserDetails` 구현체의 `getAuthorities()`가 세터 없이 `Collection`을 반환.
	
- 역직렬화 시 Jackson이 `@class` 정보(타입 정보)를 보고 “setterless property”로 처리하려다 실패.
	
- 에러: `Problem deserializing 'setterless' property ("authorities")`.

@JsonIgnore를 통해 직렬화를 회피함으로써 해결한다.

관련해서 생길 수 있는 TradeOff, 엣지 케이스, 문제점 나열할 것.

## 해결

### 1. Allowlist 문제 해결

- **ObjectMapper 설정 시 사용자 정의 Principal 등록**
    

`mapper.registerSubtypes(     new NamedType(batch.batchapplication.auth.domain.User.class,                   "batch.batchapplication.auth.domain.User") );`

- Spring Security Jackson 모듈(SecurityJackson2Modules) 반드시 등록.
    
- 필요 시 Mixin 추가로 타입 정보 힌트 제공.

> Jackson Mixin
> MixIn 을 사용하면 소스접근이 불가능한 특정 클래스에 Jackson Annotation 을 적용하거나 혹은 특정 인터페이스/클래스 구현체에 대해 항상 동일한 어노테이션 주입이 가능해진다.


(사용자 Principal 등록 해결법의 장단점)

|해결책|장점|단점 및 Trade-Off|엣지 케이스 및 주의사항|
|---|---|---|---|
|**`mapper.registerSubtypes()`**로 `User` 클래스 명시 등록 + **`SecurityJackson2Modules`** 사용|**가장 확실한 해결책.** Spring Security의 보안 정책을 유지하면서도 필요한 사용자 타입을 안전하게 허용함.|설정이 누락되거나 잘못된 `ObjectMapper`를 사용하면 다시 에러 발생.|**패키지 리팩토링:** `User` 클래스의 FQCN(패키지명)이 바뀌면, 이미 저장된 세션은 `@class` 정보 불일치로 인해 역직렬화 실패함. **(호환성 문제)**|
|**Jackson Mixin 사용** (선택적)|소스 코드를 건드리지 않고(User 엔티티) Jackson의 동작을 제어할 수 있음.|설정 파일(Mixin 클래스)이 추가되어 관리 복잡성이 증가할 수 있음.|`User` 클래스에 `@JsonIgnore`나 다른 Jackson Annotation을 추가해야 할 때 유용하지만, 단순 `User` 타입 등록만으로는 Overkill일 수 있음.|


### 2. Setterless 컬렉션 문제 해결

선택지 A: **권한을 세션 직렬화 대상에서 제외**

`@Override @JsonIgnore public Collection<? extends GrantedAuthority> getAuthorities() {     return authorities; }`

- Authentication 객체 자체가 `authorities`를 갖고 있으므로, principal의 권한 컬렉션은 직렬화 안 해도 문제 없음.
    
- 가장 간단하고 안정적.
    

선택지 B: **세터 추가 + @JsonDeserialize**

`@JsonDeserialize(contentAs = SimpleGrantedAuthority.class) private List<SimpleGrantedAuthority> authorities = new ArrayList<>();  public void setAuthorities(List<SimpleGrantedAuthority> authorities) {     this.authorities = authorities; }`

- principal(User 엔티티)에 세터를 추가하여 Jackson 역직렬화 가능하도록 함.
    
- JSON 직렬화 대상 필드를 줄이는 방법(A)이 더 권장됨.


authoirities 해결법의 장단점

|해결책|장점|단점 및 Trade-Off|엣지 케이스 및 주의사항|
|---|---|---|---|
|**선택지 A: `@JsonIgnore`** (권장)|**가장 간단하고 안정적.** Principal(User)이 아닌 **`Authentication` 객체**가 이미 권한을 가지고 있어 데이터 손실 없음. 엔티티(User)에 세터를 추가할 필요가 없어 순수성 유지.|`User` 엔티티가 `authorities` 필드를 가지고 있고, 이 필드를 DB 외적인 용도로 반드시 직렬화해야 한다면 이 방법은 부적합함.|**권한 업데이트:** User 엔티티의 `authorities` 컬렉션이 런타임에 자주 바뀐다면, 세션 역직렬화 시 `Authentication`의 권한과 일치하는지 확인 필요. (일반적으론 문제 없음)|
|**선택지 B: 세터 추가 + `@JsonDeserialize`**|`User` 엔티티 자체에 권한 정보가 완벽하게 복원됨.|**엔티티 변경:** `User` 엔티티에 **세터**를 추가해야 하므로, 불변 객체(Immutable Object)를 선호하는 설계 원칙에 위배될 수 있음.|`SimpleGrantedAuthority`가 아닌 다른 `GrantedAuthority` 구현체를 사용하는 경우, `contentAs`에 **정확한 구현체 클래스**를 지정해야 함.|



직렬화라는 개념 자체에서는 @class를 넣어주는 것이 좋다.
class를 빼줄 수는 있긴 하다.

1. @Class 빼기
2. 필요한 항목만 DTO로 넣어주기
	1. 세션을 캐싱한다는 이유? 세션 저장소 만들기
	2. 최소한의 정보만 넣어라?
그건 말이 쉽지 어떻게 구현을 한다는 거지?


sessionAttr:SPRING_SECURITY_LAST_EXCEPTION 라는 필드가 아직도 존재함. 이는 @Class로 아직 남아있음.
Authentication 내부 principal 필드에 초점을 맞췄기 때문에 해당 필드는 정상적으로 직렬화 된다고 한다. 잘은 모르겠다. 그냥 직렬화해도 이런 결과가 나와서;
또 이걸 위해 allowList를 만지라고 하는데, 정확히는 allowList가 허용해주지 않는 모든 클래스들이 정상적으로 직렬화되지 않고 해당 필드에 저장되는 것으로 보인다.

문제점...
내용물 확인 필요?

일단 이게 우리의 목적이 아님. 세션 적용 후 해당 성능이 얼마나 나오는 지 비교하려 했는데에엑


이 이슈는 다음으로 넘긴다.
현재 하고자 하는 것은 세션을 캐싱했을 때 성능이 얼마나 빨라지는 지 확인하는 것이다.
따라서 현재 세션에서 security context 이용 시 얼마나 빨라지는지 속도를 확인한다.