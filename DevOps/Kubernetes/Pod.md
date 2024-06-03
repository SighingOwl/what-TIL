# Pod란?
Pod는 쿠버네티스에서 생성 및 배포, 관리 가능한 가장 작은 컴퓨팅 단위이고 내부에 컨테이너를 1개 이상 가지고 있어 쿠버네티스는 컨테이너를 직접 관리하기 보단 파드로 컨테이너들을 묶어 관리합니다. 쿠버네티스에서 단일 파드를 생성하여 사용할 수도 있지만 이렇게 생성한 파드는 오토스케일링이나 장애 조치, 동일 워크로드 배포와 같은 작업을 자동화하기 힘든 점이 있어 보통 deployment를 사용해서 배포하게 됩니다.  파드를 배포할 때는 일반적으로 2가지 방법이 사용됩니다. 단일 컨테이너 실행 파드와 다중 컨테이너 실행 파드입니다.

| 방식 | 설명 |
| --- | --- |
| 단일 컨테이너 | 파드에 하나의 컨테이너만 사용하는 모델은 가장 일반적인 유스케이스이며 단순히 파드가 컨테이너를 감싸고 있는 형태를 이루고 있습니다. |
| 다중 컨테이너 | 이러한 형태의 파드는 밀접하게 결합되어 있고 리소스를 공유해야하는 컨테이너들로 공유된 애플리케이션을 캡슐화 할 수 있습니다. 이 때 파드 내부의 컨테이너들은 각자의 역할을 가지고 작업을 수행합니다. 예를 들어 메인으로 사용할 컨테이너가 웹 서비스를 실행한다면 init 컨테이너는 웹 서비스가 호스팅할 웹 페이지를 외부 소스에서 가져와 웹 서비스를 초기화하고 사이드카 컨테이너는 웹 서비스 로그를 수집하도록 구성하여 파드가 웹 서버 역할을 수행하도록 할 수 있습니다. |

## Pod 관리 및 lifecycle
### Kubernetes API Resource(or Object)

- 각 기능 별로 사용자가 의도한 상태의 작업을 수행하는 object
- object에 대한 기본적인 정보와 의도한 상태를 기술한 “spec” 제시
- object 생성을 위해 .yaml(yml) 파일로 kubectl에 제공
- kubectl은 API 요청 수행시 JSON 형식으로 정보 변환

```bash
kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
..
```

- kubernetes API resource 정의
    
    | Key | Description |
    | --- | --- |
    | apiVersion | Object의 Kubernetes가 지원하는 API 버전 <br> ”kubectl apt-version”으로 버전 확인 <br> - apps: 그룹 <br> - v1: 버전 core 그룹(그룹이 없는 api는 core 그룹) <br> - 개발 순서 <br> &emsp; Dev → Alpha → Beta → Stable <br> &emsp; v1alpha → v1alpha2 → v1alpha3 → v1beta1 → v1beta2 → v1 |
    | kind | 어떤 object를 생성할 것인지 종류 지정 |
    | metadata | Object 설명을 위한 메타데이터 정의 - name, namespace, label, … |
    | spec | Object 정의(선언, 리스트가 아니므로 정의하는 하위 속성의 순서는 무관) |
- kubernetes object 별 세부 설명 확인
    
    ```bash
    kubectl explain pods
    KIND:       Pod
    VERSION:    v1
    
    DESCRIPTION:
        Pod is a collection of containers that can run on a host. This resource is
        created by clients and scheduled onto hosts.
    
    FIELDS:
      apiVersion	<string>
        APIVersion defines the versioned schema of this representation of an object.
        Servers should convert recognized schemas to the latest internal value, and
        may reject unrecognized values. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
    
      kind	<string>
        Kind is a string value representing the REST resource this object
        represents. Servers may infer this from the endpoint the client submits
        requests to. Cannot be updated. In CamelCase. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
    
      metadata	<ObjectMeta>
        Standard object's metadata. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
    
      spec	<PodSpec>
        Specification of the desired behavior of the pod. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
    
      status	<PodStatus>
        Most recently observed status of the pod. This data may not be up to date.
        Populated by the system. Read-only. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
    ```
    
    - 파드에 관한 설명
        - DESCRIPTION을 확인이 중요
    - 계층적 확인
        
        ```bash
        kubectl explain pods
        kubectl explain pods.metadata
        kubectl explain pods.spec
        kubectl explain pods.spec.containers
        kubectl explain pods.spec.containers.images
        kubectl explain pods.spec --recursive
        kubectl explain pods.spec.containers --recursive
        ```
        
        - —recursive는 정확한 하위 구조를 파악할 때 사용

### Pod 관리

- **Pod**는 kubernetes object relationship이다.
    - Pod는 kubernetes의 기본적인 Application 배포 단위로, 하나 이상의 container를 포함
    - Kubernetes는 container를 개별적으로 하나씩 배포하는 것이 아닌 Pod 단위로 배포
    - Pod가 포함하는 container의 종류 - 설계 패턴
        - 실제 애플리케이션을 수행하는 runtime Container (기본)
        - 기동 시점에 처리하고 종료되는 init Container
        - 보조 역할로써 배포되는 sideCar Container
- 컨테이너가 아닌 Pod로 추상화한 이유
    - Pause container를 통해 HostOS의 namespace(Isns, Linux Kernel 기술)을 공유함으로써 Pod 내부의 컨테이너들은 Pod와 격리된 컨테이너의 장점을 유지하면서 Pod 내부에서는 자원(network, storage)을 공유하는 프로세스를 여러 개의 container로 실행할 수 있다는 장점을 갖음
        - IPC, Network, PID, File System 등이 namespace를 통해 공유
        - Pod 내부의 프로세스들은 localhost에 접근하듯 서로에 접근할 수 있게 됨
        - 이는 외부와 격리된 몇개의 docker Container를 동일한 namespace로 묶어 사용하는 기존의 아키텍처를 Pod라는 이름으로 재정의해서 사용
        - 이 구조를 Cluster 기본 구성으로 가져가기 위해서 K8s의 가장 작은 배포단위를 container가 아닌 Pod로 규정
- Pod를 사용할 때는 **ReplicaSet**을 상위에 두고 사용하는 것을 권장한다.
    - ReplicaSet은 지정된 수의 Pod를 유지하는 역할을 수행
    - 문제가 있는 Pod가 종료되었을 때 정상 Pod를 자동으로 추가한다.
    - ReplicaSet은 **Deployment**로 관리한다.
- Container 수에 따라 **Single container Pod**와 **Multi container Pod**로 구분
    - Pod는 애플리케이션 실행에 필요한 모든 종속성을 제공하는 단일 컨테이너를 호스팅
        - 단일 컨테이너 Pod는 생성이 간단하고 Kubernetes가 개별 컨테이너를 간접적으로 제어할 수 있는 방법을 제공
    - 다중 컨테이너 Pod는 서로 의존하고 동일한 리소스를 공유하는 컨테이너를 호스팅
        - Pod 내에서 컨테이너는 간단한 네트워크 연결을 설정하고 동일한 스토리지 볼륨에 액세스 가능
        - 모두 동일한 Pod에 있기 때문에 Kubernetes는 이를 단일 단위로 취급하고 관리를 단순화

### Pause Container

- Pod는 실행될 때 pause라는 이름의 Container가 먼저 실행
- Pause container의 namespace를 Pod 내부의 모든 container들이 공유해서 사용
    - container를 기술적으로 구현하기 위해 사용하는 kernel 기술인 cgroups, namespace에서의 namespace를 의미
    - cgroups는 호스트의 자원을 제한해서 각 container에 할당하는 역할을 수행
    - namespace는 각 container가 container 외부와 격리된 IPC, Network, PID, File System을 가질 수 있게 독립적인 공간을 논리적으로 형성하는 역할 수행
- SIGINT, SIGTERM 시그널을 받기 전까지 동작도 하지 않고 Pod 내에서 sleep 상태로 대기

### Pod 생성, 조회, 삭제를 위한 Kubectl 사용

- imperative syntax(명령형 문법): kubectl {create | run}을 통해 CLI로 object 생성
    
    ```bash
    kubectl create deployment deploy-name --image=image_name ~
    kubectl run pod-name --image=image_name ~
    kubectl expose   # pod와 deployment를 연결할 때 사용하는 명령
    ```
    
    - 사용하기 편리하며 help를 사용해서 문법적인 구조를 파악하기 쉽다.
        - yaml코드를 생성하는 옵션을 사용하면 전체 yaml 코드를 직접 작성하지 않아도 된다.
    - 이 방식을 선호하면 yaml 코드에 문제가 있을 때 이를 해결하는 능력이 떨어질 수 있다.
- declarative syntax(선언형 문법): YAML 코드를 작성하여 kubectl apply 하여 생성
    
    ```bash
    kubectl {create | apply} -f {mynode.yaml | URL/mynode.yaml}
    kubectl apply -f resources/ (YAML 파일의 모음(directory)을 한번에 실행)
    kubectl get pod [ -o {wide | yaml | json} ]
    kubectl describe pod pod-name
    kubectl delete pod -f mynode.yaml
    kubectl delete pod -f mynode.yaml -f mynode-sve.yaml
    kubectl delete {pod pod-name | pod/pod-name}
    kubectl edit pod pod-name
    kubectl replace -f mynode.yaml
    kubectl patch -f mynode.yaml
    ```
    

### Pod 삭제

- Pod 삭제 요청이 API 서버에 수신되면 우선 etcd 상태를 수정, 요청을 수행하는 kubelet과 Endpoints controller에 알림, 이 이벤트는 API 서버에 의해 병렬로 처리됨
    
    ![https://freecontent.manning.com/handling-client-requests-properly-with-kubernetes/](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c72d76d-4e1b-4401-8601-6e8f57d474c3/876184df-34fc-4cbc-8ae1-14d8a0bbd2ae/Untitled.png)
    
    https://freecontent.manning.com/handling-client-requests-properly-with-kubernetes/
    
- Pod 삭제 시 API Server는 Node의 kubelet에게 지시를 내려 Pod에 SIGTERM 신호를 보냄
- SIGTERM을 받으면 처리 중이던 작업을 완료하고, 새로운 요청을 받지 않도록 개발 됨
- kubelet에서 pod에 SIGTERM을 보낸 후에 일정시간동안 graceful shutdown이 되지 않는다면 강제로 SIGKILL을 보내서 pod를 강제 종료 시킴
    - 이 대기 기간은 terminationGracePeriodSeconds으로 설정 - 기본값 30초
    - 강제 삭제
        
        ```bash
        kubectl delete pod pod-name --grace-period=0 --force
        ```
        
    - 강제 삭제 전 대기 시간 확인
        
        ```bash
        kubectl get po pod-name -o yaml | grep -i terminationgrace
        ```
        
        - terminationgrace를 확인하면 강제 삭제 전 대기 시간 확인 가능
    - terminationgrace = 0으로 하면 대기 시간 없이 바로 삭제 가능
        - yaml 코드에서 컨테이너의 spec에서 terminationGracePeriodSeconds를 0으로 설정

### Pod Network

- Kubernetes는 플랫 네트워크 구조를 사용하여 호스트 간의 포트 매핑 프로세스를 제거하여 유지 관리 및 비용을 감소시킴
- **Container to Container Network**
    - 여러 컨테이너가 동일한 Pod에 있는 경우 컨테이너는 Network namespace를 공유
    - Pause container에 의해 공유되 [localhost](http://localhost) 및 port 통신이 가능
    - 단, 동일한 Pod의 각 컨테이너는 포트 충돌을 방지하기 위해 고유한 통신 포트가 있어야 함
- **Pod to Pod Network**
    - 모든 Pod는 IP 주소(eth0)와 network namespace가 있고, Pod to Pod를 연결할 수 있는 가상 이더넷(calico 제공, tunl0) 연결이 있어서 Pod(eth0) to Pod(eth0) 연결이 가능
- **Node to Node Network**
    - Cluster는 대부분의 경우 Node에 할당된 IP주소가 포함된 Routing Table을 저장
    - Routing Table은 bridge가 요청을 해결할 수 없을 때 연결된 Node에 대해 IP주소를 확인
    - 일치하는 network 콘텐츠를 적절한 Node에 연결되고 연결은 Node 수준에서 지속

### 선언형 Pod 생성

- Manifest 작성
    - myweb1-pod.yaml
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: myweb1
          labels:
            app: myweb1
        spec:
          containers:
          - name: nginx-container
            image: nginx:1.25.3-alpine
            ports:
            - containerPort: 80
        ```
        
    - myweb1-svc.yaml
        
        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: myweb-svc
        spec:
          selector:
            app: myweb1
          ports:
            - port: 8001
              targetPort: 80
          externalIPs:
            - 192.168.56.102
        ```
        
- pod 생성
    
    ```bash
    kubectl apply -f myweb1-pod.yaml
    kubectl get pod -o wide | grep myweb1
    myweb1                              1/1     Running            0                     47s     10.109.131.7     k8s-node2    <none>           <none>
    ```
    
- service 생성
    
    ```bash
    kubectl apply -f myweb1-svc.yaml
    kubectl get svc -o wide | grep myweb1
    myweb-svc                  ClusterIP   10.108.252.114   192.168.56.103   8001/TCP    10s     app=myweb1
    ```
    
- Pod, Service 삭제
    
    ```bash
    kubectl delete -f myweb1-pod.yaml
    kubectl delete -f myweb1-svc.yaml
    ```
    

### 명령형 Pod 생성

- kubectl run 명령으로 pod의 yaml 파일 생성
    
    ```bash
    kubectl run myweb2 --image=nginx:1.25.3-alpine --port=80 --dry-run=client -o yaml >> myweb2-pod.yaml
    ```
    
    - —dry-run을 사용하면 실제로 실행되고 결과 값을 알려준다.
- 생성된 myweb2-pod.yaml 확인
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: myweb2
      name: myweb2
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: myweb2
        ports:
        - containerPort: 80
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    ```
    
- —dry-run을 사용하지 않고 바로 Pod를 실행해도 된다.

### 문제가 발생한 Pod log 확인

- mysql-pod.yaml 작성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mysql57-pod
      labels:
        type: mysql57
    spec:
      containers:
      - name: mysql57-container
        image: mysql:5.7
        ports:
        - containerPort: 3306
    ```
    
- 생성한 Pod 확인
    
    ```bash
    kubectl get po -o wide | grep mysql
    mysql57-pod                         0/1     Error              0                 2m5s    10.109.131.8     k8s-node2    <none>           <none>
    ```
    
    - 어떠한 이유 때문에 Pod에 오류가 발생했다.
- log 확인
    - kubectl logs를 사용해서 로그를 확인할 수 있다.
        
        ```bash
        kubectl logs mysql57-pod
        2024-02-11 13:07:38+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
        2024-02-11 13:07:39+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
        2024-02-11 13:07:39+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
        2024-02-11 13:07:39+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
            You need to specify one of the following as an environment variable:
            - MYSQL_ROOT_PASSWORD
            - MYSQL_ALLOW_EMPTY_PASSWORD
            - MYSQL_RANDOM_ROOT_PASSWORD
        ```
        
        - mysql 컨테이너는 반드시 MYSQL_ROOT_PASSWORD가 지정되어야 하는데 그렇지 않아 오류 발생
- Manifest 수정
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mysql57-pod
      labels:
        type: mysql57
    spec:
      containers:
      - name: mysql57-container
        image: mysql:5.7
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "k8spass#"
    ```
    
    - 환경 변수로 MYSQL_ROOT_PASSWORD 추가
- MySQL → MySQL Workbench같은 외부 Client도구 연결 - service 필요
    - service manifest 작성
        
        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: mysql-svc
        spec:
          selector:
            type: mysql57
          ports:
          - port: 13306
            targetPort: 3306
          externalIPs:
          - 192.168.56.102
        ```
        
    - Mysql workbench에서 Service의 정보를 토대로 새로운 커넥션을 생성하면 외부 연결로 mysql 컨테이너에 접속 가능

### Pod lifecycle

- Pod는 정의된 lifecycle에 따라 구동
- Pod가 실행되는 동안 오류가 있다면 kubelet은 오류 처리를 위해 restart 수행
    - pod 내에서 K8s는 다양한 컨테이너 상태를 추적, Pod를 다시 정상 상태로 만들기 위해 취할 조치를 결정
- 실행 중인 Pod는 명세(spec)와 실제 상태(Status)를 모두 보유
- Pod는 Pod의 수명 중 한 번만 스케줄 되고, Pod가 Node에 스케줄 되면, Pod는 중지되거나 종료될 때까지 해당 Node에서 실행
- Pod lifecycle
    
    
    | Status | Description |
    | --- | --- |
    | Pending | Pod의 작성을 기다리고 있는 상태, 컨테이너 Image 다운로드 등에 소비되는 시간 |
    | Running | Pod가 가동 중인 상태 |
    | Suceeded | Pod 내의 컨테이너가 정상적으로 종료된 상태 |
    | Failed | Pod 내의 컨테이너 중 특정 컨테이너가 실패하여 종료된 상태 |
    | Unknown | 어떤 이유로 pod와 통신이 불가능한 상태 (일반적으로 Pod 호스트와의 통신 오류에 의해서 발생) |
- Pod Conditions
    
    
    | Type | Description |
    | --- | --- |
    | Initialized | 모든 초기화 컨테이너가 성공적으로 시작 완료한 상태 |
    | Ready | Pod는 요청이 가능한 상태 |
    | ContainerReady | Pod 안 모든 컨테이너가 준비 상태 |
    | PodScheduled | Pod가 하나의 Node로 스케줄을 완료한 상태 |
    | UnScheduled | 스케줄러가 자원 부족, 여러 제약 등으로 바로 Pod 스케줄할 수 없는 상태 |
- Pod Status
    
    
    | Status | Description |
    | --- | --- |
    | type | 이 Pod 컨디션의 이름 표시 |
    | status | 해당 컨디션이 적용 가능한지 여부 표시 |
    | lastProbeTime | Pod 컨디션이 마지막으로 Probe된 시간의 timestamp 표시 |
    | lastTransitionTime | Pod가 한 상태에서 다른 상태로 전환된 마지막 시간에 대한 timestamp 표시 |
    | reason | 컨디션의 마지막 전환에 대한 이유를 나타내는 기계가 판독 가능한 카멜표기법으로 표시 |
    | message | 마지막 상태 전환에 대한 세부 정보를 읽기 가능한 메시지 표시 |

## Pod 설계 패턴 1 - runtime container

### runtime container in Pod

- Pod가 포함하는 일반적인 애플리케이션 컨테이너를 runtimer container라고 한다.
- Pod는 애플리케이션 코드가 포함된 단일 컨테이너에 대한 wrapper 역할
- Pod가 예기치 않게 종료되면 Kubernetes는 이를 수정하지 않고 그 자리에 새 Pod를 생성하고 새로 시작된 Pod는 DNS 및 IP주소를 제외한 기존 Pod의 복제본이다.
    - 이 기능은 개발자가 애플리케이션을 설계하는 방식에 큰 영향을 끼침
- Kubernetes 아키텍처의 유연한 특성으로 인해 애플리케이션은 더 이상 지정된 Pod에만 연견될 필요가 없다. 대신 클러스터 내 어디에서나 생성된 완전히 새로운 Pod가 원할하게 자리 잡을 수 있도록 애플리케이션을 설계해야 한다.

## Pod 설계 패턴 2 - initial container

### initial container in Pod

- 한 Pod에 있는 Application container 이전에 실행되는 특수 컨테이너를 initial container라고 한다.
    - application container 실행 환경 준비
- initial container는 다양한 목적으로 사용될 수 있지만 모두 기본 프로세스인 Application container가 원하는 방향으로 실행될 수 있도록 준비하는데 있다.
- 어떤 방식으로든 메인 프로세스의 환경 초기화를 수행

### initial container 동작

- initial container는 Pod에 대한 모든 네트워킹 및 스토리지가 프로비저닝된 후에 실행
- Pod의 initial container가 실패하면 kubelet은 성공할 때까지 해당 init 컨테이너를 반복적으로 재시작
- 단, Pod restartPolicy:Never가 지정되었다면, 해당 Pod를 시작하는 동안 initial container가 실패하면 Kubernetes는 전체 Pod를 실패한 것으로 처리 뒤 종료
- init container는 애플리케이션의 소스코드들 변경할 필요 없이 애플리케이션을 실행할 수 있도록 k8s에서 환경을 구성하는 수단을 제공

### initial container 사례

- MSA 애플리케이션에 원격지의 소스로부터 최신 구성 파일을 가져오는 작업을 구성하면 initial container에 의해 애플리케이션 컨테이너는 항상 최신 구성 파일을 사용할 수 있다.
- 데이터베이스를 초기화하거나 애플리케이션 컨테이너가 연결되기 전 초기(기본) 데이터를 채우는 작업을 구성하면 initial container는 스키마 생성이나 데이터 마이그레이션과 같은 데이터베이스 설정 작업을 처리하여 데이터베이스가 애플리케이션 컨테이너와 상호 작용할 준비가 되었는지 확인할 수 있다.
- 외부 Open API 사용
    
    <img src="/images/Init_container_1.png" width="50%" height="50%" title="init container" alt="init container">    
    
    1. Local volume에 공유 영역 생성
    2. init container는 외부 리소스에 curl 요청을 보내고 해당 데이터를 Local volume에 기록
    3. Application container는 시작과 함께 동일한 Local volume에서 필요한 데이터를 읽는다.

### init container를 사용해서 외부 API로 초기 데이터 생성

- 위도, 경도로 위치를 지정하여 해당 위치의 날씨를 가져오는 파드
    - weather.yaml
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: weather-pod
          namespace: defaul드
        spec:
          volumes:
            - emptyDir: {}
              name: weather-data
          initContainers:
          - name: download-config
            image: curlimages/curl:7.85.0
            args: ["https://api.open-meteo.com/v1/forecast?latitude=37.5443878&longitude=127.03744246&current_weather=true", "-o", "/usr/share/nginx/html/index.html"]
            volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: weather-data
          containers:
            - image: nginx:1.25.3-alpine
              name: nginx-container
              ports:
                - containerPort: 80
              volumeMounts:
                - mountPath: /usr/share/nginx/html
                  name: weather-data
        ```
        
- Pod 생성 확인
    
    ```bash
    kubectl get po -o wide | grep weather
    weather-pod                         0/1     PodInitializing    0                 24s   10.111.218.71    k8s-node3    <none>           <none>
    
    kubectl get po -o wide | grep weather
    weather-pod                         1/1     Running            0                 58s   10.111.218.71    k8s-node3    <none>           <none>
    ```
    
    - Pod를 처음 실행할 때 pod 초기화가 이루어지는 것을 확인할 수 있다.
- 날씨 정보 확인
    
    ```bash
    curl 10.111.218.71
    {"latitude":37.55,"longitude":127.0625,"generationtime_ms":0.06794929504394531,"utc_offset_seconds":0,"timezone":"GMT","timezone_abbreviation":"GMT","elevation":22.0,"current_weather_units":{"time":"iso8601","interval":"seconds","temperature":"°C","windspeed":"km/h","winddirection":"°","is_day":"","weathercode":"wmo code"},"current_weather":{"time":"2024-02-12T08:00","interval":900,"temperature":7.7,"windspeed":4.6,"winddirection":225,"is_day":1,"weathercode":0}}
    ```
    

### myservice라는 서비스가 생성되기를 기다리는 Init container - kubernetes docs

- init container가 완료되면 myapp-container가 시작되고 “The app is running!” 메시지를 출력, 3600초 동안 절전 모드로 전환
- 파드와 서비스간 연결성이 없고 init container만 테스트
    - yaml 파일 생성
        
        ```bash
        kubectl run myapp-init-pod --image=busybox --dry-run=client -o yaml > myapp-init-pod.yaml
        ```
        
    - myapp-init-pod.yaml
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: myapp-pod
          labels:
            app: myapp
        spec:
          containers:
          - image: busybox:1.28
            name: myapp-container
            command: ['sh', '-c', 'echo The app is running! && sleep 3600']
          initContainers:
          - name: init-myservice
            image: busybox:1.28
             command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
        ```
        
- 파드 상태 확인
    
    ```bash
    Containers:
      myapp-container:
        ..
        State:          Waiting
          Reason:       PodInitializing
        Ready:          False
    ```
    
    - myservice가 준비되지 않아 메인 컨테이너가 대기 중임을 확인
    - 
- myservice.yaml
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: myservice
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
    ```
    
- 서비스 생성 후 파드 확인
    
    ```bash
    kubectl get po,svc -o wide | grep my
    pod/myapp-pod                           1/1     Running            0                 5m3s   10.111.218.73    k8s-node3    <none>           <none>
    service/myservice                  ClusterIP   10.100.10.244   <none>           80/TCP      39s     <none>
    ```
    
    - 서비스 생성 후에 파드가 생성 완료되어 정상 동작 중
- initcontainer는 작업이 완료 종료 된다.
    
    ```bash
    kubectl describe pod myapp-pod
    ..
    Init Containers:
      ..
        State:          Terminated
          Reason:       Completed
    ```
    

## Pod 설계 패턴 3 - sidecar container

### sidecar Container in Pod

- Sidecar container는 application container와 함께 작동하여 특정 추가 기능과 애플리케이션의 관찰 가능성 향상을 위해 로깅 및 모니터링 서비스를 구현할 수 있다.
- Sidecar container를 활용하면 코드 분리를 촉진하고 관찰 가능성을 개선하며 확장성과 보안을 단순화하여 Kubernetes 배포를 향상할 수 있다.
- application container에는 원래 목적의 기능에만 충실하고 나머지 부가적인 공통 기능들은 Sidecar container를 추가해서 사용할 수 있다.
- 예시 - 일반적인 웹서버
    
    <img src="/images/Sidecar_container_1.png" width="50%" height="50%" title="sidecar container" alt="sidecar container">    
    
    - 웹서버 컨테이너는 웹서버로서 역할에 충실하고 자신의 로그는 Pod의 파일로 남길 수 있다.
    - Sidecar container 역할인 로그 수집 컨테이너가 파일 시스템에 쌓이는 로그를 수집해서 외부의 로그 수집 시스템으로 보내는 역할을 수행

### sidecar container 사례

- MSA 환경 애플리케이션 로깅
    - 여러 MSA 환경의 application container는 들어오는 요청 처리와 같은 서비스의 핵심 기능을 처리
    - 추가로 애플리케이션에 대한 로깅 및 모니터링 기능을 Sidecar container로 구현 가능
    - Sidecar container는 수집된 로그를 중안 로깅 시스템으로 전달하거나 생성된 로그를 캡처하여 적절한 대상으로 전달하는 역할만 담당
- application container 성능 유지
    - 로깅 기능을 Sidecar container로 분리하여 application container가 본연의 작업에 집중하도록 하여 application container의 성능을 유지할 수 있다.
    - 리소스 모니터링을 위해 Sidecar container는 Prometheus 같은 도구와 통합되어 지표를 수집하고, 리소스 사용량을 모니터링하고, 마이크로서비스에 대한 성능 데이터를 수집할 수 있다.

### Sidecar container를 통한 로그 수집

- Sidecar container를 통해 수집된 로그를 Nginx container의 index.html에 적용하여 1초 간격으로 출력되게 한다.
- sidecar-pod.yaml
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
       name: log-pod
    spec:
      containers:
      - name: app-container
        image: nginx:1.25.3
        volumeMounts:
        - name: html-log
          mountPath: /usr/share/nginx/html
      - name: sidecar-container
        image: debian:10
        volumeMounts:
        - name: html-log
          mountPath: /date-log
        command: ["/bin/sh", "-c"]
        args:
          - while true; do
              date >> /date-log/index.html;
              sleep 1;
            done
      volumes:
      - name: html-log
        emptyDir: {}
    ```
    
- 로그 확인
    - app-container
        
        ```bash
        kubectl exec log-pod -c app-container -- tail -f /usr/share/nginx/html/index.html
        Mon Feb 12 09:12:09 UTC 2024
        Mon Feb 12 09:12:10 UTC 2024
        Mon Feb 12 09:12:11 UTC 2024
        Mon Feb 12 09:12:12 UTC 2024
        .
        ```
        
    - sidecar-container
        
        ```bash
        kubectl exec log-pod -c sidecar-container -- tail -f /date-log/index.html
        Mon Feb 12 09:09:39 UTC 2024
        Mon Feb 12 09:09:40 UTC 2024
        Mon Feb 12 09:09:41 UTC 2024
        Mon Feb 12 09:09:42 UTC 2024
        ..
        ```
        
    - Sidecar container에서 수집한 로그를 app-container에서 출력하는 것을 확인

### 웹 서버에서 생성된 로그를 집계 서비스로 전달하는 sidecar container

- 웹 서버에서 생성된 access.log, error.log는 영구 볼륨에 배치하여 개발자가 오류 및 버그를 추적할 수 있도록 로그에 액세스 및 웹 서버에 대한 액세스 로그, 오류 로그를 로그 집계 서비스로 전달
- blog-web.yaml
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: blog-web
      labels:
        name: blog
    spec:
      volumes:
        - name: shared-logs
          emptyDir: {}
      containers:
        - name: blog-web-container
          image: nginx:1.25.3
          volumeMounts:
            - name: shared-logs
              mountPath: /var/log/nginx
        - name: sidecar-container
          image: debian:10
          command:
            [
              "/bin/bash",
              "-c",
              "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 5; done",
            ]
          volumeMounts:
            - name: shared-logs
              mountPath: /var/log/nginx
    ```
    
- 로그 확인
    - blog-web-container
        
        ```bash
        kubectl exec blog-web -c blog-web-container -- tail -f /var/log/nginx/access.log
        10.108.82.192 - - [12/Feb/2024:09:24:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.81.0" "-"
        ```
        
    - sidecar-container
        
        ```bash
        kubectl exec blog-web -c sidecar-container -- tail -f /var/log/nginx/access.log
        10.108.82.192 - - [12/Feb/2024:09:24:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.81.0" "-"
        ```
        
    - 두 컨테이너 모두 동일한 로그가 출력된다.

## Label 활용
### Label

- 복잡하고, 다양한 Pod를 효율적인 집합으로 다루기 위한 방법으로 Label 사용
    - Label은 Key-value기반의 속성 tag로 하나 이상 설정이 가능
    - 용도에 따른 리소스 선택 시 유용 → 객체를 식별하고 그룹화
    - Pod는 Label을 가질 수 있고, Label 검색 조건에 따라서 특정 Label을 가지고 있는 Pod만을 선택할 수 있다.
    - Label을 선택하여 특정 리소스만 배포하거나 업데이트할 수 있다.
    - Label로 선택된 리소스만 Service에 연결하거나 특정 Label로 선택된 리소스에만 네트워크 접근 권한을 부여하는 등의 작업을 할 수 있다.

### Annotations

- Label과 달리 추가적인 메타정보를 저장하는 목적
    - Label: 객체를 식별하고, 그룹화
    - Annotations: API를 통해 추가적인 데이터를 저장하는 방법
- 설정 정보 전달 및 도구에 대한 정보 제공, Label과 일부 기능은 도일
    - 객체에 대한 업데이트 사유 추적
    - 특정 스케줄링 정책 전달
    - 예시 - 특정 아이콘의 URL 주소를 제공
        
        ```yaml
        metadata:
          annotations:
            examples.com/icon-url: "https://example.com/icon.png"
        ```
        

### Label 지정, 조회

- label 지정, 조회
    
    ```bash
    kubectl run mynginx1 --image=nginx:1.25.3-alpine --labels=key=value
    kubectl run mynginx2 --image=nginx:1.25.3-alpine --labels="key1=value1,key2=value2"
    kubectl run mynginx3 --image=nginx:1.25.3-alpine --labels="key1=value1,key2=value2,key3=value3"
    
    kubectl get po --show-labels | grep mynginx
    mynginx1                            1/1     Running            0                 59s    key=value
    mynginx2                            1/1     Running            0                 9s     key1=value1,key2=value2
    mynginx3                            1/1     Running            0                 1s     key1=value1,key2=value2,key3=value3
    
    kubectl get po --selector=key=value
    NAME       READY   STATUS    RESTARTS   AGE
    mynginx1   1/1     Running   0          2m40s
    ```
    
    - create —labels=<key>=<value>를 사용하면 오브젝트에 Label을 지정할 수 있다.
    - get —show-labels를 사용하면 label을 조회할 수 있다.
    - get —selector=<key>=<value>를 사용하면 해당 label을 사용하는 리소스만 조회 가능
        - —selcetor=<key>만을 사용해서 Value에 상관없이 key가 같은 리소스 조회 가능
- label을 사용한 manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        key1: value1
        key2: value2
      name: mynginx2
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: mynginx2
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    ```
    

### 특정 namespace에 label을 지정하여 파드 생성

- manifest 작성
    - Pod Manifest 틀 생성
        
        ```bash
        kubectl run label-pod-a --image=dbgurum/k8s-lab:initial --namespace=infra-team-ns1 --labels=type=infra1 --dry-run=client -o yaml > label-pod.yaml
        ```
        
    - Pod Manifest 수정
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            type: infra1
          name: label-pod-a
          namespace: infra-team-ns1
        spec:
          containers:
          - image: dbgurum/k8s-lab:initial
            name: label-pod-a
        ---
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            type: infra1
          name: label-pod-b
          namespace: infra-team-ns1
        spec:
          containers:
          - image: dbgurum/k8s-lab:initial
            name: label-pod-b
        ---
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            type: infra1
          name: label-pod-c
          namespace: infra-team-ns1
        spec:
          containers:
          - image: dbgurum/k8s-lab:initial
            name: label-pod-c
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: infra-svc1
          namespace: infra-team-ns1
        spec:
          selector:
            type: infra1
          ports:
          - port: 7777
        ```
        
        - 여러 리소스를 하나의 Manifest로 생성
- 리소스 확인
    
    ```bash
    kubectl get po,svc -o wide --show-labels  -n infra-team-ns1
    NAME              READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES   LABELS
    pod/label-pod-a   1/1     Running   0          94s   10.111.156.74   k8s-node1   <none>           <none>            type=infra1
    pod/label-pod-b   1/1     Running   0          94s   10.111.218.75   k8s-node3   <none>           <none>            type=infra1
    pod/label-pod-c   1/1     Running   0          93s   10.109.131.18   k8s-node2   <none>           <none>            type=infra1
    
    NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR      LABELS
    service/infra-svc1   ClusterIP   10.99.125.96   <none>        7777/TCP   93s   type=infra1   <none>
    ```
    

### Pod에 설정된 label을 검색 조건으로 사용

```bash
kubectl describe svc <label>
```

- 위와 같은 방법으로 kubectl 명령 뒤에 label을 지정하면 지정된 label에 해당하는 리소스만 명령을 실행

## Node Schedule 활용 - taint, tolerations
### Node Schedule

- Kubernetes 배포가 더 크고 다양해지면 스케줄링(노드 할당)관리가 더 중요해짐
    - kube-scheduler가 Pod가 할당 될 Node를 결정
- 사용자가 원하는 Node에 스케줄링
    - nodeSelector: kube-scheduler에게 지정 Node 배치 요청
    - nodeName: 해당 Node의 kubelet에게 직접 요청
    - Affinity: 다양한 조건(친밀도, Affinity)으로 Node 배치 요청
    - Tolerations: Taint가 설정된 Node 강제 허용 요청
    - schedulerName: Multi Cluster 환경인 경우

### nodeSelector를 사용해서 원하는 node에 pod를 배치

- nodeSelector를 사용하는 pod manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: sch-test1
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node1
      containers:
      - image: nginx:1.25.3
        name: sch-test1
    ```
    
- Pod가 배치된 Node 확인
    
    ```bash
    kubectl get po -o wide | grep sch
    sch-test1                           0/1     ContainerCreating   0                 11s    <none>           k8s-node1    <none>           <none>
    ```
    
    - manifest에 지정한 node에 pod가 배치된 것을 확인

### Node에 식별에 필요한 label 설정

- Node에 Label 설정
    
    ```bash
    kubectl label nodes k8s-node2 cputype=gpu
    ```
    
- 설정한 Node의 label을 사용해 Pod를 배치하는 Manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: sch-test2
    spec:
      nodeSelector:
        cputype: gpu
      containers:
      - image: nginx:1.25.3
        name: sch-test2
    ```
    
- Pod가 배치된 Node 확인
    
    ```bash
    kubectl get po -o wide | grep sch
    sch-test1                           1/1     Running            0                 4m16s   10.111.156.75    k8s-node1    <none>           <none>
    sch-test2                           1/1     Running            0                 4s      10.109.131.19    k8s-node2    <none>           <none>
    ```
    
    - 두번째로 생성한 pod가 cputype=gpu인 k8s-node2에 배치된 것을 확인
- Node Label 수정
    
    ```bash
    kubectl label nodes k8s-node2 cpu-type=gpu2 --overwirte
    ```
    
    - 이미 존재하는 키를 사용해 label을 수정할 때 —overwrite를 사용해야 한다.
- Node label 삭제
    
    ```bash
    kubectl lable nodes k8s-node2 cpu-type-
    ```
    
    - label을 삭제할 때는 “key-”를 사용
    
    #### [참고] Node의 role 설정
    
    ```bash
    kubectl label node k8s-node1 node-role.kubernetes.io/worker=worker
    ```
    
    - worker 노드의 경우 “node-role.kubernetees.io/worker”를 키로 가지는 Label을 설정하면 Role이 설정된다.
    - 노드 확인
        
        ```bash
        kubectl get no -o wide
        NAME         STATUS   ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
        k8s-master   Ready    control-plane   4d20h   v1.28.6   192.168.56.100   <none>        Ubuntu 22.04.3 LTS   6.5.0-17-generic   containerd://1.6.28
        k8s-node1    Ready    worker          4d20h   v1.28.6   192.168.56.101   <none>        Ubuntu 22.04.3 LTS   6.5.0-17-generic   containerd://1.6.28
        k8s-node2    Ready    worker          4d20h   v1.28.6   192.168.56.102   <none>        Ubuntu 22.04.3 LTS   6.5.0-17-generic   containerd://1.6.28
        k8s-node3    Ready    worker          4d19h   v1.28.6   192.168.56.103   <none>        Ubuntu 22.04.3 LTS   6.5.0-17-generic   containerd://1.6.28
        ```
        

### nodeName으로 Node를 선택

- NodeName으로 Pod를 배치하는 주체는 kube-scheduler가 아닌 kubelet이다.
    - 앞으로 지원하지 않을 예정
- nodeName을 사용하는 Pod manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: sch-test3
    spec:
      nodeName: k8s-node3
      containers:
      - image: nginx:1.25.3
        name: sch-test3
    ```
    
    - nodeSelector 대신 특정 nodeName을 사용하여 배치
- pod 확인
    
    ```bash
    kubectl get po -o wide | grep sch
    sch-test1                           1/1     Running            0                 18m    10.111.156.75    k8s-node1    <none>           <none>
    sch-test2                           1/1     Running            0                 14m    10.109.131.19    k8s-node2    <none>           <none>
    sch-test3                           1/1     Running            0                 12s    10.111.218.76    k8s-node3    <none>           <none>
    ```
    
    - 세번째로 생성된 Pod가 Manifest에 지정된 k8s-node3에 배치된 것을 확인

### Taint, Tolerations

- Master node에는 Taint 설정으로 Application pod를 할당하지 않음 - 기본
- 이를 허용하기 위해 Tolerations가 요구됨
- Taint 구성요소
    - Effect
        - NoSchedule: taint를 toleration 하지 않는 Pod들은 scheduler 안됨
        - PreferNoSchedule: taint를 toleration 하지 않더라도 만약 scheduling 될 수 있는 다른 노드가 없을 때는 이 노드에 scheduling 가능
        - NoExecute: toleration이 없는 pod들은 전부 삭제 됨
    - Key
        - 사용자가 지정하는 기준 대상 키
    - Value
        - 사용자가 지정한 키에 대한 값
    - Operator
        - 키와 값에 대한 연산자

### Taint 설정, 해제

- taint 확인
    
    ```bash
    kubectl describe no | grep -i taint
    Taints:             node-role.kubernetes.io/control-plane:NoSchedule
    Taints:             <none>
    Taints:             <none>
    Taints:             <none>
    ```
    
    - Master Node에는 기본적으로 NoSchedule이 설정되어 있다.
- Taint 설정
    
    ```bash
    kubectl taint node k8s-node1 kubernetes.io/k8s-node1:NoSchedule
    ```
    
    - Taint 확인
        
        ```bash
        kubectl describe no | grep -i taint
        Taints:             node-role.kubernetes.io/control-plane:NoSchedule
        Taints:             kubernetes.io/k8s-node1:NoSchedule
        Taints:             <none>
        Taints:             <none>
        ```
        
        - k8s-node1에 NoSchedule taint 설정
    - Taint를 걸면 기존에 스케줄 된 pod는 유지하지만 새로운 pod를 추가로 스케줄링 받지 않는다.
    - Taint된 node에 pod를 스케줄링하면 pod가 pending 상태로 존재한다.
- Taint 해제
    
    ```bash
    kubectl taint node k8s-node1 kubernetes.io/k8s-node1:NoSchedule-
    ```
    
    - Taint 해제는 해제하고자 하는 taint뒤에 “-”를 붙인다.
    - Taint 확인
        
        ```bash
        kubectl describe no | grep -i taint
        Taints:             node-role.kubernetes.io/control-plane:NoSchedule
        Taints:             <none>
        Taints:             <none>
        Taints:             <none>
        ```
        
    - Taint 해제시 해당 노드에 스케줄 되기를 기다리는 pod가 배치된다.

### Taint가 설정된 Node에 toleration을 사용한 pod 스케줄링

- Taint 설정
    
    ```bash
    kubectl label nodes k8s-node1 cputype=gpu  # 노드를 특정하기 위핸 label 설정
    kubectl taint node k8s-node1 cputype=gpu:NoSchedule
    ```
    
    - label을 사용해서 taint 설정 가능
- Tolerations가 설정된 Pod manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: sch-test4
    spec:
      nodeSelector:
        cputype: gpu
      containers:
      - image: nginx:1.25.3
        name: sch-test4
      tolerations:
      - key: "cputype"
        operator: "Equal"
        value: "gpu"
        effect: "NoSchedule
    ```
    
    - cputype이 gpu이고 NoSchedule이면 스케줄링 허용 Tolerations
- 파드 확인
    
    ```bash
    get po -o wide | grep sch
    sch-test1                           1/1     Running            0                 53m    10.111.156.75    k8s-node1    <none>           <none>
    sch-test2                           1/1     Running            0                 49m    10.109.131.19    k8s-node2    <none>           <none>
    sch-test3                           1/1     Running            0                 34m    10.111.218.76    k8s-node3    <none>           <none>
    sch-test4                           1/1     Running            0                 9s     10.111.156.76    k8s-node1    <none>           <none>
    ```
    
    - 네번째 Pod가 NoSchedule로 taint된 k8s-node1에 정상적으로 스케줄 됨

## Liveness, Readiness and Startup, Probe의 이해
### Pod Probe(진단) 서비스

- Probe는 container에서 kubelet에 의해 주기적으로 수행되는 진단을 의미
- 진단을 수행하기 위해 kubelet은 container에 의해서 구현된 Handler를 호출
- Handler
    - ExecAction: container 내에서 지정된 명령어를 실행, 명령어 상태 코드 0으로 종료되면 진단이 성공한 것으로 간주, 실패는 1
    - TCPSocketAction: 지정된 포트에서 container의 IP 주소에 대해 TCP 검사를 수행. 포트가 활성화되어 있다면 진단이 성공한 것으로 간주
    - HTTPGetAction: 지정된 포트 및 경로에서 container의 IP주소에 대한 HTTP Get 요청을 수행. 응답 상태코드가 200~399면 진단이 성공한 것으로 간주. 나머지 실패
- Probe result → Success(진단 통과), Failure(진단 실패), Unknown(진단 자체가 실패)

### Probes

- Kubernetes Probe는 컨테이너의 상태를 지속적으로 모니터링하고, 문제가 발생하면 자동으로 조치를 취하는 Kubernetes 기능
- Pod Probe(진단) 서비스 종류
    - liveness Probe: container가 동작 중인지 여부를 진단
    - readinessProbe: container가 요청을 처리할 준비가 되었는지 여부를 진단
    - startupProbe: container 내의 애플리케이션이 시작되었는지를 진단
        - 이 Probe가 활성화되면 다른 Probe는 활성화되지 못한다.
        - startupProbe가 실패하면 kubelet이 container를 죽인다.
    - 이 Probe 사용을 지정하지 않으면 기본적으로 Success 상태로 넘어간다.
- 여러 Probe를 조합해서 사용할 수 있다.
    - startupProbe을 먼저 사용하고 다른 Probe를 사용하는 2단계 Probe가 가장 이상적이다.
    - startupProbe가 진단을 성공하면 다른 Probe들이 진단에 도움이 될 수 있다.
- livenessProbe와 readinessProbe는 모두 Pod에 있는 컨테이너의 안정성과 가용성을 보장하는데 사용되는 메커니즘
    - livenessProbe는 컨테이너가 여전히 실행 중이고 올바르게 작동하는지 확인하는데 사용
        - 컨테이너가 살아 있고 응답하는지 확인하고 컨테이너가 응답하지 않거나 오류 상태에 갇히는 상황을 감지하고 복구하는데 사용
        - 실패한 컨테이너를 자동으로 다시 시작하여 애플리케이션을 계속 사용할 수 있도록 보장 가능
    - readinessProbe는 컨테이너가 들어오는 트래픽을 수락할 준비가 되었는지 확인하는데 사용
        - 컨테이너가 요청을 안전하게 처리할 수 있는 상태인지 확인
- Probe 서비스 진단 시점
    
    <img src="/images/Probe_1.png" width="50%" height="50%" title="probe" alt="probe">    
    
    - Main container가 생성될 때 진단
    - PostStart Hook
        - Pod가 완전 초기화 되기 전에 일부 작업을 수행할 수 있는 기능을 제공
        - 컨테이너가 생성된 직후에 동작
    - PreStop Hook
        - 어떤 이벤트에 의해서 container가 종료되기 전에 호출되는 Hook
        - 진행 중인 프로세스가 완료되도록 보장하는 기능 제공 - graceful shutdown 보장

### livenessProbe

- kubernetes 예제
    
    [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
    
    - livenessProbe을 사용하는 Pod Manifest
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            test: liveness
          name: liveness-exec
        spec:
          containers:
          - name: liveness
            image: registry.k8s.io/busybox
            args:
            - /bin/sh
            - -c
            - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
            livenessProbe:
              exec:
                command:
                - cat
                - /tmp/healthy
              initialDelaySeconds: 5
              periodSeconds: 5
        ```
        
- 이벤트 확인
    
    ```bash
    kubectl describe po liveness-exec
    ..
    Events:
      Type     Reason     Age                        From               Message
      ----     ------     ----                       ----               -------
      Normal   Scheduled  2m44s                      default-scheduler  Successfully assigned default/liveness-exec to k8s-node1
      Normal   Pulled     2m10s                      kubelet            Successfully pulled image "registry.k8s.io/busybox" in 2.531s (2.531s including waiting)
      Normal   Pulled     57s                        kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.635s (1.635s including waiting)
      Warning  Unhealthy  13s (x6 over 98s)          kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
      Normal   Killing    13s (x2 over 88s)          kubelet            Container liveness failed liveness probe, will be restarted
      Normal   Pulling    <invalid> (x3 over 2m13s)  kubelet            Pulling image "registry.k8s.io/busybox"
      Normal   Created    <invalid> (x3 over 2m10s)  kubelet            Created container liveness
      Normal   Started    <invalid> (x3 over 2m10s)  kubelet            Started container liveness
      Normal   Pulled     <invalid>                  kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.912s (1.912s including waiting)
    ```
    
    - livenessProbe가 진단을 하고 ‘/tmp/healthy’ 파일이 없어 진단이 fail됨
    - kubelet이 container를 죽이고 다시 시작
    - ‘/tmp/healthy’파일을 확인될 때까지 반복하다가 지정된 600초가 지나면 더이상 진단을 하지 않고 pod 재시작을 수행하지 않는다.

## 참고 자료  
[쿠버네티스 - 파드](https://kubernetes.io/ko/docs/concepts/workloads/pods/)    
[동양북스 - 쿠버네티스 입문](https://www.yes24.com/Product/Goods/85578606)    
fastcampus - 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online   