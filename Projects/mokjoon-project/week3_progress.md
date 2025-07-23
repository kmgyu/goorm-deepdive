
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
