```bash
docker exec [name] env
```

현재 컨테이너 환경변수 확인

---

프로메테우스로 메트릭 수집하기

프로메테우스에서는 다른 어플리케이션의 메트릭을 수집할 수 있다.

prometheus.yml 파일로 리스닝할 포트를 설정해줘야 하는데, 수집되는 대상은 exporter가 필요하다.
메트릭을 보내주는 기능.

스프링은 스프링 액추에이터가 있기 때문에 exporter가 내장되어 귀찮은 것을 생략할 수 있다. 대부분 exporter가 존재한다...
없으면 라우팅이랑 계속 보내주는 것을 설정해줘야 함..

prometheus.yml (참고용)
``` yml
scrape_configs:
  - job_name: 'spring-actuator-app'
    # Actuator 메트릭 엔드포인트 경로
    metrics_path: '/actuator/prometheus' 
    static_configs:
      # Docker Desktop 환경에서는 host.docker.internal을 사용합니다.
      # Linux에서는 호스트 IP 주소 또는 Docker 네트워크 설정을 따릅니다.
      - targets: ['host.docker.internal:8080']
```

현재 도커 파일로 빌드만 되고있음.

---

컨테이너 모니터링
https://joyerim.tistory.com/117

