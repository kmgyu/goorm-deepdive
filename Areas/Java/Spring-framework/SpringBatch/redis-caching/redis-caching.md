
# 주제

Redis를 세션 저장소로 이용하여 기존 세션 저장소보다 얼마나 성능 개선을 이룰 수 있는지 확인한다.

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

이때 각 대상의 metric 구조와 네임 스페이스는 다를 수 있기 때문에 서버에 따라 쿼리의 형식이 달라질 수 있다.

리스닝 대상은 Exporter라는 것을 통해 metric을 외부로 노출시킨다.
Node Exporter, Redis Exporter, Spring Actuator 등이 존재한다.

> metric
> 시계열 데이터
> 특정 시점에 어떤 지표 값이 얼마였는가를 지속적으로 기록하는 단위

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
> 크롬에서 사용되지 않을 때 리스닝을 하지 못하는 이슈가 나타나 참고만 하였음.



# 부하테스트

성능 개선을 확인하기 위해, 가상의 사용자를 생성해 인위적인 트래픽을 발생시키고 이를 볼 수 있는 작업이 필요하다.


## 고려대상

jmeter, k6, ngrinder, locust

jvm을 사용하는 언어의 경우, 무겁기 때문에 사용하기 어려움.
로컬에서 사용한다. 둘 다 사용하기 때문에 정확하게 판단이 되지 않을 가능성이 높다.

고려해야 할 도구는 k6와 locust로 좁혀졌다.
이미 환경이 갖춰져 있어 빠르게 사용할 수 있는 locust를 선택함.

## 측정 방식

가상 사용자 5000명을 이용한다.

ramp up (second) 50


로그인 작업과, 새로고침하는 작업 두 가지를 이용하여 세션 생성과 세션 생성 후 캐싱 시 속도 향상에 대해 측정한다.

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

레디스를 사용할 시 DB를 조회하고 레디스에 다시 작성하는 과정에서 Network IO가 발생하기 때문에 오버헤드로 인해 시간이 증가한 것으로 보임.

레디스는 싱글 스레드인데 순간적으로 트래픽이 늘어나서 대응을 못한 것이 원인일 수도 있다. 어쩌다 보니 순간 접속자가 급작스럽게 늘어난 상황도 확인해볼 수 있었다!
특정 시간마다 한꺼번에 사용자의 로그인 요청을 처리하는 로그를 볼 수 있었음. 이 또한 성능 하락의 주요 원인으로 보임.

# 발생한 이슈

## 레디스 세션 직렬화

스프링은 레디스를 사용할 시 자바 언어에 대한 전용 직렬화를 지원한다.
만약 JSON을 이용하여 범용성을 챙기고 싶다면 알아둬야 할 것이 존재한다.

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

이런 식으로 @class를 통해 어떤 클래스를 가리키는 지 명시하도록 직렬화하게 되는데, 별도의 설정하여 이를 가리지 않는다면 외부에서 Redis에 접근해 객체를 변조시키는 RCE 취약점이 발생한다.

그러므로 해당 부분에 대해 인지하고 있어야 한다.

Jackson Serializer 는 종류가 많지만 3가지 정도만 소개한다.

JdkSerializationRedisSerializer : 기본 Serializer
GenericJackson2JsonRedisSerializer : Json으로 직렬화해주는 Serializer
StringRedisSerializer : Redis 사용 시 문자열로 직렬화하는 Serializer

따라서 JdkSerializationRedisSerializer를 사용하거나, JSON으로 직렬화 시 보안에 주의를 기울여 추가적인 코드를 작성할 필요가 있다.


## In-memory 세션 저장소 간 성능 차이

톰캣과 레디스에서 성능 차이가 발생하였음.

네트워크 I/O가 성능에 굉장히 많은 관여를 한다.
기본 세션 저장소 사용 시 바로 생성되던 것이 청크 단위(5~10개 단위로 묶이는 것으로 보임)로 생성되는 것을 로그로 확인할 수 있음.

레디스가 싱글 스레드로 관리되기에 분산 세션 저장소를 이용한다면 이런 단점을 효과적으로 개선할 수 있을 듯?
