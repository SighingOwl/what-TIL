본 내용은 Fastcampus의 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online 강의 내용을 정리한 내용입니다.

# Kubernetes 아키텍처
## Kubernetes 구조 이해

### Kubernetes 구조

<img src="/images/kubernetes_architecture.drawio.png" width="50%" height="50%" title="kube architecture" alt="kube architecture">   


- Kubernetes architecture는 클러스터 전반의 서비스 검색(Service Discovery)을 위해 제공하기 위한 오픈소스 플랫폼
    - Kubernetes 클러스터에는 하나 이상의 control plane과 하나 이상의 컴퓨팅 Node(work node or data plane)
    - Control plane은 전체 클러스터를 중앙 관리하고, API 트래픽을 통해 각 Node와 통신하며, 사용자가 원하는 상태(Desire State) 제공을 위해 적합한 컴퓨팅 Node에 object 스케줄링
    - 각 컴퓨팅 Node는 Control plane과 통신하는 Kubelet daemon과 함께 Docker, containerd, CRI-O와 같은 컨테이너 런타임을 실행
    - 각 Node는 온프레미스 서버, 클라우드 기반 가상머신, 베어메탈 서버 등으로 구성
- **Control Plane  - Master Node**
    - API Server (REST API) - 가장 중요한 요소
        - 중앙에서 모든 것을 관리
        - 개발자가 kubectl이나 dashboard를 사용할 때 API Server에 연결된다.
    - Scheduler
        - 파드를 노드에 바인드
    - etcd (key-value DB, SSOT)
    - Controller Manager
        - 대부분의 Kubernetes 리소스 오브젝트를 관리
        - 지속적으로 제어 루프를 돌면서 리소스를 관찰
        - 리소스에 문제가 발생하면 자동으로 재시작하거나 재생성 등을 수행
    - Cloud Controller Manager - CCM
        - Kubernetes가 클라우드와 연결하기 위해 Cloud Provider API를 사용하는 클라우드 연결점
    - kubelet
    - kube-proxy
    - 클라우드 벤더의 관리형 kubernetes를 사용하면 Control Plane 부분은 클라우드에서 직접 관리한다.
- **Worker Node**
    - kube-proxy
        - kube-proxy에 한개의 노드가 구성됨
        - 네트워크와 kubelet을 CNI를 사용해 연결
        - kubernetes 런타임
            - Docker
            - containerd
                - Kubernetes 오브젝트를 생성할 수 있는 containerd-shim → shim
                - run를 통해 OS 커널과 연결
            - CRI-O
        - kubernetes 오브젝트
- **BIRD**
    - Calico의 모듈로 vRouter(가상 라우터)를 구현하고 BGP를 이용하는 동적 IP 라우팅 데몬(L3)

### **예시) Pod 생성**

1. yaml파일을 TLS 암호화로 전달
    - apiserver 도달 후
1. Authentication check
    - 적합한 계정인지 확인 - 인증
2. Authorization check
    - 사용자가 권한이 있는지 확인 - 인가
3. Admission controller
    - 인증, 인가 정보 확인
4. etcd에 pod 생성 승인 요청을 기록
    1. 기록이 끝나면 API Server로 완료 메세지 전송
5. API Server가 Scheduler에 Pod가 바인딩 될 Node 요청
    1. Scheduler가 적절한 Node를 선택하기 위한 정보를 API Server를 통해 etcd에 요청
    2. Scheduler가 Node들의 정보를 활용하여 적합한 Node를 선택
    3. 선택한 Node를 API Server로 반환
6. API Server가 선택된 Node의 kubelet에 pod 생성 요청
    1. kubelet은 kubernetes 런타임을 통해 pod container를 생성하는 지시를 내린다.
    2. Kubernetes 런타임은 OCI - Open Container Initiate으로 커널의 컨테이너 기술을 활용해 pod container를 생성
7. Pod를 생성 후 Kubelet은 CNI로 연결된 네트워크를 통해 API Server에 pod 정보를 넘겨준다.
8. API Server는 받은 정보를 etcd에 기록
    1. SSOT - Single Source of Truth
    2. 단일 소스의 진실을 계속 업데이트 한다.
9. API Server는 사용자의 요청이 있을 때마다 리소스의 상태를 알려준다.

### Pod를 통한 구조 검증

```bash
kubectl run <pod-name> --image=<image-name>...
```

1.  이 요청은 API 서버에 의해 검증
2. API 서버는 이 요청을 control plane의 Scheduler로 전달
3. API 서버는 etcd 저장소와 상호 작용할 수 있는 유일한 구성 요소이므로 Scheduler는 API 서버에 클러스터 관련 정보를 요청
4. API 서버는 Scheduler가 요청한 정보를 etcd 저장소에서 데이터를 읽어 제공
5. 정보를 받은 Scheduler는 해당 정보를 기반으로 지정된 노드에 Pod를 할당하고 이 메시지를 API 서버에 전달
6. API 서버로부터 요청을 받은 노드의 kubelet은 CRI를 통해 컨테이너 런타임과 상호 작용하여 노드에서 Pod가 생성, 실행 됨
    - CRI - Container Runtime Interface
7. Pod가 실행되는 동안 Controller Manager는 원하는 클러스터 상태(Desire State)가 Kubernetes 클러스터의 실제 상태(Current State)와 일치하는지 지속적으로 확인

### Kubernetes Cluster

- Control Plane과 Worker Node를 묶은 것을 Cluster라고 한다.
- 컨테이너화 된 애플리케이션을 실행하는데 사용되는 물리적 및 가상 Node 그룹
- K8s object 성능과 안전성을 보장할 수 있는 시스템을 구축하기 위해 모든 Node를 그룹화하는 것을 클러스터

### Kubernetes Node

- Node는 Kubernetes cluster에 소속된 단일 서버
    - 실제 workload인 Pod가 실행되는 기본 단위, Pod 호스팅
    - Pod 내부에서는 우리가 필요로 하는 애플리케이션 컨테이너를 실행
    - 컨테이너 내부에서 독립적인 애플리케이션 실행
- Control Plane은 Worker node와 클러스터 내 Pod를 관리하고, Worker Node는 애플리케이션의 구성요소인 Pod를 호스트한다. - Data Plane은 작업이 실행되는 영역
- 운영 환경에서는 일반적으로 Control Plane이 여러 서버에 걸쳐 관리를 수행하고, 클러스터는 일반적으로 여러 Node를 실행하므로 내결함성과 고가용성이 제공된다.
- Node는 “kube-controller-manager”에 의해 5초 간격으로 노드 상태를 체크하여 API 서버에 보고
    1. Node가 등록될 때 Node에 CIDR Block 할당
    2. 내부 Node 정보를 최신으로 유지, Cloud Provider와 연동 시 사용
    3. Node 상태 모니터링 (—node-monitor-period=5)
- Node가 사용 중인 리소스 용량에 대한 정보를 추적하여 API 서버에 보고 됨 - by kubelet
    - 이 정보는 Scheduler가 모든 Node의 모든 Pod에 충분한 자원이 있는지 확인할 때 사용
- **가용성 유지**
    - kubelet
        - node-status-update-frequency: 기본 10s (api-server에 노드의 상태를 게시하는 주기)
        - node-status-report-frequency: 기본 5m
    - controller-manager
        - node-monitor-period: 기본 5s (node-controller에서 node status를 동기화하는 시간)
        - node-monitor-grace-period: default 40s (node로부터 해당 시간 동안 응답을 받지 못한 경우 상태를 Not Ready 상태로 변경)
        - pod-eviction-timeout: 기본 5m (노드에서 Pod를 삭제하기 위한 대기 시간)

### Kubernetes Worker Node

- **구성 요소**
    - kubelet
    - kube-proxy
    - container runtime
    - Addons
        - CNI plugin
        - coreDNS for DNS
        - Metric Server
        - Web UI Dashboard
- **kubelet**
    - Control Plane 및 Worker Node에 모두 존재
    - Pod 실행(생성, 변경, 삭제)을 위해 control plane의 API 서버와 통신 수행
        - Worker Node를 API 서버에 등록하고, API 서버에서 podSpec을 받아서 처리하는 에이전트 역할
    - liveliness, readiness, and startup probes 기능 처리
    - Node의 container runtime을 사용하여 image를 가져오고 컨테이너를 실행하는 등의 작업 수행
    - kubelet은 container를 관리하고 Pod가 원하는 상태인지 확인하는데 중요
- **kube-proxy**
    - Service object는 Pod를 트래픽에 노출, Endpoint object에는 Pod IP 주소와 Port포함
        - kube-proxy는 Pod용 Service 구현체
    - Service를 사용하여 Pod를 노출하면 kube-proxy는 Service object와 그룹화된 Pod로 트래픽을 보내는 네트워크 규칙 생성
    - kube-proxy는 UDP, TCP, SCTP 프로토콜을 Proxy, HTTP는 no Proxy
        - 모든 노드에서 DaemonSet으로 실행
    - kube-proxy는 API 서버와 통신, 서비스와 해당 PodIP, Port에 대한 세부 정보를 가져옴
    - Service 변경 사항 및 Endpoint 모니터링 후 다양한 Mode를 사용하여 Service에 연결된 Pod로 트래픽을 Routing 하기 위한 규칙(Rule)을 생성, 업데이트
    - Mode에는 IPTables, IPVS, User space, Kernel space 포함
        - IPTables 모드를 사용하는 경우 kube-proxy는 IPTables 규칙으로 트래픽을 처리하고 로드밸런싱을 위해 백엔드 Pod를 Random 선택
- **container runtime**
    - OCI(Open Container Initiative)는 컨테이너 형식 및 런타임에 대한 표준 집합.
        - Kubernetes는 CRI-O, Docker(cri-docker), containerd, Mirantis와 같은 CRI를 준수하는 여러 Container runtime을 지원
    - kubelet은 CRI API를 사용하여 container runtime과 상호작용하여 컨테이너의 수명주기를 관리하고 모든 컨테이너 정보를 Control plane에 제공
- **Addons**
    - 활성화 시 Kubernetes cluster 성능 및 기능 향상에 도움
    - CNI plugin
    - CodeDNS(DNS 서버용): CoreDNS는 DNS 기반 Service Discovery 기능 제공
    - Metric Server(리소스 메트릭용): 클러스터에 있는 Node 및 Pod의 성능 데이터와 리소스 사용량을 수집하여 주로 HPA 구성에 사용
    - Web UI(Dashboard): Kubernetes dashboard를 통해 K8s resource object 관리

## Kubernetes 주요 구성 요소

### Master Node - control plane role

- control plane node는 각각 특정 작업을 담당하는 여러 구성요소로 구성
- 구성 요소들은 유기적인 동작을 통해 각 Kubernetes 클러스터의 상태가 사전 정의된 원하는 상태와 일치하는지 지속적으로 확인
    - kube-apiserver
    - etcd
    - kube-scheduler
    - kube-controller-manager
    - (선택) kube-cloud-controller-manager

### **kube-apiserver**

- kube-apiserver는 Kubernetes API를 사용하는 Kubernetes 클러스터의 중앙 관리 도구
    - API 요청을 관리
    - API 객체에 대한 요청 처리 및 Pod 등의 데이터 유효성 검사
    - 사용자에 대한 인증 및 권한 평가(RBAC)를 통해 승인 제어 (admission controller)
    - etcd와 통신하는 유일한 구성요소
    - Control plane과 worker node 구성 요소 간의 모든 작업 조정
- 여러 control plane 구성요소와 통신하고 모니터링 및 타사 서비스와 통신 가능
    - kubectl 시 HTTP Request(HTTP REST API)를 통해 kube-apiserver와 통신
    - 내부 구성요소인 스케줄러, 컨트롤러 등과는 gRPC를 사용하여 kube-apiserver와 통신
    - kube-apiserver와 클러스터의 다른 구성 요소 간의 통신은 클러스터에 대한 무단 액세스를 방지하기 위해 TLS(Transport Layer Security)를 통해 진행

### **etcd - key-value 기반의 오픈 소스 data storage**

- 모든 클러스터 관련 정보는 etcd 내에 재정의가 아닌 추가를 통해 저장(SSOT, 애플리케이션 데이터 제외)
- 공간 활용율을 높이기 위해(데이터 저장소 크기 최소화) 주기적으로 압축(or 파쇄) 수행
- B+tree 구조로 BboltDB 위에 구축
- 보안을 위해 암호화 기능 제공
- 강력한 일관성
    - Node 업데이트 시 강력한 일관성을 통해 클러스터의 다른 모든 Node에 즉시 업데이트
- 분산형
    - etcd는 일관성을 유지하면서 여러 노드에서 클러스터로 실행되로록 설계
- Key-value 저장 구조
    - 데이터를 키와 값으로 저장하는 비관계형 데이터베이스
- Raft Consensus Algorithm - Ralf 합의 알고리즘
    - leader-member 구성으로 고가용성 유지 (leader 선 기록 후 member에 복제)
- **동작**
    - Kubernetes 객체(Pod, Deployment, Secret 등)의 모든 구성 및 상태, 메타데이터 저장
    - API 서버는 etcd의 감시 기능(Watch())을 사용하여 객체 상태의 변화를 추적
        - etcd client가 API를 사용하여 Event 구독
    - control plane의 유일한 Statefulset 구성 요소
    - gRPC를 사용하여 키-값 API를 노출, 유일하게 API 서버와 통신
    - /registry 디렉터리 키 아래의 모든 객체를 키-값 형식으로 저장
        - Default NS 내에 Nginx Pod 생성 → /registry/pods/default/ngin
- etcd 확인
    
    ```bash
    kubectl exec -it -n kube-system etcd-k8s-master -- sh
    sh-5.2# etcdctl -h
    NAME:
    	etcdctl - A simple command line client for etcd3.
    
    USAGE:
    	etcdctl [flags]
    
    VERSION:
    	3.5.10
    
    API VERSION:
    	3.5
    ..
    ```
    
    - etcd는 기본적으로 sh를 쉘로 사용
    - etcdctl -h를 사용하면 etcd에 대한 간략한 정보를 확인 가능
- 현재 etcd의 상태 값
    
    ```bash
    kubectl -n kube-system exec -it etcd-k8s-master -- sh \
    -c "ETCDCTL_API=3 \
    ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
    ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
    ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
    etcdctl endpoint health"
    127.0.0.1:2379 is healthy: successfully committed proposal: took = 57.594797ms
    ```
    
- ectd 멤버 리스트
    
    ```bash
    kubectl -n kube-system exec -it etcd-k8s-master -- sh \
    -c "ETCDCTL_API=3 \
    ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
    ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
    ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
    etcdctl member list"
    76335a6259872c1a, started, k8s-master, https://192.168.56.100:2380, https://192.168.56.100:2379, false
    ```
    
    - 멀티 클러스터일 경우 멤버가 여러개 출력될 수 있다.
- etcd 백업 및 복원 - CKA, CKAD 기출
    
    ```bash
    kubectl -n kube-system exec -it etcd-k8s-master -- sh \
    -c "ETCDCTL_API=3 etcdctl \
    --endpoints=127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /var/lib/etcd/snapshot.db"
    ```
    
    - etcd에 대한 주기적인 백업을 크론잡을 사용해서 수행하기도 한다.
    - 날짜 단위로 구별해서 저장한다.
    
    ```bash
    kubectl -n kube-system exec -it etcd-k8s-master -- sh \
    -c "ETCDCTL_API=3 etcdctl \
    --endpoints=127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot restore /var/lib/etcd/snapshot.db"
    ```
    
    - snapshot restore를 사용해서 원하는 백업데이터를 사용해서 복원
    
    ### kube-scheduler
    
    - kube-scheduler는 worker node에 Kubernetes Pod 스케줄링(예약, 할당) 담당
        - API 서버에서 Pod 생성 이벤트를 수신하는 컨트롤러
    - Pod 배포시 CPU, 메모리, 선호도, 오염, 허용 범위, 우선 순위, 영구 볼륨(PV) 등과 같은 Pod 요구 사항 지정
    - 스케줄링 단계 (Scheduling context)
        - Scheduling cycle: Worker node 선택
        - Binding cycle: 변경 사항(생성, 변경, 삭제)을 클러스터에 적용
    - kube-scheduler의 주요 작업은 생성 요청을 식별하여 가장 적합한 Node를 선택하는 것
    - **동작**
        1. kubectl 등을 통해 Pod 생성 요청을 Kube-apiserver에게 전달
        2. kube-apiserver는 요청 정보를 etcd에 기록 (pod state)
        3. Client에게 요청 승인 알림
        4. Scheduler는 할당을 위해 Node의 정보를 kube-apiserver에게 요청, etcd에 기록(pod state)
            - Filtering을 통해 적합한 노드들을 선택
            - scoring을 통해 노드에 점수를 할당하여 노드 순위 지정
            - 순위가 높은 노드 선택 - 동일 시 랜덤 지정
        5. Pod 할당을 위해 kube-apiserver는 지정된 Node의 kubelet에게 Pod(container) 생성 요청
        6. kubelet은 해당 Node의 container runtime에게 container 시작 요청
        7. kubelet은 생성 상태를 Kube-apiserver에게 Bind pod to Node 알림
        8. kube-apiserver는 etcd에 기록 (pod state)
    
    ### kube-controller-manager
    
    - 무한 제어 루프를 실행하는 프로그램으로 모든 Kubernetes 컨트롤러를 관리하는 구성 요소
    - 지속적으로 실행되며 객체의 현재 상태와 원하는 상태를 감시
    - Kubernetes의 대부분의 resource object는 controller에 의해 관리
    - **주요 controller**
        - Deployment Controller
        - ReplicaSet Controller
        - DaemonSet Controller
        - Job / CronJob Controller
        - endpoints Controller
        - namespace Controller
        - Node Controller
        - Service Controller
        - Route Controller
        - User define Controller

## Kubernetes CNI - calico

### CNI - Container Network Interface

- CNI는 컨테이너 간의 네트워킹을 제어할 수 있는 Plugin 기반 Network architecture
- 다양한 형태의 container runtime과 orchestrator 사이의 네트워크 계층을 구현하는 공통된 인터페이스를 제공, k8s는 Pod간의 통신을 위해서 CNI를 사용
- k8s는 기본적으로 ‘kubenet’이라는 자체적인 CNI Plugin을 제공하지만 네트워크 기능이 매우 제한적임. 그 단점을 보완하기 위해 3rd-party plugin제공
- **필요성** - 여러 개의 node로 구성된 Kubernetes cluster 환경에서는
    - 각 Node에 존재하는 container network IP 대역이 동일하여 pod들이 같은 IP를 할당 받을 가능성이 높음
    - Pod의 IP가 다르게 할당되었다 하더라도 해당 Pod가 어느 Node에 존재하는지 확인 불가
        - 자신의 Node에 있는 Pod IP만 식별 가능하므로
    - 중복되지 않을 IP를 부여해줄 CNI Plugin 필요
    - CNI Plugin은 모든 worker node에게 중복되지 않는 subnet(CIDR) 부여, worker node에 할당된 Pod는 해당 subnet에 포함된 IP를 제공 받음
- CNI는 Pod 생성~삭제 시 마다 호출되는 API의 규격과 인터페이스를 정의

### CNI 구성

- CNI가 Bridge Interface를 만들고, 컨테이너 네트워크 대역대를 나누어 routing table 생성
    - container 서비스들은 기본적으로 2계층 서비스인 bridge network를 사용
- Pod들은 CNI에 의해 제공되는 고유의 IP 보유
- 클러스터 내의 모든 Pod들은 내부 네트어크가 자동으로 구성되어 Service 없이도 Pod간 통신 가능
- CNI Provider는 VXLAN(Vitrual Extensible Lan), IP-in-IP 과 같은 캡슐화된 네트워크 모델 또는, BGP(Border Gateway Protocol)와 같은 캡슐화 되지 않은 네트워크 모델을 사용하여 네트워크 모델 구현

### calico

- calico는 Kubernetes CNI 인터페이스를 준수한 Kubernetes 네트워크 모델로서 Pod, Node, 외부 네트워크 등 Kubernetes 리소스의 네트워크 통신을 담당
- kubernetes 네트워크 통신을 위한 calico 구성요소
    - Calico API server
    - Felix
    - BIRD
    - CNI plugin
    - IPAM plugin
    - calicoctl
- calico는 vRouter(가상 라우터)를 구현하고 BGP를 이용, 라우팅 정보를 공유하여 pod, service 네트워크 수행, 외부 pod 통신은 overlay또는 Direct로 통신
- calico에 대한 라우팅 정보 등을 ectd에 저장
- calico는 각 node에게 중복되지 않는 subnet(CIDR) 부여, node에 할당된 Pod는 해당 subnet에 포함된 IP를 제공받음, 이를 관리하기 위해 IPAM 구성요소 사용
    - calico는 IPAM을 이용하여 IP Block 관리
    - IPAM  - IP Address Management
        
        [What is IPAM? It's crucial for managing IP addresses – BlueCat Networks](https://bluecatnetworks.com/glossary/what-is-ipam/)
        
    - Calico IPAM 확인
        
        ```bash
        # calicoctl 설치
        kubectl apply -f https://docs.projectcalico.org/archive/v3.17/manifests/calicoctl.yaml
        kubectl get pod -n kube-system calicoctl -o wide
        NAME        READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
        calicoctl   1/1     Running   0          22s   192.168.56.102   k8s-node2   <none>           <none>
        
        # calico가 동작 중인 노드 확인
        kubectl exec -n kube-system calicoctl -- calicoctl get nodes
        NAME
        k8s-master
        k8s-node1
        k8s-node2
        k8s-node3
        
        # IPAM 확인
        kubectl exec -n kube-system calicoctl -- calicoctl ipam show
        +----------+--------------+------------+------------+-------------------+
        | GROUPING |     CIDR     | IPS TOTAL  | IPS IN USE |     IPS FREE      |
        +----------+--------------+------------+------------+-------------------+
        | IP Pool  | 10.96.0.0/12 | 1.0486e+06 | 12 (0%)    | 1.0486e+06 (100%) |
        +----------+--------------+------------+------------+-------------------+
        
        # IPAM IP block 확인
        kubectl exec -n kube-system calicoctl -- calicoctl ipam show --show-blocks
        +----------+------------------+------------+------------+-------------------+
        | GROUPING |       CIDR       | IPS TOTAL  | IPS IN USE |     IPS FREE      |
        +----------+------------------+------------+------------+-------------------+
        | IP Pool  | 10.96.0.0/12     | 1.0486e+06 | 12 (0%)    | 1.0486e+06 (100%) |
        | Block    | 10.108.82.192/26 |         64 | 4 (6%)     | 60 (94%)          |
        | Block    | 10.109.131.0/26  |         64 | 2 (3%)     | 62 (97%)          |
        | Block    | 10.111.156.64/26 |         64 | 4 (6%)     | 60 (94%)          |
        | Block    | 10.111.218.64/26 |         64 | 2 (3%)     | 62 (97%)          |
        +----------+------------------+------------+------------+-------------------+
        ```
        
- calico는 BIRD를 통해 각 Node의 라우팅 정보를 공유
    - Bird 확인
        
        ```bash
        ip -c route | grep bird
        blackhole 10.108.82.192/26 proto bird
        10.109.131.0/26 via 192.168.56.102 dev tunl0 proto bird onlink
        10.111.156.64/26 via 192.168.56.101 dev tunl0 proto bird onlink
        10.111.218.64/26 via 192.168.56.103 dev tunl0 proto bird onlink
        ```
        
        - 터널링 서비스로 구성되어 있음을 확인
- Pod 생성 시 Bind된 Node에 calieXXXXXXX 인터페이스가 자동으로 생성되어 연결 됨
- calico 확인
    
    ```bash
    kubectl -n kube-system get daemonset
    NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
    calico-node   4         4         4       4            4           kubernetes.io/os=linux   3d15h
    ```
    
    - calico는 각 노드마다 반드시 하나씩 실행되는 daemonset으로 실행된다.
    
    ```bash
    kubectl get pod -o wide -n kube-system | grep calico
    calico-kube-controllers-5fc7d6cf67-9jl2n   0/1     CrashLoopBackOff   8 (105s ago)   3d15h   10.108.82.203    k8s-master   <none>           <none>
    calico-node-5ff5s                          1/1     Running            4 (23m ago)    3d15h   192.168.56.103   k8s-node3    <none>           <none>
    calico-node-66s7n                          1/1     Running            3 (24m ago)    3d15h   192.168.56.102   k8s-node2    <none>           <none>
    calico-node-96mv2                          1/1     Running            3 (25m ago)    3d15h   192.168.56.100   k8s-master   <none>           <none>
    calico-node-t2gk9                          1/1     Running            3 (25m ago)    3d15h   192.168.56.101   k8s-node1    <none>           <none>
    ```
    
    - calico-kube-controller는 클러스터 IP를 할당 받았고 다른 pod들은 노드 ip를 받아서 사용

## 일반적인 애플리케이션 Pod 배포 과정

### 예시

1. 개발 팀에 Application 배포 요구가 전달 됨 → Node JS 테스트를 위한 source code 전달
    
    ```jsx
    var http = require('http')
    http.createServer(function (req, res) {
    	res.writeHead(200, {'Content-Type': 'text/plain'});
    	res.end("Welcome to Kubernetes~! by fastcampus." + "\n");
    }).listen(8000);
    ```
    
2. 요구 사항을 반영하여 Dockerfile 생성 → docker build를 통해 Image 생성
    
    ```docker
    FROM node:21-slim
    EXPOSE 8000
    COPY runapp.js .
    CMD node runapp.js
    ```
    
    ```bash
    docker build -t sighingowl/mynode:1.0 .
    
    ```
    
3. docker run ~ 을 통해 컨테이너화 확인
    
    ```bash
    docker run -it -d --name=runapp-nodejs -p 8000:8000 node:21-slim
    docker cp runapp.js runapp-nodejs:/runapp.js
    docker exec -it runapp-nodejs node -v   # 노드 정상 동작 확인
    docker exec -it runapp-nodejs node runapp.js
    
    # 다른 터미널에서
    curl 172.17.0.2:8000
    Welcome to Kubernetes~! by fastcampus.
    
    # 빌드한 이미지 테스트
    docker run -it -d --name=runapp-nodejs -p 8000:8000 sighingowl/mynode:1.0
    curl localhost:8000
    Welcome to Kubernetes~! by fastcampus.
    
    # 테스트 후 컨테이너 종료 및 삭제
    ```
    
4. Public or Private registry에 Image update
    
    ```bash
    docker login
    docker push sighingowl/mynode:1.0
    ```
    
5. 해당 Image를 이용한 Pod 및 Service를 생성하여 내부, 외부 연결 테스트 수행
    - mynode.yaml → 선언형
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: mynode-pod
          labels:
            app: hi-nodejs
        spec:
          containers:
          - name: nodejs-container
            image: sighingowl/mynode:1.0
            ports:
            - containerPort: 8000
        ```
        
    - Pod 생성 및 실행 확인
        
        ```bash
        kubectl apply -f mynode.yaml
        kubectl get pod -o wide | grep mynode
        mynode-pod                          1/1     Running            0                 37s   10.109.131.6     k8s-node2    <none>           <none>
        
        curl 10.109.131.6:8000  # 클러스터 IP이므로 외부 연결 불가능
        Welcome to Kubernetes~! by fastcampus.
        ```
        
    - Pod가 외부 노출이 되기 위한 서비스 생성
        - mynode-svc.yaml
            
            ```yaml
            apiVersion: v1
            kind: Service
            metadata:
              name: mynode-svc
            spec:
              selector:
                app: hi-nodejs
              ports:
              - port: 10001
                targetPort: 8000
              externalIPs:
              - 192.168.56.102
            ```
            
        - 서비스 생성 및 확인
            
            ```bash
            kubectl apply -f mynode-svc.yaml
            kubectl get pod,svc -o wide | grep mynode
            pod/mynode-pod                          1/1     Running            0                15m   10.109.131.6     k8s-node2    <none>           <none>
            service/mynode-svc                 ClusterIP   10.102.232.95   192.168.56.102   10001/TCP   70s     app=hi-nodejs
            ```
            
        - 클러스터 외부에서 연결 테스트
            
            ```bash
            curl 192.168.56.102:10001
            Welcome to Kubernetes~! by fastcampus.
            ```
            
6. 성공적으로 테스트가 끝났다면 자동 배포 환경(CI/CD)이 포함된 Products 환경에 배포

## 참고 자료
- fastcampus 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online. - Kubernetes architecture 이해