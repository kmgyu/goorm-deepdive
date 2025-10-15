
소스코드
https://github.com/kmgyu/spring_batch_instruction

# 주제

인증 방식은 크게 토큰 기반 인증, 세션 기반 인증으로 나뉜다.
이때 세션 기반 인증은 기본적으로 서버 측에서 세션 저장소를 통해 관리하게 되는데, 이 때문에 추가적인 시스템 자원을 소모하게 된다.

> 세션 저장소의 경우 서블릿 컨테이너(톰캣)에서 생성되며 GC의 관리 하에 놓이게 됨.

보통 외부에 세션 저장소를 배치하는 방식은 분산형 서버에서 sticky session에서 트래픽 불균형과 같은 문제를 해소하면서 사용자를 인식하기 위해 사용된다.
모놀리식 서버에서 단순히 Redis를 추가하는 것 만으로 성능 향상을 이룰 수 있는지 살펴본다.

또한, 이 과정에서 생기는 이슈나 취약점에 대해 알아본다.


# 프로젝트 구조

프로젝트는 간단하게 구성하였다.

**의존성**
- Spring boot
- Spring boot Security
- Spring batch
- thymeleaf

*선택 적용* (레디스 세션 사용 시 적용됨)
- Spring Redis session
- Spring Actuator
- Spring Prometheus

**DB**
- MariaDB
- Redis (선택 적용)

위와 같이 기본적인 의존성만 이용하여 프로젝트를 구성하였으며, 부하 테스트를 위한 사용자는 배치 작업을 통해 생성하였다.

사용하는 기기의 경우, 본인의 노트북을 활용해 모놀리식 서버 형태로 사용하였다.

기기 스펙(요약)은 아래와 같으니 참고.
CPU : 8코어 (16 스레드)
메모리 : 32GB

## 아키텍처 살펴보기
### 기본 세션 사용

![[non-caching-architecture.png]]

스프링에서 기본으로 제공하는 In-memory 방식의 세션 저장소는 유저가 로그인할 시, 서버에서 DB에 접근하여 사용자의 정보를 확인한 뒤 Security Context를 만드는 식으로 동작한다.

만약 세션 저장소에서 해당 사용자에 대한 Security Context(Session 정보)가 존재할 경우, DB를 조회하지 않는다.

이때, 세션 저장소는 기본적으로 톰캣의 세션 저장소를 사용하게 된다.

세션 저장소의 경우 영구 DB를 활용하거나, Redis나 Memcached와 같은 In-memory DB를 활용하는 방법도 존재한다.


### 레디스 세션 사용

세션 저장소로 레디스를 이용하는 방법이다.

**사용된 읽기 쓰기 전략**
- Look-Aside
	- 서버가 캐시를 먼저 조회, 캐시에 없을 경우 DB를 조회
- Redis를 세션 저장소로 사용, 서버는 세션을 저장하지 않음.
	- 레디스가 주 세션 저장소이기 때문에 캐싱 시의 쓰기 전략과 다르다.

스프링에서 세션 저장소를 제공했던 것과 같이 크게 두 가지 시나리오가 존재한다. (세션 만료와 같은 엣지 케이스 또는 분산 저장소와 같은 복잡한 형태는 지금 상황에서 고려하지 않도록 한다.)

1. 세션 저장소에 세션 정보가 존재하는 경우

![[redis-caching-architecture1.png]]
사용자가 로그인했을 때 서버에서는 Redis에 해당 사용자의 세션 정보를 찾게 된다.

세션 저장소에 사용자 정보가 존재한다면 해당 정보를 서버에 넘기게 되고, 서버는 해당 정보를 읽어 사용자를 인식한다.


2. 세션 저장소에 세션 정보가 존재하지 않는 경우

![[redis-caching-architecture2.png]]
사용자가 세션 저장소에 존재하지 않을 때, 서버는 DB를 조회해서 사용자의 정보를 조회한 후, 해당 정보를 토대로 Session을 생성하고 세션 저장소에 저장한다.


# 모니터링하기

서버를 구축하여 리소스를 모니터링하는 것은 중요한 과제이다.

본래 이를 위해 Spring Actuator + Prometheus를 사용하려 하였으나, 갱신이 정상적으로 되지 않아 참고용으로만 사용하였다.

Prometheus는 8월 6일 대규모 트래픽 제어에서 간단하게 소개된 바 있음.
쿼리를 이용해 시스템 자원을 확인하는 형식임.

리스닝을 통해 대상이 노출하는 metric을 수집한다.

> metric
> 시계열 데이터
> 특정 시점에 어떤 지표 값이 얼마였는가를 지속적으로 기록하는 단위

이때 각 대상의 metric 구조와 네임 스페이스는 다를 수 있기 때문에 서버에 따라 쿼리의 형식이 달라질 수 있다.

리스닝 대상은 Exporter라는 것을 통해 metric을 외부로 노출시킨다.
Node Exporter, Redis Exporter, Spring Actuator 등이 존재한다.


쿼리 예제 (스프링 액추에이터)
```
# 전체 RPS
sum(rate(http_server_requests_seconds_count[1m]))

# 프로세스 cpu 사용률
avg(process_cpu_usage) by (instance) * 100
```

다행스럽게도, 다음에 살펴볼 부하테스트의 도구인 Locust가 RPS와 응답시간을 제공한다.

따라서 부하테스트 도구의 해당 기능을 이용해 사용자 입장에서의 성능을 측정해볼 것이다.

> 시스템 성능에 대한 것은 프로메테우스를 활용할 수 있다.
> 앞서 말했듯이 크롬에서 사용되지 않을 때 리스닝을 하지 못하는 이슈가 나타나 참고만 하였음.

# 부하테스트

성능 개선을 확인하기 위해, 가상의 사용자를 생성해 인위적인 트래픽을 발생시키고 이를 볼 수 있는 작업이 필요하다.


## 고려대상

jmeter, k6, ngrinder, locust

jvm을 사용하는 언어의 경우, 무겁기 때문에 사용하기 어려움.
로컬에서 사용해야 하므로 컴퓨터의 부담을 최소화하고자 하였다.

따라서 고려해야 할 도구는 k6와 locust로 좁혀졌다.
k6는 Go, locust는 python 기반으로 작성되었다.
각각 js와 python으로 테스트 시나리오를 작성해줄 수 있다.
이미 환경이 갖춰져 있어 빠르게 사용할 수 있는 locust를 선택함.

## 측정 방식

가상 사용자 5000명을 이용한다.
초당 사용자 생성 50

가상 사용자는 생성될 때 로그인하게 되고, 0.5~3초마다 새로고침을 하여 트래픽을 유발한다.

### 코드 - 요약

*locustfile.py*
```python
class WebsiteUser(HttpUser):
    """
    하나의 Locust user는 시작 시 계정 하나를 가져와 로그인하고,
    이후 대시보드/새로고침 등의 요청을 반복합니다.
    """
    wait_time = between(0.5, 3)  # 요청 사이 랜덤 대기(초) — 실제 시나리오에 맞게 조정
    credential = None

    def on_start(self):
        # 계정 할당
        try:
            self.credential = _credentials_q.get_nowait()
        except Exception:
            if REUSE_CREDENTIALS_IF_EXHAUSTED:
                # 재사용: 랜덤 계정 생성 (user1~TOTAL_USERS)
                n = random.randint(1, TOTAL_USERS)
                self.credential = (f"user{n}", str(n))
            else:
                # fallback: make an ephemeral id
                n = random.randint(1, TOTAL_USERS)
                self.credential = (f"user{n}", str(n))

        username, password = self.credential

        # 로그인 전, CSRF 토큰이 필요한 경우 로그인 페이지를 먼저 GET
        headers = {"Accept": "application/json, text/html, */*"}
        csrf_token = None
        if USE_CSRF:
            resp = self.client.get(LOGIN_PATH, name="GET login page", headers=headers, catch_response=True)
            if resp.status_code == 200:
                csrf_token = _maybe_get_csrf_token(resp.text)
                # 쿠키(세션)는 self.client가 자동으로 보관
            else:
                resp.failure(f"Failed to GET login page: {resp.status_code}")

        # 로그인 요청
        if LOGIN_AS_JSON:
            body = {USERNAME_FIELD: username, PASSWORD_FIELD: password}
            if csrf_token:
                body[CSRF_FIELD] = csrf_token
                # CSRF가 필요하고 서버가 토큰을 헤더로 기대한다면 아래처럼 전송하도록 변경하세요:
                # headers["X-CSRFToken"] = csrf_token
            with self.client.post(LOGIN_PATH, json=body, headers=headers, catch_response=True, name="POST login") as r:
                if r.status_code in (200, 302):
                    # 200 OK or redirect on success — 테스트 환경에 맞게 성공 코드 조건 조정
                    r.success()
                else:
                    r.failure(f"Login failed: {r.status_code} - {r.text[:200]}")
        else:
            # form-encoded
            data = {USERNAME_FIELD: username, PASSWORD_FIELD: password}
            if csrf_token:
                data[CSRF_FIELD] = csrf_token
            with self.client.post(LOGIN_PATH, data=data, headers={"Content-Type": "application/x-www-form-urlencoded"}, catch_response=True, name="POST login (form)") as r:
                if r.status_code in (200, 302):
                    r.success()
                else:
                    r.failure(f"Login failed: {r.status_code} - {r.text[:200]}")

        # optional: 방문/초기화 페이지 요청 (로그인 후 redirect를 따르지 않으면 수동으로)
        # self.client.get(DASHBOARD_PATH, name="GET dashboard (after login)")

    def on_stop(self):
        # 계정 반납(원하면)
        if REUSE_CREDENTIALS_IF_EXHAUSTED:
            # 반납하면 다른 사용자가 재사용 가능
            try:
                _credentials_q.put_nowait(self.credential)
            except Exception:
                pass

    @task(5)
    def refresh_dashboard(self):
        """
        새로고침/대시보드 트래픽을 유발 — 빈도는 decorator의 인자로 조절
        """
        # 리프레시 시 약간의 랜덤 파라미터를 붙여 캐시 적중률을 낮춤
        params = {"_": str(int(time.time() * 1000)), "r": str(random.randint(1, 1000))}
        with self.client.get(REFRESH_PATH, params=params, name="GET refresh", catch_response=True) as r:
            if r.status_code == 200:
                r.success()
            else:
                r.failure(f"refresh failed: {r.status_code}")
```

사용자는 처음 생성될 시 로그인 한 후 지속적으로 새로고침을 통해 세션을 확인하도록 하여 세션 저장소의 탐색 성능을 비교한다.

### 결과

**기본 세션 저장소**

![[non-redis traffic.png]]
트래픽

![[non-redis statistics.png]]
통계

---

**레디스 세션 저장소**

로그인 요청이 느려 약 30분 간 진행함.

![[redis traffic.png]]
트래픽

![[redis statistics.png]]
통계


먼저 새로고침 등으로 세션에서 사용자를 조회하는 경우 p50, p95, p99 모두 레디스를 사용할 때 성능 개선이 있었다.


사용자가 로그인하는 경우 반대로 기본 세션 저장소를 사용하는 것이 빨랐다.

유력한 원인으로는 레디스는 싱글 스레드인데 순간적으로 트래픽이 늘어나는 상황이라서 대응을 못한 것과, 직렬화 비용으로 추정된다.


# 발생한 이슈

## 레디스 세션 직렬화

스프링은 레디스를 사용할 시 자바 언어에 대한 전용 직렬화를 지원한다.
만약 분산 환경을 사용하기 때문에 만약 JSON을 이용하여 범용성을 챙기고 싶다면 아래와 같은 이슈가 발생할 수 있다.

세션의 경우, sessionAttr:SPRING_SECURITY_CONTEXT 라는 필드로 저장되는 SecurityContext 객체는 아래와 같이 직렬화된다.

```JSON
{
    "@class": "org.springframework.security.core.context.SecurityContextImpl",
    "authentication": {
        "@class": "org.springframework.security.authentication.UsernamePasswordAuthenticationToken",
        "authorities": [
            "java.util.Collections$UnmodifiableRandomAccessList",
            []
        ],
        "details": {
            "@class": "org.springframework.security.web.authentication.WebAuthenticationDetails",
            "remoteAddress": "0:0:0:0:0:0:0:1",
            "sessionId": null
        },
        "authenticated": true,
        "principal": {
            "@class": "batch.batchapplication.auth.domain.User",
            "id": 10002,
            "username": "user1",
            "email": null,
            "password": "$2a$10$GbC/4/pIjP1JOAXqemqOLucRC73gOgKkpbntU.37uFN6WztMGj2de",
            "backdoor": "1",
            "enabled": true,
            "accountNonLocked": true,
            "credentialsNonExpired": true,
            "accountNonExpired": true
        },
        "credentials": null
    }
}
```

이런 식으로 @class를 통해 어떤 클래스를 가리키는 지 명시하도록 직렬화하게 되는데, 별도의 설정을 통해 이를 가리지 않는다면 외부에서 Redis에 접근해 객체를 변조시키는 RCE 취약점이 발생한다.


Jackson Serializer 는 종류가 많지만 3가지 정도만 소개한다.

JdkSerializationRedisSerializer : 기본 Serializer
GenericJackson2JsonRedisSerializer : Json으로 직렬화해주는 Serializer
StringRedisSerializer : Redis 사용 시 문자열로 직렬화하는 Serializer

JdkSerializationRedisSerializer 또한 역직렬화 취약점이 존재하기 때문에, JSON으로 직렬화 시 보안에 주의를 기울여 추가적인 조치를 해주어야 한다.

> @class 비활성화와, 직렬화/역직렬화 시 신뢰할 수 있는 클래스 명단을 지정하는 조치를 취해주면 된다.

## In-memory 세션 저장소 간 성능 차이

서블릿 컨테이너(톰캣)와 레디스에서 성능 차이가 발생하였음.

유력한 원인
직렬화 비용
단일 레디스 서버

기본 세션 저장소와 비교 시 육안으로 로그를 통해 유저 세션 생성 속도를 비교할 수 있었음.

보통 외부에 세션 저장소를 배치하는 방식은 다중 서버 구조에서 사용자를 인식하기 위해 사용한다.
레디스의 경우 단일 스레드 서버이기 때문에 장애나 순간적인 트래픽에 취약한 편이기 때문에, 클러스터링을 이용해 이런 약점을 보완해주어야 한다.

### Redis Clustering

레디스는 Redis Clustering을 제공한다.
데이터 샤딩을 통해 레디스의 취약점인 장애 대응을 완화시켜주는 기능이다.

특징은 다음과 같다.
- 여러 노드에 자동적인 데이터 분산(샤딩)
- 일부 노드의 실패나 통신 단절에도 계속 작동하는 가용성 제공
	- auto failover : master-replica 구조에서 기존 master가 죽게 되면 replica가 master로 승격된다.
	- replica migration : replica가 다른 master로 migrate해서 특정 master의 replica가 부족한 현상을 줄인다.
- 고성능을 보장하면서 선형 확장성을 제공
    - 분산과 확장성이 좋고 성능에 대한 보장을 해준다

[클러스터링에 대한 추가 자료](https://velog.io/@ekxk1234/Redis-Cluster)

### Lettuce의 shutdown 처리 특징

만약 Redis가 정상 종료 될 때, 클라이언트 측은 커넥션을 어떻게 관리하는가?
Lettuce는 Redis 클라이언트 중 하나이며, 동기/비동기를 지원함.

- 비동기 이벤트 루프 기반(Netty)으로 동작함.
- `LettuceConnectionFactory.destroy()` 시 내부의 `ClientResources` (EventLoopGroup, Timer 등)를 정리함.
- **모든 비동기 작업이 완료된 후 커넥션이 닫히도록 보장함**. (Connection draining)
- Spring Boot에서 `@PreDestroy` 또는 `DisposableBean.destroy()` 훅을 통해 자동 호출됨.

> 그러나, Redis Cluster 전체 노드의 graceful shutdown은 레디스 서버 수준에서 별도의 설정이 필요해진다.
> 데이터 영속성 보장과 마스터 역할을 안전한 전환을 충족할 필요가 존재함.

이런 단점들을 해결해주려면 이런 자원들을 관리해주는 무언가가 필요하며, 대부분의 클러스터링에서 이런 문제점이 나타날 것이다.
이때 분산 환경에서의 조정을 위한 솔루션으로 Zookeeper이 제시된다.

> 장애 발생 시에는 자동 재연결 시도, 비동기 예외 전파, 타임아웃 및 회복 등으로 대응하며 클러스터 환경의 경우 클러스터 토폴로지 자동 업데이트(Redis Cluster의 토폴로지 변화 시 감지 및 라우팅 정보 갱신) 등을 시도할 수 있다.

### Zookeeper 알아보기

Zookeeper는 분산 처리 환경에서 사용 가능한 데이터 저장소로, 분산 서버 간의 정보 공유, 서버 투입/제거 시 이벤트 처리, 서버 모니터링, 시스템 관리, 분산 락 처리, 장애 상황 판단 등 다양한 분야에서 활용할 수 있다.

핵심 기능
- 데이터 상태 관리
데이터를 디렉터리 구조의 트리 노드로 저장하고, 데이터가 변경되면 클라이언트에게 어떤 노드(데이터)가 변경됐는지 콜백을 통해서 알려준다.
- 세션 생명주기 및 생성 순서 관리
데이터를 저장할 때 해당 세션이 유효한 동안 데이터가 저장되는 Ephemeral Node(임시 노드)로 클라이언트의 동작 여부를 판단한다.
데이터를 저장하는 순서에 따라 자동으로 일련번호(sequence number)가 붙는 Sequence Node(순차 노드)로 분산 락 등을 구현할 수 있다.

**추천 포스트**
[medium - 분산 시스템](https://medium.com/@arneg0shua/100%EB%8C%80-%EC%9D%B4%EC%83%81%EC%9D%98-%EC%84%9C%EB%B2%84%EB%A5%BC-%ED%95%9C-%EB%AA%B8%EC%B2%98%EB%9F%BC-%EB%B6%84%EC%82%B0%EC%8B%9C%EC%8A%A4%ED%85%9C-0a9046f8cacc)
[naver d2 - redis cluster를 위한 zookeeper](https://d2.naver.com/helloworld/294797)
[medium - 파일 업로드 진행 중인 어플리케이션을 배포하기 : graceful shutdown](https://medium.com/@arneg0shua/%ED%8C%8C%EC%9D%BC-%EC%97%85%EB%A1%9C%EB%93%9C-%EB%8F%84%EC%A4%91-%EB%B0%B0%ED%8F%AC%EB%A5%BC-%EC%A7%84%ED%96%89%ED%95%B4%EB%8F%84-%EA%B4%9C%EC%B0%AE%EC%9D%84%EA%B9%8C-graceful-shutdown-%EC%A0%81%EC%9A%A9%ED%8E%B8-453ae9f29dd1)


# 셀프 피드백

> 요약 : 테스트 케이스 제대로 안 만들고 실험해서 중구난방이다.
> 결국 제대로 살펴보면 실험을 위한 구체적인 시간 같은 게 정해져있지 않았음.

로컬에서 하기 때문에 외부 프로그램 실행 시 Response Time이나 RPS에 영향을 주는 경우가 많았다. 또한 환경 격리가 제대로 이루어지지 않았음.
분산 환경은 물리적으로 독립시켜서 실험해주는 것이 권장될 듯...

실험 통제가 제대로 이루어지지 않았음.
redis는 30분동안 진행, 기본은 약 7분 진행...

실험에서 login과 새로고침이 분리되지 않았음.
이걸 독립해서 실험해주는 것이 좋을 듯.

순간 접속이 폭증하는 경우만 강조됨. 사용자가 적을 경우에 대한 비교가 없음.

리소스 모니터링이 이루어지지 않았음. 자원 소모량이 어떻게 변하는 지 확인할 수 없었다.

결국 기술적으로 깊게 파고 들어가지 못했음. 내부적인 구현 같은 것을 보려고 했는데 HttpSession 생성 과정에서 getSession 말고는 이해한 것이 없다.
그래서 getSession하면 누가 만들어주는 거임 그/아/아/앗
세션이 만료되는 상황에서 refresh token을 사용해 재발급한다거나, 장애 발생 시 더 취약성 등 다양한 취약점이 존재하는 데 그런 것들이 고려되지 않았음.

마지막에 이슈들 개선점 좋은데, 좀 주제랑 맞지 않는 것 같음. 분산 환경을 어거지로 단일 서버에서 해서 그런가 문제가 많이 보인다.

## 개선 방안

docker file로 이미지 파일로 빌드하고 AWS 환경에서 사용하기 쉽게 만든다.
AWS 환경에서 어플리케이션을 실험해볼 수 있다.
prometheus, grafana도 추가로 사용해보면 좋을 듯.

기술적으로 깊게 파고드는 문제는 실험 주제 자체가 기존에 세션 웜업이여서 자꾸 변경하다가 시간이 지체된 것이 주요 원인.
주제 자체를 좀 논리적으로 생각하고 삽질을 줄여야 한다.

