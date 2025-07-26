
프로젝트 초기화

spring boot project
- spring data jpa
- lombok
- mysql driver
- spring web
- ~~spring security~~ 로그인 등의 기능은 아직 필요 없다고 봤다. 확장할 일이 없어보임
- spring configuration processor

```gradle
dependencies {  
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'  
//    implementation 'org.springframework.boot:spring-boot-starter-security'  
    implementation 'org.springframework.boot:spring-boot-starter-web'  
    implementation 'org.springframework.boot:spring-boot-starter-webflux'  
    compileOnly 'org.projectlombok:lombok'  
    runtimeOnly 'com.mysql:mysql-connector-j'  
    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'  
    annotationProcessor 'org.projectlombok:lombok'  
    testImplementation 'org.springframework.boot:spring-boot-starter-test'  
    testImplementation 'io.projectreactor:reactor-test'  
//    testImplementation 'org.springframework.security:spring-security-test'  
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'  
}
```
이정도로 최대한 간결하게 시작했다.

생각해보니 백엔드를 할거면 프론트엔드도 있어야 한다.
같이할 사람 없나?

https://naturecancoding.tistory.com/97
우선 여기서 레퍼런스를 얻었다.

---
0724

https://nashs789.tistory.com/129

셀레니움 설치하고 일단 긁어본다.

아직 mysql 세팅은 하지 않았다.
테이블 설계 해야되는데 어떻게 할지 고민임.

# Unable to Locate Driver Error
셀레니움 세팅하는데 이런 게 떴다.

https://www.selenium.dev/documentation/webdriver/troubleshooting/errors/driver_location/
정말 감사하게도...? 셀레니움에서 이슈 해결법을 얻을 수 있다.
path 등록하니 해결되었다.

아무튼 동작하는데, 스크랩하니까 403 포비든이 뜬다.
?
백준에서 404를 띄웠는데, 공식적으로 웹 스크래핑을 금지하고 있다보니 이런 결과가 나온 것 같다.
혹시 몰라서 까먹었던 User-Agent 헤더를 붙여봤다. 성공했다.
스크래핑 시 정책을 알아볼 필요가 있다...!
롸붯.텍스트라던가... 공부해도 기억에 남지가 않는다ㅏㅏㅏ

---
0725

DB 설계를 까먹고있었다.
도메인 설계가 우선이다...
근데 이슈 트래킹 시스템은 설계를 언제하냐???
이슈 발행하고 함?

GPT
- 도메인 개념 정의(개략적 ERD)
- 초기 DB 구조 설계(기본 테이블 MVP 수준)
- 핵심 기능 플로우 정리

이걸 이슈단위로 쪼개서 개발에 들어간다...?
### 3. **이슈 중심 개발 흐름**

1. **에픽(Epic)이나 마일스톤으로 큰 기능을 정의**  
    (예: "회원 기능", "게시판 기능")
    
2. 각 에픽을 이슈(Story/Task) 단위로 분할  
    (예: "회원가입 API", "이메일 중복 검사", "JWT 발급")
    
3. 개발하면서 필요하면 **도메인/DB 구조 변경 → 마이그레이션**  
    => 이게 흔히 말하는 "리팩토링"이야.
### 4. **그럼 리팩토링은 언제 하냐?**

- 설계가 완벽하지 않기 때문에,
    
- 개발하면서 새로운 요구사항이 나오거나,
    
- 기존 설계가 비효율적이라는 걸 **실행 중에 깨달았을 때**
    
- → 그때 이슈로 "DB 리팩토링" 또는 "모델 구조 개선"을 만들어서 처리함
등등 etc 지루하고 현학적인 설명이 있었다.

그래서 설계를 미리 하긴 해야 한다.
근데 기본이면 얼마나 해야되는거임?
악!!!

---
0726
도메인
백준 + solvedAC 보기 쉽게 만들어주기
도메인이 뭘까

데이터를 재가공하는 것이 목적이다.

백준 이용자, solvedac 이용자, 문제?

API docs까지 뚫어두는 것으로 만족하자!
타임리프는 안된다 꾸짖을 갈!!!
프론트는 나중에 연동해버리기

기술
FE - ???
BackEnd - Spring Boot
RDBMS - MySQL (MariaDB가 라이센스 필요없긴 하지만 내가 쓰는 수준에선 별 차이가 없기 때문에...)

solvedac 비공식 api 참고

발생 가능한 문제들
백준에 있고 solvedac에 없는 이용자?
백준에 없고 solvedac에 있는 이용자?
DB 설계?

파이썬이 쓰고싶어요 파흑흑흑

도구 세팅
Github, Github Actions
Notion
Jenkins