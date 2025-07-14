### 프레임워크의 정의
소기의 목적을 달성하거나 복잡한 문제를 해결하고 서술하는 데 사용되는 기본 개념 구조

프레임워크하면 당연히 웹 프레임워크인 스프링이 생각난다.
닷넷, 하둡 등 그 외에도 다양한 프레임워크들이 많다.
소프트웨어 개발 모형에서도 프레임워크라는 단어를 사용한다. (애자일 프레임워크)
이렇듯 폭넓은 정의가 있기 때문에 단어 자체는 맥락으로 알아 듣는 것이 편할 것 같다. 

### 프레임워크와 라이브러리의 차이?

정보처리기사에서 제시되는 프레임워크의 특징은 다음과 같다...
프레임워크
- 모듈화
- 재사용성
- 확장성
- 제어의 역전 (IoC, Inversion of Control)

![프레임워크 - 개발자 코드 - 라이브러리](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FMvJNW%2FbtsbPW6SKz0%2FAAAAAAAAAAAAAAAAAAAAAGEt_h8BK5ku2T6tUy73UyLoja_q9gOev4jpUyOgKpCX%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DeZLH26FetINyOhDWeSy0T4Yvd8U%253D)

프레임워크와 라이브러리와의 차이점은 바로 제어의 역전에 있다.
애플리케이션의 흐름에 대한 주도권이 어디에 있는지가 차이점이라는 것이다.
예를 들면 프레임워크는 설정파일의 태그설정이나, DB 연동 방법 등에 대한 규칙을 갖고 있고, 개발자는 이를 따라야 한다.
반대로 라이브러리는 개발자가 직접 호출하기 때문에 개발자에게 주도권이 있다.

> 일련의 예시로, 리액트 또한 UI를 만드는 기능 만을 제공하기 때문에 라이브러리로 본다. [MDN-react](https://developer.mozilla.org/ko/docs/Learn_web_development/Core/Frameworks_libraries/React_getting_started)

[프레임워크 vs 라이브러리](https://sharonprogress.tistory.com/169)


### 다양한 프레임워크
- JWT
- Spring
- .Net Framework
- Qt
- Android
- Bootstrap

### 스프링 삼각형으로 알아보는 프레임워크


![스프링삼각형](https://blog.kakaocdn.net/dna/wvBhF/btsB6kIzX9d/AAAAAAAAAAAAAAAAAAAAAH75uO3RQLaHFEvh3Bojlczx_MR5oghSSqxQC5zUYA5i/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1753973999&allow_ip=&allow_referer=&signature=2DkcpC43qo3w9IeaqZvkeP%2Fz7PI%3D)

DI
A가 B 클래스를 사용하고 있을 때, A가 B를 직접 생성해 사용하는 것이 아니라 외부에서 B 클래스의 인스턴스를 생성해 주입해주는 것

- 의존성 감소
	- 변화에 강함
	- 재사용성 높아짐
	- 유지보수 용이
- 코드양 감소
- 테스트 용이

IoC
스프링에서는 IoC를 위해 DI 컨테이너를 이용한다.
한 객체가 어떤 구체 클래스에 의존하는 지는 별도의 관심사이다. Spring은 의존성 주입을 도와주는 DI 컨테이너로써 강하게 결합된 클래스를 분리하고 애플리케이션 실행 시점에 객체 간 관계를 결정해 주어 결합도를 낮추고 유연성을 확보한다.

AoP : 관점 지향 프로그래밍
비즈니스 로직으로부터 중복된 관심사를 분리하는 것에 목적을 둔다.
애플리케이션에서 코드가 중복되고, 강력하게 결합되어 있어 다른 로직과 분리할 수 없는 애플리케이션 로직.
ex) 로깅, 보안 트랜잭션 등

즉, AOP란 부가 기능을 따로 관리하는 것을 의미한다.

스프링의 DI가 의존성의 주입이라면 AOP는 코드 주입이라 할 수 있다.
여러 모듈을 개발 시 모듈에서 공통적으로 등장하는 로직이 존재한다. 예를 들어 입금 출금 이체와 같은 부분에서 보안적인 부분이나 트랜잭션 로그를 남기고자 하는 코드 부분들이 분명히 공통적으로 등장할 것이다.

PSA : 일관성 있는 서비스 추상화
서비스 추상화의 대표적 예로, JDBC를 두는데 이러한 표준 스펙 덕분에 개발자는 오라클, MySQL, MS-SQL 혹은 이외의 것을 사용하더라도 공통된 방식으로 코드를 작성할 수 있다.

그 이유는 디자인 패턴 중 하나인 어댑터 패턴을 사용했기 때문이다.

이처럼 같은 일을 하는 다수의 기술을 공통의 인터페이스로 제어할 수 있게 한 것을 서비스 추상화라 한다.

즉 잘 만든 인터페이스, 추상화가 잘된 인터페이스를 뜻한다.

https://systemdata.tistory.com/73
https://github.com/Jammini/TIL/blob/master/spring/constructor_injection.md
https://github.com/Jammini/TIL/blob/master/spring/aop.md



### 템플릿 메소드 패턴

프레임워크에서 공통된 로직의 틀을 제공하고, 개발자가 구체적인 일부만 구현하는 패턴

여러 클래스에서 공통으로 사용하는 메서드를 템플릿화 하여 상위 클래스에서 정의하고, 하위 클래스마다 세부 동작 사항을 다르게 구현하는 패턴
변하지 않는 기능(템플릿)은 상위 클래스에 만들어두고 자주 변경되며 확장할 기능은 하위 클래스에서 만들도록 하여, 상위의 메소드 실행 동작 순서는 고정하면서 세부 실행 내용은 다양화 될 수 있는 경우에 사용된다.

패턴 구조
- AbstractClass(추상 클래스) : 템플릿 메소드를 구현하고, 템플릿 메소드에서 돌아가는 추상 메소드를 선언한다. 이 추상 메소드는 하위 클래스인 ConcreteClass 역할에 의해 구현된다.
- ConcreteClass(구현 클래스) : AbstractClass를 상속하고 추상 메소드를 구체적으로 구현한다. ConcreteClass에서 구현한 메소드는 AbstractClass의 템플릿 메소드에서 호출된다.

hook 메서드
부모의 템플릿 메서드의 영향이나 순서를 제어하고 싶을 때 사용되는 메서드의 형태.

자식 클래스에서 좀 더 유연하게 템플릿 메서드의 알고리즘 로직을 다양화 가능함.
훅 메소드는 추상 메소드가 아닌 일반 메소드로 구현하는데, 선택적으로 오버라이드 하여 자식 클래스에서 제어하거나 아니면 놔두거나 하기 위해서 이다.


[https://inpa.tistory.com/entry/GOF-💠-템플릿-메소드Template-Method-패턴-제대로-배워보자](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A9%94%EC%86%8C%EB%93%9CTemplate-Method-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90) 
-> 여기서 안가져온 내용들 한트럭임. 더 옮겨야 한다.

### 헐리우드 원칙
> "Don't call us, we'll call you"

저수준 구성 요소에서 고수준 구성요소의 직접 호출을 막고, 고수준 구성요소가 저수준 구성요소를 직접 호출 하는 것은 허용한다.

의존성 부패(dependency rot) 현상 방지 가능
>의존성 부패현상이란, 고수준 구성요소가 저수준 구성요소에 의존하고, 또 그 저수준 구성요소는 고수준 구성요소에 의존하고, 고수준 구성요소가 또다른 구성요소에 의존하고... 이런식으로 의존성이 복잡하게 꼬이는 것을 말한다. 
> 시스템이 이런식으로 구현되면 시스템이 어떤식으로 디자인 된 것인지 알아보기가 어려워진다.
> https://invincibletyphoon.tistory.com/52

### Convention over Configuration
- 설정보다 관례를 따르게 하는 개발 철학
- 스프링부트, 루비온레일즈 등에서 핵심 원칙
-  개발자가 작성해야 할 설정을 줄이고 생산성을 높임

### Software Architecture에서의 위치
- 프레임워크는 아키텍처의 구성 요소(Component)**로 동작함
- MVC, Layered Architecture, Clean Architecture 등과의 관계 이해 필요
### 프레임워크의 종속성과 추상화
- 프레임워크에 너무 의존하면 이식성(portability)과 독립성 손상
  → Framework Lock-in 현상
- 이를 벗어나기 위한 Interface 기반 설계의 중요성


### 스프링에서 많이 쓰는 디자인 패턴?

## ✅ 스프링에서 많이 쓰는 디자인 패턴

### 1. **싱글톤 패턴 (Singleton Pattern)**

- **사용처**: 모든 빈(Bean)은 기본적으로 싱글톤으로 관리됨
    
- **의의**: 상태가 없는 객체(Controller, Service 등)는 한 번만 생성해서 재사용 → 메모리 낭비 줄임
    
- **주의**: 싱글톤 객체에 **공유 상태(state)**를 두면 동시성 이슈 발생
    

---

### 2. **의존성 주입 (Dependency Injection, DI) + 전략 패턴**

- **사용처**: Service가 Repository나 외부 API 전략을 주입받음
    
- **구현 방식**: 인터페이스를 통해 구현체를 갈아끼우는 방식
    

```java
@Service
public class PaymentService {
    private final PayStrategy payStrategy;
    public PaymentService(PayStrategy payStrategy) {
        this.payStrategy = payStrategy;
    }

    public void pay() {
        payStrategy.execute();
    }
}
```

→ `PayStrategy` 구현체를 바꿔가며 정책 변경 가능

---

### 3. **템플릿 메서드 패턴 (Template Method Pattern)**

- **사용처**: `JdbcTemplate`, `RestTemplate`, `AbstractController` 등
    
- **의의**: 공통된 흐름을 정의하고, 핵심 로직만 override하여 유연성 확보
    

```java
public abstract class ReportGenerator {
    public final void generate() {
        fetchData();
        formatData();
        export();
    }

    protected abstract void fetchData();
    protected abstract void formatData();
    protected void export() { ... }
}
```

→ 스프링은 이 구조를 통해 공통된 로직을 프레임워크에서 제공하고, 개발자는 핵심 부분만 구현

---

### 4. **프록시 패턴 (Proxy Pattern)**

- **사용처**: AOP, 트랜잭션 처리, 보안 처리
    
- **설명**: 실제 객체 앞에 프록시 객체를 두어 **부가 로직을 삽입**
    

예: `@Transactional`  
→ 해당 메서드 실행 전/후로 트랜잭션 시작/커밋/롤백 코드 삽입

---

### 5. **옵저버 패턴 (Observer Pattern)**

- **사용처**: 이벤트 기반 비동기 처리 (ex. `ApplicationEventPublisher`)
    
- **예시**: 회원 가입 후, 이메일 전송/로그 기록 등 후처리를 이벤트 리스너로 분리
    

```java
@Component
public class SignUpListener {
    @EventListener
    public void onUserSignUp(UserSignUpEvent event) {
        sendWelcomeMail(event.getUser());
    }
}
```

---

### 6. **팩토리 패턴 (Factory Pattern)**

- **사용처**: 빈 생성 시 동적으로 객체를 만들거나 전략을 선택
    
- **구현 방식**: FactoryBean, @Bean 메서드에서 조건 분기
    

```java
@Configuration
public class EncoderFactory {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

### 7. **빌더 패턴 (Builder Pattern)**

- **사용처**: DTO 또는 복잡한 객체 생성 시
    
- **예시**: `Lombok @Builder` 사용해 가독성 좋게 생성
    

```java
Member.builder()
      .name("홍길동")
      .email("gildong@example.com")
      .build();
```

---

## 🧩 그 외 자주 언급되는 패턴들

|패턴|사용 맥락|
|---|---|
|어댑터 패턴|외부 API를 내부 인터페이스에 맞춤|
|데코레이터 패턴|필터체인, 인터셉터 체인|
|책임 연쇄 패턴|Spring Security의 인증 필터|
|퍼사드 패턴|Service 계층에서 여러 Repository 호출을 묶음|

---

## ✅ 요약

|패턴|스프링에서의 역할|
|---|---|
|싱글톤|기본 Bean 생성 방식|
|DI + 전략|의존성 관리와 유연한 정책 주입|
|템플릿 메서드|JdbcTemplate, RestTemplate 등|
|프록시|AOP, 트랜잭션, 보안|
|옵저버|이벤트 기반 비동기 처리|
|팩토리|조건 기반 Bean 생성|
|빌더|객체 생성 가독성 향상|

---

필요하다면 각 패턴별로 **실무 사례를 붙여서 설명하는 자료**나 **패턴-애노테이션 매핑표**도 만들어드릴 수 있습니다.