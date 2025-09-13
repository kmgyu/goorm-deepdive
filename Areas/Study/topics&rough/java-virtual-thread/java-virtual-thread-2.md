# 1편 내용 요약

## 1. 배경과 등장

- 자바는 기본적으로 OS 기반 네이티브 스레드(Kernel Level Thread)를 사용
- 문제점:
    - 높은 스레드 생성 비용과 메모리 점유율
    - OS 스케줄링 과정에서 시스템 콜 발생 → 컨텍스트 스위칭 비용 증가
        
- 대안: 경량 스레드 모델(Lightweight Thread Model)
- 가상 스레드(Virtual Thread): JDK 레벨에서 구현된 경량 스레드 모델

---

## 2. 주요 특징

### (1) 저렴한 생성 비용
- 기존 스레드: 생성·소멸 비용이 비싸서 스레드 풀 활용
- 가상 스레드: 생성/종료가 수십 μs 수준으로 가볍고, 메모리 점유도 약 50KB → 요청 단위로 스레드 생성 가능
    
| 항목       | 플랫폼 스레드 | 가상 스레드 |
| -------- | ------- | ------ |
| 메모리 크기   | ~2MB    | ~50KB  |
| 생성 시간    | ~1ms    | ~10μs  |
| 컨텍스트 스위칭 | ~100μs  | ~10μs  |

---

### (2) Non-Blocking I/O 지원

- Thread per request 모델: Blocking I/O 사용 시 I/O 대기 시간이 길 수록 치명적
    
- 가상 스레드:
    - JVM 수준 스케줄링 + Continuation 활용
    - 블로킹 호출을 중단(suspend) → 다른 스레드 실행 → 완료 시 재개(resume)


---

### (3) 기존 코드와의 호환성

- VirtualThread는 `Thread`를 상속 → LSP 원칙 충족
- 기존 Thread API와 완벽히 호환
- Spring Boot 3.2+에서 `spring.threads.virtual.enabled=true` 설정만으로 적용 가능

→ Webflux/Coroutine처럼 새로운 패러다임 학습 없이 Non-blocking 처리 가능

---

## 4. 정리

- Virtual Thread는 **가볍고 빠르며, Non-blocking I/O를 자연스럽게 지원하는 경량 스레드**
    
- 장점:
    - 저렴한 생성 비용 → 풀 필요 없음
    - JVM 스케줄링으로 System call 오버헤드 제거
    - 기존 코드와 높은 호환성 → 러닝 커브 낮음
- 단점:
	- 스레드 제한이 없어 메모리 관리 필요
	- CPU 작업을 많이 할 경우 장점이 사라짐.
        
- 적용 시점:
    - Thread per request 구조에서 I/O 블로킹이 문제인 경우
    - Webflux/Coroutine 러닝 커브가 부담되는 경우

---

# 학습 내용

오늘 알아볼 내용

JVM의 가상 스레드 스케줄링
가상 스레드의 작업 단위 : Continuation


# 동작 과정 살펴보기

## Java Thread

- 플랫폼 스레드 (또는 네이티브 스레드)
- OS에 의해 스케줄링
- 커널 스레드와 1:1 매핑
- 작업 단위 Runnable

어플리케이션은 커널과 유저 영역으로 나뉘어 실행됨

유저 영역에서 커널 영역을 사용하기 위해선 커널 영역(OS), 유저 영역(JVM) 사이를 통신하는 과정 필요
이를 위해 JNI(Java Native Interface) 사용
각 스레드는 1대1로 매핑

![[virtual-thread_java-platform-thread.png]]
*[Image from](https://velog.io/@zenon8485/Java21-Virtual-Thread) 기존 스레드 구조*

플랫폼 스레드 생성 시
1. 유저 영역에 생성된다.
2. JNI를 통해 커널 스레드 생성을 요청한다.
3. 커널 스레드 생성 및 스케줄링

커널 스레드 생성 과정을 자바에서 확인하기 위해 1편 테스트 코드의 `Thread::start`를 부분을 확인한다.
```java
public static void main(String[] args) {
	List<Thread> threads = IntStream.range(0, 1_000_000)
		.mapToObj(i -> new Thread(() -> {})) // do nothing
		.toList();
	
	threads.forEach(Thread::start);
	
}
```

start 메서드에 내부
`start0()`라는 메서드를 통해 커널 스레드를 생성 요청함.

![[virtual-thread-start.png]]
![[virtual-thread-start-native.png]]
`start0()` 메서드는 native 키워드를 사용함.
이를 통해 JNI를 이용하는 것을 확인 가능

### JNI

> 위에서 start0이라는 메서드가 native 키워드를 사용하는 것을 확인하였다. 그러나 실제 구현되는 코드는 확인해보지 않았다.

native 메서드의 실제 구현은 C/C++로 이루어지기 때문이다.

구현 코드
내부 구현 함수와 JNI 메서드를 매핑하는 코드
https://github.com/openjdk/jdk/blob/221e1a426070088b819ddc37b7ca77d9d8626eb4/src/java.base/share/native/libjava/Thread.c

start0 구현체 (2933 라인부터)
https://github.com/openjdk/jdk/blob/221e1a426070088b819ddc37b7ca77d9d8626eb4/src/hotspot/share/prims/jvm.cpp


#todo
스레드 C/C++ 코드 딥다이브
https://code-run.tistory.com/59

추가로 공부해볼 것?
CDS 아카이브
https://blog.igooo.org/123


## 가상 스레드

동작 방식
- JVM에 의해 스케줄링
- 캐리어 스레드와 1:N 매핑
- 작업 단위 Continuation


### JVM 스케줄링


![[virtual-thread-comment.png]]
주석을 통해 OS 대신 JVM에 의해 스케줄링 된다는 것을 확인 가능


Virtual Thread는 생성 시 유저 영역에 생성됨
기존에는 시작 시 커널 영역에 직접 스레드 생성을 요청했었다.
따라서 커널 스레드와 가상 스레드 사이에서 관리를 해주는 어떠한 스케줄러가 존재한다는 것을 예상 가능

![[virtual-thread-architecture.png]]
*[Image from](https://velog.io/@zenon8485/Java21-Virtual-Thread) 가상 스레드 구조*


#todo 
다시 올라가보면 기존 스레드들을 플랫폼 스레드라고 표현했다.
왜일까?
1:1로 커널 스레드와 매핑되는 실제 스레드이기 때문.
플랫폼 스레드는 스레드 풀로 관리된다.
*VirtualThreadFactory에서도 캐리어 스레드를 관리하는 코드가 있던 것으로 기억. 해당 코드 찾아서 삽입해두기*


VirtualThread의 구조를 알아보기 위해 코드 탐색 해본다.

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
이 값이 어떤 값으로 처리되는지 확인하기 위해 생성자에서 타입을 확인한다.

별도 파라미터로 받지 않을 경우, DEFAULT_SCHEDULER 사용


**스케줄러 특징**
static 변수로 선언
createDefaultScheduler()를 통해 생성

static 변수이기 때문에 모든 VirtualThread는 동일한 스케줄러를 공유

ForkJoinPool 메커니즘으로 스케줄링

**createDefaultScheduler 확인하기**
![[virtual-thread-create-default-scheduler.png]]
CarrierThread라는 것을 사용함.

#todo 캐리어 스레드 딥다이브

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


#### Fork/Join Framework

#todo https://burningfalls.github.io/java/what-is-fork-join-framework/
대규모 병렬 처리를 위해 설계된 고성능 멀티스레딩 프레임워크
앞서 등장한 포크조인 풀은 포크/조인 프레임워크의 컴포넌트이다.

Java 7에서 소개됨.
분할 정복 알고리즘을 기반으로 작업을 더 작은 작업으로 분할하고, 이를 병렬로 처리한 다음, 최종 결과를 결합하는 방식으로 작동
CPU 코어를 효율적으로 활용하여 복잡한 연산을 빠르게 처리하는 것을 가능케 함.

주요 컴포넌트
- ForkJoinPool
- ForkJoinTask

**ForkJoinPool** 
Work-Stealing 알고리즘을 기반으로 하는 특별한 유형의 스레드 풀
Fork/Join 프레임워크의 핵심

작업을 수행할 워커 스레드를 관리하며, 작업의 분할과 실행을 조정함.


**ForkJoinTask**
분할 가능한 작업
이 클래스의 인스턴스는 ForkJoinPool에 의해 실행된다.

핵심 메서드
- fork() : 작업을 비동기적으로 시행. 즉, 작업을 ForkJoinPool의 작업 큐에 추가하고 즉시 반환
- join() : 작업 결과가 준비될 때까지 기다린 다음, 결과 반환


**동작 방식**
1. 작업 분할(Fork)
복잡한 작업을 더 작은 작업으로 분할한다.
분할은 문제를 더 작게 만들어 해결 가능한 수준에 도달할 때까지 재귀적으로 발생한다.
    
2. 작업 실행
분할된 작업은 병렬로 실행된다.
각 스레드는 자신의 작업 큐를 갖고 있으며, 다른 스레드의 작업 큐에서 작업을 훔쳐올 수 있는 `work-stealing` 알고리즘을 사용한다.

3. 결과 결합(Join)
모든 하위 작업이 완료되면, 그 결과들은 최종 결과를 형성하기 위해 결합된다.


> **Work Stealing**
> 멀티스레드를 위한 병렬 컴퓨팅 스케줄링 전략
> OS를 기준으로 설명됨. 개념은 동일하다.
> #todo https://en.wikipedia.org/wiki/Work_stealing
> 각 프로세서는 작업 큐(Working Queue)를 각각 가지고 있으며, 각각 태스크를 담아 순차적으로 처리한다. 만약 해당 작업 실행 과정에서 다른 작업과 병렬로 처리될 수 있는 새로운 작업이 생성될 경우, 즉시 해당 프로세스 대기열에 추가된다.
> 만약 대기 상태의 프로세서가 존재할 경우, 다른 프로세서의 워커큐에서 태스크를 *훔친다.*
> 모든 프로세서가 작업 상태에 있다면 스케줄링 오버헤드는 발생하지 않는다. (즉, 훔치기는 발생하지 않는다.)
> 반대되는 전략으로 **Work-Sharing**이 있다.
> 스케줄러가 바쁜 프로세서에서 새로 생성된 작업(스레드 또는 태스크)을 활용도가 낮은 프로세서로 적극적으로 마이그레이션하여 부하를 분산하고 전체 응답 시간을 개선하려고 시도하는 병렬 컴퓨팅 스케줄링 패러다임이라고 한다.
> 관련 논문이 존재하나 자료가 충분히 존재하지 않음.
> #todo https://ieeexplore.ieee.org/document/7462221/

JVM 스케줄링의 이유
- 스레드는 생성 및 스케줄링 시 커널 영역 접근
  그 과정에서 시스템 콜 오버헤드 및 비용 발생
- 버추얼 스레드는 커널 영역 접근 없이 단순 Java 객체 생성
- 즉, Virtual Thread는 생성 시 시스템 콜 X

#todo JVM 스케줄링 시 메모리에선 어떤 일이 일어날까?
힙, 스택 그림으로 보여주기...




### Continuation 작업 단위

Continuation 개념 자체는 오래 전부터 사용되던 프로그래밍 패러다임
https://en.wikipedia.org/wiki/Continuation

> 컴퓨터 사이언스에서, Continuation은 컴퓨터 프로그램의 제어 상태에 대한 추상적 표현.
> LISP이라는 함수형 프로그래밍 언어와 Java의 가상 스레드, C++ Fiber 등에서도도 지원한다.

![[virtual_thread_kotlin_coroutine.png]]
[image from](https://techblog.woowahan.com/7349/)

코틀린 코루틴도 Continuation을 사용하도록 동작

코틀린 컴파일러가 suspend()를 코루틴을 사용하는 새로운 클래스로 실행하는 방식
일반적인 함수는 Caller가 호출하면 모든 라인 수행, 리턴 시 제어권 반환 및 종료

suspend()는 중단이 가능해짐
코루틴 개념을 적용해서 Caller가 호출하면 어느 정도 실행 중 중단 지점에서 Caller로 제어권 반환
Caller가 다시 진행, 코루틴은 중단 지점을 기록해뒀다가 다시 진행
Caller로 반환 시 콜러는 다시 자기 일 했다가 코루틴 다시 줌. 이때 코루틴은 자기 작업을 다시 진행

Continuation의 기본적인 동작 방식
- 실행 가능한 작업 흐름
- 중단 가능
- 중단 지점으로부터 재실행 가능

> 즉 일종의 작업 실행에 대한 컨텍스트를 나타내는 메타 데이터이다.

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

> #todo yield() 말고도 프로세스 실행을 위해 사용하는 여러 메소드들이 존재한다.
> 이 메소드들에 대해 선행적으로 간단히 설명하고, 내부 동작 과정을 설명할 것.
> 프로세스 생명주기 그림을 가져와서 각각 대응되는 개념들을 매칭시키는 것도 좋을 것 같음.

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
> #todo 작동을 안 합니다.
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


Continuation이 그림으로 어떻게 스케줄링 되는지 알아보자.
작업 큐에는 RunContinuation이라는 람다가 들어가게 된다.

![[virtual_thread_temp1.png]]

yield는 현재 작업을 중단하고 제어권을 반환하는 메서드이다.
yield를 명시적으로 주어야 할까? -> #todo 이 표현에 대해서는 좀 '더' 잘 설명해야 할 필요가 있음.
언제 yield가 되는지 살펴보자

void park()를 통해 작업을 중단 시킨다.
이때 try문에서 yieldContinuation();이 실행된다.

이때 Virtual Thread의 메서드는 패키지 프라이빗이기 때문에 외부 호출이 어려움.

자바 유틸 패키지의 LockSupport.park()가 이를 도와준다.
현재 스레드가 VirtualThread일 때 park를 호출하는 역할을 한다.

#todo LockSupport 클래스의 park 코드

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


#todo 중간요약 타임
JVM에 의한 스케줄링
Continuation 작업단위

Virtual Thread는 어떤 스케줄러를 사용하더라?
포크조인 풀!

Continuation 중단 메서드 호출시에는?
yield(park)

### 시나리오

시나리오를 가정하여 기존 스레드 모델과 가상 스레드 모델의 트랜잭션이 어떻게 스케줄링되는지 확인한다.

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

> 톰캣의 경우 프로토콜핸들러커스터마이저라는 빈을 수정해서 ExecutorService를 바꿔줘야 한다. 이를 충분히 인지시키기 바람.
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
환경의 스펙이 좋지 않을수록 성능 차이 극명
Thread vs VirtualThread
AWS EC2
- 1core cpu
- 1GB ram
Spring MVC
JVM heap 256mb
부하 테스트 도구 : ngrinder
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

> cpu 작업의 경우 일반 스레드 작업과 차이가 없다. 결국 non-blocking 작업의 경우 io 사용으로 인해 중단되면서 생긴 유휴시간을 최대한 활용하기 위해 쓰게 되는 것인데 cpu 사용량이 많은 작업의 경우 비동기 blocking과 같은 안티 패턴이 되어버린다.

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
- 변경 가능하다면 java.util의 ReentrantLock을 사용하도록 변경


> #todo 그렇다면 Pinned Thread 현상은 언제 생깁니까?


No Pooling
- 생성 비용이 저렴하기 때문
- 사용할 때마다 생성
- 사용완료 후 GC

따라서 오히려 스레드를 풀링하는 것이 병목현상이 될 수 있음.
주석에서도 무제한으로 설정하도록 되어있다.

> #todo 왜 스레드 풀을 설정하면 오히려 병목 현상이 생기는 것입니까? 어떤 곳에서 병목현상이 생기게 됩니까? 스레드 풀의 작동 방식이 어떤 것이기에 그런 것입니까?


CPU bound task
- 결국 Carrier Thread 위에서 동작하므로 성능 낭비
- nonblocking의 장점을 활용하지 못함
> 앞서 설명된 방법이다.

경량 스레드 특징 재확인                                                                                 
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

> #todo 배압 조절 기능이 무엇입니까?
> 기존 스레드는 어떤 알고리즘으로 배압을 조절합니까?
> 가상 스레드의 배압 조절을 할 수 있습니까? 아니면 배압 조절의 대안이 존재합니까?





# Reference

자바 스레드 딥다이브
https://code-run.tistory.com/59

고루틴
https://ykarma1996.tistory.com/188