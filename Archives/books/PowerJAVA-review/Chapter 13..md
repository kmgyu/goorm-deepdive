제너릭 프로그래밍
다양한 종류의 데이터를 처리할 수 있는 클래스와 메소드를 작성하는 기법
Object 타입 변수를 사용하는 것보다 안전하고 사용하기 쉽다.

제너릭 사용 이전?
Object 타입으로 받아서 처리하는 다형성을 이용함.
Object 타입 변수가 해당 객체를 포인팅해준다.

이때 아래, 문제점 존재 해결책으로 등장
- 데이터 꺼낼 시 항상 형변환 필요
- 기존 데이터와 다른 형변환 가능, 오류 발생

`generic<T>`
형식작성. 이때 T는 타입 매개 변수라고 한다.

제너릭 메서드
```
public static <T> T getLast(T[] a) {
...
}
```

다음과 같이 일반 클래스 메서드에서도 타입 매개 변수를 사용해 제너릭 메서드를 정의 가능.
이 경우, 타입 매개 변수 범위는 메서드 내부로 제한됨

타입 매개 변수는 반드시 메서드의 수식자와 반환형 사이에 위치되어야 함.
제너릭 메서드 호출 위해서는 실제 타입을 꺾쇠 안에 적어줘도 되지만 그냥 일반 메서드처럼 호출해도 된다?

> 꺾쇠 생략은 기억해두는 게 좋을 듯...

---

컬렉션

자료구조
컬렉션은 제네릭 기법으로 구현됨
배열은 가장 기초적 형태의 컬렉션

컬렉션의 종류?
컬렉션 계층구조를 확인하라
책에서는 Queue, Deque, Stack 같은 것들 안 나옴. 다른 거 참고해야할듯...

특징
- 제너릭 사용
- 기초 자료형 저장 불가. 클래스만 사용 가능
- 기본 자료형 저장 시 오토 박싱 지원

Collection 인터페이스 메서드
> Map은 인터페이스 계층 구조에서 Collection이랑 다르게 되어있다!
> 그래서 다를 수 있다.

- isEmpty(), contains(), containsAll(`Collection<?> C`)
- add(), addAll(`Collection<? extends E> from`)
- remove, removeAll, retainAll, clear
- iterator(), stream(), parallelStream()
- size()
- toArray()

쓰레드도 지원하나봄. 쩐당

for문, for-each도 있는데, Iterator를 이용한 기법도 존재함.

```java
List<String> list = blahblahblah;
Iterator e = list.iterator();
while(e.hasNext()) {
	String s = (String) e.next();
}
```
- hasNext() - 미방문 원소 존재시 true
- next() - 다음 원소 반환
- remove() - 최근 반환된 원소 삭제
주요 메서드 3가지

---

ArrayList vs Vector
ArrayList vs LinkedList

---

Set?
TreeSet은 Red Black Tree를 이용해 구현함.
HashSet은 말그대로 해시 테이블로 구현함.
LinkedHashSet은 순서 불명확성을 제거함.
Ordered 느낌인듯

---
Map?

키-값을 하나의 쌍으로 묶어 저장하는 자료구조
python의 dictionary
c++에도 map이란 자료구조 존재

put(), get()
더 있는데 책에선 이것만 설명함.

얘도 아마 해싱이랑 희소배열 쓸거임.

---

Queue
구현체
ArrayDeque
LinkedList
PriorityQueue

Deque라는 인터페이스가 Queue를 상속받고 그 구현체가 ArrayDeque였던 것으로 기억.
LinkedList와 ArrayDeque 비교하면 성능 차이가 있었던 것으로 기억함.

---

Collections 클래스
정렬, 섞기, 탐색 알고리즘 가지고있는 유틸 클래스
팀 소트 사용했었나?
기본이 머지소트 기반으로 해서 안정 정렬하고 더 빠른 거 사용하려면 Sorting 하는 클래스였나 필요했던 거 같은데... 많이 안봐서 그런가 기억이 가물가물하다.

소팅 기준을 Comparable`<?>` 인터페이스로 정할 수 있다.
Comparator도 있는데 둘이 사용법이 달랐음... 둘이 해주는 건 똑같다...!

섞기(Shuffling)
자료구조 내의 원소를 랜덤하게 섞는다. 테스트할 때도 유용하게 사용 가능함.

탐색
이진 탐색은 진짜 유명한 탐색 알고리즘임
lowerbound, upperbound는 직접 구현해야 한다. 흑흑


