# Volume Mount

## Kubernetes Storage Volume 이해
### Volume

- Kubernetes는 Storage Volume정의를 통해 Pod의 VoIume을 정의한다.
    - Volume 자체를 정의하는 기능은 없다.
    - Pod 내부의 Volume 기술을 정의하는 것
- Pod 컨테이너 내에 저장된 파일은 별도 설정이 없으면 Host의 임시 디스크에 보관
    - Pod 삭제 및 재시작 시 임시 디스크의 데이터도 함께 손실 됨
    - 멀티컨테이너 Pod는 내부 컨테이너 간 데이터 공유 수행이 필요
        - Sidecar container가 작업 수행시 데이터 공유가 필요할 때 사용
- Kubernetes volume 추상화는 이러한 문제를 해결하는 메커니즘을 제공
    - 데이터 손실 관리
    - 데이터 공유 방법 관리
    - 네트워크가 아닌 데이터 공유의 관점에서 해결하려고 함
- VoIume은 Podspec에 포함하여, Pod 내 filesystem에 mount 됨
    - Podspec → .spec.volumes 구문으로 볼륨 기술 및 이름 지정
    - .spec.containers[*].voIumeMounts. 컨테이너 내부 FS에 mount df -h로 확인
        - volumeMounts에 실제 경로가 입력되어야 한다.
        - 이름은  .spec.volumes에서 지정한 이름과 같아야 한다.
    - VoIume은 데이터 공유 및 지속성에 도움 되고. 일부 API 기능을 통해 데이터 이전 효과도 지원
        - DB
- VoIume의 임시 볼륨과 영구 볼륨으로 구분 기준
    - 상태 비저장 애플리케이션의 경우 이 경우 임시 볼륨을 사용 (데이터를 보존할 필요 없음)
    - 상태 비저장 애플리케이션은 컨테이너가 존재하는 동안 파일 시스템에 데이터를 쓸수 있지만 이는 애플리케이션의 수명 주기 측면에서 중요하지 않다.

### Volume 사용 예

- Tomcat & Logstash
    - Tomcat
        - 웹 애플리케이션
        - 로그(/usr/local/tomcat/logs)
        - 로그 파일 손상이 동작에 영향 없음.
    - Logstash
        - Tomcat 로그 분석을 위해 사용
        - 수집된 로그를 EIasticsearch에 전달
        - ELK 스택을 통해 Tomcat 로그분석
    - 임시 볼륨 기법
        - ELK 스택을 통해 Tomcat 로그분석을 한다면 Tomcat의 /usr/local/tomcat/logs와 emptyDir 볼륨을 통해 Logstash 컨테이너의 /mnt 간에 파일 시스템 공유가 필요하다.
    - 영구 볼륨 기법
        - Logstash 컨테이너의 /mnt 에 저장된 데이터를 EIasticsearch Pod 로 보낸다면, 이 Pod는 상태 저장이어야 함
        - 상태 저장 Pod는 데이터 보존을 위해 임시 볼륨이 아닌 영구 볼륨을 사용해야 한다. (동일 Pod든, 아니든)

### Volume 방식

| 이름 | 설명 |
| --- | --- |
| emptyDir | Pod가 생성될 때 생성되고 Pod가 삭제될 때 삭제되는 임시 볼륨 |
| hostPath | Node의 로컬 볼륨 |
| awsElasticBlockstore | AWS EBS 볼륨 |
| azureDisk | Microsoft Azure 볼륨 |
| cephfs | Ceph 볼륨 |
| cinder | 오픈스택 Cinder 볼륨 |
| gcePersitentDisk | GCE 볼륨 |
| gitRepo | Git Repository의 콘텐츠를 생성 초기에 저장한 볼륨 |
| iSCCI | iSCSI 볼륨 |
| NFS | NFS(네트워크 파일 시스템) 볼륨 |

## emptyDir 활용
### emptyDir - 임시 볼륨 기법(non-persistent storage)

- Pod 생성시 Podspec에 의해 Pod 내부에존재하는Volume 생성
- 임시(ephemeral Data) 데이터를 저장하는데 사용되는 비어있는 디렉터리 제공 방식
- 동일 Pod 내의 모든 컨테이너의 접근이 가능하고 내부 데이터 공유를 위해 사용
    - init container, sidecar container를 활용할 때 사용
- Pod와 lifecycle이 같다.
- emptyDir Volume은 Pod를 실행하는 Node 환경에 따라 성능 변화
    - Node의 환경: 디스크 유형으로 HDD, SSD, Network Storage 등
    - 디스크 대신 메모리 사용도 가능

### Multi container pod에 emptyDir 활용

- emptyDir을 적용한 Multi container Pod manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: temp-pod1
      name: temp-pod1
    spec:
      volumes:
      - name: temp-vol
        emptyDir:{}
      containers:
        - image: dbgurum/k8s-lab:initial
        name: temp-container1
        volumeMounts:
        - name: temp-vol
          mountPath: /mount1
      - image: dbgurum/k8s-lab:initial
        name: tmpe-container2
        volumeMounts:
        - name: temp-vol
          mountPath: /mount2
    ```
    
- 컨테이너 내부에서 볼륨 파악
    
    ```bash
    kubectl exec -it temp-pod1 -c temp-container1 -- bash
    [root@temp-pod1 /]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    ..
    /dev/sda1        75G   17G   58G  23% /mount1
    ..
    
    [root@temp-pod1 mount1]# echo 'welcome to fastcampus. temp-pod1' > k8s-1.txt
    ```
    
    - 정상적으로 emptyDir이 mount1 경로로 마운트 된것을 확인
    - mount1 디렉토리에 임의 파일 생성 후 temp-container2에서 확인 해본다.
    
    ```bash
    kubectl exec -it temp-pod1 -c tmpe-container2 -- bash
    [root@temp-pod1 /]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    ..
    /dev/sda1        75G   17G   58G  23% /mount2
    ..
    
    [root@temp-pod1 mount2]# ls
    k8s-1.txt
    ```
    
    - temp-container2에는 /mount2로 emptyDir이 마운트 되어 있다.
    - 마운트된 디렉토리의 이름은 다르지만 같은 Volume을 사용하고 있으므로 temp-container1에서 생성한 파일을 확인할 수 있다.
- 임시 볼륨이므로 pod를 삭제하면 볼륨에 있던 데이터도 모두 삭제된다.
    - 같은 Pod를 재생성해서 확인하면 볼륨에는 아무것도 남아있지 않다.

### Multi container Pod에 emptyDir 메모리에 구성

- emptyDir에 메모리 설정이 포함된 pod manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: temp-mem-pod
    spec:
      volumes:
      - name: memory-vol
        emptyDir:
          medium: Memory
          sizeLimit: 1Gi
      containers:
      - image: dbgurum/k8s-lab:initial
        name: mem-container
        volumeMounts:
        - name: memory-vol
          mountPath: /mem-mount
    ```
    
    - emptyDir
        - medium: memory를 설정하면 emptyDir가 메모리에 마운트된다.
        - sizeLimit: emptyDir의 용량을 제한, 미설정시 기본 2Gi로 설정 및 제한
- 생성된 파드에 내부에 접속해 마운트 확인
    
    ```bash
    df -h
    Filesystem      Size  Used Avail Use% Mounted on
    ..
    tmpfs           1.0G     0  1.0G   0% /mem-mount
    ..
    ```
    
    - 이전에 임시 볼륨을 마운트 했을 때 Filesystem은 /dev/sda1이었지만 메모리에 마운트 했을 때는 tmpfs로 되어 있다.

### emptyDir을 이용하여 Pod 외부에서 웹 소스 제공

- Pod 외부에서 웹 소스를 가져오는 Pod manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myweb-pod
      labels:
        app: myweb
    spec:
      containers:
      - name: web-source
        image: alpine
        args: ["tail", "-f", "/dev/null"]
        volumeMounts:
        - name: source-volume
          mountPath: /source
      - name: webserver
        image: nginx:1.25.3-alpine
        volumeMounts:
        - name: source-volume
          mountPath: /usr/share/nginx/html/
      volumes:
       - name: source-volume
         emptyDir: {}
    ```
    
- 생성한 Pod에 접속해서 페이지 확인
    
    ```bash
    kubectl run web-test -it --rm --restart=Never --image=curlimages/curl -- curl 10.109.131.38
    <html>
    <head><title>403 Forbidden</title></head>
    <body>
    <center><h1>403 Forbidden</h1></center>
    <hr><center>nginx/1.25.3</center>
    </body>
    </html>
    pod "web-test" deleted
    ```
    
    - 403 Forbidden 페이지가 출력된다.
- kubectl cp를 사용해서 index.html을 pod 내부로 복사
    
    ```bash
    kubectl cp index.html myweb-pod:/source/index.html -c web-source
    ```
    
    - index.html 복사 후 다시 페이지 확인
        
        ```bash
        kubectl run web-test -it --rm --restart=Never --image=curlimages/curl -- curl 10.109.131.38
        <html>
          <head>
            <title>Kubernetes Running Sample App -emptyDir Test-</title>
              <style>body {margin-top: 40px; background-color: #333;} </style>
            <meta http-equiv="refresh" content="3" >
        </head>
        <body>
          <div style=color:white;text-align:center>
            <h1> Kubernetes, myWeb Pod Application. </h1>
            <h2> Good fastcampus! </h2>
              <p>Application is now good running on a Pod in kubernetes.</p>
            </div>
          </body>
        </html>
        pod "web-test" deleted
        ```
        
    - 사전에 작성한 index.html과 동일한 내용이 출력
- 외부에서 페이지를 볼 수 있도록 Nodeport 서비스 생성
    
    ```bash
    kubectl expose po myweb-pod --name=myweb-svc --type=NodePort --port=80 --target-port=80
    ```
    
- 아래와 같이 로컬에서 접속할 수 있다.
    
    <img src="/images/kube_volume_1.png" width="75%" height="75%" title="kube volume 1" alt="kube volume 1">      
    

### Tomcat & Logstash

- tomcat pod
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: tomcat
    spec:
      replicas: 1
      selector:
        matchLabels:
          run: tomcat
      template:
        metadata:
          labels:
            run: tomcat
        spec:
          containers:
          - image: tomcat
            name: tomcat
            ports:
            - containerPort: 8080
            env:
            - name: UMASK
              value: "0022"
            volumeMounts:
            - name: tomcat-log
              mountPath: /usr/local/tomcat/logs
          - image: logstash:7.17.16
            name: logstash
            volumeMounts:
            - name: tomcat-log
              mountPath: /mnt
            args: ["-e input { file { path => \"/mnt/localhost_access_log.*\" } } output { stdout{ codec => rubydebug } elasticsearch { hosts => [\"http://elasticsearchsvc.default.svc.cluster.local:9200\"] } }"]
          volumes:
            - name: tomcat-log
              emptyDir: {}
    ```
    
    - tomcat의 /usr/local/tomcat/logs를 Logstash의 /mnt에 붙임
- tomcat container에 접속해서 log 파일 확인
    
    ```bash
    kubectl exec -it tomcat-8fb6fc75d-r4gqr -c tomcat -- ls /usr/local/tomcat/logs
    catalina.2024-02-14.log  localhost_access_log.2024-02-14.txt
    ```
    
- logstash container에 접속해서 동일한 log 파일이 있는지 확인
    
    ```bash
    kubectl exec -it tomcat-8fb6fc75d-r4gqr -c logstash -- ls /mnt
    catalina.2024-02-14.log  localhost_access_log.2024-02-14.txt
    ```
    
    - tomcat에서 확인한 로그와 동일한 파일이 마운트한 디렉토리에서 확인 가능

## hostPath 활용
### hostPath

- hostPath는 영구볼륨으로 Pod가 생성된 Node 파일시스템의 FiIe 및 Directory에 Volume으로 mount
- 데이터가 특정 Node에 저장되므로 Pod가 재생성되어 Node가 바뀌면 이전 Node에 저장되어 있던 hostPath VoIume의 데이터를 읽지 못함.
- Node 내 저장된 파일을 Pod 들 간 공유해야 하는 경우에 사용되지만, 단일 노드에만 생성되므로 테스트 상황에 적합
- Pod가 삭제되어도 내용이 삭제되지 않는다
- Kubernetes docs(공식 문세에 나온 주의 사항)
    
    <img src="/images/kube_volume_2.png" width="75%" height="75%" title="kube volume 2" alt="kube volume 2">      
    
### hostPath 값

| 값 | 내용 |
| --- | --- |
|  | 빈 문자열 (기본값)은 이전 버전과의 호환성을 위한 것으로, hostPath 볼륨은
마운트 하기 전에 아무런 검사도 수행되지 않는다. |
| DirectoryOrCreate | 만약 주어진 경로에 아무것도 없다면, 필요에 따라 Kubelet이 가지고 있는 동일한
그룹과 소유권, 권한을 0755로 설정한 빈 디렉터리 생성 |
| Directory | 주어진 경로에 디렉터리가 있어야 함 |
| FileOrCreate | 만약 주어진 경로에 아무것도 없다면, 필요에 따라 Kubelet이 가지고 있는 동일한
그룹과 소유권, 권한을 0644로 설정한 빈 파일 생성 |
| File | 주어진 경로에 파일이 있어야 함 |
| Socket | 주어진 경로에 UNIX 소캣이 있어야 함 |
| CharDevice | 주어진 경로에 문자 디바이스가 있어야 함 |
| BlockDevice | 주어진 경로에 블록 디바이스가 있어야 함 |

### hostPath를 사용한 pod 생성

- hostPath 설정 pod manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: host-pod1
    spec:
      containers:
      - name: container
        image: dbgurum/k8s-lab:initial
        volumeMounts:
        - name: host-path
          mountPath: /mount1
      volumes:
      - name: host-path
        hostPath:
          path: /DATA1/fastcampus
          type: DirectoryOrCreate
    ```
    
- 컨테이너 내부로 들어가 볼륨 확인
    
    ```bash
    kubectl exec -it host-pod1 -- bash
    ```
    
- Pod가 생성된 노드에서 hostPath로 지정한 디렉토리 확인
    
    ```bash
    cd /DATA1/fastcampus/
    ls
    k8s-1.txt
    ```
    
    - 이 디렉토리는 pod가 삭제되어도 노드에 존재하는 디렉토리이므로 삭제되지 않는다.
- Pod 삭제 후 재생성 했을 때 동일한 노드에 생성되면 hostPath에 저장된 파일을 그대로 사용할 수 있지만 그렇지 않으면 사용할 수 없다.

### NFS 구성

- pod를 생성할 때 nodeSelector를 사용해 특정 Node에 배치되도록 설정
- 이전 pod의 manifest에 nodeSelector 추가 → k8s-node2를 위한 Pod도 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: host-pod1
    spec:
      containers:
      - name: container
        image: dbgurum/k8s-lab:initial
        volumeMounts:
        - name: host-path
          mountPath: /mount1
      volumes:
      - name: host-path
        hostPath:
          path: /DATA1/fastcampus
          type: DirectoryOrCreate
    ```
    
- container에 접속해서 마운트한 디렉토리에 임의의 파일 생성
- 모든 워커 노드에 NFS 구성
    
    ```bash
    sudo apt-get update
    sudo apt-get -y install nfs-kernel-server
    sudo systemctl enable --nw nfs-server
    sudo systemctl restart nfs-server
    sudo systemctl status nfs-server
    ```
    
- 1번 노드를 NFS 서버로 설정
    
    ```bash
    sudo vi /etc/exports
    # /DATA1 *(rw,sync,no_root_squash,no_subtree_check,insecure) 권한 설정을 /etc/exports에 추가
    
    sudo netstat -nlp | grep 2049
    sudo systemctl restart nfs-server
    sudo systemctl status nfs-server
    
    sudo mount -t nfs k8s-node1:/DATA1 /DATA1 # 다른 워커 노드에서 실행
    ```
    
- 마운트 확인
    
    ```bash
    df -h
    Filesystem        Size  Used Avail Use% Mounted on
    ..
    k8s-node1:/DATA1  7.3G     0  6.9G   0% /DATA1
    ..
    ```
    
    - 다른 워커 노드에서 마운트를 확인하면 k8s-node1의 /DATA1이 마운트 된 것을 확인
- 파일 공유 확인
    
    ```bash
    sudo touch /DATA1/fastcampus/k8s-nfs.txt  # nfs 서버가 아닌 다른 워커 노드에서
    
    ls /DATA1/fastcampus
    k8s-1.txt  k8s-nfs.txt 
    ```
    
    - 모든 노드에서 동일한 디렉토리에 같은 파일을 확인할 수 있다.

### NFS를 요청하는 Pod 생성

- NFS pod Manifest
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nfs-nginx
    spec:
      containers:
      - name: nfs-nginx
        image: nginx:1.25.3-alpine
        volumeMounts:
        - name: nfs-vol
          mountPath: /usr/share/nginx/html
      volumes:
      - name : nfs-vol
        nfs:
          path: /DATA1
          server: 192.168.56.101
    ```
    
- NFS 마운트 디렉토리에 Index.html 생성
- 페이지 확인
    
    ```bash
    curl 10.109.131.41
    <html>
      <head>
        <title>Kubernetes Running Sample App -emptyDir Test-</title>
          <style>body {margin-top: 40px; background-color: #333;} </style>
        <meta http-equiv="refresh" content="3" >
    </head>
    <body>
      <div style=color:white;text-align:center>
        <h1> Kubernetes, myWeb Pod Application. </h1>
        <h2> Good fastcampus! </h2>
          <p>Application is now good running on a Pod in kubernetes.</p>
        </div>
      </body>
    </html>
    ```
    
    - nfs pod의 ip 주소로 페이지 요청을 하면 nfs의 디렉토리의 Index.html을 사용한다.

### 로그 파일을 hostPath로 공유할 문제가 있는 pod의 로그 파일을 감사

- nginx 로그를 hostPath에 공유하도록 설정한 pod → k8s-node1을 사용
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: weblog-pod
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node1
      containers:
      - name: nginx-web
        image: nginx:1.23.1-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: host-path
          mountPath: /var/log/nginx
      volumes:
      - name: host-path
        hostPath:
          path: /DATA1/nginx-log
          type: DirectoryOrCreate
    ```
    
- k8s-node1의 /DATA1 디렉토리에 nginx-log 디렉토리 생성 확인
    
    ```bash
    student@k8s-node1:/DATA1$ ls
    bin  certs  chisel  compose  docker_config  fastcampus  index.html  lost+found  nginx-log  portainer.db  portainer.key  portainer.pub  tls
    ```
    
    - /DATA1/nginx-log에 남는 로그는 pod가 삭제되더라도 유지된다.
- 로그 감사
    
    ```bash
    awk '$4>"[14/Feb/2024:12:20:30]" && $4<"[14/Feb/2024:12:30:30]"' access.log | awk '{ print $1 } ' | sort | uniq -c | sort -r | more
    ```
    
    - awk를 사용해서 감사
    - 로그는 10개의 패턴을 사용하는데 명령에 사용한 $4은 4번째 패턴이고 이는 날짜다.
    - 첫번째 awk의 결과에서 첫번째 패턴인 IP주소만 출력

### MySQL DB

- MySQL DB의 데이터 저장 경로를 hostPath로 Node에 저장하여 지속
- MySQL의 /var/lib/mysql을 hostPath에 마운트한 pod
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mydata-pod
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node1
      containers:
      - name: mydata-container
        image: mysql:8.0
        volumeMounts:
        - name: host-path
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
      volumes:
      - name: host-path
        hostPath:
          path: /DATA1/mysql-data
          type: DirectoryOrCreate
    ```
    
- MySQL DB에 dummy data 추가
    
    ```bash
    kubectl exec -it mydata-pod -- bash
    mysql -uroot -p
    ```
    
    ```sql
    create database k8sdb;
    use k8sdb;
    create table k8s (c1 int, c2 varchar(20));
    insert into k8s values (1, 'k8s');
    insert into k8s values (2, 'fastcampus');
    select * from k8s;
    exit
    ```
    
- MySQL Pod를 삭제 후 mysql-data 확인
    
    ```bash
    student@k8s-node1:/DATA1/mysql-data$ ls
     auto.cnf        binlog.index   client-cert.pem     '#ib_16384_1.dblwr'  '#innodb_redo'   mysql        performance_schema   server-cert.pem   undo_001
     binlog.000001   ca-key.pem     client-key.pem       ib_buffer_pool      '#innodb_temp'   mysql.ibd    private_key.pem      server-key.pem    undo_002
     binlog.000002   ca.pem        '#ib_16384_0.dblwr'   ibdata1              k8sdb           mysql.sock   public_key.pem       sys
    ```
    
    - MySQL pod를 삭제해도 데이터는 그대로 남아있다.
    - 이 데이터는 다른 워커 노드에 MySQL pod 생성 되어도 NFS로 공유되고 있어 그대로 사용할 수 있다.

## PV & PVC 이해와 활용
### PV & PVC

- hostPath 방식은 리소스 관리 및 보안 측면에서 권장하지 않는다.
- Kubernetes는 PVobject로 Storage를 추상화하고 PVC object로 Storage를 할당 받아 사용할 수 있게 구현
- PV(PersistentVolume)는 관리자나 Storage Class에 의해 생성되는 VoIume
- PVC(Persistent Volume Claim)는 사용자가 Volume을 사용하기 위해 PV에요청
    - PV는 스토리지 그 자체이고. PVC는 사용자, 개발자가 PV에게 VoIume 할당 요청하는 것
    - 용량(capacity)과 권한(accessModes) 설정을 통해 요청

### PV & PVC lifecycle

- 프로비저닝(provisioning)
    - PV를 만드는 단계
    - 이 단계에서는 PV를 미리 만들고 사용하는 정적(static) 방법과 요청이 있을 때 PV를 만드는 동적(dynamic) 방법이있다.
- 바인딩(Binding)
    - 생성한 PV를 PVC와 연결하는 단계
    - PVC는 스토리지 용량과 접근방법에 따라 PV와 연결, 대응되는 PV가 생길 때까지 Pending
    - PV와 PVC는 1:1 관계
- 사용(Using)
    - PVC는 Pod에서 사용되고, Pod는 PVC를 볼륨으로 인식하여 사용하는 단계
- 회수(Reclaiming)
    - 사용이 끝난 PVC가 초기화되는 단계
    - Retain(default) : PVC가 삭제될 때 대응되는 PV 상태는 bound → Released로 바뀌며 다른 PVC가 연결될 수 없는 상태이며 PV속 데이터는 유지
    - DeIete : PVC가 삭제될 때 대응되는 PV도 같이 삭제
    - Recycle : PVC가 삭제될 때 대응되는 PV상태는 bound →  Pending으로 변경, 다른 PVC대기

### accessMode

- PV 생성 시 설정하는 접근 권한
- ROX(ReadOnlyMany)- 읽기만 가능, 모든 노드에서 접근이 가능함
- RWX(ReadWriteMany)- 읽기-쓰기 가능, 모든 노드에서 접근이 가능함
- RWO(ReadWriteOnce) - 단일 노드에서 읽기-쓰기로 마운트 가능함

### PV 생성 - PV → PVC → Pod

- pv manifest → pv1: 1G, RWO, pv2: 1G, ROX, pv3: 2G, RWX
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv1
    spec:
      capacity:
        storage: 1G
      accessModes:
      - ReadWriteOnce
      local:
        path: /DATA2
      nodeAffinity:
        required:
          nodeSelectorTerms:
          - matchExpressions:
            - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}
    ```
    
    - capacity와 accessMode는 PVC가 PV를 요청할 때 사용하는 대상
    - 마운트할 경로
        - local, NFS, cloud를 할 것인지 설정
    - Node를 식별할 nodeAffinity가 필요하다.
        - 예시는 nodeSelector를 사용하며 key가 [kubernetes.io/hostname이고](http://kubernetes.io/hostname이고) 값이 k8s-node1인 노드를 선택해서 프로비저닝
- 생성한 pv 확인
    
    ```bash
    kubectl get pv
    NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                 STORAGECLASS   REASON   AGE
    persistentvolume/pv1            1G         RWO            Retain           Available                                                 7s
    persistentvolume/pv2            1G         ROX            Retain           Available                                                 7s
    persistentvolume/pv3            2G         RWX            Retain           Available                                                 7s
    ```
    
    - 현재 pv들은 Retain 상태로 회수되어 있다.
- PVC manifest
    
    ```yaml
    apiVersion: v1             # pv2
    kind: PersistentVolumeClaim
    metadata:
      name: pvc1
    spec:
      accessModes:
      - ReadOnlyMany
      resources:
        requests:
          storage: 1G
    ---
    apiVersion: v1             # pv1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc2
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1G
    ---
    apiVersion: v1             # pending - 요청한 용량에 적절한 PV가 없기 때문
    kind: PersistentVolumeClaim
    metadata:
      name: pvc3
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 5G
    ---
    apiVersion: v1             # pv3
    kind: PersistentVolumeClaim
    metadata:
      name: pvc4
    spec:
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 1G
    ```
    
- 생성한 PVC 확인
    
    ```bash
    ubectl get pv,pvc
    NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
    persistentvolume/pv1            1G         RWO            Retain           Bound    default/pvc2                                  6m21s
    persistentvolume/pv2            1G         ROX            Retain           Bound    default/pvc1                                  6m21s
    persistentvolume/pv3            2G         RWX            Retain           Bound    default/pvc4                                  6m21s
    
    NAME                         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/pvc1   Bound     pv2      1G         ROX                           5s
    persistentvolumeclaim/pvc2   Bound     pv1      1G         RWO                           5s
    persistentvolumeclaim/pvc3   Pending                                                     5s
    persistentvolumeclaim/pvc4   Bound     pv3      2G         RWX                           5s
    ```
    
    - 각 PVC는 요청한 accessMode와 capacity에 맞는 PV에 바인딩 되었다.
    - pvc3는 요청한 용량을 만족하는 PV가 없어 pending 상태이다.
- PVC를 볼륨으로 사용하는 Pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mynode-pod
    spec:
      containers:
      - image: sighingowl/mynode:1.0
        name: mynode-container
        ports:
        - containerPort: 8000
        volumeMounts:
        - name: mynode-path
          mountPath: /mynode
      volumes:
      - name: mynode-path
        persistentVolumeClaim:
          claimName: pvc1
    ```
    
    - volumes에 사용할 볼륨으로 persistentVolumeClaim으로 설정하고 사용할 PVC를 지정
- Pod에 접속해서 임의파일 생성 후 PV에 가서 확인
    
    ```bash
    student@k8s-node1:/DATA2$ ls
    k8s-pvc.txt  lost+found
    ```
    

### PV, PVC 자동 연결이 아닌 수동 지정(Label 사용)

- PV, PVC의 자동 연결은 조건에 맞는 것을 자동으로 찾아 연결하는 방법을 사용한다.
- Manifest에서 필요할 경우 Label을 사용해 수동으로 PV, PVC를 지정할 수 있다.
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv4
      labels:
        name: pv4
    spec:
      capacity:
        storage: 1Gi
      accessModes:
      - ReadWriteOnce
      local:
        path: /data_dir
      nodeAffinity:
        required:
          nodeSelectorTerms:
          - matchExpressions:
            - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc5
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
      storageClassName: ""
      selector:
        matchLabels:
          name: pv4
    ```
    
- 생성된 PV, PVC 확인
    
    ```bash
    kubectl get pv,pvc
    NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
    ..
    persistentvolume/pv4            1Gi        RWO            Retain           Bound    default/pvc5                                  44s
    
    NAME                         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    ..
    persistentvolumeclaim/pvc5   Bound     pv4      1Gi        RWO                           8s
    ```
    
    - Label에서 지정한 PV에 PVC가 바인딩된 것을 확인

### NFS를 이용하는 PV

- PV의 볼륨을 NFS로 지정하여 사용
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-nfs
    spec:
      capacity:
        storage: 1G
      volumeMode: Filesystem
      accessModes:
      - ReadWriteOnce
      persistentVolumeReclaimPolicy: Recycle
      mountOptions:
      - hard
      nfs:
        path: /DATA1
        server: 192.168.56.101
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc-nfs
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1G
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: mynode-nfs-pod
    spec:
      containers:
      - name: mynode
        image: sighingowl/mynode:1.0
        ports:
        - containerPort: 8000
        volumeMounts:
        - name: testpath
          mountPath: /mount1
      volumes:
      - name : testpath
        persistentVolumeClaim:
          claimName: pvc-nfs
    ```
    
    - PV
        - mountOptions를 사용해 볼륨으로 사용할 nfs를 설정 가능
        - nfs를 사용하는 경로와 서버 ip 주소를 설정
- 생성된 PV, PVC 확인
    
    ```bash
    kubectl get pv,pvc
    NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
    ..
    persistentvolume/pv-nfs         1G         RWO            Recycle          Bound    default/pvc-nfs                               29s
    ..
    
    NAME                            STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/pvc-nfs   Bound     pv-nfs   1G         RWO                           2m9s
    ..
    ```
    
    - nfs를 사용하는 PV에 PVC가 바인딩 됨
- kubectl edit로 PV 용량 변경
    
    ```bash
    kubectl edit pv pv-nfs
    ..
    capacity:
        storage: 5G
    ..
    ```
    
    - kubectl edit로 PV 용량을 바꾸면 즉시 적용된다.
    - 이렇게 PV, PVC를 생성하는 것을 정적인 방법이다.

## 동적 볼륨 storageClass
### 스토리지 정적 프로비저닝 - Static Provisioning

- 정적 프로비저닝은 관리자가 수동으로 PV를 생성
- 해당 PV의 용량, 액세스 모드 및 저장소 유형과 같은 하위 저장소의 세부 정보를 지정
- 이러한 PV는 사용자가 생성하는 PVC 생성으로 요청되어 PVC의 요구 사항과 일치하는 사용가능한 PV에 Binding

### 스토리지 동적 프로비저닝 - Dynamic Provisioning

- 동적 프로비저닝(Dynamic provisioning)은 PVC 요청에 대한 PV 생성을 자동화
    - 정적 프로비저닝은 사전에 PV를 생성 후 PVC 요청을 해야하지만 동적 프로비저닝은 미리 PV를 생성하지 않아도 된다.
- 관리자는 PV 생성하기 위한 템플릿인 StorageClass를 정의
- 각 StorageClass는 PV를 생성하는 책임이 있는 프로비저너(Provisioner)를 지정
    - provisoner는 일종의 driver
    - AWS의 프로비저너나 GCE-pd가 해당
- 사용자가 특정 StorageClass를 참조하는 PVC를 생성하면 프로비저너는 PVC의 요구사항과 일치하는 PV를 자동으로 생성하여 관리자의 수동 개입을 보류 시킨다.
- 프로비저너(provisioner)는 일반적으로 CSP가 제공하는 볼륨 플러그인을사용
    - 예) provisioner: [kubernetes.io/gce-pd](http://kubernetes.io/gce-pd)

### 동적 프로비저닝, StorageClass 전략

- 운영환경에서는 storageClass의 성능을 최대로 높게 구성
- 개발환경에서는 storageClass를 느리지만 안전한 volume에 저장하도록 구성
- 운영정책에 따라 storageClass를 정의하여 그에 맞는 PV를 자동으로 생성할 수 있다.
- storageCIass 생성을 위해 OpenEBS local hostpath 설치
    - OpenEBS hostpath는 Pod가 실행되는 Node의 디렉토리(hostpath)를 Pod의 볼륨으로 할당
    - provisioner : openebs.io/local

### Openebs driver를 이용한 storageClass 생성

- Openebs sc는 첫번째 사용자(소비자)가 볼륨을 Binding할 때까지 대기
    - PVC 생성 후 Pod가 생성되면 볼륨 연결
    - 자동으로 PV가 생성
    - PVC.STATUS가 Pending → Bound로 변경
- openebs-operator와 openebs-sc 설치
    
    ```bash
    kubectl apply -f https://openebs.github.io/charts/openebs-operator-lite.yaml
    kubectl apply -f https://openebs.github.io/charts/openebs-lite-sc.yaml
    ```
    
- storageClass 확인
    
    ```bash
    kubectl get sc
    NAME               PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
    openebs-device     openebs.io/local   Delete          WaitForFirstConsumer   false                  37s
    openebs-hostpath   openebs.io/local   Delete          WaitForFirstConsumer   false                  37s
    ```
    
    - StorageClass를 설치 후 확인하면 프로비저너와 회수 정책, 볼륨 바인드 모드 확인 가능
- Openebs를 사용하는 PVC 생성
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: openebs-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
      storageClassName: "openebs-hostpath"
    ```
    
    - openebs를 사용하기 위해 storageClassName을 지정해야 한다.
- 생성한 sc, pvc 확인
    
    ```bash
    kubectl get sc,pvc | grep openebs
    storageclass.storage.k8s.io/openebs-device     openebs.io/local   Delete          WaitForFirstConsumer   false                  7m35s
    storageclass.storage.k8s.io/openebs-hostpath   openebs.io/local   Delete          WaitForFirstConsumer   false                  7m35s
    persistentvolumeclaim/openebs-pvc   Pending                                      openebs-hostpath   13s
    ```
    
    - pvc는 Pod가 생성되어 볼륨을 연결하게 될 때까지 Pending 상태 유지
- openebs driver를 이용한 storageClass 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mynode-sc-pod
    spec:
      containers:
      - image: sighingowl/mynode:1.0
        name: mynode-container
        ports:
        - containerPort: 8000
        volumeMounts:
        - name: mynode-path
          mountPath: /mynode
      volumes:
      - name: mynode-path
        persistentVolumeClaim:
          claimName: openebs-pvc
    ```
    
    - PVC의 이름을 openebs를 사용하기 위해 만든 PVC의 이름을 사용
- openebs를 사용하는 pod를 생성 후 pv, pvc 확인
    
    ```bash
    kubectl get pv,pvc | grep openebs
    persistentvolume/pvc-df61ac98-6d43-47d7-8f77-14b64fcd6831   1Gi        RWO            Delete           Bound    default/openebs-pvc   openebs-hostpath            30s
    persistentvolumeclaim/openebs-pvc   Bound     pvc-df61ac98-6d43-47d7-8f77-14b64fcd6831   1Gi        RWO            openebs-hostpath   6m10s
    ```
    
    - Pod의 상태가 running이 되면 pvc의 상태가 Bound로 변경이 된다.
        - PV를 확인하면 PVC의 요청에 맞는 PV가 생성되어 있다.
- PV가 생성된 노드에서 openebs 볼륨의 위치 확인
    
    ```bash
    sudo find / -name k8s-sc.txt
    /var/lib/kubelet/pods/b600181f-8623-4df1-8950-5d513b49a346/volumes/kubernetes.io~local-volume/pvc-df61ac98-6d43-47d7-8f77-14b64fcd6831/k8s-sc.txt
    /var/openebs/local/pvc-df61ac98-6d43-47d7-8f77-14b64fcd6831/k8s-sc.txt
    ```
    
    - 사전에 containr에서 PV에 생성한 파일로 위치를 찾아 봄
    - /var/openebs/local/ 아래에 PV 볼륨이 생성된 것을 확인

### Amazon EKS 기반의 Storage(EBS) 사용

### EKS Storage - EBS, EFS

- EBS는 가용 영역에서 자동 볼륨 복제를 사용하여 블록 수준 스토리지를 생성
- 하나 이상의 EBS 볼륨을 단일 EC2 인스턴스에 연결 가능
- 필요에 따라 EC2 인스턴스 간에 EBS 볼륨 이동 가능
- EBS 볼륨은 EC2 인스턴스를 사용해 데이터베이스를 실행하는 데 사용 가능