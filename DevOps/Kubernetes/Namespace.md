# Namespace

- Namespace(ns)는 Kubernetes cluster 내에서 리소스(pod, Service 등)를 구분(격리)하기 위한 가상의 논리적 공간(그룹, 파티셔닝)을 제공 scope
    - 서비스(애플리케이션, 팀, 환경 등 목적에 따른 구분) 단위의 Namespace 구분은 전체 프로젝트 운영, 관리 등의 측면에서 유리
    - 개발 / 테스트 / QA / 운영 등의 목적에 따른 ns 구분
    - 팀 내부의 업무별 구분 blogTeam-dev / bIogTeam-test / bIogTeam-prod
    - ns를 통해 격리할 수 있기 때문에 애플리케이션들을 서로 간섭없이 실행 가능
    
    ### Namespace에 종속되거나 종속되지 않는 Kubernetes object
    
    - 일반적으로 사용되는 deployments, pod 등의 api-resources는 ns에 의해 구분
        - kubectl apl-resources —namespaced=true
        - kubectl api-resources —namespaced=false
    - Node, PV등은 Kubernetes에서 사용되는 저수준의 object로 ns에 의해 구분되지 않음
    - ns에 의해 관리되는 object가 아닌 클러스터 전반에 걸쳐 사용되는 object라는 의미

## Namespace(ns)별 접근 제어(RBAC및) 자원소비 제어(ResourceQuota )설정 가능

- 클라스터 및 ns 수준의 RBAC 구성(RoleBinding, ClusterRoleBinding)을 통해 리소스, 사용자
간의 접근 제어 구현, 이를 통한 기본 보안 강화
- 민감한 데이터나 리소스에 대한 무단 액세스를 방지하는 동시에 여러 팀이 Namespace 내에서
작업할 수 있도록 허용
- ResourceQuota는 ns가 사용 가능한 최대 리소스 양을 지정하여 특정 애플리케이션의 자원
독점으로 인한 다른 애플리케이션의 상대적 성능 저하를 방지
- 주로 CPU, Memory, Pod 수 등을 제한 좀 더 효율적인 용량 관리를 통해 성능향상에 기여
- ResourceQuota은 특정 Namespace에 사용가능한 총 리소스이며, LimitRange는 Namespace 내에서 실행되는 컨테이너에 대한 제한을 할당하는데 사용됩니다.

## 서로 다른 팀이나 프로젝트가 서로 방해하지 않고 Kubernetes 클러스터에서
워크로드를 실행할수 있다. ns는 가상(or 하위) 클러스터

- ns 내에서의 이름 중복은 허용하지 않지만, 서로 다른 ns 내에서는 중복된 이름 허용
- ns로 구분해도 리소스 간의 통신(송수신)은 기본적으로 허용
- 제한은 NetworkPoIicy를 통해 Pod 내부로 들어오거나(lngress)외부로 나가는(Egress) 트래픽을 허용하고 거부하는 정책 설정
- WhiteIist 설정 값으로 목록이 작성되면 Pod, Namespace, Node 등 이외에는 이 Pod에 트래픽을 보낼 수가 없다

## 기본제공 Namespace

- default: 지정된 Namespace가 없는 오브젝트를 위한 기본 ns (주로 테스트 목적)
- kube-system: Kubernetes의 주요 구성요소를 위한 ns
- kube-public: 모든 사용자가 공개적으로 접근할수 있는 object(시스템프로세스)를 위한 ns
- Kube-node-Iease: 클러스터가 스케일링 될 때 노드의 heartbeat와 리더 선출과 같은 시스템 핵심 기능과 관련된 lease 오브젝트에 대한 Namespace
    - 분산 시스템의 경우 공유 리소스를 잠그고 노드 간의 활동을 조절하는 "리스(lease)"가 필요

## namespace 생성

- namespace 생성 및 확인
    
    ```bash
    kubectl create namespace dev-ns
    kubectl create namespace ops-ns
    
    kubectl get ns
    NAME                   STATUS   AGE
    default                Active   11d
    dev-ns                 Active   7s
    kube-node-lease        Active   11d
    kube-public            Active   11d
    kube-system            Active   11d
    kubernetes-dashboard   Active   10d
    openebs                Active   4d6h
    ops-ns                 Active   4s
    portainer              Active   10d
    ```
    
- namespace manifest
    
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      creationTimestamp: "2024-02-19T12:35:41Z"
      labels:
        kubernetes.io/metadata.name: dev-ns
      name: dev-ns
      resourceVersion: "446088"
      uid: 8612fd73-4816-4efb-9d5a-d64f315badd6
    spec:
      finalizers:
      - kubernetes
    status:
      phase: Active
    ```
    
- 현재 namespace 확인 및 사용 namespace 변경
    
    ```bash
    kubectl config get-contexts
    CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
    *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
    ```
    
    - NAMESPACE를 확인했을 때 아무것도 없으면 defaut namespace이다.
    - namespace 변경
        
        ```bash
        kubectl config set-context --current --namespace dev-ns
        kubectl config get-contexts
        CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
        *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   dev-ns
        ```
        
        - namespace를 변경 후 확이하면 NAMESPACE에 변경한 namespace가 출력된다.
    - namespace 검증
        
        ```bash
        kubectl config view --minify | grep namespaces:
        ```

## 다른 네임스페이스에 object 생성

- -n 옵션을 사용해서 특정 namespace에 object 생성
    
    ```bash
    kubectl create deployment nstest -n ops-ns --image=nginx --port=80 --replicas=2
    kubectl expose deployment nstest --name=netest-svc --port=80 --target-port=80
    Error from server (NotFound): deployments.apps "nstest" not found
    ```
    
    - deployment를 ops-ns에 생성하였지만 svc를 -n 없이 dev-ns에 생성하려고 시도하면 오류가 발생한다.
    
    ```bash
    kubectl -n ops-ns expose deployment nstest --name=netest-svc --port=80 --target-port=80
    ```
    
    - deployment 생성과 동일하게 -n 옵션을 사용해서 ops-ns에 svc를 생성하면 오류가 발생하지 않는다.
    
    ```bash
    kubectl get po,svc -o wide | grep nstest
    No resources found in dev-ns namespace.
    
    kubectl -n ops-ns get po,svc -o wide | grep nstest
    pod/nstest-865bf8c9cf-7rrks   1/1     Running   0          3m27s   10.111.218.98   k8s-node3   <none>           <none>
    pod/nstest-865bf8c9cf-wrfcn   1/1     Running   0          3m27s   10.111.156.79   k8s-node1   <none>           <none>
    service/netest-svc   ClusterIP   10.110.196.30   <none>        80/TCP    85s   app=nstest
    ```
    
    - 현재 ns가 dev-ns이므로 생성한 deploy와 svc를 확인하기 위해서 -n 옵션을 사용해 ops-ns에서 조회해야 한다.
    - curl 사용은 ns를 지정하지 않아도 된다.
        
        ```bash
        curl 10.111.218.98
        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
        html { color-scheme: light dark; }
        ```
        

## 같은 이름의 object를 다른 ns에 생성

- ops-ns에 생성한 deploy, svc와 동일한 이름의 deploy, svc를 dev-ns에 생성
    
    ```bash
    kubectl create deployment nstest --image=nginx --port=80 --replicas=2
    kubectl expose deployment nstest --name=netest-svc --port=80 --target-port=80
    ```
    
- dev-ns, ops-ns에 생성한 deploy,svc 확인
    
    ```bash
    kubectl get deploy,po,svc -A | grep nstest
    dev-ns                 deployment.apps/nstest                         2/2     2            2           30s
    ops-ns                 deployment.apps/nstest                         2/2     2            2           8m15s
    dev-ns                 pod/nstest-865bf8c9cf-6xvwv                         1/1     Running   0              30s
    dev-ns                 pod/nstest-865bf8c9cf-ztj6m                         1/1     Running   0              30s
    ops-ns                 pod/nstest-865bf8c9cf-7rrks                         1/1     Running   0              8m14s
    ops-ns                 pod/nstest-865bf8c9cf-wrfcn                         1/1     Running   0              8m14s
    ```
    
    - 동일한 이름의 object가 서로 다른 ns에 존재하는 것을 아무 문제가 없다.
- pod-to-pod test
    
    ```bash
    kubectl run dns-test -it --rm --image=alpine -- sh
    
    wget -qO- --timeout=3 http://netest-svc
    wget: bad address 'netest-svc'
    
    wget -qO- --timeout=3 http://netest-svc.ops-ns
    <!DOCTYPE html>
    <html>
    <head>
    
    wget -qO- --timeout=3 http://netest-svc.dev-ns
    <!DOCTYPE html>
    <html>
    <head>
    ```
    
    - defalut ns에서 Pod를 생성하여 DNS를 활용해 다른 ns의 오브젝트에 접근
    - 이때 dns에 namespace를 지정하지 않으면 현재 ns에 조회를 시도한다.

## namespace 통신이 가능한 이유

- DNS 검증
    
    ```bash
    kubectl run -it --rm dns-verify --restart=Never --image=busybox -- nslookup 10.111.218.98
    Server:		10.96.0.10
    Address:	10.96.0.10:53
    
    98.218.111.10.in-addr.arpa	name = 10-111-218-98.netest-svc.ops-ns.svc.cluster.local
    
    kubectl run -it --rm dns-verify --restart=Never --image=busybox -- nslookup 10.111.156.79
    Server:		10.96.0.10
    Address:	10.96.0.10:53
    
    79.156.111.10.in-addr.arpa	name = 10-111-156-79.netest-svc.ops-ns.svc.cluster.local
    ```
    
- nslookup
    
    ```bash
    nslookup netest-svc.ops-ns.svc.cluster.local
    Server:		127.0.0.53
    Address:	127.0.0.53#53
    
    ** server can't find netest-svc.ops-ns.svc.cluster.local: SERVFAIL
    ```
    
    - nslookup을 생성한 다른 네임스페이스의 svc를 dns로 찾으면 svc가 있음에도 찾을 수 없다는 문제가 발생한다.
    - /etc/resolv.conf에서 kubernetes의 nameserver를 추가한다.
        
        ```bash
        nameserver 127.0.0.53
        nameserver 10.96.0.10
        options edns0 trust-ad
        search .
        ```
        
- kubernetes nameserver 추가 후 nslookup
    
    ```bash
    nslookup netest-svc.ops-ns.svc.cluster.local
    ;; Got SERVFAIL reply from 127.0.0.53, trying next server
    Server:		10.96.0.10
    Address:	10.96.0.10#53
    
    Name:	netest-svc.ops-ns.svc.cluster.local
    Address: 10.110.196.30
    ;; Got SERVFAIL reply from 127.0.0.53, trying next server
    ```
    
- svc DNS로 curl을 시도 가능
    
    ```bash
    curl netest-svc.ops-ns.svc.cluster.local
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    ```
    
- clusterIP로 curl은 불가능
    - IPtable 포트 주소로 등록되어 있는 정보로 애플리케이션 구동하는 것이 목적
    - ping이 날아가지 않는다.

## 참고 자료
FastCampus - 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online.      
[Kubernetes Docs - 네임스페이스](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/namespaces/)      