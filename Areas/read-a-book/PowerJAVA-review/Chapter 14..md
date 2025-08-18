람다식

자바8에서 함수가 1급 객체가 되었음.
1급 객체의 특징?
- 변수에 저장 가능
- 매개 변수로 받을 수 있음
- 반환 가능

함수가 1급 객체가 되면서 스트림 API 쓰기 용이해졌다.
스트림 API 사용시 병렬 프로그래밍 쉽게할 수도 있음.

람다식
익명함수
`(arguments) -> (body)` 구문으로 작성됨.
0개 이상 매ㅑ개변수 가짐
매개 변수 형식을 명시적으로 선언 가능.
(int a)는 (a)와 동일.
단일 매개변수, 타입이 유추 가능한 경우 괄호 사용할 필요 없다.
i.e. `a -> return a*a`와 같은 표현
body에 하나 이상의 문장 존재 시 중괄호

어디서 쓰임?
함수 인터페이스의 컨텍스트
메서드를 다른 메서드에 전달해야할 때 등

GUI 작성 시 이벤트 리스너에 사용함.
스레드 작성시 runnable 인터페이스 구현하는 것을 간단하게 무명 메소드로만 정의해 thread 클래스로 전달함.

---

동작 매개 변수화(behavior parameterization)
610p

이 기법은 **개발 패턴**임.
사용자 요구를 담은 코드 블럭 생성, 프로그램의 다른 부분에 전달하는 것
변화하는 요구사항 대응 및 최소한의 노력으로 구현 및 유지 관리가 간단한 방법으로 만들어진 기법

예제 코드
사과농장에서 사용할 어플리케이션에 추가될 기능으로 녹색사과 필터링을 만든다.

```
//Color 정의
enum Color {RED,GREEN}

//녹색사과 필터링
Public static List<Apple> filterGreenApples(List<apple> inven){
	List<Apple> result = new ArrayList<>();
	for(Apple a: inven){
		if(GREEN.equals(apple.getColor()){
			Result.add(apple);
            }
        }
		return result;
}
```

  
2.빨간사과 필터링 요청으로 색상 필터링 메소드로 변경

```
Public static List<Apple> filterApples(List<apple> inven, Color color){
	List<Apple> result = new ArrayList<>();
	for(Apple a: inven){
		if(apple.getColor().equals(color)){
			Result.add(apple);
            }
        }
		return result;
}
```

3. 무게에 따른 필터링 요청으로 메소드 변경

```
Public static List<Apple> filterApples(List<apple> inven, Color color
						, int weight, boolean flag){
	List<Apple> result = new ArrayList<>();
	for(Apple a: inven){
		if((flag&&apple.getCoor().equals(color))||
        	(!flag&&apple.getWeight()>weight)){
            result.add(apple);
        }
		return result;
}
```

**4. 동작 파라미터화**

참 또는 거짓을 반환하는 함수 프레디케이트를 사용해 선택조건을 결정하는 인터페이스 정의

```
//무거운 사과 선택하는 메소드
public class AppleHeavyWeightPredicate implements ApplePredicate{
	public boolean test(Apple apple){
    	return apple.getWeight() > 150;
        }
}

//녹색 사과 선택
public class AppleGreenColorPredicate implements ApplePredicate{
	public boolean test(Apple apple){
    	return GREEN.equals(apple.getColor());
        }
}

public class FilteringApples{
	public static void main(String.. args){
    	List<Apple> inven = Arrays.asList(new Apple(80,GREEN),
        				new Apple(120,RED),
            				new Apple(155,GREEN));
            
        List<Apple> heavyApples =
        	filterApples(inven, new AppleHeavyWeightPredicate());
        List<Apple> greenApples =
        	filterApples(inven, new AppleGreenColorPredicate());
       }
}

Public static List<Apple> filterApples(List<apple> inven, ApplePredicate p){
	List<Apple> result = new ArrayList<>();
	for(Apple a: inven){
		if(p.test(apple)){
            result.add(apple);
        }
		return result;
}
```

5. 익명클래스 사용

```
List<Apple> redApples = filterApples(inven, new ApplePredicate(){
	public boolean test(Apple apple){
    	return RED.equals(apple.getColor());
        }
}
```

6. 람다 표현식 사용

```
List<Apple> result = 
	filterApples(inven, (Apple apple)-> RED.equals(apple.getColor()));
```

예제의 경우, 모던 자바 인 액션이라는 서적에서 나옴

우리 책에도 예제가 있다. 더 짧고 쉬움
근데 쓰기 귀찮음...
#todo

무려 디자인 패턴으로 소개됨.

https://burningfalls.github.io/java/what-is-behavior-parameterization/

---

함수형 인터페이스
ActionListener, Comparator 같은 하나의 추상 메서드만을 가진 인터페이스들

람다식과 함수형 인터페이스는 불가분의 관계
람다식을 올바르게 컴파일하려면 반드시 함수형 인터페이스가 정의되어야 하며, 대응되는 함수형 인터페이스가 있어야 의미있음.
함수형 인터페이스는 람다식으로 구현 가능.

built-in 인터페이스들 (java.util.function)
- `Supplier<T>` - 유형 T 객체 반환
- `Consumer<T>` - 유형 T 객체의 특정 동작 수행
- `BiConsumer<T, U>` - 2개의 매개 변수를 가지는 Consumer 인터페이스
- `Predicate<T> `- 유형 T 입력 기반, Bool 값 반환
- `Function<T, R>` - 유형 T를 입력받고, 유형 R을 반환
- `BiFunction<T, U, R>` - 2개의 매개 변수를 가지는 Function 인터페이스

BiFunction, UnaryOperator, BinaryOperator 등도 존재함. 추가로 찾아보는 것이 좋을 것임.

---

메서드 참조

기존 메소드를 참조해 간단한 람다식을 만들 때 자주 사용

```
list.forEach(s -> System.out.println(s));
```
위 코드에서 람다식이 하는 일은 println 호출
호출되는 메서드만을 보낼 수 없을까에 대한 아이디어 발생
메서드 참조라는 개념 도입, 메서드 자체를 참조한다.
메서드 호출과는 다름.

메서드 참조에서는 메서드가 실행되면 안됨. 따라서 새로운 표기법을 만들었다.

```
System.out::println;
```
위와 같이 println()을 참조 가능하다.
람다식의 축약버전이라고 할 수 있음.

> 파이썬에서는 함수도 객체여서 바로 참조 가능했던 것 같은데 자바는 이런 방식 쓰는구만

메서드 참조 종류
4가지 종류 존재
- 정적 메서드 참조
- 특정 객체 인스턴스 메서드 참조
- 특정 유형 인스턴스 메소드 참조
- 생성자 참조
623페이지


---

스트림 API

여기서의 stream은 입출력이 아닌 ArrayList와 같은 컬렉션에서 시작되는 스트림을 의미한다.
입출력에서와 같이 스트림은 한 번에 하나씩 생성되고 처리되는 일련의 데이터임.
스트림 api 이용 시 메서드를 이용하여 입력 스트림에서 항목을 하나씩 읽고 처리한 후, 항목을 출력 스트림으로 쓸 수 있음. 한 메서드의 출력 스트림은 다른 메서드의 입력 스트림이 될 수 있다.

> 글만 봤는데 이해 못했다. 뎃?

스트림 소스가 중간 처리 연산을 겪고 종단 연산까지의 진행이 스트림 파이프 라인

UNIX OS에서는 예전부터 스트림이 많이 사용됨.
대부분 명령어들이 표준 입력에서 데이터 읽고 처리 후, 표준 출력으로 결과 내보냄.
이런 명령어를 파이프로 연결하면 복잡한 작업이 수행 가능했음.
java 8은 이 아이디어 기반으로 스트림 api 추가함.

> bash나 cmd의 파이프 붙여서 명령어 수행하는 것처럼 자바 stream api도 그 일환이라고 생각할 수 있을 것 같음...

장점
- 컬렉션에서 탐색 시 간단하게 처리 가능
- 큰 컬렉션을 효율적으로 처리하려면 멀티 코어 아키텍처를 활용하는 것이 좋음.
  parrarelStream() 사용 시 스트림 API가 자동으로 쿼리를 여러 개의 코어를 활용하는 코드로 분해함. 즉, 병렬 프로그래밍을 지원함.

스트림 연산
데이터 처리를 위한 각종 연산을 제공함.

소스 생성 - 처리 연산 - 종말 연산
3단계로 볼 수 있다.

생성 단계 : 스트림 객체 생성. 배열/컬렉션으로 생성 가능
처리 단계 : 입력 데이터를 출력 데이터로 가공
종말 단계 : 처리된 데이터를 모아 결과를 만듬

각 단계별로 사용 가능한 메서드가 나뉨.
처리 연산의 경우, 상태 있음과 상태 없음이 구분된다.

메서드 종류는 구글 어시스턴트가 생성해줬다.
1. 중간 연산 (Intermediate Operations):

- **[map](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=map&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIChAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 각 요소에 함수를 적용하여 새로운 스트림을 생성합니다. 예를 들어, 문자열 스트림의 각 요소를 대문자로 변환할 수 있습니다.
- **[filter](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=filter&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQICxAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 조건에 맞는 요소만 포함하는 새로운 스트림을 생성합니다. 예를 들어, 짝수만 포함하는 숫자 스트림을 생성할 수 있습니다.
- **[sorted](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=sorted&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIDBAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 요소를 정렬하여 새로운 스트림을 생성합니다.
- **[distinct](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=distinct&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIEBAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 중복된 요소를 제거한 새로운 스트림을 생성합니다.
- **[limit](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=limit&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIDRAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 지정된 개수만큼의 요소만 포함하는 새로운 스트림을 생성합니다.
- **[skip](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=skip&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIDhAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 지정된 개수만큼의 요소를 건너뛰고 새로운 스트림을 생성합니다.
- **[flatMap](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=flatMap&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIERAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 각 요소를 Stream으로 변환하고, 이들을 하나의 Stream으로 평탄화합니다. 

2. 최종 연산 (Terminal Operations):

- **[forEach](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=forEach&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIJhAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 각 요소에 대해 주어진 동작을 수행합니다.
- **[reduce](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=reduce&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIJxAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 스트림의 요소들을 결합하여 결과를 생성합니다. 예를 들어, 모든 숫자의 합을 계산할 수 있습니다.
- **[collect](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=collect&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIKBAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 스트림의 요소들을 컬렉션으로 수집합니다. 예를 들어, 리스트나 맵으로 수집할 수 있습니다.
- **[count](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=count&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQILBAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 스트림의 요소 개수를 반환합니다.
- **[min](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=min&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIKhAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 스트림에서 최소값을 반환합니다.
- **[max](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=max&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIKxAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 스트림에서 최대값을 반환합니다.
- **[anyMatch](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=anyMatch&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQILRAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 조건에 맞는 요소가 하나라도 있는지 확인합니다.
- **[allMatch](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=allMatch&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQILhAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 모든 요소가 조건에 맞는지 확인합니다.
- **[noneMatch](https://www.google.com/search?cs=1&sca_esv=5ae40a1c31d56792&sxsrf=AE3TifPOhwQDkbfarcVMr3F3E20x_8j_xQ%3A1755506375128&q=noneMatch&sa=X&ved=2ahUKEwjXtIrv-pOPAxVZUGwGHRVtEtoQxccNegQIKRAB&mstk=AUtExfD1QVh-DeBORqJhGl8ISnp3lePT3gk7jihDvVzryPgvpYcQM2DqSQXaawom2QSLR096v3I6-UtJLcWO6Dk6JXRGMOmQmBl_iVBNX8oqmPzV4X20UM__ybJmcjeiL-U2XA6QsA5htS5YCa9bJ1r8gHWo3PxM11MRgPHeuzEkUr1ynd-25zUL066BmihUgsXlwOo-UU8tVSNubXw_2JGF7fUb_0dYL54WiJzAC5fCpn-HdxZ0dTYGdchHVxcd5eSNKMYrNBgHDS0YTo4TySxfHq7VQpwte1ccAtcj_T0gTGBltg&csui=3)():** 조건에 맞는 요소가 없는지 확인합니다.

