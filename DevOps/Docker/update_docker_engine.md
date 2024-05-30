# Update Docker Engine
## Docker 엔진 업데이트 이점
새로운 엔진이 릴리스 될 때마다 릴리스 노트나 Docker docs에서 이전 버전 대비 변경점을 확인할 수 있다. 이를 살펴보면 추가된 기능이나 버그 수정, 패키지 업데이트, 삭제된 기능과 API 변경점과 같은 내용을 찾을 수 있다. Docker Engine의 업데이트는 수시로 릴리스 되서 변경점을 확인하여 보안 취약점 개선, 성능 개선과 같은 이점을 활용하는 것을 권장한다.   

아래 테이블은 26.0.0 버전 릴리스 이후 마이너 버전 릴리스와 패치가 이루어진 날짜를 보여준다. 26.0.0이 릴리스 된 이후로  1~2주에 한 번 새로운 버전이 릴리스 될 정도로 빈번히 새로운 버전이 업데이트 되었다.

|릴리스 버전|릴리스 날짜|
|-|-|
|26.1.3|2024-05-16|
|26.1.2|2024-05-08|
|26.1.1|2024-04-30|
|26.1.0|2024-04-22|
|26.0.2|2024-04-18|
|26.0.1|2024-04-11|
|26.0.0|2024-03-20|     

먼저 26.0.0 버전의 릴리스 노트를 확인하면 메이저 업데이트여서 보안 패치, 새로 추가된 점, 버그 수정 및 개선 사항, API 변경, 패키지 업데이트, 제거된 사항이 눈에 띈다. docker image ls 명령에서 다중 플랫폼 이미지에서 중복된 entry를 생성하지 않는 점과 같이 새로 추가된 점과 DNS 서버가 루프백일 때 요청이 외부 DNS 서버로 포워딩되는 문제를 내부 네트워크에만 연결되도록 버그 수정, TLS 없는 TCP 연결 기능 사용 중지나 1.24 이전 API 사용 중지와 같은 변경사항을 릴리스 노트에서 확인 할 수 있다. 26.0.1이나 26.0.2는 사소한 버그 수정한 버전이다.
26.1.0은 명령에 OpenTelemetry 유틸리티와 기본 instrumentation에 대한 구성이 추가되었고 windows 컨테이너에서 windows-dns-proxy를 사용할 수 있는 기능이 추가되었다. 이 기능은 Docker 엔진 27.0에 기본 기능으로 추가 예정이고 windows-dns-proxy 플래그는 추후 삭제 예정이다.
- 기능 플래그 : 코드 자체를 업데이트 하지 않고 소프트웨어의 특정 기능을 활성화하거나 비활성화하는 것
> 출처 : [Jetbrain CI/CD 개념 - 기능 플래그란?](https://www.jetbrains.com/ko-kr/teamcity/ci-cd-guide/concepts/feature-flags/)   
출처 : [Docker Engine 26.1 Release Notes](https://docs.docker.com/engine/release-notes/26.1/)     
출처 : [Docker Engine 26.0 Release Notes](https://docs.docker.com/engine/release-notes/26.0/)

이처럼 Docker Engine은 지속적으로 새로운 업데이트를 릴리스하며 최신 Docker Engine을 사용할 때 다음과 같은 이점을 준다.
1. 기존 기능의 개선 및 새로운 기능
    - 플랫폼의 기능과 유용성을 개선하는 새로운 기능 및 개선 사항이 포함된 업데이트를 정기적으로 릴리스하고 새로운 docker 기능을 도입하여 모든 작업의 workflow를 단순화
    - Docker Document를 확인하면 기존 예시와 새로운 예시를 비교하여 어떻게 단순화된 것을 볼 수 있다.
2. 버그 수정
    - 예기치 못한 버그가 보고되면 이를 해결하여 안정성 및 성능을 개선하는 수정사항을 제공
    - 원활한 docker 작업을 보장받을 수 있다.
3. 보안 패치
    - 보안 취약성을 지속적으로 조사하여 잠재적 악용 위험을 최소화
    - 컨테이너화 된 애플리케이션의 전반적인 보안 태세를 개선
    - 주로 커널 관련 보안 패치가 이루어진다.
4. 성능 개선
    - 성능 최적화가 포함된 업데이트로 작업을 더 빠르고 효율적으로 만든다.
    - 컨테이너 시간 단축, 네트워킹 및 I/O 성능 향상, 전반적인 리소스 활용도 향상
    - 애플리케이션 최적화 가능
5. 최신 기술과의 호환성
    - Docker는 docker 서비스만을 이용하기보단 주로 다른 기술들과 결합해서 사용한다. 최신 기술과 호환성을 위해서는 지속적으로 docker 업데이트가 필요하다.
6. 커뮤니티 및 생태계 지원
    - Docker 기반 플러그인 및 통합을 활용해 컨테이너화 된 애플리케이션을 빌드 및 관리하기 위한 옵션을 사용 가능
7. 유지 관리 및 오랜 기간 동안의 지원 (LTS - Long Term Support)
    - 일부 버전에 LTS를 지정하는 관리체계를 따른다.
    - LTS 버전은 버그 수정, 보안 패치를 포함한 확장된 유지 관리 및 지원을 받아 중요한 프로덕션 환경에 적합
    - 장기적인 요구 사항에 대해 안정적이고 잘 지원되는 docker 환경을 보장
> 출처 : Fastcampus - 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online

## Docker Engine Update
1. 기존에 실행 중인 컨테이너들을 stop
    
    ```bash
    docker ps
    CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                   NAMES
    6b64dd68a462   nginx:1.25.3-alpine   "/docker-entrypoint.…"   7 seconds ago   Up 6 seconds   0.0.0.0:8001->80/tcp, :::8001->80/tcp   webserver
    
    docker stop 6b64dd68a462
    6b64dd68a462
    ```
    
2. 현재 사용 중인 기존 버전의 docker 엔진을 삭제
    
    ```bash
    sudo apt-get update
    sudo apt -y remove docker-ce
    ```
    
3. 최신 버전의 docker 엔진 설치
    - Docker Engine의 새로운 버전에 대한 GPG 키와 레포지토리를 추가하고 최신 버전의 docker engine 설치
    
    ```bash
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    docker --version
    Docker version 26.1.3, build b72abbb
    ```
    
4. 기존 버전에서 운영했던 컨테이너 가동
    
    ```bash
    docker start webserver
    webserver
    ```
    
5. if, error 발생 시 원인 파악, 문제 해결 → 중지된 컨테이너 start