<details>
<summary>컨테이너 기술이란?</summary>
<div markdown="1">

### 컨테이너 기술
- 컨테이너는 애플리케이션을 언제든 실행 가능하도록 필요한 모든 요소들을 하나의 런타임 환경으로 패키징한 호스트 OS상의 논리적 공간
    - Docker에서 Dockerfile을 사용해서 빌드한 이미지
- 애플리케이션과 종속 항목을 하나로 묶어 실행하게 해주는 운영 시스템을 가상화한 경량의 격리된 프로세스
    - MicroVM이라고도 함
    - OS 수준의 가상화 제공
    - Stateless : 다른 컨테이너에 영량을 주지 않는 독립성을 가공
- 로컬이나 온프레미스, 클라우드에서 언제든지 빠르고 효율적으로 배포 가능 및 높은 확장성을 가지고 있다.
- 서버 구성, OS 설치, 네트워크, 개발 도구 구성과 같은 반복적이고 불편한 작업에 시간을 낭비하지 않고 개발자는 애플리케이션 개발 그 자체에 집중할 수 있다.
    - 개발, 테스트, 운영을 모두 같은 환경에서 실현할 수 있다.
    -  Snowflake: 눈송이 서버
       - 개발, 테스트, 운영 서버가 모두 조금씩 다른 서버를 의미

   
### 컨테이너 특징
- 개발한 최소한의 이미지를 통해 실행되므로 경량
    - 컨테이너 이미지 생성의 Best Practice 중 하나는 이미지 경량화
        - MSA에 적합하다.
- 언제든 프로세스 수준의 속도로 빠르게 실행 가능하며 한번에 여러 개의 컨테이너를 동시에 실행 가능
    - 컨테이너 오케스트레이션
    - Kubernetes가 가장 대표적인 컨테이너 오케스트레이션 도구
    - Docker에서는 docker compose를 통해 구현
- 개인, 온프레미스, 클라우드 환경이든 어떤 OS, 어떤 환경에서도 동작 가능한 이식성을 가지고 있다.
- 컨테이너 자체의 환경에 대한 관리만 요구되므로 지속적인 서버관리 비용을 절감할 수 있다.
    - 애플리케이션에 포인트를 잡아 관리 가능
    - 플랫폼에 집중할 수 있다.
- 개발팀과 운영팀의 업무 분리로 각자의 업무와 세분화된 관리에 집중할 수 있다.
    - DevOps Workflow 구성에 최적
    - 운영팀 : 개발팀에서 원하는 이미지를 빌드, 인프라 제공, 개발팀의 이미지를 관리, CICD
    - 개발팀 : 운영팀에서 제공하는 인프라에 소스를 올림.

### 컨테이너 사례
- 대규모 애플리케이션 서비스, 여러 기업의 다양한 능플리케이션 환경, 모바일 앱 서비스
    - 구글 웹, 앱 서비스
        - 일주일에 업다운하는 컨테이너 수가 20억개 정도
    - 에어비앤비 추천 서비스
    - 넷플릭스 추천 서비스
    - 당근마켓 딥러닝 기반 추천 서비스
    - 엔씨소프트 게임 서비스
    - 삼성전자 헬스 케이 서비스
    - 타다 배차 서비스
    - 토스 금융 서비스

### 컨테이너 타입
- 컨테이너 패키징 메커니즘
    - 시스템
    - 애플리케이션
    - 라우터
- 시스템 or OS 컨테이너
    - 호스트 OS 위에 Ubuntu와 같은 배포판 리눅스 이미지를 통해 배포되는 컨테이너
    - 또다른 VM 형태이며 내부에 다양한 애플리케이션 및 라이브러리 도구를 설치, 실행 가능
    - LXC, LXD, OpenVZ, Linux VServer, BSD Jails
- 애플리케이션 컨테이너
    - Docker 컨테이너의 주요 목적
    - 단일 애플리케이션 실행을 위해 해당 서비스를 패키징하고 실행하도록 설계
        - 일반적인 OS의 경우 PID 1는 Systemd 프로세스이지만 애플리케이션 컨테이너의 경우 대상 서비스를 의미한다.
        - nginx 컨테이너의 PID 1은 nginx이다.
    - 3-tier 애플리케이션 같은 경우 각 tier(frontend-backend-DB)를 개별 컨테이너로 실행하여 연결
    - Docker container Runtime

### Docker
- 애플리케이션의 실행에 필요한 환경을 하나의 이미지로 모아두고, 이미지를 사용하여 다양한 환경에서 애플리케이션 실행 환경을 구축 및 운용하기 위한 오픈소스 플랫폼
- 여러 계층의 애플리케이션를 컨테이너로 분리, 연결하여 실행하는 MSA 아키텍처 프로젝트에 유용
    - 각 컨테이너를 API를 사용해서 연결
- 애플리케이션의 인프라를 이미지로 제공
    - Public or Pri폼ate하게 공유 가능
    - Github과 유사한 방식(open share)의 Docker Hub에서 제공
        - AWS ECR, GCP GCR에서도 같은 기능 제공
- 제공된 이미지를 기반으로 애플리케이션 서비스를 제공 및 컨테이너화 가능
    1. 애플리케이션 인프라 구성 - Dockerfile
    2. 애플리케이션 패키징 - Docker Image
    3. 이미지 공유 - Docker Hub
    4. 애플리케이션 배포 - Docker Container
</div>
</details>
   
<details>
<summary>Docker 컨테이너 가상화와 VM 가상화 비교 </summary>
<div markdown="1">

### 가상화

- 일반적으로 서버, 스토리지, 네트워크, 애플리케이션 등을 가상화
    - 하드웨어 리소스의 효율적 사용
    - 효율적인 자원 활용, 자동화된 IT 관리, 빠른 재해 복구
- 물리적인 하드웨어 유지 관리 대신 소프트웨어저그올 추상화된 가상화를 통해 제한된 부분을 쉽게 관리 유지
- 하이퍼바이저 기반의 가상머신을 통해 수생
    - Vmware
    - VirtualBox

### 컨테이너 가상화 vs VM 가상화 vs Hypervisor
#### 공통점
- 실행하고자 하는 애플리케이션 프로세스 및 종속성, 소스 등을 이미지화 하여 HostOS와 격리된 환경 제공
#### 차이점
##### 컨테이너
![Container](/images/_Container.drawio.png)
- 컨테이너는 컨테이너 관리 소프트웨어 위에서 컨테이너가 동작
- OS, 디렉토리, IP 주소와 같은 시스템 자원을 애플리케이션이 점유하고 있는 것처럼 보임
- 컨테이너 가상화는 VM 가상화에 비해 경량이면서 HostOS의 커널을 공유하는 OS 수준의 가상화
    - 별도의 커널이 필요없어 경량이다.
    - Docker 명령 몇 줄로 바로 실행가능
    - 원하는 애플리케이션 환경을 빠르게 번들링하여 패키징
##### VM 가상화
![Host visualization](/images/_Host_visualization.drawio.png)
- VM 가상화는 실제 HostOS와 동일한 역할을 수행하는 별도의 GuestOS+Kernel를 두고 원하는 애플리케이션을 설치하는 하드웨어 수준의 가상화
    - 부팅이 필요
    - 가상화를 수행하기 위한 오버헤드가 크다.
        - CPU 자원
        - 디스크 용량
        - 메모리 사용량
- 가상화 소프트웨어를 사용하여 간편한 가상 환경 구축 가능
    - VirtualBox
    - VMware
- 개발 환경 구축에 주로 사용
##### Hypervisor
![hypervisor](/images/_hypervisor_visualization.drawio.png)
- Hypervisor를 사용한 가상화는 호스트 OS 없이 Hypervisor가 하드웨어를 직접 제어하므로 자원 효율성이 높다.
- 가상 환경은 Hypervisor 위에서 동작한다.
- VM 가상화와 동일하게 가상 환경마다 별도의 OS가 필요하므로 가상 환경 시작에 오버헤드가 크다.
- MS의 Windows의 Hyper-V
    - Hyper-V를 사용해서 WSL2로 Windows에서 Linux 커널을 네이티브에 가깝게 사용가능
    - [WSL2](https://learn.microsoft.com/ko-kr/windows/wsl/about)

### 컨테이너화 기술
- 리눅스 컨테이너 기술은 LXC(Linux Container)를 이용한 시스템 컨테이너화로 시작
    - OS 수준의 가상화 도구
    - cgroup, namespace이라는 리소스 관리 장치를 사용하여 분리된 환경을 제공
    - chroot를 사용하여 특정 디렉토리를 루트 디렉토리로 바꾸는 형식으로 데이터 영역을리분리
- 컨테이너 기반 Docker는 초기에 LXC를 활용해 컨테이너를 생성
    - 지속적인 발전으로 containerd, runC를 이용하는 방식으로 변경
    - runC : 커널 기술의 공유를 통해 컨테이너 생성을 지원
    - containerd : 생성된 컨테이너의 라이프사이클 관리를 지원
    - dockerd : 사용자 환경에서 명령을 전달
</div>
</details>

<details>
<summary>Play with Docker</summary>
<div markdown="1">

### Docker 컨테이너 놀이터

[Play with Docker | Docker](https://www.docker.com/play-with-docker/)

- Docker가 웹에서 제공하는 인스턴스 형태의 Docker 랩실을 제공
- 한 인스턴스 당 4시간이며 무료

### 사용

1. Docker Hub 계정으로 로그인한 후 세션에서 새로운 인스턴스 추가
    
    ![play_with_docker](/images/play_with_docker.png)
    
2. 터미널에서 docker 명령 테스트
</div>
</details>