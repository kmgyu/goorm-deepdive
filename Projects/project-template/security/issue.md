### flyway 일 안함
DB 스키마 버전 관리를 위해 flyway를 이용했는데, 하이버네이트와 충돌이 일어났다. 사실 그냥 일 안하는 거다. history 테이블도 만들어지고 작동한다.

```
# flyway, hibernate 간 충돌 방지  
spring.jpa.hibernate.ddl-auto=validate  
spring.flyway.enabled=true
```

이런 식으로 세팅해줘야 함...

flyway 작동도 안하고 답답해서 그냥 직접 sql 실행해줬다.
그 와중에 ddl이랑 달라서 또 틀림

ddl이 실행이 안된다. 끄아ㅔ에에엑

---
0901
드디어!!! 문제를 찾았다.

```properties
spring.application.name=spring_template  
  
# flyway, hibernate ? ?? ??  
spring.jpa.hibernate.ddl-auto=none  
spring.flyway.enabled=true  
spring.flyway.locations=classpath:db/migrations  
logging.level.org.flywaydb=DEBUG  
  
spring.profiles.include=secret
```
flyway locations 세팅이 잘못되어있었다. s를 빼먹음
ddl-auto가 충돌나는 것은 팩트같음;
경로 설정도 필수... 라고 한다....
validate 혹은 none 설정을 해주는 것이 좋다. validate 하면 빌드 할 때 충돌나니 오히려 더 좋다. (flyway가 일 안하면 혼자서 자폭함)


추가로, `spring.flyway.baseline-on-migrate=true` 세팅을 해주어야 한다.
이렇게 해주어야 자동으로 최신 버전을 감지하고 쿼리를 실행한다.
안해주면 계속 V1 부터 실행하고 오류 두다다다 쏟아낸다...
default가 false라서 세팅해줘야 한다. create-drop하는 환경이면 상관없지만.