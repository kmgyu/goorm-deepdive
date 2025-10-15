프로메테우스를 연동하는 법부터 시작한다.

https://www.baeldung.com/spring-boot-prometheus

https://toss.tech/article/how-to-work-health-check-in-spring-boot-actuator


프로메테우스를 사용하려면 메트릭이라는 것을 프로메테우스에 제공해주어야 한다.

이를 위해 스프링-액추에이터와 프로매테우스를 위한 마이크로미터 의존성을 추가해준다.
마이크로미터 자체는 액추에이터에 내장되어있긴 한데, 프로메테우스 연동을 위해 추가로 의존성을 넣어줘야 하는 것 같음.

```
implementation 'org.springframework.boot:spring-boot-starter-actuator'  
runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
```


도커로 이미지를 설치해 사용해주었다.

prometheus.yml이라는 세팅 파일이 필요한데, 도커 실행시에는 로컬 파일을 도커 컨테이너에 마운트 시켜주면 적용된다.

```
/etc/prometheus/prometheus.yml
```
컨테이너 내부는 이 경로다. 로컬은 잘.. 이어주자!


![[prometheus-test.png]]

실행해준 모습. 잘 작동한다.

