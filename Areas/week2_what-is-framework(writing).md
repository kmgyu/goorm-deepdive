# 프레임워크

## 정의
소기의 목적을 달성하거나 복잡한 문제를 해결하고 서술하는 데 사용되는 기본 개념 구조

*프레임워크라고 하면 가장 먼저 웹 프레임워크인 스프링이 떠오른다.*  
*그러나 닷넷, 하둡과 같은 플랫폼 기반 프레임워크부터, 소프트웨어 개발 방법론에서 사용되는 애자일 프레임워크까지 다양한 맥락에서 이 용어가 사용된다.*

## 프레임워크와 라이브러리의 차이?
프레임워크와 라이브러리는 개발자가 반복적인 일을 하지 않게 만들어준다는 점에서 유사하다. 어떤 차이가 있을까? 이미 학습한 내용일 수 있지만, 프레임워크를 이해하기 위해 핵심적인 차이점을 다시 정리해보자.

정보처리기사에서 제시되는 프레임워크의 특징은 다음과 같다.
- 모듈화
  인터페이스에 의한 캡슐화를 통해 모듈화를 강화하고 설계와 구현의 변경에 따르는 영향을 극소화한다.
- 재사용성
  반복적으로 사용할 수 있는 컴포넌트를 정의할 수 있게 하여 재사용성을 높여준다.
- 확장성
  다형성을 통해 애플리케이션이 프레임워크의 인터페이스를 넓게 사용할 수 있도록 한다. 프레임워크는 애플리케이션 서비스나 특성이 변하더라도 그 영향에서 독립적으로 유지되며 이로 인해 프레임워크는 다양한 상황에서 재사용될 수 있는 장점을 가진다.
- **제어의 역전 (IoC, Inversion of Control)**

![프레임워크 - 개발자 코드 - 라이브러리](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FMvJNW%2FbtsbPW6SKz0%2FAAAAAAAAAAAAAAAAAAAAAGEt_h8BK5ku2T6tUy73UyLoja_q9gOev4jpUyOgKpCX%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DeZLH26FetINyOhDWeSy0T4Yvd8U%253D)
> 제어 흐름의 주체를 시각적으로 나타내는 사진

프레임워크와 라이브러리와의 차이점은 바로 제어의 역전에 있다.
애플리케이션의 흐름에 대한 주도권이 어디에 있는지가 차이점이라는 것이다.
예를 들면 프레임워크는 설정파일의 태그설정이나, DB 연동 방법 등에 대한 규칙을 갖고 있고, 개발자는 이를 따라야 한다.
반대로 라이브러리는 개발자가 직접 호출하기 때문에 개발자에게 주도권이 있다.

> 예를 들어, React는 사용자 인터페이스(UI)를 구성하는 기능만을 제공하므로 일반적으로 라이브러리로 분류된다. (코드 컨벤션, 파일 구조 등을 강제하지 않음.) [MDN-react](https://developer.mozilla.org/ko/docs/Learn_web_development/Core/Frameworks_libraries/React_getting_started)

[프레임워크 vs 라이브러리](https://sharonprogress.tistory.com/169)
https://hypers84.tistory.com/2461

### 다양한 프레임워크
- JWT
- Spring
- .Net Framework
- Qt
- Android
- Bootstrap

# 스프링 삼각형으로 알아보는 스프링

![스프링삼각형](https://blog.kakaocdn.net/dna/wvBhF/btsB6kIzX9d/AAAAAAAAAAAAAAAAAAAAAH75uO3RQLaHFEvh3Bojlczx_MR5oghSSqxQC5zUYA5i/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1753973999&allow_ip=&allow_referer=&signature=2DkcpC43qo3w9IeaqZvkeP%2Fz7PI%3D)
*스프링의 주요 특징을 삼각형으로 표현한 것, 개발 철학과도 관련이 있다.*

## 스프링 개발 철학
- **모든 레벨에서 선택권을 제공한다.**
  Spring은 설계 의사결정을 최대한 늦게 하도록 연기할 수 있게 한다. 예를 들어, 코드를 변경하지 않고 configuration을 통해 지속성 제공자를 바꿀 수 있다. 이는 다른 많은 인프라 문제와 타사 API와의 통합에 대해서도 마찬가지다.
- **다양한 관점을 수용한다.**
  Spring은 유연성을 포용하며 일을 처리하는 방법에 대한 의견을 제시하지 않는다. 또한, Spring은 서로 다른 관점을 갖는 넓은 범위의 애플리케이션 요구 사항을 제공한다.
- **이전 버전과의 강력한 호환성을 유지한다.** 
  Spring의 진화는 버전 간에 변경 사항이 거의 없도록 조심스럽게 관리되어 왔다. Spring은 신중히 선택된 범위 안에서 JDK 버전과 서드 파티 라이브러리를 제공하는데, 이는 스프링에 의존하는 애플리케이션과 라이브러리의 유지보수를 쉽게 하기 위해서이다.
- **API 설계에 관심을 기울인다.** 
  Spring 팀은 많은 생각과 시간을 들여 API를 만든다. 이렇게 만들어진 API는 직관적이며, 스프링의 여러 버전뿐만 아니라 수 년에 걸쳐서도 유지된다.
- **코드 퀄리티에 대해 높은 기준을 설정한다.** 
  스프링 프레임워크는 의미있고, 최신이며, 정확한 javadoc을 중요하게 생각한다. 그렇기에 Spring은 패키지 간 순환 의존성이 없는 클린한 코드 구조를 갖는 몇 안되는 프로젝트 중 하나이다.

[https://woonys.tistory.com/215](https://woonys.tistory.com/215) [WOONY's 인사이트:티스토리]


## POJO(Plain Old Java Object)
자바 언어 사양 외에 어떠한 제한에도 묶이지 않은 자바 오브젝트
Java EE 등의 중량 프레임워크들을 사용하게 되면서 해당 프레임워크에 종속된 무거운 개체들을 만들게 된 것에 반발하며 사용하게 된 용어.
필요에 따라 재활용할 수 있는 방식으로 설계된 자바 오브젝트를 뜻함.

## IoC
스프링에서는 IoC 구현을 위해 대표적으로 DI를 사용하며, 이를 지원하기 위한 구조로 DI 컨테이너를 이용한다.
Spring은 의존성 주입을 도와주는 DI 컨테이너로써 강하게 결합된 클래스를 분리하고 애플리케이션 실행 시점에 객체 간 관계를 결정해 주어 결합도를 낮추고 유연성을 확보한다.

### DI 컨테이너
> IoC 컨테이너, DI 컨테이너, 스프링 컨테이너 등 혼용이 되지만 IoC가 더 큰 개념임.
#### DI (Dependency Injection, 의존성 주입)
A가 B 클래스를 사용하고 있을 때, A가 B를 직접 생성해 사용하는 것이 아니라 외부에서 B 클래스의 인스턴스를 생성해 주입해주는 것

그래서 의존성이 뭔데.
```
interface object = new class();
```
다음과 같이 직접 클래스를 호출하는 형태로 인터페이스와 클래스가 직접 연결되어 있는 형태를 말한다. (연관되어 있을 경우 자체를 의존성이라고 말하나, 여기서 말하는 것은 직접적인, 강한 의존성을 말한다.)
이럴 경우, 기능이 수정되거나 클래스 자체를 바꿔야 할 때 개발자가 해당 코드의 영향을 직접 모두 제어해야 한다. (클래스 수정 시 생성자 모두 수정, 기능 수정으로 인한 오류 등...)

menuService에 의존성을 주입하는 예제
```java
public class MenuController {
	private final MenuService menuService;
... }
```


```java
public class MenuController {
	...
	public MenuController(MenuService menuService) {
		this.menuService = menuService
	}
	...
}
```

DI의 장점
- 의존성 감소
	- 변화에 강함
	- 재사용성 높아짐
	- 유지보수 용이
- 코드양 감소
- 테스트 용이

DI 컨테이너의 역할?
- 외부에서 객체를 생성하고 관리하면서 의존관계를 연결해주는 것을 DI 컨테이너라고 한다.
- IoC 컨테이너라고도 하는데 의존 관계 주입에 초점을 맞추어 주로 DI 컨테이너라 한다.

**의존성 주입 관련 어노테이션**
스프링에서는 별도로 프레임워크에서 자동으로 생성자를 주입할 수 있도록 어노테이션 또한 제공한다.
총 3개의 어노테이션이 존재하는데, 각 어노테이션마다 차이점이 존재하니 각 특징들을 인지하고 사용해야 한다.

어노테이션 소개에 앞서, 스프링 빈에 대해 짧게 짚고 넘어간다.
##### Spring Bean이란?
빈(Bean)은 스프링 컨테이너에 의해 관리되는 재사용 가능한 소프트웨어 컴포넌트이다. 또한 인스턴스화된 객체를 의미하며, 스프링 컨테이너에 등록된 객체를 스프링 빈이라고 한다.

개발자가 객체 생성을 직접 하지 않아도 되며, 스프링 컨테이너가 주입까지 관리하는 객체라고 볼 수 있다.

```xml
<!-- applicationContext.xml -->
<bean id="helloService" class="com.example.myapp.di.HelloService"/>
```

```java
// 자바 코드
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
IHelloService helloService = (IHelloService) context.getBean("helloService");
```

위 XML 설정은 `HelloService` 클래스를 `helloService`라는 이름으로 스프링 컨테이너에 등록한 것이다.  
이후 자바 코드에서는 `ApplicationContext`를 통해 컨테이너에서 해당 빈을 가져오며, `new` 키워드 대신 컨테이너가 관리하는 객체를 사용한다.

이렇게 xml 파일 빈 태그를 추가하면 helloService 인스턴스를 스프링 컨테이너가 관리하게 된다.

https://dev-wnstjd.tistory.com/440

##### @AutoWired
스프링 사용 시 기본으로 사용하는 의존성 주입 어노테이션이다.

@AutoWired는 기본적으로 등록되어 있는 빈 중에서 타입이 일치하는 빈을 찾아서 주입한다. 또는 등록된 빈의 이름과 어노테이션이 붙어 있는 변수의 이름과 일치하는 것을 찾아 주입한다.

예를 들어 아래와 같은 경우에는 Test 타입인 빈을 먼저 찾고 많이 Test 타입인 빈이 여러 개 등록되어 있다면 test라는 이름을 가진 빈을 주입하게 되어 있다.

```java
@Autowired
private Test test;
```

옵션으로 required를 입력할 수 있다. 반드시 주입해야하는지  여부를 true/false로 넣을 수 있는데, default 값은 true이다.
만일 false를 넣을 경우 빈 객체를 주입하는데 문제가 있어도 에외를 발생시키지 않고 빈을 주입시키지 않는다.

```java
@Autowired(required = false)
private Test test;
```

##### @Resource
@Resource는 기본적으로 이름을 통해 빈을 찾아 주입시킨다.
아래와 같이 작성했을 때 Test 타입의 빈을 먼저 찾는 것이 아닌 test이름을 가진 빈을 먼저 찾는다. 만약이 이름이 중복된다면 Test 타입인지 확인 후 주입시킨다.

```
@Resource
private Test test;
```

@Resource 어노테이션은 여러 가지 옵션이 있지만 name 옵션을 통해 주입시킬 빈의 이름을 명시할 수 있다.
변수의 이름과는 별도로 작용하기 때문에 변수의 이름과 다르게 설정할 수 있다.

```
@Resource(name = "testBean")
private Test test;
```

##### @Inject
@Inject는 @Autowired와 동일하게 동작해 의존성 주입을 한다. 
차이점은 @Inject는 자바 자체에 있는 어노테이션으로 자바를 사용하면 사용할 수 있는 어노테이션이다.
@Autowired의 경우 스프링 프레임워크를 사용할 경우 사용할 수 있는 어노테이션이다.

사용 방법은 @Autowired와 동일하다.

@Inject 또한 타입으로 해당 빈을 찾아 주입 시키고 만일 해당 타입의 빈이 여러 개라면 그 다음에 이름으로 찾아 주입시킨다.

```
@Inject
private Test test;
```


>단 이와 같은 어노테이션으로 스프링 프레임워크에서 의존성 주입을 하기 위해서는 Component Scan 기능을 활성화 시켜야 한다.
>
>스프링부트 어플리케이션의 경우 이러한 Component Scan에 대한 기능을 이미 @SpringBootApplication 안에 다 포함시켜놔서 빈으로 등록만 되어 있으면 Component Scan 기능을 활성화 시키지 않고도 빈 등록만으로 의존성 주입 어노테이션을 사용할 수 있다.
https://pamyferret.tistory.com/33

스프링부트가 아닌 스프링 프레임워크에서는 아래와 같이 Bean 설정 파일에 어노테이션을 통해 Component Scan 기능을 활성화 시킬 수 있다.

```java
@Configuration @ComponentScan("com.pamyferret.test.component") public class AppConfig {}
```

또는 xml 파일을 통해 빈을 등록하고 scan 하도록 할 수 있다.

```xml
<bean id="accountService" class="AccountService"></bean>
<bean id="userService" class="UserService"></bean>
```


[DI컨테이너의 사용 이유?](https://velog.io/@wogud7587/Spring-1-DI-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)


## AoP (관점 지향 프로그래밍)
> 애플리케이션의 핵심 비즈니스 로직과 관련 없는 부가적인 기능들을 모듈화하여 코드의 중복을 줄이고 유지보수성을 향상시키는 것을 목적으로 한 프로그래밍 패러다임

비즈니스 로직으로부터 중복된 관심사를 분리하는 것에 목적을 둔다.
ex) 로깅, 보안 트랜잭션 등 비즈니스 로직 관점에서 바라볼 때, 입금, 출금, 계좌 확인 등에서 지속적으로 사용되는 중복된 관심사를 분리하는 것

즉, 부가 기능을 따로 관리하는 것을 의미한다.

스프링의 DI가 의존성의 주입이라면 AOP는 코드 주입이라 할 수 있다.

대표적으로 @Transactional 어노테이션으로 트랜잭션 처리를 공통으로 넣어주는 것이 있다.

### AOP 용어 정리
- Aspect
  횡단 관심사(Cross-Cutting Concern)를 모듈화한 단위이며, 하나 이상의 Advice와 (선택적으로) Pointcut을 포함한다.
  일반적으로 @Aspect를 선언한 클래스를 지칭한다.
  여러 객체에 공통으로 적용되는 기능을 트랜잭션이나 보안 등이 그 예시이다.
- Advice
  Advice는 Join Point에 실행될 부가 기능(공통 로직)을 담은 코드 조각이다.  
  Advice는 실행 시점에 따라 `@Before`, `@After`, `@Around` 등으로 구분되며, 메서드 호출 전후 또는 예외 발생 시점에 동작할 수 있다.
- Weaving
  Advice를 핵심 로직 코드(target object)에 적용하는 과정, 이 과정을 통해 프록시 객체가 생성됨.
  Spring AOP는 런타임 시점에 프록시 객체를 생성해 weaving을 수행한다.
- Target
  Aspect가 적용될 실제 객체(클래스) 또는 그 객체의 메서드
  스프링 AOP에서는 프록시를 통해 타깃 객체 메서드 실행 전후에 Advice를 연결함.
- Join point
  Advice를 적용할 수 있는 실행 지점
  스프링 AOP에서는 메서드 실행 지점만 지원
- Pointcut
  Join Point 중에서 Advice가 적용될 대상을 선택하기 위한 표현식
  스프링 AOP에서는 AspectJ 표현식(예: `execution(* com.example..*(..))`)을 통해 Pointcut을 정의한다.

DI와는 컨셉 자체가 다르니 혼동하지 말 것.

> https://dev-ljw1126.tistory.com/345#Crosscutting-Concerns-(%ED%9D%A9%EC%96%B4%EC%A7%84-%EA%B4%80%EC%8B%AC%EC%82%AC)
> https://adjh54.tistory.com/133

### 구현 방법
1. 컴파일 이용 방법
2. 바이트 코드 조작
3. 프록시 패턴
스프링은 이중 프록시 패턴을 기반으로 하여 AOP를 지원한다.
프록시를 통해 핵심 로직을 구현한 객체에 접근하게 되면 핵심 로직을 실행하기 전,후에 공통된 기능을 적용하는 방식으로 AOP를 구현한다.

모든 요청(request)는 서블릿(Servlet)에 도달하기 전에 필터(Filter)가 먼저 해당 요청을 가로채 필터링하고 필터링된 출력값이 서블릿으로 간다. 그리고 서블릿을 거쳐 내부 로직을 처리하는 중에 입/출력단에서 다시 한번 인터셉터(Interceptor)가 동작하는데,이 필터와 인터셉터 계층에게 공통기능을 맡김으로서 개발자가 원하는 특정 로직에 공통 기능을 적용할 수 있다.

![위키피디아-UML의 프록시](https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Proxy_pattern_diagram.svg/1024px-Proxy_pattern_diagram.svg.png)
*UML의 프록시 - [위키피디아](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9D%EC%8B%9C_%ED%8C%A8%ED%84%B4)*



### 템플릿 메소드 패턴

> 프레임워크에서 공통된 로직의 틀을 제공하고, 개발자가 구체적인 일부만 구현하는 패턴

여러 클래스에서 공통으로 사용하는 메서드를 템플릿화 하여 상위 클래스에서 정의하고, 하위 클래스마다 세부 동작 사항을 다르게 구현하는 패턴
변하지 않는 기능(템플릿)은 상위 클래스에 만들어두고 자주 변경되며 확장할 기능은 하위 클래스에서 만들도록 하여, 상위의 메소드 실행 동작 순서는 고정하면서 세부 실행 내용은 다양화 될 수 있는 경우에 사용된다.

패턴 구조
- AbstractClass(추상 클래스) : 템플릿 메소드를 구현하고, 템플릿 메소드에서 돌아가는 추상 메소드를 선언한다. 이 추상 메소드는 하위 클래스인 ConcreteClass 역할에 의해 구현된다.
- ConcreteClass(구현 클래스) : AbstractClass를 상속하고 추상 메소드를 구체적으로 구현한다. ConcreteClass에서 구현한 메소드는 AbstractClass의 템플릿 메소드에서 호출된다.

hook 메서드
부모의 템플릿 메서드의 영향이나 순서를 제어하고 싶을 때 사용되는 메서드의 형태.

자식 클래스에서 좀 더 유연하게 템플릿 메서드의 알고리즘 로직을 다양화 가능함.
훅 메소드는 추상 메소드가 아닌 일반 메소드로 구현하는데, 선택적으로 오버라이드 하여 자식 클래스에서 제어하거나 아니면 놔두거나 하기 위해서 이다.

https://velog.io/@meme2367/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%8A%B8%EB%9D%BC%EC%9D%B4%EC%95%B5%EA%B8%80Spring-Triangle-IoC-AOP-PSA
[https://inpa.tistory.com/entry/GOF-💠-템플릿-메소드Template-Method-패턴-제대로-배워보자](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A9%94%EC%86%8C%EB%93%9CTemplate-Method-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90) 
해당 글을 요약한 것임. 예제 코드가 많으니 읽어보는 것이 좋다.

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
