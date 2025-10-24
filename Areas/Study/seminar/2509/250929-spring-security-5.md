준원님

spring security

anonymous / logout / requestcache / SavedRequest

# anonymous()
익명 인증 사용자와 인증되지 않은 사용자 간 실제 개념적 차이는 없으며, 액세스 제어 속성 구성 시 더 편리한 방법을 제공

익명 인증 객체를 security Holder에 넣어두면 인증 여부 관계없이 Authenticate 객체에 있다는 것을 알 수 있다.
> null인지 분간할 수 없는데, 익명 사용자 객체를 사용해서 오류가 났는지 인증이 안 된건지 분간할 수 있다.

인증 사용자와 익명 인증 사용자를 구분해 어떤 기능을 수행할 때 유용할 수 있으며, 익명 인증 객체를 세션에 저장하지 않는다.

익명 인증 사용자의 권한을 별도 운용할 수 있다. 인증된 사용자가 접근할 수 없도록 구성 가능함.

요약:
완전 편하게 코드 짜기
로그인, 접근 권한 편하게 짤 수 있음.

API 및 구조
대부분 디폴트를 사용하게 됨.
필터 체인에서 아래와 같이 사용 가능
```java
anonymous(ano -> ano
.chains
)
```
principal(), authorities()등이 있어서 권한을 지정해줄 수 있다.

`HttpServletReqeust`에서 익명으로 들어오게 되면 null이 되서 not anonymous가 뜨게 된다.
따라서 익명 요청에서 Authentication을 획득하려면 `@CurrentSecurityContext`를 사용해주면 된다. (필드에)
이렇게 되면 CurrentSecurityContextArgumentResolver가 가로채서 해결해준다.

Authentication의 경우 유저는 정상작동, 익명은 null
`@AuthenticationPrincipal` 또한 동일하다.
필드 타입은 UserDetails
`@CurrentSecurityContext`는 정상적으로 작동했다.
필드 타입은 SecurityContext

null인 이유는 ServletReqeustMethodArgumentResolver에서 resolveArgument를 해준다.
userPrincipal을 구해줄 때 null로 변함.

 Authentication 타입?은 AuthenticationPrincipalArgumentResolver 클래스가 작동한다고 함. 근데 디버깅에서 잡히지 않는다??

`@AuthenticationPrincipal`?
저 이름 긴 Resolver 동작한다.
그런데 매개변수 타입이 UserDetails이고 return 값이 String이어서 타입이 맞지 않아 null을 반환한다.
principal에서 Object가 들어옴.
> 반환할 때 String을 반환한다면 애초에 Object로 주고 타입을 변환시켜주면 되는 게 아닐까?
## AnonymousAuthenticationFilter

SecurityContextHolder에서 Authentication 객체가 없을 경우 감지하고 새 객체 생성
익명 유저 권한 생성해주기 위함!

---
# 로그아웃

디폴트 URL은 GET /logout
CSRF 기능을 비활성화하거나 RequestMatcher 사용 시 get, put, delete도 가능하다.

로그아웃 필터를 거치지 않고 로그아웃 하도록 구현할 수도 있다?
#todo 노션 가서 확인해보기

logoutSuccessHandler
addLogoutHandler

## LogoutFilter

RequestMatcher에서 매칭되는지 확인하고 핸들러로 보내줌...

## 요청 캐시 RequestCache / SavedRequest

메서드 확인해보셈~

### RequestCache
인증 절차 문제로 리디렉션 후 이전 요청 정보를 담고있는 SavedReqeust 객체를 쿠키 혹은 세션에 저장.
필요 시 다시 가져와 실행하는 캐시 매커니즘

### SavedRequest
로그인과 같은 인증 절차 후 사용자를 인증 이전 원래 페이지로 안내하여 이전 요청과 관련된 여러 정보를 저장.

흐름도

인증받지 않은 상태로 접근 시 RequestCache가 작동해서 인증 이전 요청 정보를 저장하고 로그인으로 리디렉션 때린다.

SavedRequest도 확인하셈


### RequestCache API
customParam=y라는 이름의 매개변수가 있는 경우에만 HttpSession에 저장된 SavedRequest를 꺼내오도록 설정 가능

요청을 저장하지 않도록 하려면 NullRequestCache 구현도 가능하다고 함.


### RequestCacheAwareFilter
이전에 저장한 웹 요청을 다시 불러오는 역할
SavedRequest가 현재 Request와 일치 시 이 요청을 필터 체인의 doFilter 메소드에 전달. savedRequest 없으면 원래 request를 그대로 진행

저장이 안되어 있으면 그냥 request로 간다.
저장되어 있다면, 과거의 요청을 현재의 요청과 비교하여 일치한다면 저장된 요청을 실행한다.

---

이슈?

어떤 사용자가 A라는 권한의 계정으로 로그인
중간에 관리자가 이사람의 권한을 B로 변경하게 된다.

ZooKeeper처럼 캐시를 서버에서 무효화시키는 전략
트리거 형식으로 강제로 만료시키고 refreshToken을 쓰게 만든다.
세션의 경우에는 추가적으로 세션을 제거해줘야 한다.

JWT는 액세스토큰을 블랙리스트로 만들어버린다.

또 다른 전략도 있음.
무결성이 깨졌는지 판단할 때 이 전략 사용 가능.
버저닝 전략
캐시 생존 시간이랑 헷갈렸당 ㅎ
버전 값을 확인해서 불일치한다면 그 경우 접속할 수 없다는 식으로 안내하는 방식

캐시 무효화 전략
TKR 산정? TTL 인듯
블랙리스트
버저닝

구현은 어떻게 할까?
토큰 같은 경우, payload에 하나 만들어주면됨
서버는 블랙리스트를 관리하는 테이블을 하나 만들어줘야 한다.
이런 것을 해주기 싫다면 캐시값을 죽여버리는 전략을 사용하면 된다.

민성님의 경우 중요한 값은 TTL을 짧게 줬었다.
웹소켓에서 이걸 줄 수 있는 것으로 알고 있다고 함. 그런 요청을 보낼 수 있다?

보안 면에서도 이게 사용될 수 있다.
인증 토큰만 주면 서비스 접속이 가능하니 관리자 입장에선 접근불가능하게 없애버릴 수 있다.
블랙리스트 형태로 이걸 관리해줄 수 있음.
데이터 변경 말고도 접근 통제에 쓰는 식으로도 가능하다.



> 그럼 현재 브라우저를 닫아버려도 캐시가 남나?
> 로그인 성공하면 그 페이지 들어가나?
> 쿠키는 얼마나 보존될 수 있나? 정책에 따라 달라지나?

max-age 같은 설정을 없애면 꺼진 상태에서도 영속적으로 남게 세팅할 수 있다.
결국 쿠키 정책에 대한 문제이다.

자동로그인이 그 예시?

개발자 도구 가져와서 네트워크 확인해보면 디스크 캐시에서 가져오는 게 있음. 캐시 로딩 같은 것들...
특정 데이터 로드 시 시간을 볼 수 있다. 굉장히 빠르다!
여기서 병목 현상을 캐치할 수도 있음.
보통 permissions가 오래 걸린다.
성능적으로 많이 개선할 수 있는 여지가 이런 곳에서 발생 가능

어플리케이션에서 볼 수 있음.

ARC?라는 거에서 캐시 관리해줄 수도 있고 크롬도 캐시 날리기 지원하긴 함.
로컬 스토리지 삭제 시 자동로그인 풀린다. 히하
