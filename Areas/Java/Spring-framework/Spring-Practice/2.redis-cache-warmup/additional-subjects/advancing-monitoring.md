[[using-prometheus]]
기존 모니터링은 너무 허약해서 로그파일도 제대로 못 넘길 것 같다.

prometheus는 쿼리 자체를 띄우는 것은 사전 적용하지 못하고, 웹에서 쿼리를 띄우기 전까지는 메트릭을 수집하지 않는다고 한다. (이건 사전에 실행하도록 할 수 있는 것 같음.)

promQL이 찾아봐야 하는 불편함이 존재해서 prometheus+grafana를 통해 좀 더 편하게 볼 수 있지 않을까 하여 관련 자료를 찾아봤다.

[참고자료](https://www.devkuma.com/docs/prometheus/docker-compose-install/)

prometheus와 grafana를 사용할 것이므로 아예 붙여줬다!

아래의 세팅파일을 참고하여 필요한 부분 이외엔 주석 처리해주었다.
```yml
global:
  scrape_interval: 15s     # scrap target의 기본 interval을 15초로 변경 / default = 1m
  scrape_timeout: 15s      # scrap request 가 timeout 나는 길이 / default = 10s
  evaluation_interval: 2m  # rule 을 얼마나 빈번하게 검증하는지 / default = 1m

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'       # 기본적으로 붙여줄 라벨
  query_log_file: query_log_file.log # prometheus의 쿼리 로그들을 기록, 없으면 기록안함

# 규칙을 로딩하고 'evaluation_interval' 설정에 따라 정기적으로 평가한다.
rule_files:
  - "rule.yml"  # 파일 위치는 prometheus.yml 이 있는 곳과 동일 위치
  - "rule2.yml" # 여러개 가능

# 매트릭을 수집할 엔드포인드로 여기선 Prometheus 서버 자신을 가리킨다.
scrape_configs:
  # 이 설정에서 수집한 타임시리즈에 `job=<job_name>`으로 잡의 이름을 설정한다.
  # metrics_path의 기본 경로는 '/metrics'이고 scheme의 기본값은 `http`다
  - job_name: 'monitoring-item' # job_name 은 모든 scrap 내에서 고유해야함
    scrape_interval: 10s      # global에서 default 값을 정의해주었기 떄문에 안써도됨
    scrape_timeout: 10s       # global에서 default 값을 정의해주었기 떄문에 안써도됨
    metrics_path: '/asdf'     # 옵션 - prometheus가 metrics를 얻기위해 참조하는 URI를 변경할 수 있음 | default = /metrics
    honor_labels: false       # 옵션 - 라벨 충동이 있을경우 라벨을 변경할지설정(false일 경우 라벨 안바뀜) | default = false
    honor_timestamps: false   # 옵션 - honor_labels이 참일 경우, metrics timestamp가 노출됨(true일 경우) | default = false
    scheme: 'http'            # 옵션 - request를 보낼 scheme 설정 | default = http
    params:                   # 옵션 - request요청 보낼 떄의 param
      user-id: ['myemail@email.com']

    # 그 외에도 authorization 설정 
    # service discovery 설정(sd)

    # 실제 scrap 하는 타겟에 관한 설정
    static_configs:
      - targets: ['localhost:9090', 'localhost:9100', 'localhost:80'] ## prometheus, node-exporter, cadvisor  
        labels: # 옵션 - scrap 해서 가져올 metrics 들 전부에게 붙여줄 라벨
          service : 'monitor-1'
    
    # relabel_config - 스크랩되기 전의 label들을 수정
    # metric_relabel_configs - 가져오는 대상들의 레이블들을 동적으로 다시작성하는 설정(drop, replace, labeldrop)

```


rule을 통해 알림이 가게 만들수도 있다고 한다.
*rule.yml 예시*
```
groups:
- name: alerts # 파일 내에서 unique 해야함
  rules:

  # Alert for any instance that is unreachable for >5 minutes.
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  # Alert for any instance that has a median request latency >1s.
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
s
```

---

# Grafana 추가하기

이제 기존 docker compose의 `services:` 아래에 다음 설정을 추가한다.
```yml
grafana:
    image: grafana/grafana
    container_name: grafana
    # user: "$GRA_UID:$GRA_GID"
    ports:
      - 3000:3000 # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)
    volumes:
      - ./grafana/volume:/var/lib/grafana
    restart: always
    networks:
      - promnet
```

> monitoring_promnet이란 도커 네트워크를 별개로 사용한다. 원한다면 networks 설정을 변경해주자.


![[grafana-dashboard.png]]

Grafana의 특징은 import dashboard를 통해 대시보드 템플릿을 가져올 수 있다는 것이다.

어떻게 설정해야 할 지 막막했는데, 이것이 정말 큰 도움이 되었다.
import 해주는 것은 이 [자료](https://velog.io/@sojukang/%EC%84%B8%EC%83%81%EC%97%90%EC%84%9C-%EC%A0%9C%EC%9D%BC-%EC%89%AC%EC%9A%B4-Prometheus-Grafana-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81-%EC%84%A4%EC%A0%95#grafana-%EC%8B%A4%ED%96%89)를 참고하자.

스프링 대시보드들은 다음과 같다.
[Spring Boot Obeservability](https://grafana.com/grafana/dashboards/17175-spring-boot-observability/) - 2023 마지막 업데이트
[JVM (Micrometer)](https://grafana.com/grafana/dashboards/4701-jvm-micrometer/) - 2023 마지막 업데이트
[Spring Boot HikariCP / JDBC](https://grafana.com/grafana/dashboards/6083-spring-boot-hikaricp-jdbc/) - 2019 마지막 업데이트
[Spring boot 3.x Statistics](https://grafana.com/grafana/dashboards/19004-spring-boot-statistics/) - 2023 마지막 업데이트
[Spring Boot Cache (3.x)](https://grafana.com/grafana/dashboards/21303-cache/) - 2024 마지막 업데이트


