
준영님 발표

PostgreSQL

객체 관계형 DBMS??
엄... 특징이 뭐징

오픈소스 RDBMS 유형 중 하나

# 특징
트랜잭선, ACID 지원
스키마

# 구조
클라이언트/서버 모델 사용
서버는 DB 파일 관리, 클라이언트 앱 연결 수용
각 커넥션에 대해 새 프로세스를 fork


```bash
brew services start postgresql@15
```
다음과 같이 실행 가능


기본적인 명령어 사용법 알려주신다.

MySQL, MariaDB에서 본 것과는 달리 PostgreSQL은 사용자 생성 시 조금 간결하다.
```
create user jung with password '1234';
```
어디서 접속하는지... 이런 것들은 다 세팅 안한다.

`SERIAL` -> `AUTO_INCREMENT`랑 같음.
`JSONB` -> PostgreSQL도 JSON 지원한다. 이건 BSON임!
검색, 필터링, 수정, 인덱스 활용에 용이하다....?
JSON 타입도 문자열 형태로 넣으면 자동으로 JSON 저장해줌
그럼 클라이언트에 보내줄 땐 어떻게 보내주려나?

->, ->> 연산자로 JSON 컬럼 접근 가능
`->`는 단순 JSON 객체
`->>`는 텍스트 반환

배열도 지원.
ANY(tags)? tags 배열 안에 특정 문자같은 데이터 포함되어있는지 검사? like 와 비슷하다고 함.

update + insert(insert on conflict)
삽입 도중 유니크 값 중복으로 인한 충돌 시, 업데이트와 삽입을 동시에 할 수 있다.

 

기업에서 많이 쓰는 이유?
JSON 활용 가능
성능도 더 좋다.
JSONB를 활용하는 데 어떻게??
일반적 sql에서는 json 지원을 안한다.

배열 삽입, update, insert 동시에 한다던지 같은 여러 기능을 지원함.




JSON, 배열 쓰는 것
객체지향 개념을 강하게 지원해야 ORDBMS라고 분류한다.

MySQL, MariaDB는 지원을 제대로 해주지는 않음.


그럼 어떤 경우에 써야 할까?

JSON 삽입, 배열 삽입

