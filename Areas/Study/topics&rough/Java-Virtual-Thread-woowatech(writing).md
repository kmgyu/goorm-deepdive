https://www.youtube.com/watch?v=BZMZIM-n4C0&list=PLp4Yd2HKNbKvyTNhmxDIaPZDGWdHKkER-&index=31

https://tech.kakaopay.com/post/ro-spring-virtual-thread/

https://mangkyu.tistory.com/317
# 1. Virtual Thread 소개

전사 게이트웨이 시스템의 구조


![[virtual_thread1.png]]


안정성과 처리량에 대한 고려 필요
전사 게이트웨이 시스템은 타 시스템의 앞부분에 위치하여 사용자 인증과 같은 전처리 수행, 결과를 뒷단 시스템으로 흘려보낸다.

다양한 시스템의 트래픽이 한 곳에 몰리게 된다.
높은 트래픽 발생, 높은 처리량을 가질 수 있는 기술 고려가 필요함.

선택지
- 코틀린 코루틴
- 자바 프로젝트 Loom
  정식 feature 아님
  레퍼런스 부족
  구성할 수 있는 프로젝트 부족

코틀린 코루틴

JDK21의 가상 스레드가 정식 feature로 추가됨
이것을 딥다이브 했다


2018년 Project Loom으로 시작된 경량 스레드 모델
2023년 JDK21에 정식 feature로 추가

- 스레드 생성 및 스케줄링 비용이 기존 스레드보다 저렴
- 스레드 스케줄링을 통해  Non-blocking I/O 지원
- 기존 스레드 상속하여 코드 호환


## 저렴한 스레드 생성 비용

기존 자바 스레드는 생성 비용이 크다
따라서 미리 스레드를 생성해 스레드 풀에 저장해준다.
스레드 풀의 존재 이유

사용 메모리 크기가 크다.
최대 2MB를 사용하게 되면서 많은 메모리 차지

OS에 의해 스케줄링됨
스케줄링 과정에서 항상 OS와 통신하기 때문에 System call이 발생함
따라서 시스템 콜 오버헤드 존재

> System call
> 자바에서 커널 영역의 기능을 사용할 때 호출되는 인터페이스


Virtual Thread는 생성 비용 작다.

- 스레드 풀 개념 존재하지 않음
  요청 올 때마다 생성 및 파괴
- 사용 메모리 크기가 작다
  수십 kb 까지만 사용하도록 설계됨
- OS가 아닌 JVM 내 스케줄링
  시스템 콜로 인한 오버헤드 발생하지 않음.

장점

|             | Thread | Virtual Thread |
| ----------- | ------ | -------------- |
| 메모리 사이즈     | ~2MB   | ~50KB          |
| 생성 시간       | ~1ms   | ~10μs          |
| 컨텍스트 스위칭 시간 | ~100μs | ~10μs          |

으심. 시행해본다.

가설
Virtual Thread는 생성 및 실행이 빠르다.

실험방법
스레드 1,000,000개 실행

```java
public static void main(String[] args) {
	List<Thread> threads = IntStream.range(0, 1_000_000)
		.mapToObj(i -> new Thread(() -> {})) // do nothing
		.toList();
	
	threads.forEach(Thread::start);
	
}
```



```java
// 가상 스레드 버전
public static void main(String[] args) {
	List<Thread> threads = IntStream.range(0, 1_000_000)
		.mapToObj(i -> new Thread.ofVirtual().unstarted(() -> {})) // do nothing
		.toList();
	
	threads.forEach(Thread::start);
	
}
```

우테코에서는 각각 31.632 초 소요, 0.375초 소요
약 98.8%의 성능 개선이 이루어짐

실제로 스레드 생성 속도가 빨라졌다.

메모리 크기/시스템 콜 때문인지는 확인 불가.


## Non-blocking I/O 지원

증가하는 I/O blocking time
Thread per request 모델에서 특히 병목이 되는 blocking time

> TODO : 블로킹, 논블로킹 io 그림 다시 가져올 것

![[virtual_thread_blokingio.png]]
blocking i/o

 ![[virtual_thread_nonblokingio.png]]
non-blocking i/o

blocking time을 획기적으로 줄여주기 때문에 굉장히 중요한 패러다임으로 자리잡았다.

spring webflux가 대표적
netty의 이벤트 루프를 활용, 블로킹 i/o의 시간을 획기적으로 줄임
블로킹 i/o 시간을 줄여 사용 대비 높은 퍼포먼스를 보일 수 있다.

virtual thread 또한 지원하나, webflux의 방식과는 많이 다름.

두 가지 개념을 활용해 논블로킹을 구현
- JVM 스레드 스케줄링
  앞에서 설명?
- Continuation 활용

경량 스레드를 처음보는 사람은 생소할 것.
이 두가지 요소도 뒤에서 살펴볼 것



실험 2
가설
Virtual Thread로 구성된 서버는 nonblocking io로 동작한다.

실험방법
Tomcat 스레드 10개
10초 소요 API 호출
100회 동시 호출
*QUIZ?*
소요시간은 100초 예상됨

실제 결과
일반 Thread
130초

Virtual Thread
10초

결론
기존 스레드 대비 92.2%
I/O 블로킹 요청을 동시 처리
즉, non blocking i/o로 작동한다고 예측 가능


## 기존 스레드 상속

기존 스레드를 상속해서 기존 자바 앱에 완벽하게 호환되도록 설계됨

VirtualThread는 BaseVirtualThread를 상속하고, BaseVirtualThread는 Thread를 상속함.

SOLID - LSP에 따르면 Thread를 사용하는 곳에선 완벽하게 호환되어야 한다.

```
@Bean
public ExecutorService executorService() {
	return Executors.newVirtualThreadPerTaskExecutor()
}
```
executorService 또한, newVirtualThreadPerTaskExecutor()와 같이 단순히 버추얼스레드의 ExecutorService로 바꾸는 것으로 사용가능

리액티브나 코루틴과 같은 새로운 것을 배울 필요 없이 바로 적용 가능한 장점이 된다.

JDK 19에서는 prev-feature로 fiber라는 클래스로 만들어져있었음.
자바가 스레드를 상속하는 방법으로 바뀌면서 추상화에 얼마나 진심인지 알 수 있다.


중간 요약

스레드 생성/스케줄 속도
논블로킹 io
thread 상속


---

# 2. Virtual Thread 동작 원리

## 일반 스레드 특징

Thread
- 플랫폼 스레드 (또는 유저 스레드)
- OS에 의해 스케줄링
- 커널 스레드와 1:1 매핑
- 작업 단위로 Runnable 사용

어플리케이션은 커널과 유저 영역으로 실행됨

유저 영역에서 커널 영역을 사용하기 위해선 커널 영역(OS), 유저 영역(JVM) 사이를 통신하는 과정이 필요하다.
이를 위해 Java Native Interface (JNI)를 사용해야 한다.
각 스레드는 1대1로 매핑된다.

![[virtual_thread_thread_jni.png]]

![[virtual_thread_thread_jni_temp.png]]
*그림 원래 3단계로 나뉨. 직접 그리거나 가져올 것...*

플랫폼 스레드 생성 시
1. 유저 영역에 생성된다.
2. JNI를 통해 커널 스레드 생성을 요청한다.
3. 커널 스레드 생성 및 스케줄링

기존 코드의 `Thread::start`를 참고.
```java
public static void main(String[] args) {
	List<Thread> threads = IntStream.range(0, 1_000_000)
		.mapToObj(i -> new Thread(() -> {})) // do nothing
		.toList();
	
	threads.forEach(Thread::start);
	
}
```

start 메서드에 들어가보면 `start0()`라는 메서드를 통해 커널 스레드를 생성 요청함.

더 들어가면
`private native void start0();` 라는 메서드 확인 가능
이를 통해 JNI를 이용하는 것을 확인 가능

Virtual Thread
- 가상 스레드
- JVM에 의해 스케줄링
- 캐리어 스레드와 1:N 매핑
- 작업 단위 Continuation

JVM과 캐리어 스레드 먼저 살펴보자.

![[virtual_thread_temp0.png]]
VirtualThread 주석 보면 JVM에 의해 스케줄링 된다는 것을 확인 가능

그림으로 살펴보기

Virtual Thread 생성 시 유저 영역에 생성됨
시작 시 기존에는 커널 영역에 직접 스레드 생성을 요청했었다.
어떠한 스케줄러가 존재한 다는 것을 예쌍 가능

코드로 확인

virtualThread의 start 메서드 살펴보자.
submitRunContinuation()이라는 코드 호출을 확인가능
thread를 실행하기 위해 task submit -> 작업 스케줄링 코드인 것을 확인 가능
들어가보면 모르는 필드가 존재한다.
scheduler, runContinuation
이 두개가 처음 보는 놈들
scheduler는 JVM 내 가상 스레드 스케줄링 담당이라고 예상 가능
필드 선언부로 가보면 Executor 타입의 scheduler가 있는 것을 볼 수 있음.

이 값이 어떤 값으로 처리되는지 확인하기 위해 생성자로 넘어감

별도 파라미터로 받지 않을 경우, DEFAULT_SCHEDULER 사용


**스케줄러 특징**
static 변수로 선언됨, createDefaultScheduler()를 통해 생성됨
static 변수이기 때문에 모든 VirtualThread는 동일한 스케줄러를 공유

ForkJoinPool 메커니즘으로 스케줄링

**createDefaultScheduler 확인하기**
CarrierThread라는 것을 사용함. 워커 스레드(일반 스레드)
포크조인 워커 스레드
parallelism이라는 값을 사용가능한 프로세서 수로 설정. = 워커 스레드 수



따라서 포크조인 풀을 사용한다는 것을 알 수 있다
프로세서 수 만큼의 캐리어 스레드를 워커 스레드로 사용한다.

포크조인 풀은 Work Stealing 방식으로 작업을 수행

Work Stealing 매커니즘?
워커 스레드는 워커 큐를 각각 가지고 있고, 각각 태스크를 담아 순차적으로 처리하는 방식
본인 워커큐가 비어있을 경우, 남의 워커큐에서 태스크를 훔쳐옴.
따라서 워크 스틸링 방식



JVM 스케줄링 이유
굳이 이렇게 해야됨?
- 스레드는 생성 및 스케줄링 시 커널 영역 접근
  그 과정에서 시스템 콜 오버헤드 및 비용 발생
- 버추얼 스레드는 커널 영역 접근 없이 단순 Java 객체 생성
- 즉, Virtual Thread는 생성 시 시스템 콜 X


## Continuation 작업 단위

오래 전부터 사용되던 프로그래밍 패러다임

![[virtual_thread_kotlin_coroutine.png]]
[image from](https://techblog.woowahan.com/7349/)

코틀린 코루틴도 Continuation을 사용하도록 동작

코틀린 컴파일러가 suspend()를 코루틴을 사용하는 새로운 클래스로 실행하는 방식
일반적인 함수는 Caller가 호출하면 모든 라인 수행, 리턴 시 제어권 반환 및 종료

suspend()는 중단이 가능해짐
코루틴 개념을 적용해서 Caller가 호출하면 어느 정도 실행 중 중단 지점에서 Caller로 제어권 반환
Caller가 다시 진행, 코루틴은 중단지점을 기록해뒀다가 다시 진행
Caller로 반환 시 콜러는 다시 자기 일 했다가 코루틴 다시 줌. 이때 코루틴은 자기 작업을 다시 진행

Continuation의 기본적인 동작 방식
- 실행 가능한 작업 흐름
- 중단 가능
- 중단 지점으로부터 재실행 가능


```java
public void runnable() {
	System.out.println("start");
	
	System.out.println("running");
	
	System.out.println("end");
}
```
일반적인 runnable은 실행하면 끝까지 수행

```java
public void continuation() {
	System.out.println("start");
	// stop
	System.out.println("running");
	// stop
	System.out.println("end");
}
```

continuation은 중단 가능

![[virtual_thread_continuation_scope.png]]

Continuation은 관리를 위한 Scope가 존재함

Continuation1이 실행되는 과정이라고 보면 됨
실행 중 중단되면 작업 중단
stack memory의 스택 포인터에 기록 (cont1)
기록된 메모리를 힙으로 이동

실행 제어권을 호출자로 되돌림

Continuation2 실행 시 cont2가 스택으로 올라옴
cont2가 중단지점 도달 시 작업 중단, cont2(스택포인터)에 위치 기록

Continuation1을 다시 스택에 올리고 중단지점부터 다시 수행


예제
```java
public static void main(String[] args) {
	ContinuationScope continuationScope = new ContinuationScope("Virtual Thread");
	Continuation continuation1 = new Continuation(continuationScope, () -> {
	System.out.println("Continuation1 running 1");
	Continuation.yield(continuationScope);
	System.out.println("Continuation1 running 2");
	})
	
	Continuation continuation2 = new Continuation(continuationScope, () -> {
	System.out.println("Continuation2 running 1");
	Continuation.yield(continuationScope);
	System.out.println("Continuation2 running 2");
	})
	
	continuation1.run();
	continuation2.run();
	continuation1.run();
	continuation2.run();
}
```
yield를 통해 제어권이 main으로 돌아오고 main에서 수동으로 다시 실행하는 모습

virtualthread에서 continuation을 어떻게 사용하나?
필드로 Continuation cont;를 가지고 있다.
Runnable runContinuation이라는 필드도 가지고 있다. Continuation을 실행해주는 람다로 기억하면 된다.

생성자에서, cont는 VthreadContinuation이라는 타입으로 받은 태스크를 Continuation으로 만들어 사용한다.
runContinuation는 버추얼 스레드의 프라이빗 메서드로 runContinuation이 존재하여, 그것을 사용함.

runContinuation은 단순이 cont.run()을 통해 continuation을 실행한다.

*여기서 코드 까가지고 다 가져오기*


virtualThread 스타트할 때 submitRunContinuation을 실행했었다. 다시 돌아가서 확인해보자.
스케줄러는 포크조인풀, 런 컨티뉴에이션은 무엇인가라고 했었다.
단순히 Continuation을 실행하는 작업이다라는 것을 알 수 있다.


Continuation이 그림으로 어떻게 동작하는 지도 찾아보자.
워크큐에 런 컨티뉴에이션이라는 람다가 들어가는구나를 알 수 있다.

![[virtual_thread_temp1.png]]

yield를 명시적으로 주어야 할까?
언제 yield가 되는지 살펴보자

void park()를 통해 작업을 중단 시킨다.
이때 try문에서 yieldContinuation();이 실행된다.

Virtual Thread의 메서드는 패키지 프라이빗이기 때문에 외부 호출이 어려움.

자바 유틸 패키지의 LockSupport.park()가 이를 도와준다.
현재 스레드가 VirtualThread일 때 park를 호출한다.

이때 else문에서 U.park을 호출한다. 일반 스레드에서 호출할 시 이걸 사용함. U는 unsafe의 park 메서드를 의미함
기본적으로 일반 스레드에서 블로킹할 때 사용한다.

일반 스레드는 커널 스레드를 사용하기 때문에 일반 스레드를 block 시킬 시 커널 스레드도 같이 block시켜줘야 한다.
> 왜요?

그래서 native 붙음

Thread.sleep()
Mono.block()
CompletableFuture.get()
과 같이 스레드 블락시킬 시 쓰는 ㄴ메서드

기존 lockSupport는 호출 시 스레드를 블락시키는 함수였음.
그런데 버추얼 스레드 추가 이후(JDK21)로는 버추얼 스레드인 경우 버추얼 스레드의 park 사용하도록 변경
블로킹 되던 것이 버추얼 스레드 분기가 생기면서 버추얼 스레드 사용되고 있으면 continuation의 yield 를 호출한다는 것을 알 수 있다.

JDK17과 JDK21 살펴보면 다르다.

WorkQueue를 보면 Continuation1이 실행되고 있고, Cont1.yiled() 상황 가정해보자.
cont1이 중단되고 힙메모리로 넘어가서 워크큐에서 제거됨.
continuation1 중단 시 continuation2가 이어서 작업을 실행함
여기는 그림이 있었다.


Continuation 사용 이유
- Thread는 작업 중단을 위해 커널 스레드를 중단
- Virtual Thread는 작업 중단을 위해 continuation yield
- 작업이 block 되어도 실제 스레드는 중단되지 않고 다른 작업 처리 -> NIO처럼 동작
- 커널 스레드 중단이 없으므로 시스템 콜 x -> 컨텍스트 스위칭 비용이 낮음


중간요약 2
JVM에 의한 스케줄링
Continuation 작업단위

Virtual Thread는 어떤 스케줄러를 사용하더라?
포크조인 풀!

Continuation 중단 메서드 호출시에는?
yield(park)

![[Pasted image 20250908135954.png]]
커널 스레드 및 톰캣 스레드 2개 밖에 없는 상황 가정

그리고 다른 톰캣 및 DB, 혹은 api


2번째 리퀘스트가 I/O 블로킹 만나게 되면 locksupport park를 사용한다.
커널 스레드를 블락하게 될 것.

만약 세번째 리퀘스트가 들어오게 된다면 톰캣 스레드가 존재하지 않아 대기하게 될 것.

![[Pasted image 20250908140053.png]]
![[Pasted image 20250908140104.png]]

이후 두번째 리퀘스트 완료 후 3번째 리퀘스트가 할당받게 될 것

이것이 일반적인 thread per request 방식이다.

virtual thread로 변경하면 어떻게 될까?
한 라인이면 된다.

![[Pasted image 20250908140142.png]]


기존 스레드 모델과 비교해보자.
Request2를 받게되면 Virtual2가 마찬가지로 lock support의 park를 호출할 것이다.

![[Pasted image 20250908144436.png]]
continuation에 yield하고 재 스케줄링 해주면 된다.


![[Pasted image 20250908144445.png]]
자연스럽게 virtual 스레드는 api와 연동이 된다.

![[Pasted image 20250908144513.png]]
새로운 요청이 오게되면 생성되는 새로운 스레드는 캐리어 스레드에 등록되서 요청을 처리할 수 있게 된다.

![[Pasted image 20250908144551.png]]
만약 첫번째 요청의 response가 완료되면 자연스럽게 두번째 가상스레드를 할당해준다.
우리가 알고있는 non-blocking io와 비슷한 방식으로 동작한다!

Continuation + JDK 라이브러리 리팩토링 = Nonblocking

서버 환경에서 보여주었다.



# 성능 테스트

가상 스레드가 일반 스레드 대비 얼마나 성능 향상이 될까?
어떤 상황에서 적용해야 이점을 볼 수 있을까?

환경
구릴수록 성능 차이 극명
Thread vs VirtualThread
AWS EC2
- 1core cpu
- 1GB ram
Spring MVC
JVM heap 256mb
ngrinder
- vUser: 200
I/O bound / cpu bound
- api 호출
- cpu 연산
테스트의 경우 i/o, cpu로 나눠서 테스트함



I/O bound 작업은 기존 151%, cpu bound는 93% 성능의 TPS를 보임

일반 Thread 방식에 비해
- I/O bound 작업은 더 높은 처리량
  non blocking 방식이라.
- CPU bound 작업은 더 낮은 처리량
  platform thread 위에서 결국 동작.
  가상 스레드 스케줄링, 생성 비용이 낭비가 된다.
- Thread 서버는 특정 vuser수 부터 장애 발생
  최대 처리량에서 일반 스레드 서버가 낮다.


WebFlux vs Virtual Thread
AWS EC2
- 1core cpu
- 1GB ram
Spring WebFlux
JVM heap 256mb
ngrinder
- vUser: 500
api 호출

I/O Bound만 처리
VirtualThread는 211% 차이가 난다.

vUser 수가 일정 이상 넘어가면 WebFlux는 줄어들게 된다.
유저수가 많고 메모리도 낮기 때문에 컨텍스트 스위칭에서 병목이 걸린다.

극한 상황에서 차이점이 있다고 결론내릴 수 있음.

요약
I/O Bound 작업 효율이 높아진다.
제한된 사양에서 최대 처리량을 보일 수 있다.


# 주의사항

Blocking carrier thread
(Pin 현상)

> virtual thread가 carrier thread에 영구적으로 고정되는 현상.

캐리어 스레드를 block하면 Virtual Thread 활용 불가
- synchronized
- parallelStream

성능 상 손해를 볼 수 있음.

VM Optoin으로 감지 가능
-Djdk.tracePinnedThreads=short, full

이걸 지원하도록 만드는 방법이 있다?
virtual thread의 synchronized, parallelStream 사용 부분을 다른 락인 ReentrantLock을 사용하게 할 수 있다.
51:40 부근

- 병목 가능성 존재
- 사용 라이브러리 release 점검
- 변경 가능하다면 java.util의 ReentrantLock을 사용하도록 qusrud


No Pooling
- 생성 비용이 저렴하기 때문
- 사용할 때마다 생성
- 사용완료 후 GC

따라서 오히려 스레드를 풀링하는 것이 병목현상이 될 수 있음.
주석에서도 무제한으로 설정하도록 되어있다.

CPU bound task
- 결국 Carrier Thread 위에서 동작하므로 성능 낭비
- nonblocking의 장점을 활용하지 못함


경량 스레드
- 수백만개의 스레드 생성 컨셉
- Thread Local을 최대한 가볍게 유지해야 함
  무거운 객체를 넣을 시, Virtual Thread의 이점을 살릴 수 없다.
- 쉽게 생성 및 소멸
- JDK preview ScopedValue
  preview feature로 Thread Local을 대체하는 개념.
  살펴보면 좋을 수 있다.

배압
Virtual Thread는 배압 조절 기능이 없다.
- 유한 리소스의 경우 배압을 조절하도록 설정 (DB 커넥션, 파일)
  제한된 개수를 쓰는 것들은 DB 커넥션이 부족해서 문제가 되는 경우가 발생 가능하다.
- 충분한 성능테스트 필요

기존에는 스레드로 인해 강제로 배압조절이 되었었다. 스레드가 부족해지면 요청을 처리하지 못하니 자연스럽게 처리량이 줄어들어 배압이 조절됨.
그러나 가상 스레드는 서버가 가질 수 있는 최대치를 가지려 노력할 것임. 따라서 하드웨어 성능을 충분히 테스트하지 않고 사용 시 하드웨어 상에 문제가 발생할 수 있다.



# 결론

- Virtual Thread는 가볍고, 빠르고, nonblocking인 경량 스레드
- Virtual Thread는 JVM 스케줄링 + Continuation
- Thread per request 사용중이고, I/O blocking time이 주된 병목이 경우 고려 가능
- 쉽게 적용 가능
	- Reactive가 러닝 커브로 부담되는 경우
	- Kotlin coroutine이 러닝 커브로 부담되는 경우



# 질문들

## 코틀린 코루틴 vs 가상 스레드

가상 스레드는 결국 대기시간을 줄이기 위한 패러다임
코루틴은 루틴. 메서드나 함수라고 볼 수 있다. 메서드의 대기시간을 어떻게 줄이느냐에 대한 패러다임

서로 패러다임이 다르고, 가상 스레드의 경우 thread per request를 줄이기 위한 모델. 그래서 thread 단위로 작업을 쪼개서 스케줄링하는 방식을 사용함.
코루틴은 메서드 단위로 작업을 쪼개서 사용한다.
작업 단위가 그래서 작기 때문에 가상 스레드에 비해 더 높은 동시성을 가짐

코틀린의 경우 특정 코루틴을 종료, 캔슬 가능.
가상 스레드는 구조화된 동시성을 지원하지 않음.

단위, 상황에 맞게 사용해야 한다!


## 지금 사용중인 스레드 풀 모델에서 스레드 풀을 경량 스레드로 바꾸는 과정?

톰캣
tomcat protocol handler customizer로 사용 가능

스레드 풀을 바꾼다?
executor Service 또는 threadFactory를 만들면된다.

> 스프링부트 3.2+부터는 다음과 같이 application.properties를 지정해주면 사용해줄 수 있다.

```properties
spring.threads.virtual.enabled=true
```



## 가상 스레드를 실제 운영 환경에서 사용하거나 사용할 계획?

실행 환경은 안정성 중요. 하드웨어, 유한 리소스를 고려해야 하기 때문에 심도있게 테스트 후 적용이 필요함. 아직 안했대용


## WebFlux는 역사 속으로 사라지는가

웹플럭스도 패러다임이 다르다고 봐야 함.
함수형 프로그래밍 지원, 배압 조절 지원

각자 장단점이 존재함.
함수형 프로그래밍 적용해놓은 시스템에서는 더 좋은 프로그래밍 스타일이라고 봐야 한다.


### java virtual thread는 1 request당 1thread 할당. 그렇다면 가상 스레드 할당의 제한은?

제한 없음.
그래서 반드시 배압 조절해야함.
너무 많아지면 리소스, 처리량 등을 주의해야 함.


## DB cp를 주의해야 하는데, 어떻게 해결?
> CP 자체가 세마포어 역할을 해서 락을 걸어준다는데, 테스트하거나 레퍼런스 찾아본 결과 딜레이가 되어서 익셉션이 터지는데 어떻게 해결?

애플리케이션 코드로 배압을 조절해야 할 듯
어떻게 적용할지는 고민해볼 영역
사용자도 배압의 고려가 필요


## Network I/O가 없는 서버에서 사용이 얼마나 많은 차이?

기본 값으로 써도 된다? -> 안티패턴
network io가 없다면 쓰면 안된다.


## 스레드 로컬처럼 가상 스레드의 컨텍스트 유지 기술?
priview feature의 scoped value


## 가상 스레드를 mysql jdbc driver와 같이 사용할 경우 오히려 성능 하락 가능, R2DBC처럼 별도 구현체 기다려야 하는지?
Mysql Connector에 PR이 된 것이 있다. (2024. 4. 23. 기준)
완전히 패러다임이 바뀌는 것이 아니기 때문에 Synchronized만 잘 개선이 되면 Mysql Driver에서도 사용 가능하다.


## Thread 100초 -> 가상 스레드 10초 개선은 io 처리가 더 가능해서 줄어든 것이라고 보면 되는지?
Non-blocking IO로 동작하기 때문에 다음값 기다리고 다음값 받고 해서 10초간 완료 되었다.


## 버추얼 스레드가 캐리어 스레드에 1:N 매칭된다는 것은 워킹 큐를 상정한 것인지?
실제로 워크 큐에 여러 개의 버추얼 스레드의 continuation이 담길 수 있음.
그래서 워크 큐가 있으면 여러개의 continuation이 있으면 순서대로 처리한다.
그래서 1:N임.


## 가상 스레드 활용할 경우 작업 별 다른 스레드 사용할 수 있을 것 같은데, 해당 경우 스레드 로컬 활용 불가?
스레드 로컬 활용 가능
결국 캐리어 스레드에 스레드로컬을 담아둠

## 메모리 관리에 있어서 여러 작업들이 스케줄이 되면 JVM은 gc로 관리하기 때문에 OS 가상 메모리 활용이 제한될 것 같은데, 메모리적으로 어려운 점 존재하는지?
일반 스레드 대비 수백분의 1 사이즈. 훨씬 많이 만들 수 있다.
아무리 작아도 무제한으로 만들면 메모리를 많이 사용. 적절한 배압 조절 필요

