# Monitoring

- 모니터링은 시간의 흐름에 따른 시스템 및 여러 구성 요소의 동작과 출력 등을 관찰하고 확인하는 작업을 통해 자원의 효율적인 사용을 식별, 평가
- 모니터링 전략을 세우려면 무엇을, 왜, 어떻게 모니터링해야 하는지 우선 고려
- 모니터링은 활용률 및 처리량 같은 Metric에 중점을 두고 시스템 전반의 성능을 알 수 있다.
    - 메모리 사용량 급증, 캐시 적중률 감소, CPU 사용량 증가 등을 관찰
- 모니터링은 애플리케이션 중심의 MSA 같은 복잡한 아키텍처를 진단하기에 어려움이 있다.
- 몇 가지 Metric 간의 상관 관계는 애플리케이션 장애 진단에 충분한 정보를 제공하지 못하기 때문에 복잡한 MSA 환경의 애플리케이션에 대한 가시성 있는 관찰을 위해 Observability 도구가 요구
    - 관찰 가능성 → 모니터링, 로깅, 추적, 시각화

## cadvisor

- 컨테이너 리소스 Metric 정보를 수집하는 도구
- Google에서 제공하고 관리하는 오픈 소스 컨테이너 모니터링 도구
- Docker 컨테이너와 다른 컨테이너 플랫폼에도 기본적으로 지원 가능
- cadvisor는 docker hostos에서 실행 중인 컨테이너에 대한 정보를 수집하고 해당 데이터를 처리한 후 내보내는 단일 컨테이너 데몬으로 구성
- cadvisor는 이전 리소스 사용량, 리소스 격리 매개 변수 및 각 컨테이너 머신 전체에 대한 네트워크 통계를 기록
- 각 컨테이너는 독립적인 개별 환경이고, 이를 모니터링하여 각 애플리케이션의 정보를 수집하여 현재의 성능과 개선점을 진단
- cadvisor는 컨테이너에 포함된 애플리케이션 metric을 수집하여 활성 및 읽기 연결 수, 애플리케이션에 적절한 CPU 및 메모리 할당이 있는지를 알 수 있다.
- Metric data는 전용 웹 인터페이스 또는 Google Big Query, Elastic Search, Kafka, Prometheus, Redis와 같은 다른 수집 데이터 분석 가능 도구와 연동 가능
    
    ### 컨테이너 실행
    
    ```bash
    docker run \
    --restart=always \
    --volume=/:/rootfs:ro \
    --volume=/var/run:/var/run:rw \
    --volume=/sys/fs/cgroup:/sys/fs/cgroup:ro \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --volume=/dev/disk/:/dev/disk:ro \
    --publish=9559:8080 \
    --detach=true \
    --name=cadvisor \
    --privileged \
    --device=/dev/kmsg \
    gcr.io/cadvisor/cadvisor:latest
    ```
    
    - 컨테이너 실행 후 웹 인터페이스에 접속하면 컨테이너마다 지표가 출력되는 것을 확인할 수 있다.
        
        <img src="/images/Container_monitoring.png" width="50%" height="50%" title="container monitoring" alt="container monitoring">
        
- Observability tool, Prometheus + Grafana
    - 모니터링 : 인프라 로그 메트릭을 검사하여 작업 및 인사이트를 수행
    - 로그 : 특정 시간에 발생한 이벤트 기록
    - 추적 : 원인 관련 이벤트의 요청 흐름을 캡처하는데 사용
    - 시각화 : 모든 정보를 차트 등의 시각적 효과를 통해 제시하여 빠른 인사이트를 제공
    - Prometheus는 Node exporter를 사용한 push 방식의 모니터링을 사용한다.
        - docker compose를 사용해서 cadvisor의 정보를 프로메테우스로 전달하도록 할 수 있고 이것을 grafana로 시각화 역시 가능