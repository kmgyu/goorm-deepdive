도메인 협력관계 -> 도메인의 협력 관계 정의
클래스 다이어그램 -> 클래스 및 구현체, 인터페이스 등 정의
객체 다이어그램 -> 실제로 서버에 올라가면 참조가 어떻게 되는지에 대해.


JUnit 사용법
@Test 어노테이션
Assertions.assertThat(variable).isEqualTo(Data);
형식으로 값 검증
이게 테스트의 단위가 되는듯?

구현체가 추상에도 의존하고 구체클래스에도 의존하면 DIP 지키려다 둘 다 의존하는 망해버린 사례
policy 변경한다 하면 다른 구체 클래스로 이걸 바꾸게 됨.

-> 그럼 인터페이스만 의존하는데 어떻게 기능이 있냐고!!!
누군가 클라이언트에 구현 객체를 대신 생성해서 주입해주면(DI해주면) 된다

---

관심사의 분리
AppConfig 등-장
애플리케이션 전체 동작 방식 구성(config)위해 구현 객체 생성, 연결하는 책임을 가지는 별도의 클래스를 만든다.


```java
public class AppConfig {

	public MemberService memberService () {
	return new MemberServiceImpl(new MemoryMemberRepository());
	}
	
}
```
DI를 위한 팩토리

```java
public class MemberServiceImpl implements MemberService {
	private final MemberRepository memberRepository;

	public MemberServiceImpl(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}

	...
}


```
*MemberServiceImpl의 코드*

이렇게 되면 MemberServiceImpl은 MemberRepository 인터페이스에만 의존하면 된다. 구체 클래스를 몰라도 되는 상황이 나온다.
- DIP 완성
- 관심사의 분리

뒤에서는 AppConfig가 MemberServiceImpl / Repository도 생성하고 DI 주입히킨다.


@BeforeEach
테스트 실행전에 무조건 실행해야한다.
어디서 본 것 같다. 이거 앞 뒤로 붙이는 거면 데코레이터 패턴인가?

비즈니스로직이 바뀌어도 AppConfig만 수정해도 되도록 만들어야 한다~
중복도 제거가능하다.
new classA(new C, new D) 이런 형식도 팩토리로 만들어서 c(), d() 형식으로 그냥 메서드 넣어주면 된다.

ioc 컨테이너는 어셈블러, 오브젝트 팩토리라고 불리기도 한다..