
사전 지식

| 구분      | blocking                                                                                   | non-blocking                                                                                                |
| ------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| **동기**  | **동기 + blocking**전통적인 방식. 메소드 호출 → 스레드가 결과 나올 때까지 멈춤.예: `read()` 호출 시 데이터가 도착할 때까지 대기      | **동기 + non-blocking**결과를 기다리긴 하지만, 준비 안 되어 있으면 즉시 리턴(EAGAIN). 호출자가 루프 돌며 재시도(polling).                      |
| **비동기** | **비동기 + blocking**콜백은 있지만 내부적으로 blocking I/O를 별도 스레드에 맡겨서 처리.예: 스레드풀에서 블로킹 호출 후 콜백으로 결과 반환 | **비동기 + non-blocking**가장 이상적인 고성능 방식. 스레드가 묶이지 않고, 완료 시 이벤트/콜백으로 알려줌.예: epoll + 콜백, 코틀린 코루틴, Node.js 이벤트 루프 |
![sync-async-blocking-nonblocking](https://blog.kakaocdn.net/dna/1HN9m/btsakdV6qwm/AAAAAAAAAAAAAAAAAAAAAAGW3OS731SSnlXrlG5h1ohZuq0jlsKK9MDAGius_RLk/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&allow_ip=&allow_referer=&signature=0jja0fIYBgSZvULQVqmXnhNMqCI%3D)
[image from](https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC) 동기/비동기, blocking/non-blocking

# 소개

만약 운영중인 서비스가 현재 감당할 수 있는 트래픽을 초과한다면 어떻게 대응할 수 있을까?
1. 스케일 아웃, 스케일 업 등으로 1차 대응
2. 병목 현상 확인 및 해결

> MSA 아키텍처와 같은 분산 환경에서는 항상 스레드에 대한 고민을 안고 가게됨.

정보 처리 기사에서 스레드 수준의 구분
- 커널 수준 스레드(KLT, Kernel Level Thread)
- 사용자 수준 스레드(ULT, User Level Thread)

자바에서는 OS가 관리하는 native 스레드를 제공한다.

따라서 하드웨어에 종속적이며, 시스템 콜과 컨텍스트 스위칭을 발생시킨다.

> System call
> 자바에서 커널 영역의 기능을 사용할 때 작업을 요청하는 것
> JNI (java natural interface)가 중계함.

이런 비용을 최소화하기 위해 나타난 대안이 경량 스레드 모델이다.

![[light-weight-thread-model.png]]
*[image from](https://medium.com/@Frevib/threads-vs-lightweight-threads-f85b9b43a684), 경량 스레드 모델*

자바의 가상 스레드는 경량 스레드 모델을 JDK 레벨에서 구현한 것


**가상 스레드**
기존 스레드의 컨텍스트 스위칭 비용과, 시스템 콜 시간을 줄여 성능을 높인 경량화된 스레드


# 특징
2018년 Project Loom으로 시작된 경량 스레드 모델
2023년 JDK21에 정식 feature로 추가

- 기존 스레드에 비해 저렴한 생성 및 컨텍스트 스위칭 비용
- 스레드 스케줄링을 통한 Non-blocking I/O 지원
- 기존 스레드 상속을 통한 코드 호환

---

## 저렴한 생성 비용

### 기존 스레드

생성 시 비용 문제
- 스레드 풀의 존재 이유

메모리 점유율 문제
- 스레드 크기 (스택 사이즈) 의 경우, 2mb (windows 1mb)를 점유. 많은 메모리를 차지

OS에 의한 스케줄링
- 스케줄링 과정에서 항상 OS와 통신, System call 발생
  -> System call 오버헤드 존재

### 가상 스레드

저렴한 생성 비용
- 스레드 풀 개념이 존재하지 않음
- 매 요청마다 스레드 생성 및 파괴

메모리 점유율 감소
- 수십 kb까지만 사용하도록 설계됨

JVM 내 스케줄링
- 시스템 콜로 인한 오버헤드 발생하지 않음.


### 비교표

|             | Thread | Virtual Thread |
| ----------- | ------ | -------------- |
| 메모리 사이즈     | ~2MB   | ~50KB          |
| 생성 시간       | ~1ms   | ~10μs          |
| 컨텍스트 스위칭 시간 | ~100μs | ~10μs          |

### 테스트를 통한 검증

```java
public static void main(String[] args) { // native thread
	List<Thread> threads = IntStream.range(0, 1_000_000)
		.mapToObj(i -> new Thread(() -> {})) // do nothing
		.toList();
	
	threads.forEach(Thread::start);
	
}
```

```java
// 가상 스레드 버전
public static void main(String[] args) { // virtual thread
	List<Thread> threads = IntStream.range(0, 1_000_000)
		.mapToObj(i -> Thread.ofVirtual().unstarted(() -> {})) // do nothing
		.toList();
	
	threads.forEach(Thread::start);
	
}
```

![[virtual-thread-test1.png]]
![[virtual-thread-test2.png]]

속도가 확연히 차이나는 것을 볼 수 있다.

> *참고*
> 로컬에서 돌리는 한계로 인해 스레드는 10만개, 가상 스레드는 100만개임.

메모리 크기/시스템 콜 때문에 성능 향상이 일어났는지지 현재 단계에서는 확인 불가


---

## Non-blocking I/O 지원


![sync-async-blocking-nonblocking](https://blog.kakaocdn.net/dna/1HN9m/btsakdV6qwm/AAAAAAAAAAAAAAAAAAAAAAGW3OS731SSnlXrlG5h1ohZuq0jlsKK9MDAGius_RLk/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&allow_ip=&allow_referer=&signature=0jja0fIYBgSZvULQVqmXnhNMqCI%3D)
[image from](https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC) 동기/비동기, blocking/non-blocking

Blocking time은 I/O 시간이 길어질 수록 그만큼 늘어나게 됨.
Blocking time은 Thread per request 모델에서 특히 병목 현상의 원인.

Non-Blocking I/O는 이런 Blocking time을 줄여주기 때문에 굉장히 중요한 패러다임

JVM 진영에서는 Spring Webflux, Kotlin Coroutine이 대표적


> Thread per request 모델
> 요청마다 스레드를 생성하는 방식

> Blocking time
> 스레드가 특정 연산 수행을 위해 대기하는 시간


![event-loop](https://blog.kakaocdn.net/dna/c0uk4N/btr5QptBybu/AAAAAAAAAAAAAAAAAAAAAK0qb6SA70_1NxkC7FMWzh4Uqs0x-2ferFKjU2bgNLN3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&allow_ip=&allow_referer=&signature=EO6cy2aeyJ%2FWCjzu7QOmC3qFpRo%3D)
[image from](https://gnidinger.tistory.com/entry/WebFlux%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84-Netty-%EA%B7%B8%EB%A6%AC%EA%B3%A0)

Spring Webflux의 경우 netty의 이벤트 루프를 활용해 Blocking I/O의 시간을 획기적으로 줄인다.

가상 스레드의 경우, Webflux의 방식과는 다른 방식으로 Non-Blocking I/O를 지원

가상 스레드는 두 가지 개념을 활용해 Non-Blocking I/O를 구현함.

- JVM 스레드 스케줄링
- Continuation 활용


### 테스트를 통한 검증
#todo
Virtual Thread로 구성된 서버는 nonblocking io로 동작한다.

실험방법
Tomcat 스레드 10개
10초 소요 API 호출
100회 동시 호출

실제 결과
일반 Thread
130초

Virtual Thread
10초

결론
기존 스레드 대비 92.2%
I/O 블로킹 요청을 동시 처리
즉, non blocking i/o로 작동한다고 예측 가능

---
## 기존 스레드 상속

![[virtual-thread-hierarchy.png]]
VirtualThread의 계층도
*Thread - yield()에서 VirtualThread 클래스를 확인 가능. package-private로 내부적으로만 구현하도록 숨겨져있다.*

기존 스레드를 상속해서 기존 자바 앱에 완벽하게 호환되도록 설계됨

VirtualThread는 BaseVirtualThread를 상속하고, BaseVirtualThread는 Thread를 상속함.

SOLID - LSP에 따르면 Thread를 사용하는 곳에선 완벽하게 호환되어야 한다.

```java
@Bean
public ExecutorService executorService() {
	return Executors.newVirtualThreadPerTaskExecutor()
}
```
executorService 또한, newVirtualThreadPerTaskExecutor()와 같이 단순히 가상 스레드의 ExecutorService로 바꾸는 것으로 사용 가능하다.

> Executor Service?
> ExecutorService는 작업(Runnable, Callable) 등록을 위한 인터페이스이다. ExecutorService는 Executor를 상속받아서 작업 등록 뿐만 아니라 실행을 위한 책임도 갖는다.

![executor service](https://blog.kakaocdn.net/dna/bcGfch/btrGEhU3Y4W/AAAAAAAAAAAAAAAAAAAAAHIBH_XZAX-UYQJ6sMtiKvZ88jUdq0brBdyMFN0rCh0H/img.gif?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&allow_ip=&allow_referer=&signature=hOW0FlCyUhinUKoc4AtroV8IeuY%3D)
[image from](https://mangkyu.tistory.com/259)

![[virtual-thread_ExecutorService.png]]

Spring Webflux의 리액티브, 코틀린의 코루틴과 같은 새로운 것을 배우지 않고 바로 적용할 수 있어 학습 곡선이 낮은 것이 장점이 된다.


JDK 19에서는 prev-feature로 fiber라는 클래스로 만들어져 있었다.
*스레드를 상속하는 구조로 바뀌면서 패러다임과 개발 철학에 얼마나 진심인지 알 수 있다.*




---

# 동작 과정

Java Thread
- 플랫폼 스레드 (또는 유저 스레드)
- OS에 의해 스케줄링
- 커널 스레드와 1:1 매핑
- 작업 단위로 Runnable 사용

어플리케이션은 커널과 유저 영역으로 나뉘어 실행됨

유저 영역에서 커널 영역을 사용하기 위해선 커널 영역(OS), 유저 영역(JVM) 사이를 통신하는 과정이 필요하다.
이를 위해 JNI(Java Native Interface)를 사용함.
각 스레드는 1대1로 매핑된다.

![[virtual-thread_java-platform-thread.png]]
*[Image from](https://velog.io/@zenon8485/Java21-Virtual-Thread) 기존 스레드 구조*

플랫폼 스레드 생성 시
1. 유저 영역에 생성된다.
2. JNI를 통해 커널 스레드 생성을 요청한다.
3. 커널 스레드 생성 및 스케줄링

테스트 코드의 `Thread::start`를 참고.
```java
public static void main(String[] args) {
	List<Thread> threads = IntStream.range(0, 1_000_000)
		.mapToObj(i -> new Thread(() -> {})) // do nothing
		.toList();
	
	threads.forEach(Thread::start);
	
}
```

start 메서드에 들어가보면 `start0()`라는 메서드를 통해 커널 스레드를 생성 요청함.

![[virtual-thread-start.png]]
![[virtual-thread-start-native.png]]
`start0()` 메서드는 native 키워드를 사용함. native 키워드는 사용자의 
이를 통해 JNI를 이용하는 것을 확인 가능


가상 스레드
- 가상 스레드 (진짜임)
- JVM에 의해 스케줄링
- 캐리어 스레드와 1:N 매핑
- 작업 단위 Continuation

---

## JVM에 의한 스케줄링


![[virtual-thread-comment.png]]
주석을 통해 OS 대신 JVM에 의해 스케줄링 된다는 것을 확인 가능


Virtual Thread는 생성 시 유저 영역에 생성됨
기존에는 시작 시 커널 영역에 직접 스레드 생성을 요청했었다.
따라서 커널 스레드와 가상 스레드 사이에서 관리를 해주는 어떠한 스케줄러가 존재한다는 것을 예상 가능

![[virtual-thread-architecture.png]]
*[Image from](https://velog.io/@zenon8485/Java21-Virtual-Thread) 가상 스레드 구조*

```java
/**  
 * Schedules this {@code VirtualThread} to execute.  
 * * @throws IllegalStateException if the container is shutdown or closed  
 * @throws IllegalThreadStateException if the thread has already been started  
 * @throws RejectedExecutionException if the scheduler cannot accept a task  
 */@Override  
void start(ThreadContainer container) {  
    if (!compareAndSetState(NEW, STARTED)) {  
        throw new IllegalThreadStateException("Already started");  
    }  
  
    // bind thread to container  
    assert threadContainer() == null;  
    setThreadContainer(container);  
  
    // start thread  
    boolean addedToContainer = false;  
    boolean started = false;  
    try {  
        container.onStart(this);  // may throw  
        addedToContainer = true;  
  
        // scoped values may be inherited  
        inheritScopedValueBindings(container);  
  
        // submit task to run thread  
        // 여기서 Continuation을 호출
        // 주석을 통해 여기서 스레드 실행과 태스크 submit 하는 것을 확인 가능
        submitRunContinuation();  
        started = true;  
    } finally {  
        if (!started) {  
            afterDone(addedToContainer);  
        }  
    }  
}
```

```java
/**  
   * Submits the runContinuation task to the scheduler. For the default scheduler,
   * and calling it on a worker thread, the task will be pushed to the local queue,
   * otherwise it will be pushed to an external submission queue.
   * @throws RejectedExecutionException  
 */
 private void submitRunContinuation() {  
    try {  
        scheduler.execute(runContinuation);  
    } catch (RejectedExecutionException ree) {  
        submitFailed(ree);  
        throw ree;  
    }  
}
```

주요 개념들의 등장
scheduler, runContinuation

scheduler는 JVM 내 가상 스레드 스케줄링 담당이라고 예상 가능
![[virtual-thread-field-chunk.png]]
*VirtualThread 필드 선언부*

Executor 타입의 scheduler가 있는 것을 볼 수 있음.


![[virtual-thread-constructor.png]]
이 값이 어떤 값으로 처리되는지 확인하기 위해 생성자로 넘어감

별도 파라미터로 받지 않을 경우, DEFAULT_SCHEDULER 사용


**스케줄러 특징**
static 변수로 선언
createDefaultScheduler()를 통해 생성

static 변수이기 때문에 모든 VirtualThread는 동일한 스케줄러를 공유

ForkJoinPool 메커니즘으로 스케줄링

**createDefaultScheduler 확인하기**
![[virtual-thread-create-default-scheduler.png]]
CarrierThread라는 것을 사용함.
캐리어 스레드는 일반 자바 스레드와 동일함. 워커 스레드임.
ForkJoinWorkerThread로 사용되는 것을 볼 수 있음.

parallelism이라는 값을 사용가능한 프로세서 수로 설정함.
즉, 이는 사용가능한 스레드 숫자임.


```java
// 잘린 parallelism 분기문
if (parallelismValue != null) {  
    parallelism = Integer.parseInt(parallelismValue);  
} else {  
    parallelism = Runtime.getRuntime().availableProcessors();  
}  
if (maxPoolSizeValue != null) {  
    maxPoolSize = Integer.parseInt(maxPoolSizeValue);  
    parallelism = Integer.min(parallelism, maxPoolSize);  
} else {  
    maxPoolSize = Integer.max(parallelism, 256);  
}
```


따라서 포크조인 풀을 사용한다는 것을 알 수 있다
프로세서 수 만큼의 캐리어 스레드를 워커 스레드로 사용한다.

> ForkJoinPool?
> Work-Stealing 알고리즘을 기반으로 하는 특별한 유형의 스레드 풀


> Work Stealing 매커니즘?
> 워커 스레드는 워커 큐를 각각 가지고 있고, 각각 태스크를 담아 순차적으로 처리하는 방식
> 본인 워커큐가 비어있을 경우, 남의 워커큐에서 태스크를 훔쳐옴.
> 따라서 워크 스틸링 방식


JVM 스케줄링의 이유
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

Continuation은 관리를 위한 Scope가 존재

현재 그림은 Continuation1이 실행되는 과정
1. Continuation1이 진행, Runnable1 실행
2. yield()로 작업 중단
	1. stack memory의 스택 포인터에 현재 작업 내용 기록(cont1)
	2. cont1은 Heap 메모리로 내려감
3. 실행 제어권을 호출자로 전환
4. Continuation2가 제어권을 받음
	1. cont2가 스택 메모리로 올라옴
5. cont2가 중단지점 (yield()) 도달 시 작업 중단
	1. stack memory의 스택 포인터에 현재 작업 내용 기록(cont2)
	2. cont2은 Heap 메모리로 내려감
6. Continuation1을 다시 스택에 올리고 중단지점부터 다시 수행


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


virtualthread에서 continuation을 사용하는 방식

![[virtual-thread-field-chunk.png]]
필드에 다음과 같은 변수를 가짐.
Continuation cont
Runnable runContinuation
runContinuation은 Continuation을 실행해주는 람다로 기억하면 된다.

생성자에서, cont는 VthreadContinuation이라는 타입으로 받은 태스크를 Continuation으로 만들어 사용한다.
runContinuation는 가상 스레드의 프라이빗 메서드로 runContinuation이 존재하여, 그것을 사용함.

![[virtual-thread-runContinuation.png]]
runContinuation은 단순히 cont.run()을 통해 continuation을 실행한다.


virtualThread 스타트 시 submitRunContinuation을 실행했었다.
JVM에 의한 스케줄링 섹션에 다시 돌아가서 확인해보자.


submitRunContinuation에서 사용하던 변수들인 scheduler는 ForkJoinPool 방식이었으며, RunContinuation은 알 수 없는 것이었다.
이제 단순히 Continuation을 실행하는 작업이다라는 것을 알 수 있다.


Continuation이 그림으로 어떻게 동작하는 지도 알아보자.
작업 큐에는 RunContinuation이라는 람다가 들어가게 된다.

![[virtual_thread_temp1.png]]

yield는 현재 작업을 중단하고 제어권을 반환하는 메서드이다.
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

그래서 native 붙음

Thread.sleep()
Mono.block()
CompletableFuture.get()
과 같이 스레드 블락시킬 시 쓰는 메서드

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














webflux vs virtual-thread
https://www.baeldung.com/java-reactor-webflux-vs-virtual-threads

virtual thread
https://0soo.tistory.com/259


우아한 테크블로그
https://techblog.woowahan.com/15398/#:~:text=Go%EC%9D%98%20goroutine%EC%9D%84%20%EC%95%84,%ED%83%80%EC%9E%84%EC%9D%84%20%EB%82%AE%EC%B6%94%EB%8A%94%20%EA%B0%9C%EB%85%90%EC%9E%85%EB%8B%88%EB%8B%A4.

카카오 테크블로그
https://tech.kakaopay.com/post/coroutine_virtual_thread_wayne/#:~:text=%EC%BD%94%EB%A3%A8%ED%8B%B4%EC%9D%80%20%EC%BD%94%ED%8B%80%EB%A6%B0,%EC%A0%81%EC%9D%80%20%EC%9E%90%EC%9B%90%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%A9%EB%8B%88%EB%8B%A4.