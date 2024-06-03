# Kubernetes?
쿠버네티스는 컨테이너화 된 워크로드와 서비스 관리를 자동화하여 시스템 운영을 수월하게 하는 오픈소스 컨테이너 오케스트레이션 도구다. 컨테이너가 실행 중에 장애가 발생하면 재시작하고 버전 업데이트가 필요하면 업데이트 정책에 따라서 컨테이너 업데이트를 실행하며 서비스를 생성하는 것만으로 로드밸런싱을 가능하게 합니다. 또한 서비스의 부하에 따라 컨테이너의 수를 조절하는 수평 확장을 지원하여 원활한 서비스 운영을 돕기도 합니다. 이런 특징 때문에 여러 서비스가 모여 하나의 애플리케이션을 구성하는 MSA나 자동화된 운영이 필요한 DevOps에 적합한 도구입니다.

## 컨테이너 오케스트레이션이란?
컨테이너를 하나의 서버에서 실행시킬 때는 컨테이너를 관리하는데 부담이 크지 않지만 실제 서비스는 여러 컨테이너들이 장애를 대비해 여러 서버에 분산되어있어 모든 컨테이너를 일일이 수동으로 관리하는 것은 효율적이지 않습니다. 이 때 필요한 것이 컨테이너 오케스트레이션입니다. 컨테이너 오케스트리에이션은 대규모 애플리케이션을 배포할 때 컨테이너의 생명 주기 관리, 네트워크, 스토리지 관리, 컨테이너 스케줄린 관리를 자동화 하여 인프라 관리를 간소화하고 운영 오버헤드를 줄일 수 있는 프로세스입니다.     

### 컨테이너 오케스트레이션 도구

| 도구 | 설명 |
| --- | --- |
| Kubernetes | 현재 컨테이너 오케스트레이션 표준으로 자리 잡은 도구 <br> 오픈소스 소프트웨어이며 거대한 커뮤니티가 활성화 되어 있어 커뮤니티의 지원을 받기 용이 <br> 대규모 애플리케이션 도입 사례가 풍부|
| Docker Swarm | Docker가 지원하는 클러스터링 기능이며 여러 컨테이너를 멀티호스트 환경에서 작동 및 관리 가능 <br> Docker 1.12 이전 버전에서는 Docker swarm이 Docker와 분리되어 있었지만 현재는 포함되어 있음. |
| Apache Mesos | 오픈소스 클러스터 오케스트레이션 도구 <br> 몇 천대의 대규호 호스트 클러스터링 지원 <br> 여러 호스트의 CPU, 메모리, 디스크를 추상화 하여 하나의 리소스 풀로 다룰 수 있음 <br> Mesos를 컨테이너 오케스트레이션에 사용하기 위해서 별도의 컨테이너 관리 프레임워크가 필요합니다. 이때 사용하는 대표적인 프레임워크가 Marathon이며 장기간 동안 지속해야하는 애플리케이션의 실행이나 모니터링이 가능|

## Kubernetes를 활용한 컨테이너 오케스트레이션
Kubernetes를 사용하는 주 목적은 요구된 상태를 유지시키는데에 있습니다. 만일 일정 개수의 컨테이너가 유지되어야 하는 요구조건이 있고 컨테이너가 장애로 인해 종료되면 Kubernetes는 요구조건을 충족하기 위해 다른 컨테이를 새로 실행하여 개수를 채우는 작업을 수행합니다.   
### Kubernetes 오케스트레이션의 기능

| 기능 | 설명 |
| --- | --- |
| Service discovery & Load Balancing | 파드에 고유한 IP 주소와 단일 DNS 이름 할당 <br> 파드 간 로드밸런싱 수행 가능 |
| Automated rollouts and rollbacks | 새로운 버전의 애플리케이션이나 애플리케이션 설정 변경시 업데이트 정책을 통해 점진적으로 업데이트를 수행하며 다운타임이 발생하지 않도록 함. 만일 업데이트 과정에서 오류가 발생하면 자동으로 변경 사항을 롤백. |
| Self healing | 파드의 상태를 확인하여 장애가 생긴 컨테이너를 재시작하고 노드에 문제가 발생하면 파드들을 정상 노드로 스케줄링을 수행합니다. 서비스 제공 준비가 되지 않은 파드에 대해서 클라이언트가 액세스하는 것을 방지 |
| Automatic bin packing | 리소스 요구 사항에 따라 컨테이너를 할당하고 효율적으로 리소스를 활용하며 가용성을 유지 |
| Storage orchestration | 로컬, 네트워크, CSP(Cloud Service Provider)에 위차한 스토리지를 kubernetes에 마운트 |
| Secret and configuration management | 애플리케이션에 필요한 비밀이나 구성 정보, 환경 변수과 같은 정보를 이미지를 다시 빌드하거나 외부에 노출시키지 않고 클러스터 내부에서 안전하게 관리 및 업데이트 가능 |
| Horizontal scaling | 명령이나 CPU 사용량에 따라 자동으로 애플리케이션을 수평 확장 가능 |

> 출처 : [쿠버네티스](https://kubernetes.io/ko/)

### Kubernets 설계 사상

| 설계 사상 | 설명 |
| --- | --- |
| 선언적 구성 기반의 배포 환경 | 요구하는 상태를 선언하고 현재 상태와 비교를 지속하여 자동 복구되도록 배포 <br> 요구 상태 선언은 YAML 파일을 사용 |
| 기능 단위의 분산 | 각각의 기능들이 개별적인 구성요소로서 독립적으로 분산되고 컨트롤러로 관리 <br> Node, Deployment, ReplicaSet, Namaspace |
| 클러스터 단위의 중앙 제어 | 전체 물리 리소스를 클러스터 단위로 추상화하여 관리 <br> 컨트롤 플레인을 통해 클러스터를 관리 |
| 동적 그룹화 기능 | 구성 요소들에 Label과 Annotation을 키-값 단위로 설정 |
| API 기반의 상호 작용 | 구성 요소들은 kube-apiserver를 통해서만 상호 접근이 가능 |


## 참고 자료
- [쿠버네티스란 무엇인가?](https://kubernetes.io/ko/docs/concepts/overview/)
- [베스픽 - 핫한 게 아니라 대세입니다. '쿠버네티스' 이해하기](https://blog.bespinglobal.com/post/%ED%95%AB%ED%95%9C-%EA%B2%8C-%EC%95%84%EB%8B%88%EB%9D%BC-%EB%8C%80%EC%84%B8%EC%9E%85%EB%8B%88%EB%8B%A4-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0/)
- [seongjin.me - 쿠버네티스와 컨테이너 오케스트레이션, 그리고 핵심 설계 사상](https://seongjin.me/kubernetes-core-concepts/)
- fastcampus 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online. - 컨테이너 서비스 개요