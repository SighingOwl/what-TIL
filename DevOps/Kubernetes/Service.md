# Service

## Service object가 필요한 이유
### Kubernetes Service

- Service는 네트워크 추상화 object로, 생성된 Pod에 동적 접근이 가능하고, 이를
통해 애플리케이션을 클러스터 내의 네트워크 서비스로 노출할 수 있다.
- Service는 IP 주소 또는 DNS 이름을 통해 특정 포트에 직접 액세스할 수 있도록
Pod를 논리적으로 그룹화 할 수 있다.
    - Kubernetes 는 Pod 에게 고유한 IP 주소를 할당하고, Service를 생성하여 Pod 집합에 대한
    단일 DNS 명을 부여하여 부하 분산을 수행할 수 있다.
        - plugin : calico를 사용
            - IP 주소를 슬라이스, 관리
            - Endpoint controller에 의해서 관리
        - 서비스를 생성하면 DNS에 등록이 되고 DNS를 활용해서 부하분산이 가능
        - Service는 연결된 Pod의 IP를 엔드포인트로 가지고 있고 이것을 활용해서 무작위로 트래픽을 전달하는 부하분산을 수행
            - Endpoint는 service가 패킷을 전달하고자 하는 pod들의 집합
- Pod는 가상 Ethernet device가 있지만 serivce는 없다.

### Service가 필요한 이유

- Pod에 문제가 생기면 Kubernetes는 그 Pod를 삭제 후 재생성 한다.
    - kubelet에 의해 Restart
    - 기존 Pod IP가 같을 확률은 매우 낮다.
- Kubernetes 설계 사상은 Pod는 언제든 장애에 의해서 Down 될 수 있다.
    - 신규 Pod는 새로운 IP가 부여되기 때문에 애플리케이션을 매번 수정을 해야 하는 불편함을
    없애기 위해 Service를 연결하여 사용하게 설계 되었다.
- 클러스터의 외부에서 내부의 Pod 애플리케이션 접근을 위한 단일 진입점 역할
- Proxy(Load Balancer)의 역할처럼 연결된 Pod 들에 트래픽을 전달하는 object가 필요
- Service object 는 연결된 각각의 Pod로 CIient 요청 트래픽을 포워딩해주는 Proxy와 같다.

### Service가 연결되는 방법

- Service는 가상IP와 port으로 생성
- kube-proxy는 이 Service의 가상IP를 구현하고 port와 함께 관리하는 역할
    - IP-table과 함께 작업
- Service object는 Pod를 트래픽에 노출, Endpoint object에는 Pod IP 주소와 port 포함
- Service를 사용하여 Pod를 노출하면 kube-proxy는 Service object와 그룹화된 Pod로
트래픽을 보내는 네트워크 규칙(RuIes) 생성
- kube-proxy는 Service 변경사항 및 Endpoint 모니터링 후 설정된 Mode를 사용하여
Service에 연결된 Pod로 트래픽을 Routing 하기 위한 규칙(Rule)을 생성, 업데이트
    - Kube-proxy는 API server를 모니터링
    - lptables mode (기본): kube-proxy가 iptables를 관리하는 역할만 수행하고, 클라이언트에서 직접 트래픽을 받지않고.클라이언트의 모든 요청은 iptables를 거쳐서 Pod로 직접 전달
        - 애플리케이션 pod가 수시로 변경이 될 때마다 IP table을 업데이트 해야하는 단점이 있다.
    - IPVS mode: IPVS(IP VirtuaI Server)는 Netfilter Framework 기반으로 구현된 Linux KerneI LeveI에서 동작하는 Layer4 Load Balancing 도구(모듈)
        - 현업에서 많이 사용
        - RR(round-robin)
        - LC(Ieast connection)
        - DH(destination hashing)
        - SH(source hashing)
        - SED(shortest expected delay)

### Service type

- ClusterlP : Service를 클러스터 내부 가상 IP를 구현하여 노출
- NodePort : 고정 포트 (30000~32767)로 각 Node의 IP에 Service를 외부에 노출
- LoadBalancer : 클라우드 공급자(CSP)의 로드밸런서를 사용하여 Service를 외부에 노출
- ExternalName : 일반적인 selector 연결이 아닌 외부 서비스에 대한 DNS name을 제공해
내부 파드가 외부의 특정 도메인에 접근하게 하기 위한 리소스
- lngress : ngress를 사용하여 Service를 외부에 노출, lngress는 서비스 유형은 아니지만, 클러스터의 진입점 역할 수행

## Service type - clusterIP
### ClusterIP

- CIusterIP 서비스는 외부에 노출될 필요가 없는 애플리케이션의 내부 통신을 위한 타입
    - case1 - wordpress to mysql
    - case2 - frontend to backend, DB
    - case3 - redis DB의 master to slave
- backend 서비스에서 DB 서비스로 데이터 요청 처리를 하는 내부 연결
- 애플리케이션인 경우 ClusterlP 서비스를 사용하여 연결
    - app: backend Label이 붙은 Pod 를 선택하기 위한 selector 사용
    - Service는 클라이언트가 Service에 접근하기 위해 포트 80을 노출
    - backend 애플리케이션이 실행되는 Pod의 포트 8080으로 트래픽을 전달
    - type을 CIusterIP로 지정, 미지정시 기본 값

### clusterIP type service 생성

- clusterIP type service manifest
    
    ```bash
    kubectl expose pod mynode-pod --name=mynode-svc --type=ClusterIP --port=9001 --target-port=8000 --dry-run=client -o yaml > mynode-svc.yaml
    ```
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: mynode
      name: mynode-svc
    spec:
      ports:
      - port: 9001
        protocol: TCP
        targetPort: 8000
      selector:
        run: mynode
      type: ClusterIP
    ```
    
- 생성된 clusterIP service 확인
    
    ```bash
    kubectl get po,svc -o wide --show-labels | grep mynode
    pod/mynode-pod   1/1     Running   0          3m34s   10.109.131.24   k8s-node2   <none>           <none>            run=mynode
    service/mynode-svc   ClusterIP   10.105.197.55   <none>        9001/TCP   8s      run=mynode   run=mynode
    ```
    
- 서비스를 ClusterIP로 생성한 경우 클러스터 내부에서는 접속이 가능하지만 외부에서는 접속 불가하다.

## Service type - NodePort
### NodePort

- NodePort는 애플리케이션에 대한 외부 연결을 활성화하여 CIusterIP 서비스의 기능을 확장
- NodePort 서비스를 생성하면 클러스터의 모든 Node에 특정한 포트(30000~32767)를 열어
외부에서 접근, 생성된 Service의 ClusterlP로 트래픽 Routing
- kube-proxy가 이를 forwarding 하도록 netfilter의 chain rule 수정
- 웹 애플리케이션이나 API와 같이 클러스터 외부에서 액세스해야 하는 애플리케이션에 적합
    - 외부에서 <NodelP>:<NodePort>로도 접근 가능
    - CIuster 내부에서 <내부IP>:<포트> 접속 가능
- NodePort는 클러스터의 모든 Node에 특정한 포트(30000~32767)를 open
    
    - 기본값은 랜덤으로 포트번호 부여
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nodeport-svc
    spec:
      type: NodePort
      selector:
        app: backend
       ports:
       - name: web
         port: 9000
         targetPort: 8000
         protocol: TCP
    ```
    
    nodePort:port# 지정
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nodeport-svc
    spec:
      type: NodePort
      selector:
        app: backend
       ports:
       - name: web
         port: 9000
         targetPort: 8000
         protocol: TCP
         nodePort: 30001
    ```
    
    1. NodePort가 가장 먼저 개방
    2. Service port가 두번째로 개방
    3. Pod port가 마지막으로 개방

### NodePort 생성

- NodePort type service manifest
    
    ```bash
    kubectl expose pod mynode-pod2 --name=mynode-svc2 --type=NodePort --port=9002 --target-port=8000 --dry-run=client -o yaml > mynode-svc2.yaml
    ```
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: mynode-pod2
      name: mynode-svc2
    spec:
      ports:
      - port: 9002
        protocol: TCP
        targetPort: 8000
      selector:
        run: mynode-pod2
      type: NodePort
    ```
    
- 생성된 파드와 서비스 확인
    
    ```bash
    kubectl get po,svc -o wide --show-labels | grep mynode
    pod/mynode-pod2   1/1     Running   0          2m31s   10.111.218.82   k8s-node3   <none>           <none>            run=mynode-pod2
    service/mynode-svc2   NodePort    10.111.154.36   <none>        9002:30246/TCP   20s     run=mynode-pod2   run=mynode-pod2
    ```
    
- NodePort로 생성한 서비스는 nodeport의 Port를 사용해서 클러스터 내외부에서 접속이 가능

### Iptable 확인

- iptables로 nodeport 확인
    
    ```bash
    sudo iptables -t nat --list KUBE-NODEPORTS -n | column -t | grep mynode
    KUBE-EXT-ZH6IDACFKHNMCG77  tcp             --   0.0.0.0/0    0.0.0.0/0    /*  default/mynode-svc2        */  tcp  dpt:30246
    ```
    
- iptables로 mynode-svc2의 Chain을 사용해서 확인
    
    ```bash
    sudo iptables -t nat --list KUBE-EXT-ZH6IDACFKHNMCG77 -n | column -t | grep mynode
    **KUBE-MARK-MASQ**             all                        --   0.0.0.0/0    0.0.0.0/0    /*  masquerade  traffic  for  default/mynode-svc2  external  destinations  */
    ```
    
    - KUBE-MARK-MASQ는 Netfilter Mark로 패킷의 Source IP 주소를 Node IP 주소로 변경하는 역할 수행
- iptables로 서비스 확인
    
    ```bash
    sudo iptables -t nat --list KUBE-SERVICES -n | column -t | grep mynode
    KUBE-SVC-ZH6IDACFKHNMCG77  tcp            --   0.0.0.0/0    10.111.154.36   /*  default/mynode-svc2                             cluster  IP          */     tcp   dpt:9002
    KUBE-SVC-VENBE3DPT3RRD5AR  tcp            --   0.0.0.0/0    10.105.197.55   /*  default/mynode-svc                              cluster  IP          */     tcp   dpt:9001
    ```
    
- iptables로 mynode-svc2 chain을 사용해서 검색해서 서비스가 어떤 pod로 연결되는지 확인
    
    ```bash
    sudo iptables -t nat --list KUBE-SVC-ZH6IDACFKHNMCG77 -n | column -t | grep mynode
    KUBE-MARK-MASQ             tcp                        --   !10.96.0.0/12  10.111.154.36  /*  default/mynode-svc2  cluster  IP                  */  tcp  dpt:9002
    KUBE-SEP-LI57BC53M7L76IV6  all                        --   0.0.0.0/0      0.0.0.0/0      /*  default/mynode-svc2  ->       10.111.218.82:8000  */
    ```
    

### NodePort 트래픽 분산 기능

- pod, svc 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myweb1
      labels:
        app: myweb
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node1
      containers:
      - name: container
        image: dbgurum/k8s-lab:v1.0
        ports:
        - containerPort: 8080
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: myweb2
      labels:
        app: myweb
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node2
      containers:
        - name: container
          image: dbgurum/k8s-lab:v1.0
          ports:
          - containerPort: 8080
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: myweb-svc
    spec:
      selector:
        app: myweb
      ports:
      - port: 9000
        targetPort: 8080
        nodePort: 30090
      type: NodePort
    ```
    
- 생성한 파드, 서비스 확인
    
    ```bash
    kubectl get po,svc -o wide | grep myweb
    pod/myweb1   1/1     Running   0          7m37s   10.111.156.83   k8s-node1   <none>           <none>
    pod/myweb2   1/1     Running   0          24s     10.109.131.25   k8s-node2   <none>           <none>
    service/myweb-svc    NodePort    10.101.74.14   <none>        9000:30090/TCP   24s     app=myweb
    ```
    
- 부하 분산 테스트
    
    ```bash
    curl 192.168.56.101:30090/hostname
    Hostname : myweb2
    curl 192.168.56.101:30090/hostname
    Hostname : myweb2
    curl 192.168.56.101:30090/hostname
    Hostname : myweb2
    curl 192.168.56.101:30090/hostname
    Hostname : myweb2
    curl 192.168.56.101:30090/hostname
    Hostname : myweb2
    curl 192.168.56.101:30090/hostname
    Hostname : myweb1
    curl 192.168.56.101:30090/hostname
    Hostname : myweb1
    curl 192.168.56.101:30090/hostname
    Hostname : myweb2
    ```
    
    - myweb1, myweb2를 랜덤으로 트래픽이 전달되는 것을 확인

### 부하 분산 정책 설정

- Policy type
    - externalTrafficPolicy:{Local | Cluster}
        - Local 정책으로 변경하면 Cluster 정책처럼 무작위 트래픽 분산이 되지 않고, 처음 들어온 node의 Pod로만 지속적으로 연결 됨.
        - kube-proxy는 proxy요청을 Local endpoint로만 proxy하고 트래픽을 다른 Node로 전달 안함)
    - internalTrafficPolicy:{Local | Cluster}
        - Service에 그룹화된 Pod 중 동일 Node에 존재하는 Pod간의 트래픽 정책
- 동작 중인 서비스에 정책 추가
    
    ```bash
    kubectl patch svc myweb-svc -p '{"spec":{"externalTrafficPolicy":"Local"}}'
    ```
    
    - patch를 사용해도 되고 kubectl edit를 사용해서 Manifest를 수정해도 된다.
- externalTrafficPolicy:Local로 수정하면 처음 트래픽이 전달된 노드로만 계속 전달된다.

## Service type - LoadBalancer

### LoadBalancer

- 클라우드 서비스 공급자(CSP - AWS, GCE)에서 제공하는 외부 Load Balancer 를 이용해
클러스터 외부에서 접근 가능하도록 노출하는 서비스로 Node 앞에 위치해 각 Node들로
트래픽을 분산하는 역할
- LoadBaIancer 서비스는 NodePort 서비스 위에 적용, L4 LoadBaIancer가 생성되고,
CIusterIP 서비스 및 NodePort 서비스가 암시적으로 생성됨
- PubIic으로 사용 가능한 IP 주소 및 DNS 주소를 제공하여 Private IP 주소 및 NodePort
포트를 통해 클러스터의 Node에 트래픽을 Load Balancing 하고 전달
- LoadBaIancer 서비스는 웹 애플리케이션이나 API와 같이 높은 트래픽 양을 처리해야 하는 애플리케이션에 유용
- 클라우드 환경이 아닌 온프레미스(VM) 환경에서 LoadBaIancer를 사용할 경우 제공한 PubIic 주소를 제공할 수 있는 MetalLB(bare metal load balancer) 모듈을 설치해주어야 한다.
    - MetalLB 없이 로드밸런서 서비스를 만들면 서비스는 외부 IP를 계속 pending 중인 상태가 된다.

## VM기반의 LoadBalancer 사용을 위한 MetalLB 구성

### MetalLB

- 온프레미스 클러스터에서는 AWS, GCP, Azure와 같이 Load BaIancer를 제공하지
않으므로 부하분산 기능 및 외부 연결용 IP 주소를 제공하는 리소스가 필요
- MetalLB는 Bare MetaI 환경(가상화하지 않고 사용하는 고성능 물리 서버)에서
사용할 수 있는 Load BaIancer 기능을 제공하는 CNCF의 오픈소스 프로젝트
- MetaILB는 네트워크 로드 밸런서를 제공, "외부 IP 주소 풀"을 사용하여 온프레미스
및 VM 등의 가상 환경 클러스터에서 외부 서비스에 접근이 가능

### MetalLB 특징

- 온프레미스환경에서"type: LoadBalancer" 의 EXTERNAL-IP=<pending>을 해소
- MetaILB의 목적은 LoadBaIancer IP로 전달되는 트래픽을 클러스터 Node로 유도
- 트래픽이 Node에 도달하면 MetalLB의 역할은 끝나고, 그 다음은 클러스터의 CNI에서 처리
- 제공되는서비스IP(EXTERNAL-IP)는ping에 작동하지 않고, 서비스에 대한 동작은 제공된 IP를 통해 애플리케이션에 액세스해서 확인 가능
- MetaILB 설치
    
    [MetalLB, bare metal load-balancer for Kubernetes](https://metallb.universe.tf/installation/)
    
    - Kubernetes manifest 사용
    - Kustomize 사용
    - HeIm 사용

### MetalLB 모드

- Layer2 mode
    
    [](https://metaiib.universe.tf/concepts/Iayer2/)
    
    - 클러스터 노드 중 하나가(리더 노드) 외부에서 접속 가능한 IP의 ARP request를 네트워크 인터페이스에 할당해서 처리, 구성은 단순하지만 대용량 처리에 부하가 심하여 테스트용으로 권장
    - ARP는 주소 결정 프로토콜(Address Resolution Protocol)로 |p주소를 MAC 주소와 매칭 시기기 위한 프로토콜이다
- BGP mode
    - 운영 환경에서는 고가용성 구성이 가능한 BGP 모드 권장
    
    [MetalLB, bare metal load-balancer for Kubernetes](https://metallb.universe.tf/concepts/bgp/)
    
- 고급 AddresspooI 구성
    
    [MetalLB, bare metal load-balancer for Kubernetes](https://metallb.universe.tf/configuration/_advanced_ipaddresspool_configuration/)
    

### MetalLB L2 Mode

[MetalLB, bare metal load-balancer for Kubernetes](https://metallb.universe.tf/concepts/layer2/)

- 사용할 IP 주소의 대역 설정을 통해 IP 제공
    - IPAddressPool 대역이 사용자와 Cluster Node 모두 접근 가능한 대역으로 지정
- 설치 후 필수 사항 "EXTERNAL-IP 대역" 및 "설정모드"지정을 위한 configmap 배포
- Iayer2 모드는 Node의 NIC에 IP를 바인딩하지 않고, 로컬 네트워크의 ARP 요청에 응답하여 컴퓨터의 MAC주소를 클라이언트에 제공하는 방식으로 작동

### MetalLB 구성요소

- ControIIer(Deployment)
    - 구성된 IP pool에서 로드 밸런싱을 수행할 EXTERNAL-IP주소를 할당하는 역할
- Speaker(DaemonSet)
    - Layer2 or BGP모드를 통해 할당된 IP를 알리는 역할 수행. speaker Pod는 node IP를 사용
- 컨트롤러 및 스피커 서비스 계정
    - 서비스에는 컨트롤러 및 스피커와 구성 요소 작동에 필요한 RBAC 사용 권한이 포함

### MetalLB L2 Mode 생성

<img src="/images/kube_service_1.png" width="75%" height="75%" title="kube service 1" alt="kube service 1">     

- Type: LoadBaIancer 생성하면 MetalLB L2 모드의 ControIIer는 API server를 통해 EXTERNAL-IP를 특정 노드의 리더 Speaker pod에 할당
- EXTERNAL-IP를 리더 Speaker pod가 소유하고 있다는 것을 ARP를 이용하여 모든 노드의 Speaker pod에 전파

### MetalLB 설치

- kube-system 네임스페이스의  kube-proxy configmap 편집
    
    ```bash
    kubectl edit configmap -n kube-system kube-proxy
    ```
    
    ```yaml
    ...
    37     ipvs:
    38       excludeCIDRs: null
    39       minSyncPeriod: 0s
    40       scheduler: ""
    41       strictARP: true
    42       syncPeriod: 0s
    
    ```
    
    - ipvs의 strictARP가 기본값으로 false를 가지는데 이를 true로 변경
- MetalLB manifest를 사용해서 MetalLB를 kubernets cluster에 배포
    
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
    ```
    
- Metallb 리소스 확인
    
    ```bash
    kubectl get all -n metallb-system -o wide
    NAME                              READY   STATUS    RESTARTS   AGE     IP               NODE         NOMINATED NODE   READINESS GATES
    pod/controller-786f9df989-rp87v   1/1     Running   0          2m35s   10.111.218.83    k8s-node3    <none>           <none>
    pod/speaker-j5gqc                 1/1     Running   0          2m35s   192.168.56.102   k8s-node2    <none>           <none>
    pod/speaker-j6c6r                 1/1     Running   0          2m35s   192.168.56.101   k8s-node1    <none>           <none>
    pod/speaker-jvvtn                 1/1     Running   0          2m34s   192.168.56.103   k8s-node3    <none>           <none>
    pod/speaker-xxzrf                 1/1     Running   0          2m35s   192.168.56.100   k8s-master   <none>           <none>
    
    NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
    service/webhook-service   ClusterIP   10.100.69.130   <none>        443/TCP   2m35s   component=controller
    
    NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE     CONTAINERS   IMAGES                             SELECTOR
    daemonset.apps/speaker   4         4         4       4            4           kubernetes.io/os=linux   2m35s   speaker      quay.io/metallb/speaker:v0.13.12   app=metallb,component=speaker
    
    NAME                         READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                                SELECTOR
    deployment.apps/controller   1/1     1            1           2m35s   controller   quay.io/metallb/controller:v0.13.12   app=metallb,component=controller
    
    NAME                                    DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                                SELECTOR
    replicaset.apps/controller-786f9df989   1         1         1       2m35s   controller   quay.io/metallb/controller:v0.13.12   app=metallb,component=controller,pod-template-hash=786f9df989
    ```
    
    - Speaker Pod들은 모두 Host의 IP 주소를 사용한다.
        - Speaker Pod들은 daemonset으로 실행되어 각 노드마다 하나씩 동작 중이다.
    - Controller는 Deployment로 배포
- 로드밸런서 서비스 테스트
    
    ```bash
    kubectl get po,svc -o wide | grep myweb
    pod/myweb   1/1     Running   0          61s   10.109.131.26   k8s-node2   <none>           <none>
    service/myweb-svc    LoadBalancer   10.100.20.42   <pending>     8080:31191/TCP   57s     run=myweb
    ```
    
    - MetalLB를 설치했지만 여전히 svc의 외부 IP는 pending 상태이다.
    - 이유는 아직 IP Pool을 구성하지 않았기 때문에 접근 가능한 IP 대역을 모른다.

### 고급 IP Pool(addresspool) 구성

- IP 대역 설정 Manifest
    - 192.168.56.150/26 대역을 가지는 ip pool 구성
        
        ```yaml
        apiVersion: metallb.io/v1beta1
        kind: IPAddressPool
        metadata:
          name: ip-pool-01
          namespace: metallb-system
        spec:
          addresses:
          - 192.168.56.150/26
        ```
        
    - 192.168.56.200/27 대역을 가지는 ip pool 구성
        
        ```yaml
        apiVersion: metallb.io/v1beta1
        kind: IPAddressPool
        metadata:
          name: ip-pool-02
          namespace: metallb-system
        spec:
          addresses:
          - 192.168.56.200/27
        ```
        
    - L2 pool manifest
        
        ```yaml
        apiVersion: metallb.io/v1beta1
        kind: L2Advertisement
        metadata:
          name: network-l2-lb-01
          namespace: metallb-system
        spec:
          ipAddressPools:
          - ip-pool-01
          - ip-pool-02
        ```
        
        - Speake가 어떤 대역의 IP를 관리할 것인지 설정
- ip-pool과 L2 pool 배포 후 loadbalancer svc 확인
    
    ```bash
    kubectl get po,svc -o wide | grep myweb
    pod/myweb   1/1     Running   0          11m   10.109.131.26   k8s-node2   <none>           <none>
    service/myweb-svc    LoadBalancer   10.100.20.42   192.168.56.128   8080:31191/TCP   11m     run=myweb
    ```
    
    - LoadBalancer svc의 외부 IP가 할당 된 것을 확인

### VM 기반의 LoadBalancer 테스트

- deployment 배포
    
    ```bash
    kubectl create deployment metallb-deploy --image=traefik/whoami --replicas=3 --port=80
    ```
    
- Deployment 노출을 위한 svc 배포
    
    ```bash
    kubectl expose deployment metallb-deploy --name=metallb-deploy-svc --type=LoadBalancer --port=80 --target-port=80
    ```
    
- Pod, svc 확인
    
    ```bash
    kubectl get po,svc -o wide | grep metallb
    pod/metallb-deploy-86c69d6b5d-864mc   1/1     Running   0          41s   10.111.156.84   k8s-node1   <none>           <none>
    pod/metallb-deploy-86c69d6b5d-8d8xk   1/1     Running   0          41s   10.111.218.85   k8s-node3   <none>           <none>
    pod/metallb-deploy-86c69d6b5d-m7prv   1/1     Running   0          41s   10.109.131.27   k8s-node2   <none>           <none>
    service/metallb-deploy-svc   LoadBalancer   10.104.205.242   192.168.56.128   80:30942/TCP   7s      app=metallb-deploy
    ```
    
    - svc의 외부 IP가 Metallb가 동작함에 따라 할당 됨을 확인
- 로드밸런서 동작 확인
    
    ```bash
    curl 192.168.56.128
    Hostname: metallb-deploy-86c69d6b5d-m7prv
    IP: 127.0.0.1
    IP: ::1
    IP: 10.109.131.27
    IP: fe80::8848:f2ff:fe3e:351b
    RemoteAddr: 10.108.82.192:32522
    GET / HTTP/1.1
    Host: 192.168.56.128
    User-Agent: curl/7.81.0
    Accept: */*
    
    curl 192.168.56.128
    Hostname: metallb-deploy-86c69d6b5d-m7prv
    IP: 127.0.0.1
    IP: ::1
    IP: 10.109.131.27
    IP: fe80::8848:f2ff:fe3e:351b
    RemoteAddr: 10.108.82.192:63473
    GET / HTTP/1.1
    Host: 192.168.56.128
    User-Agent: curl/7.81.0
    Accept: */*
    
    curl 192.168.56.128
    Hostname: metallb-deploy-86c69d6b5d-8d8xk
    IP: 127.0.0.1
    IP: ::1
    IP: 10.111.218.85
    IP: fe80::c41e:86ff:fe7d:ebac
    RemoteAddr: 10.108.82.192:23905
    GET / HTTP/1.1
    Host: 192.168.56.128
    User-Agent: curl/7.81.0
    Accept: */*
    
    curl 192.168.56.128
    Hostname: metallb-deploy-86c69d6b5d-864mc
    IP: 127.0.0.1
    IP: ::1
    IP: 10.111.156.84
    IP: fe80::9c57:eaff:fe75:59d1
    RemoteAddr: 10.108.82.192:38856
    GET / HTTP/1.1
    Host: 192.168.56.128
    User-Agent: curl/7.81.0
    Accept: */*
    
    curl 192.168.56.128
    Hostname: metallb-deploy-86c69d6b5d-m7prv
    IP: 127.0.0.1
    IP: ::1
    IP: 10.109.131.27
    IP: fe80::8848:f2ff:fe3e:351b
    RemoteAddr: 10.108.82.192:32131
    GET / HTTP/1.1
    Host: 192.168.56.128
    User-Agent: curl/7.81.0
    Accept: */*
    
    curl 192.168.56.128
    Hostname: metallb-deploy-86c69d6b5d-8d8xk
    IP: 127.0.0.1
    IP: ::1
    IP: 10.111.218.85
    IP: fe80::c41e:86ff:fe7d:ebac
    RemoteAddr: 10.108.82.192:1298
    GET / HTTP/1.1
    Host: 192.168.56.128
    User-Agent: curl/7.81.0
    Accept: */*
    ```
    
    - 세번째로 나오는 IP가 트래픽을 보낼때마다 랜덤으로 다른 IP가 출력되는 것을 확인
    - 각 IP는 deployment로 배포한 pod의 IP이다.
- kubectl get events -w를 사용하면 IP를 할당 받는 과정을 확인할 수 있다.

## Amazon EKS 기반의 LoadBalancer 사용
### EKS LoadBalancer

<img src="/images/kube_service_2.png" width="75%" height="75%" title="kube service 2" alt="kube service 2">     

- 일반적으로 LoadBaIancer Service는 모든 노드 앞에 로드 밸런서를 추가하여
NodePort Service를 확장한다.
- Amazon EKS에서는 Kubernetes가 Load BaIancer를 요청하면고 모든 노드를
자동으로 등록한다.
- Load Balancer는 지정된Service의 pod가 실행되고 있는 위치를 감지하지않고,
모든 작업자 노드가 로드 밸런서에 백엔드 인스턴스로 추가된다.
- EKS는 기본적으로 Classic Load BaIancer가 LoadBaIancer 서비스에 사용된다.
- Load BaIancer는 노출된 포트를 통해 이를 노드로 라우팅한다.
- 서비스에 주석을 추가하여 Network Load BaIancer로 변경할 수 있다.
    - [service.beta.kubernetes.io/aws-load-balancer-type:](http://service.beta.kubernetes.io/aws-load-balancer-type:) external
    - service.beta.kubernetes. io/aws-load-balancer-nlb-ta rget-type: instance

## Service type - Ingress
### Ingress

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- lngress object는 클러스터 외부에 있는 HTTP 및 HTTPS 경로를 서비스에 노출하고 트래픽 규칙을 정의
- lngress object는 일반적으로 로드밸런서를 통해 수신 규칙 및 요청을 이행하는 lngress Controller를 사용(lngress 사용시 전제 조건)
    - NGINX ControIIer, Envoy ControIIer, Traefik lngress ControIIer 등
    - AWSLoad Balancer ControIIer (EKS 사용시)
- lngress object를 사용하면 사용하는 로드밸런서의 수를 줄일 수 있다.
    - lngress object 및 Controller를 사용하면 기존의 서비스당 로드밸런서 하나에서 lngress당 로드 밸런서 하나로 전환하고 여러 서비스로 라우팅 할 수 있다.
        - MSA에 사용하면 유용
    - 트래픽은 경로 기반 라우팅을 사용하여 적절한 서비스로 라우팅이 가능
- L7, http/https 서비스지원
    - https는 TLS + 인증서
    - 인증서을 kubernetes secret에 저장된 키값으로 사용
- lngress는 실제로 서비스 유형이 아닌 여러 서비스 앞에 있으며 "스마트라우터"
- 클러스터의 진입점 역할
- Kubernetes 내부 처리 과정
    - lnternet(external traffic) → lNGRESS(routingrule) → one more Services → one more web APP Pods → container
    - lngress는 단일 접점과 서비스 라우팅 역할
    - Service는 Pod 로드 밸런싱 역할
    - Pod는 비즈니스로직(App) 처리하는 역할 수행
- lngress는 클러스터 외부의 L7(HTTP 및 HTTPS)경로를 클러스터 내의 Service object에
대한 inbound traffic access(rules)를 사용하여 쉽게 연결 가능
- Ingress manifest 예시
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: minimal-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx-example
      rules:
      - http:
          paths:
          - path: /testpath
            pathType: Prefix
            backend:
              service:
                name: test
                port:
                  number: 80
    ```
    
    - 다른 Kubernetes object보다 manifest가 다소 복잡하다.

### Ingress Container 설치

[Installation Guide - Ingress-Nginx Controller](https://kubernetes.github.io/ingress-nginx/deploy/)

- VMware를 사용하는 온프레미스는 bare metal를 선택해 설치
- bare metal ingress controller 배포
    
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
    ```
    
- Ingress-nginx namespace의 리소스 확인
    
    ```bash
    kubectl get all -n ingress-nginx
    NAME                                            READY   STATUS      RESTARTS   AGE
    pod/ingress-nginx-admission-create-7lfdq        0/1     Completed   0          92s
    pod/ingress-nginx-admission-patch-v9mqz         0/1     Completed   0          92s
    pod/ingress-nginx-controller-6dc9c5fb7c-tj8xb   1/1     Running     0          92s
    
    NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
    service/ingress-nginx-controller             NodePort    10.105.43.202   <none>        80:32701/TCP,443:32216/TCP   92s
    service/ingress-nginx-controller-admission   ClusterIP   10.108.99.73    <none>        443/TCP                      92s
    
    NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/ingress-nginx-controller   1/1     1            1           92s
    
    NAME                                                  DESIRED   CURRENT   READY   AGE
    replicaset.apps/ingress-nginx-controller-6dc9c5fb7c   1         1         1       92s
    
    NAME                                       COMPLETIONS   DURATION   AGE
    job.batch/ingress-nginx-admission-create   1/1           15s        92s
    job.batch/ingress-nginx-admission-patch    1/1           16s        92s
    ```
    
    - ingress-nginx-controller svc를 확인하면 외부에서 접속을 위해 NodePort 유형의 서비스로 동작 중이다.
        - HTTP, HTTPS를 모두 지원하기 위해 각각 포트 매핑이 되어 있다.

### Ingress 구성

- ingress와 연결될 pod, svc 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: hello-app-pod
      labels:
        app: hello
    spec:
      containers:
        - name: hello-container
          image: dbgurum/ingress:hi
          args:
            - "-text=Hello, Fastcampus."
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: hello-app-svc
    spec:
      selector:
        app: hello
        ports:
          - port: 5678
    ```
    
- ingress 구성
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: http-hello
      annotations:
        ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx
      rules:
      - http:
          paths:
          - path: /hello-app
            pathType: Prefix
            backend:
              service:
                name: hello-app-svc
                port:
                  number: 5678
    ```
    
- ingress 구성 후 세부 정보 확인
    
    ```bash
    kubectl describe ingress http-hello
    Name:             http-hello
    Labels:           <none>
    Namespace:        default
    Address:          192.168.56.102
    Ingress Class:    nginx
    Default backend:  <default>
    Rules:
      Host        Path  Backends
      ----        ----  --------
      *
                  /hello-app   hello-app-svc:5678 (10.109.131.31:5678)
    Annotations:  ingress.kubernetes.io/rewrite-target: /
    Events:
      Type    Reason  Age                From                      Message
      ----    ------  ----               ----                      -------
      Normal  Sync    15s (x2 over 47s)  nginx-ingress-controller  Scheduled for sync
    ```
    
    - address가 Pod가 생성된 Node의 IP 주소와 같은 여부와 Rules table에서 Path와 backend가 지정한 대로 연결되었는지 확인
- Ingress 동작 확인
    
    ```bash
    curl 192.168.56.102:32701/hello-app
    Hello, Fastcampus.
    ```
    
    - Ingress에 제공된 IP:Ingress Controller Port/ingress를 구성할 때 설정한 경로를 사용해서 접근

### Ingress 구성에서 2개 이상의 경로 설정

- 여러 pod와 서비스를 생성하는 manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: hello-pod
      labels:
        app: hello
    spec:
      containers:
        - name: hello-container
          image: dbgurum/ingress:hi
          args:
            - "-text=Hello, Fastcampus."
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: welcome-pod
      labels:
        app: welcome
    spec:
      containers:
        - name: welcome-container
          image: dbgurum/ingress:hi
          args:
            - "-text=Welcome, Fastcampus."
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: hello-svc
    spec:
      selector:
        app: hello
      ports:
        - port: 5678
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: welcome-svc
    spec:
      selector:
        app: welcome
      ports:
        - port: 5678
    ```
    
- 규칙에 2개 이상의 경로를 가지는 Ingress Manifest
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: hello-welcome
      annotations:
        ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx
      rules:
      - http:
          paths:
          - path: /hello
            pathType: Prefix
            backend:
              service:
                name: hello-svc
                port:
                  number: 5678
          - path: /welcome
            pathType: Prefix
            backend:
              service:
                name: welcome-svc
                port:
                  number: 5678
    ```
    
- ingress 세부 정보 확인
    
    ```bash
    kubectl describe ingress hello-welcome
    Name:             hello-welcome
    Labels:           <none>
    Namespace:        default
    Address:          192.168.56.102
    Ingress Class:    nginx
    Default backend:  <default>
    Rules:
      Host        Path  Backends
      ----        ----  --------
      *
                  /hello     hello-svc:5678 (10.109.131.33:5678)
                  /welcome   welcome-svc:5678 (10.111.218.88:5678)
    Annotations:  ingress.kubernetes.io/rewrite-target: /
    Events:
      Type    Reason  Age                    From                      Message
      ----    ------  ----                   ----                      -------
      Normal  Sync    3m12s (x2 over 3m41s)  nginx-ingress-controller  Scheduled for sync
    ```
    
    - Rules를 확인하면 2개의 경로에 각각 연결된 svc를 확인 가능
    - 만일 위와 같은 Rules가 출력되지 않는다면 pod, svc manifest의 label이나 ingress manifest의 경로와 service 이름을 확인해서 수정한다.
- ingress 동작 확인
    
    ```bash
    curl 192.168.56.102:32701/hello
    Hello, Fastcampus.
    
    curl 192.168.56.103:32701/welcome
    Welcome, Fastcampus.
    ```
    

### Ingress 이름 기반 가상 호스팅

- 이름을 특정할 수 있는 Pod, Svc manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: svc-pod-1
      labels:
        app: svc
    spec:
      containers:
        - name: svc1-container
          image: dbgurum/ingress:hi
          args:
            - "-text=This is the SERIVCE page of fastcampus.k8s -1-"
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: svc-pod-2
      labels:
        app: svc
    spec:
      containers:
        - name: svc2-container
          image: dbgurum/ingress:hi
          args:
            - "-text=This is the SERIVCE page of fastcampus.k8s -2-"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: svc-page
    spec:
      selector:
        app: svc
      ports:
        - port: 5678
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: cloud-pod-1
      labels:
        app: cloud
    spec:
      containers:
        - name: cloud1-container
          image: dbgurum/ingress:hi
          args:
            - "-text=This is the CLOUD page of fastcampus.k8s -1-"
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: cloud-pod-2
      labels:
        app: cloud
    spec:
      containers:
        - name: cloud2-container
          image: dbgurum/ingress:hi
          args:
            - "-text=This is the CLOUD page of fastcampus.k8s -2-"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: cloud-page
    spec:
      selector:
        app: cloud
      ports:
        - port: 5678
    ```
    
- 이름 기반 가상 호스트 ingress manifest
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: fastcampus-http
      annotations:
        ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx
      rules:
      - host: svc.fastcampus.k8s
        http:
          paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: svc-page
                port:
                  number: 5678
    	- host: cloud.fastcampus.k8s
        http:
         paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: cloud-page
                port:
                  number: 5678
    ```
    
    - 이전 Ingress와 다른 점은 rules에 “host” 부분이 추가 되었다.
- ingress 세부 정보 확인
    
    ```bash
    kubectl describe ing fastcampus-http
    Name:             fastcampus-http
    Labels:           <none>
    Namespace:        default
    Address:          192.168.56.102
    Ingress Class:    nginx
    Default backend:  <default>
    Rules:
      Host                Path  Backends
      ----                ----  --------
      svc.fastcampus.k8s
                          /   svc-page:5678 (10.109.131.34:5678,10.111.218.89:5678)
                          /   cloud-page:5678 (10.109.131.35:5678,10.111.156.89:5678)
    Annotations:          ingress.kubernetes.io/rewrite-target: /
    Events:
      Type    Reason  Age                From                      Message
      ----    ------  ----               ----                      -------
      Normal  Sync    33s (x2 over 76s)  nginx-ingress-controller  Scheduled for sync
    ```
    
    - Rules에서 서비스 별로 연결된 2개의 파드 IP 주소를 확인할 수 있다
- ingress 동작 확인
    
    ```bash
    curl http://192.168.56.102:32701 -H "Host: svc.fastcampus.k8s"
    This is the SERIVCE page of fastcampus.k8s -2-
    curl http://192.168.56.102:32701 -H "Host: svc.fastcampus.k8s"
    This is the SERIVCE page of fastcampus.k8s -2-
    curl http://192.168.56.102:32701 -H "Host: svc.fastcampus.k8s"
    This is the SERIVCE page of fastcampus.k8s -2-
    curl http://192.168.56.102:32701 -H "Host: svc.fastcampus.k8s"
    This is the SERIVCE page of fastcampus.k8s -1-
    curl http://192.168.56.102:32701 -H "Host: svc.fastcampus.k8s"
    This is the SERIVCE page of fastcampus.k8s -1-
    ```
    
    - 이름 기반 가상 호스팅 중인 Ingress로 접속 할 때는 -H 옵션을 사용해서 헤더를 지정해야 한다.
    - 서비스에 2개의 파드가 연결되어 있어 트래픽이 랜덤으로 전달 된다.

### Ingress 생성시 버그로 인한 오류 해결

```bash
kubectl delete validatingwebhookconfiguration ingress-nginx-admission
```

- webhook error가 발생하면 위 명령을 사용한다.

## 참고 자료
Fastcampus 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online.