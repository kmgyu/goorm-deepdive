
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