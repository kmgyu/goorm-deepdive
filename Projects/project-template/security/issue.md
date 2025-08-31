DB 스키마 버전 관리를 위해 flyway를 이용했는데, 하이버네이트와 충돌이 일어났다.

```
# flyway, hibernate 간 충돌 방지  
spring.jpa.hibernate.ddl-auto=validate  
spring.flyway.enabled=true
```

이런 식으로 세팅해줘야 함...

flyway 작동도 안하고 답답해서 그냥 직접 sql 실행해줬다.
그 와중에 ddl이랑 달라서 또 틀림

ddl이 실행이 안된다. 끄아ㅔ에에엑