구현 위주

OAuth만 쓰는 것이 좋나? with GPT

OAuth 인증 시스템을 직접 구현한다면, **사용자 계정 테이블과 외부 OAuth 공급자(provider) 연결 정보를 분리**하는 방식이 가장 많이 사용돼. 일반적으로 다음과 같은 구조로 설계해.

---

### 1. users (내부 사용자 계정)

내 서비스에서 관리하는 핵심 사용자 정보

|컬럼명|타입|설명|
|---|---|---|
|user_id|BIGINT (PK)|내부 사용자 고유 ID|
|email|VARCHAR|이메일(일부 OAuth에서는 제공 안 할 수도 있음)|
|name|VARCHAR|사용자 이름|
|profile_image|VARCHAR|프로필 이미지 URL|
|created_at|DATETIME|생성일시|
|updated_at|DATETIME|수정일시|

→ 핵심은 `user_id`가 내부 시스템에서 공통적으로 참조되는 기준 키라는 점.

---

### 2. oauth_providers (OAuth 공급자 테이블)

내가 지원하는 OAuth 공급자 정의

|컬럼명|타입|설명|
|---|---|---|
|provider_id|BIGINT (PK)|공급자 고유 ID|
|provider_name|VARCHAR|ex: GOOGLE, KAKAO, NAVER, GITHUB|
|client_id|VARCHAR|내 앱에서 발급받은 client_id|
|client_secret|VARCHAR|내 앱에서 발급받은 client_secret|
|auth_url|VARCHAR|인증 URL|
|token_url|VARCHAR|토큰 발급 URL|
|user_info_url|VARCHAR|사용자 정보 요청 URL|

---

### 3. user_oauth_accounts (사용자별 OAuth 연결 계정)

어떤 사용자가 어떤 OAuth 공급자 계정을 사용했는지 기록

|컬럼명|타입|설명|
|---|---|---|
|id|BIGINT (PK)|연결 고유 ID|
|user_id|BIGINT (FK → users.user_id)|내부 사용자 ID|
|provider_id|BIGINT (FK → oauth_providers.provider_id)|공급자 ID|
|provider_user_id|VARCHAR|공급자 측 사용자 고유 ID (Google sub, Kakao id 등)|
|access_token|VARCHAR|접근 토큰(보안 때문에 암호화/단기 저장 권장)|
|refresh_token|VARCHAR|리프레시 토큰(옵션)|
|expires_at|DATETIME|토큰 만료 시각|
|connected_at|DATETIME|최초 연결 시각|

→ 이 테이블이 있어야 사용자가 **구글/카카오/깃허브 여러 개 계정을 연결**해도 하나의 `user_id`로 묶을 수 있음.

---

### 4. 세션/토큰 관리 (선택)

만약 자체 JWT 등을 발급한다면 별도 테이블을 둠

|컬럼명|타입|설명|
|---|---|---|
|token_id|BIGINT (PK)|토큰 고유 ID|
|user_id|BIGINT|사용자 ID|
|jwt_token|TEXT|발급한 JWT|
|revoked|BOOLEAN|폐기 여부|
|issued_at|DATETIME|발급 시각|
|expires_at|DATETIME|만료 시각|

---

### 설계 포인트

1. **users**는 내부 서비스 기준 → 이걸 중심으로 모든 데이터 연결
    
2. **user_oauth_accounts**를 두어 다중 로그인(구글, 카카오 등) 연결 가능
    
3. `provider_user_id`는 OAuth 공급자가 보장하는 고유 ID라 중복 체크에 핵심
    
4. `access_token`/`refresh_token`은 보안 때문에 DB 보관을 최소화하거나 암호화 저장 필요
    
5. 토큰을 DB에 저장하지 않고 **stateless(JWT)** 방식으로만 운영하는 것도 가능

---

JWT는 "내 서비스가 자체 발급하는 토큰"이야.  
흐름은 보통 이렇게 돼:

1. OAuth 로그인 성공 → 유저를 내 `users` 테이블에 매핑
    
2. 내 서비스가 자체 **JWT 발급** (`sub=user_id`, `roles`, `exp` 등 포함)
    
3. 클라이언트는 **내 JWT**를 API 요청 시 사용
    

장점:

- **Stateless 인증** → 서버에 세션 저장 필요 없음
    
- **내 권한 체계와 통합** → JWT payload에 roles/claims 포함 가능
    
- **보안 제어권 확보** → 만료 시간, 서명 알고리즘, 토큰 폐기 정책을 내가 직접 관리

---

현실적인 조합
JWT는 "내 서비스가 자체 발급하는 토큰"이야.  
흐름은 보통 이렇게 돼:

1. OAuth 로그인 성공 → 유저를 내 `users` 테이블에 매핑
    
2. 내 서비스가 자체 **JWT 발급** (`sub=user_id`, `roles`, `exp` 등 포함)
    
3. 클라이언트는 **내 JWT**를 API 요청 시 사용
    

장점:

- **Stateless 인증** → 서버에 세션 저장 필요 없음
    
- **내 권한 체계와 통합** → JWT payload에 roles/claims 포함 가능
    
- **보안 제어권 확보** → 만료 시간, 서명 알고리즘, 토큰 폐기 정책을 내가 직접 관리

---



Spring boot tutorials
https://spring.io/guides/tutorials/spring-boot-oauth2

티스토리 - 스프링부트 + OAuth 2.0 작동 방식
https://developer-nyong.tistory.com/60

스프링부트 튜토리얼 2
https://samtao.tistory.com/38


Google OAuth API Scenario
https://developers.google.com/identity/protocols/oauth2?hl=ko