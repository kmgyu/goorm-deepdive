
연관 지식

![[Pasted image 20250910124453.png]]
[image from](https://developer.ibm.com/articles/l-async/) IBM async


| 구분      | blocking                                                                     | non-blocking                                                              |
| ------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **동기**  | 메소드 호출 → 스레드가 결과 나올 때까지 멈춤.<br>예: `read()` 호출 시 데이터가 도착할 때까지 대기              | 결과를 기다리긴 하지만, 준비 안 되어 있으면 즉시 리턴(EAGAIN). <br>호출자가 루프 돌며 재시도(polling).     |
| **비동기** | 콜백은 있지만 내부적으로 blocking I/O를 별도 스레드에 맡겨서 처리.<br>예: 스레드풀에서 블로킹 호출 후 콜백으로 결과 반환 | 스레드가 묶이지 않고, 완료 시 이벤트/콜백으로 알려줌.<br>예: epoll + 콜백, 코틀린 코루틴, Node.js 이벤트 루프 |
![sync-async-blocking-nonblocking](https://blog.kakaocdn.net/dna/1HN9m/btsakdV6qwm/AAAAAAAAAAAAAAAAAAAAAAGW3OS731SSnlXrlG5h1ohZuq0jlsKK9MDAGius_RLk/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&allow_ip=&allow_referer=&signature=0jja0fIYBgSZvULQVqmXnhNMqCI%3D)
[image from](https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC) 동기/비동기, blocking/non-blocking

# 소개

 
스레드 수준의 구분
- 커널 수준 스레드(KLT, Kernel Level Thread)
- 사용자 수준 스레드(ULT, User Level Thread)


![[virtual-thread_java-platform-thread.png]]
자바는 OS가 관리하는 native 스레드를 제공

문제점
- 하드웨어에 종속적
- 시스템 콜과 컨텍스트 스위칭을 발생

해당 문제점은 스레드를 많이 사용할 수록 두드러짐

> System call
> 자바에서 커널 영역의 기능을 사용할 때 작업을 요청하는 것
> JNI (java natural interface)가 중계함.

이런 비용을 최소화하기 위해 나타난 대안 중 하나가 경량 스레드 모델

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

생성 비용이 저렴하다는 것을 확인 가능

> *참고*
> 로컬에서 돌리는 한계로 인해 스레드는 10만개, 가상 스레드는 100만개임.

메모리 크기/시스템 콜 때문에 성능 향상이 일어났는지 현재 단계에서는 확인 불가


---

## Non-blocking I/O 지원


![sync-async-blocking-nonblocking](https://blog.kakaocdn.net/dna/1HN9m/btsakdV6qwm/AAAAAAAAAAAAAAAAAAAAAAGW3OS731SSnlXrlG5h1ohZuq0jlsKK9MDAGius_RLk/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1759244399&allow_ip=&allow_referer=&signature=0jja0fIYBgSZvULQVqmXnhNMqCI%3D)
*[image from](https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC) 동기/비동기, blocking/non-blocking*

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

Virtual Thread로 구성된 서버는 nonblocking io로 동작한다. 해당 성능을 확인해본다.

실험방법
Tomcat 스레드 10개
10초 소요 API 호출
100회 동시 호출

*api*
```java
@RestController  
public class SlowController {  
  
    @GetMapping("/api/slow")  
    public String slow() throws InterruptedException {  
        Thread.sleep(10_000);  
        return "OK";  
    }  
}
```

*application properties*
```properties
spring.application.name=virtual_thread_example
server.port=8080

# Tomcat: 최대 워커 스레드 10개로 제한 (플랫폼 스레드 수)
server.tomcat.threads.max=10
server.tomcat.accept-count=100

# 스프링 MVC 가상 스레드 사용
spring.threads.virtual.enabled=true

# 로깅
logging.level.org.springframework.web=INFO
logging.level.org.apache.catalina.core=INFO
```

*test*
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)  
class VirtualThreadLoadTest {  
  
    @LocalServerPort  
    int port;  
  
    static RestClient restClient = RestClient.builder().build();  
  
    // if you want to test just blocking synchronous, just edit properties  
    @Test  
    void nonBlockingIo_under_10_tomcat_threads_and_100_parallel_calls() throws Exception {  
        String baseUrl = "http://localhost:" + port;  
        int parallel = 100;  
  
        ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();  
        List<CompletableFuture<ResponseEntity<String>>> futures = new ArrayList<>(parallel);  
  
        long started = System.nanoTime();  
        for (int i = 0; i < parallel; i++) {  
            CompletableFuture<ResponseEntity<String>> f = CompletableFuture.supplyAsync(() ->  
                    restClient.get().uri(baseUrl + "/api/slow").retrieve().toEntity(String.class), executor);  
            futures.add(f);  
        }  
  
        List<ResponseEntity<String>> results = futures.stream().map(CompletableFuture::join).toList();  
        long elapsedMs = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - started);  
  
        // 모든 응답이 200 OK 이고 바디가 OK 인지 확인  
        results.forEach(r -> {  
            assertThat(r.getStatusCode().is2xxSuccessful()).isTrue();  
            assertThat(r.getBody()).isEqualTo("OK");  
        });  
  
        // 벽시계 시간: 10초 + 약간의 오버헤드 내로 수렴하면 가상 스레드/논블로킹 처리로 판단  
        // 톰캣 워커 스레드가 10개인데도 전체가 10초 언저리로 끝나면 블로킹 워커가 병목이 아님을 관찰  
        System.out.println("Elapsed: " + elapsedMs + " ms");  
        assertThat(elapsedMs).isLessThan(15_000); // 여유치 포함  
  
        executor.shutdown();  
    }  
}
```

예상 결과
일반 Thread
100초

Virtual Thread
10초

로컬에서 실제 실행 결과
일반 Thread
101.56초 (101560 ms)

Virtual Thread
11.96초 (11965 ms)

결론
기존 스레드 대비 약 84.8%
I/O 블로킹 요청을 동시 처리
즉, non blocking i/o로 작동한다고 예측 가능

---
## 기존 스레드 상속

![[virtual-thread-hierarchy.png]]
VirtualThread의 계층도
*Thread - yield()에서 VirtualThread 클래스를 확인 가능. package-private로 내부적으로만 구현하도록 숨겨져있다.*

기존 스레드를 상속해서 기존 자바 앱에 완벽하게 호환되도록 설계됨

VirtualThread는 BaseVirtualThread를 상속하고, BaseVirtualThread는 Thread를 상속함.

SOLID - LSP원칙에 따르면 Thread를 사용하는 곳에선 완벽하게 호환되어야 한다.

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

JDK 19 prev-feature에서는 fiber라는 thread와 독립적인 개념으로 실험했었는데, 새로운 클래스 도입 대신 상속을 택함. 새로운 타입 대신 Thread 모델을 그대로 확장했다는 점에서 패러다임과 개발 철학의 일관성이 드러난다.

### SpringBoot에서의 적용 방법

스프링 부트는 3.2 이상부터 가상 스레드를 적용할 수 있도록 만들어져있다.
간단하게 properties 또는 yml에 세팅을 작성해주기만 하면 된다.

```properties
# 스프링 MVC 가상 스레드 사용
spring.threads.virtual.enabled=true
```

---
# 결론

- Virtual Thread는 가볍고, 빠르고, nonblocking인 경량 스레드
- Virtual Thread는 JVM 스케줄링 + Continuation
- Thread per request 사용중이고, I/O blocking time이 주된 병목인 경우 고려 가능
- 쉽게 적용 가능
	- Reactive가 러닝 커브로 부담되는 경우
	- Kotlin coroutine이 러닝 커브로 부담되는 경우

---

# Reference

우아한 테크블로그
https://techblog.woowahan.com/15398/#:~:text=Go%EC%9D%98%20goroutine%EC%9D%84%20%EC%95%84,%ED%83%80%EC%9E%84%EC%9D%84%20%EB%82%AE%EC%B6%94%EB%8A%94%20%EA%B0%9C%EB%85%90%EC%9E%85%EB%8B%88%EB%8B%A4.

카카오 테크블로그
https://tech.kakaopay.com/post/coroutine_virtual_thread_wayne/#:~:text=%EC%BD%94%EB%A3%A8%ED%8B%B4%EC%9D%80%20%EC%BD%94%ED%8B%80%EB%A6%B0,%EC%A0%81%EC%9D%80%20%EC%9E%90%EC%9B%90%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%A9%EB%8B%88%EB%8B%A4.

webflux vs virtual-thread
https://www.baeldung.com/java-reactor-webflux-vs-virtual-threads

virtual thread
https://0soo.tistory.com/259