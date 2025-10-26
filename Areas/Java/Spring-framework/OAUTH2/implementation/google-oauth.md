구글의 경우 client 정보를 json 파일 형식으로 다운로드 가능.

내용을 보면 다음과 같은 내용 존재

우리 서비스임을 증명
client_id
client_secret

인가 코드 uri
auth_uri

AT(access token) 발급 uri
token_uri


auth_provider_x509_cert_url?
> 안나옴.


스코프 설정하기(데이터 액세스)

가장 자주 사용되는 정보
- 기본 계정 이메일 주소 확인
- 개인 정보 보기
- google에서 내 개인 정보를 나와 연결

체크하지 않더라도 이건 기본으로 제공한다.
다른 정보들은 별도 체크 또는 직접 범위 추가로 작성해줘야 한다.


나머지 정보들 가져다 쓰기
앱 게시 또는 테스트 사용자를 추가해야 추가적 스코프 정보를 받을 수 있다.

프로덕션의 경우 구글에 승인을 받아야 한다.
따라서 추가적 스코프를 받으려면 사용자 민감 정보를 받는 별도 승인 절차를 받아야 함.
체크리스트에서 민감한 범위, 제한된 범위같은 형식으로 보여준다. 이 정보들이 있을 경우 배포 시 승인절차 필요.


리디렉션 URL
리디렉션을 통해 코드를 받을 라우팅 정보를 넣어줘야 한다.
백엔드에서 모두 처리할 경우 백엔드의 url을 넣어줘야 함.
요청을 보내는 쪽으로 인가코드를 보내주게 된다. 따라서 배포까지 진행 시 도메인 url도 추가해줘야 함.

---

# oauth 흐름

토큰 가져오기, 프로필 가져오기 다 각각의 요청임.

api 형식
header
- https://oauth2.googleapis.com/token
body
- form-data 형식
- code=THE_CODE&client_id=xxx&client_secret&....&redirect_uri=

사용자 프로필 가져오기
특징
- openid와 picture는 별도 scope, 콘솔 설정 없이도 기본적으로 제공
- email, profile은 scope에서 요청 시 console 설정 없이도 제공
api 형식
header
- https://openidconnect.googleapis.com/v1/userinf
이게 가장 최신버전이며 개발자들이 많이 사용하는 링크임. 다른 2개의 링크도 존재한다.
- https://www.googleapis.com/userinfo/v2/me
- https://www.googleapis.com/oauth2/v3/userinfo

