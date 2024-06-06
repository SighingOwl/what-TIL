# ConfigMap & Secret
Kubernetes는 Pod 생성에 사용되는 YAML 파일과 “설정 값”을 분리할 수 있는 ConfigMap, Secret 제공하며 주로 ConfigMap은 설정 값을 Secret은 노출되서는 안되는 값을 넣어 줄 때 사용합니다.

### ConfigMap, Secret이 필요한 이유

- 개발, 테스트, 운영 환경에 사용되는 각기 다른 환경 값의 분리가 필요
    - 개발 환경은 SSH를 통한 보안 접근 해제
    - 테스트 환경은 지정된 USER 및 KEY를 이용한 SSH 보안 접근 사용
    - 운영환경은 테스트 환경과 다르게 지정된 USER 및 KEY를 이용한 SSH 보안 접근 사용
    - 각각 환경에 사용할 이미지를 따로 관리하면?
        - 이미지 관리를 각 환경에 맞게 따로 관리하는 것은 비효율적
        - 이미지의 용량을 고려해야 할 때 관리 부담이 커진다.
- 애플리케이션 Image는 동일하게 사용하고 필요한 환경 구성 값을 ConfigMap으로 만듦
- 기일이 요구되는(민감한) 데이터는 Secret 으로 생성해 Pod에 적용해서 사용
- 애플리케이션 레벨이 아닌 Object 레벨의 환경 설정 관리를 위해 필요
    - 애플리케이션 레벨에서 환경 구성 변경 시 소스코드와의 의존성이 높고. 설정 값 변경 시마다 코드 수정이 불가피하여 개발 속도가 지연될 수 있다.
    - 민첩성을 강조하는 CIoud Native 환경에서의 애플리케이션 개발은 애플리케이션 코드와 분리된 ConfigMap, Secret object 사용을 통해 설정값들을 신속하게 적용 가능
- ConfigMap, Secret은 중앙관리 방식으로 애플리케이션 코드에서 구성 및 민감한 정보를 분리하도록 설계된 Kubernetes API object → kubectl api-resources

### 환경 변수가 필요한 이미지를 사용한 pod 실행

- 사전에 빌드한 API_KEY를 필요로 하는 애플리케이션 이미지를 사용
    
    ```bash
    kubectl run myweb1 --image=sighingowl/mynode:2.0 --port 8000
    ```
    
- 실행할 떄 환경변수로 API_KEY를 주지 않아 API_KEY가 지정되지 않았다는 경고 메시지 출력
    
    ```bash
    curl 10.109.131.48:8000
    API_KEY is not set in the environment variables.
    ```
    
- API_KEY 정보를 가지는 configmap 생성
    
    ```bash
    kubectl create cm api-key --from-literal=API_KEY=k8spass#
    ```
    
- 생성한 configmap 확인
    
    ```bash
    kubectl get cm
    NAME               DATA   AGE
    api-key            1      6s
    ```
    
    ```bash
    kubectl describe cm api-key
    ..
    Data
    ====
    API_KEY:
    ----
    k8spass#
    ..
    ```
    
    - configmap은 인자를 별도의 암호화 없이 평문 그대로를 저장한다.
- configmap을 사용한 pod manifest
    
    ```bash
    kubectl run cmtest-pod --image=sighingowl/mynode:2.0 --port=8000 --dry-run=client -o yaml > cmtest-pod.yaml
    ```
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: cmtest-pod
      name: cmtest-pod
    spec:
      containers:
      - image: sighingowl/mynode:2.0
        name: cmtest-pod
        ports:
        - containerPort: 8000
    		envFrom:
        - configMapRef:
            name: api-key
    ```
    
    - 환경 변수 참조를 configmap에서 하는 것으로 설정
- Pod 접속 테스트
    
    ```bash
    curl 10.111.156.101:8000
    Welcome to fastcampus Kubernetes~! by kevin.
    ```
    
    - configmap을 통해 API_KEY를 전달 받아 실행하였을 때 정상적으로 접속이 가능
- CLI 명령으로 configmap 내용 변경
    
    ```bash
    kubectl edit configmaps api-key
    ```
    
    - configmap을 수정해도 바로 Pod에 적용되지 않는다.
    - pod를 강제 재시작을 수행 후 configmap의 내용이 pod에 적용된다.
        
        ```bash
        kubectl replace --force -f cmtest-pod.yaml
        kubectl exec -it cmtest-pod -- env | grep -i api
        API_KEY=k8s123#
        ```
        

## ConfigMap 활용
### ConfigMap

- configMap은 key-value 쌍으로 기밀이 아닌 일반적인 데이터를 저장하는 경우 사용
- 주로 Pod 내 컨테이너가 사용할 구성(환경 변수, 커맨드-라인 인수, 구성 파일 등)을 저장
- configMap에 데이터저장은 1MiB 이하로만 가능
- 개발과 운영에 사용되는 서로 다른 설정을 적용하기 위해 사용
- configMap, Secret을 Pod에 동시적용 가능
- 생성
    - 선언형 YAML코드 작성 : kubectl api-resources | grep configmap
    - 명령형 : kubectl create configmap(or cm) configmap_name {설정1 | 설정2 | 설정3}
        - 실정1 : --from-literal key=value [--from-literal key2=value2]..
        - 실정2 : --from-file file_name (file_name 파일 수에 다수의 설정 값 지장)
        - 실정3 : --from-env-file file_name (key=value의 환경 변수가 저장된 파일)
    - 변경할 수 없는 ConfigMap 생성
        - immutable: true 실정
        - 실행 중인 애플리케이션을 의도치 않은 중단 유발을 차단하고자 하는 경우
        - 불변 실정으로 ConfigMap에 대한 APIserver의 감시 중단, 클러스터 성능 향상에 기여
- Pod에 적용 방식
    - Pod 속 환경 변수로 등록
        - envFrom.configMapRef.name
    - Pod Volume으로 mount
        - volumes.configMap.name
        - mountPath : 해당 디렉터리의 전체를 업데이트 하는 방식
        - subPath : 해당 디렉터리 전체가 아닌 지정된 파일만 업데이트 하는 방식
        - mount된 configMap, Secret은 업데이트시 Pod에 자동 반영됨
            - mountPath로 등록된 configmap만 가능
            - subPath로 등록된 configmap은 불가능
    - Pod 속 컨테이너의 환경변수 값으로 KEY 지정
        - valueFrom.configMapRef.name
        - valueFrom.configMapRef.key

### 다중 환경 값 저장

- kubectl 명령으로 configmap 생성
    
    ```bash
    kubectl create configmap k8s-env \
    --from-literal orchestrator=kubernetes \
    --from-literal runtime=containerd
    ```
    
    - —dry-run 옵션을 사용하면 cm의 yaml 파일을 얻을 수 있다.
- 생성한 cm 확인
    
    ```bash
    kubectl describe cm k8s-env
    ..
    Data
    ====
    orchestrator:
    ----
    kubernetes
    runtime:
    ----
    containerd
    ..
    ```
    
    - 데이터가 평문으로 된 키-값 쌍으로 저장되어 있다.
    - kubectl get cm <cm-name> -o yaml을 사용하면 cm이 yaml 파일로 변환되어 키-값 쌍을 확인할 수 있다.
- cm을 사용하는 pod 생성
    
    ```yaml
                                                                                                                            1,1           All
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: cm-envfrom-pod
      name: cm-envfrom-pod
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: cm-envfrom-pod
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: k8s-env
    ```
    
- Pod에 configmap에 저장한 설정값이 제대로 들어갔는지 확인
    
    ```bash
    kubectl exec cm-envfrom-pod -- env | grep orchestrator
    orchestrator=kubernetes
    
    kubectl exec cm-envfrom-pod -- env | grep runtime
    runtime=containerd
    ```
    
- configmap immutable: true 설정
    
    ```bash
    kubectl edit cm k8s-env
    ```
    
    ```yaml
    apiVersion: v1
    data:
      orchestrator: kubernetes
      runtime: containerd
    immutable: true
    kind: ConfigMap
    metadata:
      creationTimestamp: "2024-02-15T07:57:04Z"
      name: k8s-env
      namespace: default
      resourceVersion: "241018"
      uid: 0b84b9b2-2952-4f61-beb6-a2e29be7cdc8
    ```
    

### 다중 환경 값을 볼륨으로 mount - —from-literal

- configmap을 볼륨에 마운트하면 configmap의 키-값 쌍이 마운트한 경로에 파일로 생성
- config 값을 파일로 적용할 때 사용
    - nginx.conf 파일 같은 것을 사용할 때 유용
- configmap을 볼륨으로 마운트했을 때 키-값을 수정하면 pod 재시작 없이 적용가능
- configmap을 볼륨에 마운트
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: cm-volume-pod
      name: cm-volume-pod
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: cm-volumbe-pod
        ports:
        - containerPort: 80
        volumeMounts:
        - name: cm-volume
          mountPath: /etc/config
      volumes:
      - name: cm-volume
        configMap:
          name: k8s-env
    ```
    
- Pod가 configmap을 볼륨으로 마운트 했는지 확인
    
    ```bash
    kubectl exec cm-volume-pod -- ls -al /etc/config
    total 0
    drwxrwxrwx    3 root     root            95 Feb 15 09:27 .
    drwxr-xr-x    1 root     root            52 Feb 15 09:27 ..
    drwxr-xr-x    2 root     root            41 Feb 15 09:27 ..2024_02_15_09_27_31.4280137625
    lrwxrwxrwx    1 root     root            32 Feb 15 09:27 ..data -> ..2024_02_15_09_27_31.4280137625
    lrwxrwxrwx    1 root     root            19 Feb 15 09:27 orchestrator -> ..data/orchestrator
    lrwxrwxrwx    1 root     root            14 Feb 15 09:27 runtime -> ..data/runtime
    ```
    
    - 날짜, 시간으로 이름이 구성된 디렉토리가 생성
    - ..data가 날짜, 시간으로 이름이 구성된 디렉토리의 심볼릭 링크로 생성
    - configmap의 키가 링크로 생성되었고 이 링크는 configmap의 값이 저장된 경로로 링크되어 있다.
        
        ```bash
        kubectl exec cm-volume-pod -- cat /etc/config/orchestrator
        kubernetess
        kubectl exec cm-volume-pod -- cat /etc/config/runtime
        containerd
        ```
        
- configmap 수정 후 pod 적용 확인
    
    ```bash
    # apiVersion: v1
    # data:
      # orchestrator: k8s
      # runtime: containerd
    ..
    
    kubectl exec cm-volume-pod -- cat /etc/config/orchestrator
    k8s
    ```
    
    - orchestrator를 수정했을 때 pod에 동일하게 적용된다.

### 다중 환경 값을 볼륨으로  mount - —from-file

- redis의 구성 파일을 mount
    
    ```bash
    REDIS_PRIMARY_SERVICE_HOST=10.0.0.11
    REDIS_PRIMARY_SERVICE_PORT=6379
    REDIS_PRIMARY_PORT=tcp://10.0.0.11:6379
    REDIS_PRIMARY_PORT_6379_TCP=tcp://10.0.0.11:6379
    REDIS_PRIMARY_PORT_6379_TCP_PROTO=tcp
    REDIS_PRIMARY_PORT_6379_TCP_PORT=6379
    REDIS_PRIMARY_PORT_6379_TCP_ADDR=10.0.0.11
    ```
    
    ```bash
    kubectl create cm redis-config --from-file=redis.conf
    ```
    
- redis configmap 확인
    
    ```bash
    kubectl describe cm redis-config
    ..
    Data
    ====
    redis.conf:
    ----
    REDIS_PRIMARY_SERVICE_HOST=10.0.0.11
    REDIS_PRIMARY_SERVICE_PORT=6379
    REDIS_PRIMARY_PORT=tcp://10.0.0.11:6379
    REDIS_PRIMARY_PORT_6379_TCP=tcp://10.0.0.11:6379
    REDIS_PRIMARY_PORT_6379_TCP_PROTO=tcp
    REDIS_PRIMARY_PORT_6379_TCP_PORT=6379
    REDIS_PRIMARY_PORT_6379_TCP_ADDR=10.0.0.11
    ..
    ```
    
- redis config를 사용하는 pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: cmfile-volume-pod
      name: cmfile-volume-pod
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: cmfile-volumbe-pod
        ports:
        - containerPort: 80
        volumeMounts:
        - name: cmfile-volume
          mountPath: /opt/redis/config
      volumes:
      - name: redis-volume
        configMap:
          name: redis-config
    ```
    
- 생성된 Pod에서 redis.conf 확인
    
    ```bash
    ubectl exec cmfile-volume-pod -- cat /opt/redis/config/redis.conf
    REDIS_PRIMARY_SERVICE_HOST=10.0.0.11
    REDIS_PRIMARY_SERVICE_PORT=6379
    REDIS_PRIMARY_PORT=tcp://10.0.0.11:6379
    REDIS_PRIMARY_PORT_6379_TCP=tcp://10.0.0.11:6379
    REDIS_PRIMARY_PORT_6379_TCP_PROTO=tcp
    REDIS_PRIMARY_PORT_6379_TCP_PORT=6379
    REDIS_PRIMARY_PORT_6379_TCP_ADDR=10.0.0.11
    ```
    

### 환경 값을 파일로 저장  - —from-env-file

- 환경 값을 가져와서 configmap 생성
    - kubernetes 예제 속성 파일을 다운받아 configmap으로 생성
    
    ```bash
    kubectl create configmap game-config --from-file=./game-config/
    
    kubectl describe cm game-config
    ..
    Data
    ====
    game.properties:
    ----
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
    ui.properties:
    ----
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
    ```
    
- 환경 값 파일을 환경 변수로 등록 - —from-env-file
    
    ```bash
    kubectl create configmap game-confige-env-file --from-env-file=game-config/game-env-file.properties
    
    kubectl describe cm game-confige-env-file
    ..
    Data
    ====
    allowed:
    ----
    "true"
    enemies:
    ----
    aliens
    lives:
    ----
    3
    ```
    
- 환경 변수 파일을 configmap으로 하는 pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: cmfile-volume-pod2
      name: cmfile-volume-pod2
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: cmfile-volumbe-pod2
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: game-config-env-file
    ```
    

### Nginx의 index.html을 configmap으로 생성, Pod에 제공

- index.html 자체를 configmap으로 생성
    
    ```bash
    kubectl create cm index-cm --from-file index.html
    ```
    
- cm 확인
    
    ```bash
    kubectl get cm index-cm -o yaml
    apiVersion: v1
    data:
      index.html: |
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
    kind: ConfigMap
    metadata:
      creationTimestamp: "2024-02-15T10:03:01Z"
      name: index-cm
      namespace: default
      resourceVersion: "261735"
      uid: e945823c-04e3-406a-96c2-6021231f93bf
    ```
    
- index.html을 받을 Pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: myweb-cm--pod
      name: myweb-cm-pod
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: myweb-cm-pod
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-index-file
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-index-file
        configMap:
          name: index-cm
    ```
    
- pod의 index.html 확인
    
    ```bash
    curl 10.111.218.96
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
    
    - pod의 index.html이 nginx의 기본 index.html이 아닌 configmap으로 넘겨준 index.html이다.

## Pod 설계 패턴 4 - Ambassador
### Ambassador Pod design pattern

- Ambassador는 Pod내에 Proxy 역할을 하는 컨테이너를 추가하는 패턴
- Pod내에서 외부 서버에 접근하거나 외부에서 Pod 애플리케이션에 접근 할 때, Pod 내부의 Ambassador 컨테이너(proxy)에 접근하도록 설정하고, 실제로 외부와의 연결은 Proxy에서 처리하는 방식
- 외부 서비스에 대한 균일한 인터페이스를 제공하여 외부 서비스를 더 쉽게 관리하고 확장하는 데 사용
- Ambassador 컨테이너는 기본 애플리케이션 컨테이너를 대신하여 네트워킹, 인증, 로깅, 모니터링과 같은 특정 교차 문제를 처리하는 일에 적합

### 외부에서 Application container에 요청을 전달하는 Ambassador

- nginx Pod의 nginx.conf를 수정하여 reverse proxy 구성으로 변경
    - nginx.conf
        
        ```bash
        events {
          worker_connections	1024;
        }
        http {
          upstream flaskapp {
            server localhost:9000;
        	}
        
          server {
            listen 80;
        
            location {
              / {
              proxy_pass	http://flaskapp;
              proxy_redirect	off;
            }
          }
        }
        ```
        
    - nginx.conf를 configmap으로 생성
- ambassador 패턴을 적용한 pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: flaskapp-pod
      name: flaskapp
    spec:
      containers:
      - image: nginx:1.25.3-alpine
        name: proxy-container
        volumeMounts:
        - name: nginx-proxy-config
          mountPath: /etc/nginx
        ports:
        - containerPort: 80
          protocol: TCP
      - image: dbgurum/py_flask:2.0
        name: flaskapp-container
        ports:
        - containerPort: 9000
          protocol: TCP
      volumes:
      - name: nginx-proxy-config
        configMap:
          name: nginx-conf
    ```
    
    - proxy-container는 nginx를 구동하고 nginx.conf를 configmap에서 가져와 사용한다.
    - nginx container가 80번 포트로 외부에서 요청을 받으면 9000번 포트로 flaskapp-container로 전달
- 외부 노출을 위한 svc 생성
    
    ```bash
    kubectl expose po flaskapp --name=flaskapp-svc --type=NodePort --port=80 --target-port=80
    ```
    
- 파드, 서비스 확인
    
    ```bash
    kubectl get pod,svc -o wide | grep flask
    pod/flaskapp   2/2     Running   0          2m8s   10.109.131.56   k8s-node2   <none>           <none>
    service/flaskapp-svc   NodePort    10.97.134.43   <none>        80:31539/TCP   9s      run=flaskapp-pod
    ```
    
- 외부에서 proxy container를 사용해서 트래픽을 전달하는 기법

## Secret 활용
### Secret

- Secret 은 비밀번호, OAuth 토큰, API 키, TLS 인증서, ssh 키와 같은 민감한 CredentiaI이 요구되는 데이터를 base64로 인코딩하여 저장, 관리한다.
    - 암호화를 하는 것은 아니다.
    - github에 secret을 올리게되면 누구든지 디코딩해서 사용할 수 있어 올리지 않도록 주의해야 한다.
- Secret 에 데이터 저장은 1MiB 이하로만 가능
- 사용 방법은 ConfigMap과 유사
- Pod에 Secret 적용시 자동 Decoding 되어 애플리케이션에서 바로 사용 가능
- 사용 가능한 비밀
    - docker-registry : Docker 레지스트리에 사용할 시크릿
    - generic : 로컬 파일, 디렉토리 또는 리터럴 값에서 Secret을 생성
    - tls : TLS Secret을 생성

### 암호 저장용 Secret 만들기

- generic type 시크릿 생성
    
    ```bash
    kubectl create secret generic my-pwd --from-literal=mypwd=k8spass#
    ```
    
- secret 확인
    
    ```bash
    kubectl get secrets my-pwd -o yaml
    apiVersion: v1
    data:
      mypwd: azhzcGFzcyM=
    kind: Secret
    metadata:
      creationTimestamp: "2024-02-15T12:20:30Z"
      name: my-pwd
      namespace: default
      resourceVersion: "284533"
      uid: fac07a1a-682d-4d96-b909-734c9a5d84ad
    type: Opaque
    ```
    
    - mypwd를 키로 가지고 인코딩된 값을 가진 비밀을 확인 가능
- secret을 참조하는 Pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: secret-pod
    spec:
      containers:
      - name: my-container
        image: busybox
        args: ['tail', '-f', '/dev/null']
        envFrom:
        - secretRef:
            name: my-pwd
    ```
    
- 생성한 pod에서 secret 확인
    
    ```bash
    kubectl exec secret-pod -- env | grep -i mypwd
    mypwd=k8spass#
    ```
    
    - Pod의 환경 변수에서 secret을 확인 가능

### 웹에 DB 관련 Secret 제공

- secret을 환경 변수로 등록
    
    ```bash
    kubectl create secret generic web-db-secret \
    --from-literal=rootpw=k8spwd \
    --from-literal=database=fastcampusdb \
    --from-literal=user=k8suser \
    --from-literal=password=k8spass#
    ```
    
    - db secret을 사용하는 pod 생성
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: web-db-secret-pod1
        spec:
          containers:
          - name: my-container
            image: busybox
            args: ['tail', '-f', '/dev/null']
            envFrom:
            - secretRef:
                name: web-db-secret
        ```
        
    - Pod에서 환경 변수로 등록한 Secret 확인
        
        ```bash
        kubectl exec -it web-db-secret-pod1 -- env | grep rootpw
        rootpw=k8spwd
        ```
        
- secret을 volume으로 마운트
    - Secret을 볼륨으로 마운트하는 pod 생성
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: web-db-secret-pod2
        spec:
          containers:
          - name: my-container
            image: busybox
            args: ['tail', '-f', '/dev/null']
            volumeMounts:
            - name: web-db-sec
              mountPath: /secrets
          volumes:
          - name: web-db-sec
            secret:
              secretName: web-db-secret
        ```
        
    - Pod에서 볼륨으로 마운트한 secret 확인
        
        ```bash
        kubectl exec -it web-db-secret-pod2 -- ls -al /secrets
        total 0
        drwxrwxrwt    3 root     root           160 Feb 15 12:37 .
        drwxr-xr-x    1 root     root            66 Feb 15 12:37 ..
        drwxr-xr-x    2 root     root           120 Feb 15 12:37 ..2024_02_15_12_37_37.1530931528
        lrwxrwxrwx    1 root     root            32 Feb 15 12:37 ..data -> ..2024_02_15_12_37_37.1530931528
        lrwxrwxrwx    1 root     root            15 Feb 15 12:37 database -> ..data/database
        lrwxrwxrwx    1 root     root            15 Feb 15 12:37 password -> ..data/password
        lrwxrwxrwx    1 root     root            13 Feb 15 12:37 rootpw -> ..data/rootpw
        lrwxrwxrwx    1 root     root            11 Feb 15 12:37 user -> ..data/user
        ```
        
        - pod의 /secrets 디렉토리에 비밀 값과 링크되어 있는 키들을 확인 가능
- Secret을 특정 환경변수만 전달 - env.valueFrom.secretkeyRef
    - 특정 환경변수만 전달하는 pod 생성
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: web-db-secret-pod3
        spec:
          containers:
          - name: my-container
            image: busybox
            args: ['tail', '-f', '/dev/null']
            env:
            - name: ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: web-db-secret
                  key: rootpw
            - name: DATABASE_NAME
              valueFrom:
                secretKeyRef:
                  name: web-db-secret
                  key: database
        ```
        
    - 특정 환경 변수만 전달 받는 pod 생성 후 환경 변수 확인
        
        ```bash
        kubectl exec -it web-db-secret-pod3 -- env | grep ROOT_PASSWORD
        ROOT_PASSWORD=k8spwd
        
        kubectl exec -it web-db-secret-pod3 -- env | grep DATABASE_NAME
        DATABASE_NAME=fastcampusdb
        ```
        
        - 특정한 환경 변수만 전달 된 것을 확인

### ConfigMap과 Secret 동시에 Pod에 적용

- ConfigMap과 Secret을 동시에 참조하는 Pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: conf-sec-pod1
    spec:
      containers:
      - name: my-container
        image: busybox
        args: ['tail', '-f', '/dev/null']
        envFrom:
        - configMapRef:
            name: k8s-env
        - secretRef:
            name: web-db-secret
    ```
    
- 생성한 Pod에서 환경 변수 확인
    
    ```bash
    kubectl exec -it conf-sec-pod1 -- env
    ..
    database=fastcampusdb
    password=k8spass#
    rootpw=k8spwd
    user=k8suser
    runtime=containerd
    orchestrator=kubernetes
    ..
    ```
    
    - ConfigMap과 Secret에 포함된 환경 변수를 확인 가능

### 파일이 키가 되고 파일 내용이 값이 되는 ConfigMap, Secret 동시 적용

- ConfigMap, Secret 생성
    
    ```bash
    kubectl create configmap k8s-env-cm --from-file=./k8s-env-cm.txt
    kubectl create secret generic web-db-sec --from-file=web-db-sec.txt
    ```
    
- Pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: conf-sec-pod2
    spec:
      containers:
      - name: my-container
        image: busybox
        args: ['tail', '-f', '/dev/null']
        env:
        - name: K8S_ENV
          valueFrom:
            configMapKeyRef:
              name: k8s-env-cm
              key: k8s-env-cm.txt
        - name: WEB_DB_CONN
          valueFrom:
            secretKeyRef:
              name: web-db-sec
              key: web-db-sec.txt
    ```
    

## Secret 고급 활용
### OpenSSL활용, Https를 위한 Secret

- ConfigMap, Secret 생성
    - 자격증명 생성
        
        ```bash
        openssl genrsa -out https.key 2048
        openssl req -new -x509 -key https.key -out https.cert -days 360 -subj /CN=*.k8s.io
        ```
        
        - 자격 증명을 포함하는 generic Secret 생성 - TLS를 생성할 때는 tls secret을 만들어야 하지만 연습이라서 generic으로 생성
            
            ```bash
            kubectl create secret generic k8s-https --from-file=https.key --from-file=https.cert
            ```
            
        - Secret 확인
            
            ```bash
            kubectl describe secrets k8s-https
            ..
            Data
            ====
            https.cert:  1111 bytes
            https.key:   1704 bytes
            ```
            
    - SSL용 nginx.conf 생성
        - cust-nginx-config.conf
            
            ```bash
            server {
                    listen                  8080;
                    listen                  443 ssl;
                    server_name             www.k8s.com;
                    ssl_certificate         certs/https.cert;
                    ssl_certificate_key     certs/https.key;
                    ssl_protocols           TLSv1 TLSv1.1 TLSv1.2;
                    ssl_ciphers             HIGH:!aNULL:!MD5;
                    gzip on;
                    gzip_types text/plain application/xml;
                    location / {
                            root    /usr/share/nginx/html;
                            index   index.html index.htm;
                    }
            }
            ```
            
        - sleep-interval
            
            ```bash
            vi sleep-interval
            5
            ```
            
    - 설정 값으로 ConfigMap 생성 및 확인
        
        ```bash
        kubectl create cm k8s-config --from-file=./config
        kubectl describe cm k8s-config
        ..
        Data
        ====
        cust-nginx-config.conf:
        ----
        server {
          listen                   8080;
          listen                   443 ssl;
          server_name            www.k8s.com;
          ssl_certificate        certs/https.cert;
          ssl_certificate_key  certs/https.key;
          ssl_protocols          TLSv1 TLSv1.1 TLSv1.2;
          ssl_ciphers            HIGH:!aNULL:!MD5;
          gzip on;
          gzip_types text/plain application/xml;
          location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
          }
        }
        
        sleep-interval:
        ----
        5
        ..
        ```
        
- 생성한 ConfigMap, Secret을 사용해 OpenSSL 연결을 생성하는 Pod 생성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: k8s-https
    spec:
      containers:
      - image: dbgurum/k8s-lab:env
        env:
        - name: INTERVAL
          valueFrom:
            configMapKeyRef:
              name: k8s-config
              key: sleep-interval
        name: html-generator
        volumeMounts:
        - name: html
          mountPath: /var/htdocs
      - image: nginx:alpine
        name: web-server
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
          readOnly: true
        - name: config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        - name: certs
          mountPath: /etc/nginx/certs/
          readOnly: true
        ports:
        - containerPort: 80
        - containerPort: 443
      volumes:
      - name: html
        emptyDir: {}
      - name: config
        configMap:
          name: k8s-config
          items:
          - key: cust-nginx-config.conf
            path: https.conf
      - name: certs
        secret:
          secretName: k8s-https
    ```
    
- 포트 포워드 테스트
    - 443 포트 서비스 게시 - 1번 터미널
        
        ```bash
        kubectl port-forward k8s-https 8443:443 &
        410701
        
        Forwarding from 127.0.0.1:8443 -> 443
        Forwarding from [::1]:8443 -> 443
        Handling connection for 8443
        ```
        
        - 연결을 시도할 때마다 Handling connection for 443 메시지가 출력된다.
        - Https 포트인 443 포트에서 8443 포트로 포트 포워딩을 설정
    - https 접속 수행 - 2번 터미널 → 1번 터미널과 같은 노드에 접속
        
        ```bash
        curl https://localhost:8443 -k -v
        *   Trying 127.0.0.1:8443...
        * Connected to localhost (127.0.0.1) port 8443 (#0)
        * ALPN, offering h2
        * ALPN, offering http/1.1
        * TLSv1.0 (OUT), TLS header, Certificate Status (22):
        * TLSv1.3 (OUT), TLS handshake, Client hello (1):
        * TLSv1.2 (IN), TLS header, Certificate Status (22):
        ..
        < Accept-Ranges: bytes
        <
        Be free and open and breezy!  Enjoy!  Things won't get any better so
        get used to it.
        * Connection #0 to host localhost left intact
        ```
        
        - curl을 사용하면 https 프로토콜을 사용해서 접속을 수행

### TLS 활용, Ingress service에 TLS Secret 연결

- TLS 연결을 위해 *.crt와 *.key를 생성한다.
- 이 값으로 Secret을 생성하여 lngress에 적용한다.
- 각 연결은 curl -k [https://](https://nn/)~~를 통해 접속한다.
- phpServer와 goapp을 배포하는 deployment 배포
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: tlsapp-php
      labels:
        app: php
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: php
      template:
        metadata:
          labels:
            app: php
        spec:
          containers:
          - name: phpserver-container
            image: dbgurum/phpserver:2.0
            ports:
            - containerPort: 80
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: tlsapp-go
      labels:
        app: go
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: go
      template:
        metadata:
          labels:
            app: go
        spec:
          containers:
          - name: go-container
            image: dbgurum/goapp:1.0
            ports:
            - containerPort: 9090
    ```
    
    - deployment를 노출할 svc 배포
        
        ```bash
        kubectl expose deployment tlsapp-php --name=tlsapp-php-svc --port=80 --target-port=80
        kubectl expose deployment tlsapp-go --name=tlsapp-go-svc --port=80 --target-port=9090
        ```
        
- Ingress service에 연결, OpenSSL을 이용한 자체 서명 인증서 생성
    - OpenSSL 인증서 생성
        
        ```bash
        openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out server.crt -keyout server.key -subj "/CN=php.k8s.io,goapp.k8s.io"
        ```
        
    - 인증서를 포함하는 tls Secret 생성 및 확인
        
        ```bash
        kubectl create secret tls k8s-tls-secret --cert=server.crt --key=server.key
        
        kubectl describe secrets k8s-tls-secret
        Name:         k8s-tls-secret
        Namespace:    default
        Labels:       <none>
        Annotations:  <none>
        
        Type:  kubernetes.io/tls
        
        Data
        ====
        tls.crt:  1151 bytes
        tls.key:  1704 bytes
        ```
        
        - tls 옵션으로 Secret을 생성할 때는 —cert와 —key를 반드시 입력해야한다.
    - Ingress 생성
        
        ```yaml
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: app-tls-ingress
          annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
            nginx.ingress.kubernetes.io/ssl-redirect: "true"
        spec:
          ingressClassName: nginx
          tls:
          - hosts:
            - php.k8s.io
            - goapp.k8s.io
            secretName: k8s-tls-secret
          rules:
            - host: php.k8s.io
              http:
                paths:
                - path: /
                  pathType: Prefix
                  backend:
                    service:
                      name: tlsapp-php-svc
                      port:
                        number: 80
            - host: goapp.k8s.io
              http:
                paths:
                - path: /
                  pathType: Prefix
                  backend:
                    service:
                      name: tlsapp-go-svc
                      port:
                        number: 80
        ```
        
        - nginx.ingress.kubernetes.io/ssl-redirect: "true"
            - https로 접속하면 http로 트래픽이 전달되는 것을 확인하기 위해 사용
        - 생성된 ingress 확인
            
            ```bash
            kubectl get ing
            NAME              CLASS   HOSTS                     ADDRESS          PORTS     AGE
            app-tls-ingress   nginx   php.k8s.io,goapp.k8s.io   192.168.56.102   80, 443   33s
            
            kubectl describe ing app-tls-ingress
            Name:             app-tls-ingress
            Labels:           <none>
            Namespace:        default
            Address:          192.168.56.102
            Ingress Class:    nginx
            Default backend:  <default>
            TLS:
              k8s-tls-secret terminates php.k8s.io,goapp.k8s.io
            Rules:
              Host          Path  Backends
              ----          ----  --------
              php.k8s.io
                            /   tlsapp-php-svc:80 (10.109.131.58:80,10.111.156.106:80,10.111.218.98:80)
              goapp.k8s.io
                            /   tlsapp-go-svc:80 (10.109.131.59:9090,10.111.156.107:9090,10.111.218.99:9090)
            Annotations:    nginx.ingress.kubernetes.io/rewrite-target: /
                            nginx.ingress.kubernetes.io/ssl-redirect: true
            Events:
              Type    Reason  Age                  From                      Message
              ----    ------  ----                 ----                      -------
              Normal  Sync    114s (x2 over 118s)  nginx-ingress-controller  Scheduled for sync
            ```
            
- curl을 사용해서 Https 접속 테스트
    
    ```bash
    curl https://goapp.k8s.io:32701/ -k --resolve goapp.k8s.io:32701:127.0.0.1
    curl https://php.k8s.io:32701/ -k --resolve php.k8s.io:32701:127.0.0.1
    ```
    

### SealedSecret을 통한 Secret 암호화

#### SealedSecret

- GitOps 사용시 Git에 저장되는 Secret 구성값(Data)은 단순base64 인코딩 값이므로 쉽게 디코딩 가능하다. 민감한 데이터의 노출될 위험이 높다.
- SeaIedSecret은 사용 중인 클러스터에서 실행되는 ControIIer에 의해서만 암호화/복호화가 가능하다. 따라서, 권한의 문제가 아닌 ControIIer가 있는 해당 클러스터에서만 해독 가능하다
- 동작 방식
    - 사용자는 SeaIedSecret으로 공개키 암호화 방식의 Secret 암호화 수행
    - SeaIedSecret Controller는 Kubernetes Secret으로 복호화 수행
    - 이 작업들은 SealedSecret ControIIer내에 저장된 인증서(Privatekey)를 통해 처리
    - 예기치 않은 상황에 대비해 이 내장된 인증서를 로컬로 저장해서 사용 가능하다.
        - Controller에 장애가 생길 경우 복호화가 되지 않는 심각한 문제가 발생한다.
- SealedSecret과 Secret은 동일한 네임스페이스에 동일한 이름을 가져야 한다.
- 동일한 이름의 sealedsecret으로 새로운 값을 업데이트가 가능하다.

#### SealedSecret 생성

- SealedSecret 설치
    - kubeseal, controller 설치
        
        ```bash
        KUBESEAL_VERSION='0.24.5'
        wget "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION:?}/kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz"
        tar -xvzf kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz kubeseal
        sudo install -m 755 kubeseal /usr/local/bin/kubeseal
        ```
        
    - SealedSecret controller 설치
        
        ```bash
        kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.5/controller.yaml
        ```
        
        - controller 실행 확인
            
            ```bash
            kubectl get pods -n kube-system | grep seal
            sealed-secrets-controller-7d9d5cf989-c6qkx   1/1     Running   0             34s
            ```
            
- SealedSecret controller가 보유한 인증서 사용
    - Secret 생성
        
        ```bash
        kubectl create secret generic secret-from-sealedsecret --dry-run=client --from-literal=superpwd=k8spass#111 -o yaml > mysecret-1.yaml
        ```
        
        - 이 상태는 인코딩만 되어있는 상태이다.
    - kubeseal을 사용해서 secret 암호화
        
        ```bash
        cat mysecret-1.yaml | kubeseal -o yaml > mysecret-sealed-1.yaml
        ```
        
        ```bash
        apiVersion: bitnami.com/v1alpha1
        kind: SealedSecret
        metadata:
          creationTimestamp: null
          name: secret-from-sealedsecret
          namespace: default
        spec:
          encryptedData:
            superpwd: AgAKhmw/+gO7LNtuc/nQxQecYBYw09OBUj21/h36gTUOspW4aqp0AGgCRqoAUgC4OXaCOw/uYeLhFl/r1hbItK3bNjOSLSPgjvE4EVVWL04njk6GJdpsjvup0uREEDmiJwmKC9lQ4Fy44SFG6Iw9eraxayLDFmHboZVFrkpHUNGaNHJH9YPordFvgXwr46MREQKRDtcbr6TBwADhfQulQpSHfy7i9DvCcfe0I7pXOINPU+b8GPI47WQhl4bNeeQtyydXmBSl4KVCLclqZ1cR7i7HHlhcSXDzGIeHXvn3zvMjLUqhmGxc+G8HkPKqH5+IPyZbfzi0q8TJ5JX3MG7YgSXjMEFL88rLWrxh9NWGH8bc/VRbeaRf7cqiCDHOVAke1sG3b/MUZCzfWuFlPwPCYFTNv7gKwSe/1rk02tIaC4rxviB24vdvCQJrSPnKATuDRBMutS5oLQq3v48/fUYZXOVNJkEiQg1tmuKeJzBwTouvaXp3c74VCRiRFdzBvvp0MeWmRVhmURGGMEELpwsAV+hm9K9UkpzDq82V1onLmPdlwQMx7tpHVhezgm/3PVpxtQnfnJJwnNMEHGdQx+hYhOcjNgjOBcy2M9hfPWtsn4G1ioScgkUp13m0MX9P21pLvP15G9sHWdAigccqBT9PQ+kSpO1X36dorMlPmdDyXUWxjsgcradcgvXUoyP9mKjc3RVXH/BXE9z9qdKwrg==
          template:
            metadata:
              creationTimestamp: null
              name: secret-from-sealedsecret
              namespace: default
        ```
        
        - secret에 인코딩된 값이 암호화된 값으로 바뀌었다.
    - 생성된 sealedsecret 확인
        
        ```bash
        kubectl get sealedsecret
        NAME                       AGE
        secret-from-sealedsecret   8s
        
        kubectl get secrets
        NAME                       TYPE                DATA   AGE
        ..
        secret-from-sealedsecret   Opaque              1      34s
        ..
        ```
        
        - sealedsecret도 secret이므로 secret으로 확인해도 조회가 된다.
        - 암호화된 값 복호화
            
            ```bash
            kubectl get secret secret-from-sealedsecret -o jsonpath='{.data.superpwd}' | base64 -d
            k8spass#111
            ```
            
- controller가 보유한 인증서 저장하기
    
    ```bash
    kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system --fetch-cert > mycert.pem
    ```
    
    - 예기치 못한 상황을 대비해 인증서를 로컬에 저장할 때 사용
    - Local에 저장한 인증서로 SealedSecret 생성
        
        ```bash
        kubectl create secret generic local-sealedsecret --dry-run=client --from-literal=superpwd=k8spass#333 -o yaml | kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system --format yaml --cert mycert.pem > mysecret-sealed-3.yaml
        ```
        
    - sealedsecret 배포 후 복호화
        
        ```bash
        kubectl apply -f mysecret-sealed-3.yaml
        
        kubectl get secret local-sealedsecret -o jsonpath='{.data.superpwd}' | base64 -d
        k8spass#333
        ```
        
- 주기적인 암호화 키 교체 - —re-encrypt
    
    ```bash
    kubeseal --re-encrypt < 'sealedsecret manifest' > tmp.yaml && mv tmp.yaml
    ```
    

## etcd 암호화로 Secret 암호화하기
### etcd encryption

- etcd에 저장되는 모든 API 리소스들은 암호화 기능을 지원 (기본은 평문)
- Secret에 저장된 인코딩 값은 etcd를 들여다보면 평문(plaintext)으로 확인
- etcd 암호화를 위한 설정
    - Encryption Configuration 리소스 생성
        
        [kube-apiserver Encryption Configuration (v1)](https://kubernetes.io/docs/reference/config-api/apiserver-encryption.v1/)
        
        - resources: 암호화를 원하는 리소스(object)타입 지정
        - provider: 암호화 방식을 정의하여 해당 방식으로 암호화 됨. ldentity: {}는 암호화 지원 안함.
    - kube-apiserver 에 암호화기능활성화
        - --encryption-provider-config 추가 → EncryptionConfiguration 리소스경로와 파일명
        - 활성화 시 재시작 시간 필요

### etcd 암호화 환경 구성, etcd 관리를 위한 etcdctl 설치

- etcdctl 다운로드 및 설치, 실행 테스트
    
    ```bash
    export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest|grep tag_name | cut -d '"' -f 4)
    wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz
    
    tar xvzf etcd-v3.5.12-linux-amd64.tar.gz
    cd etcd-v3.5.12-linux-amd64/
    sudo mv etcd etcdctl etcdutl /usr/local/bin
    
    etcd --version
    etcd Version: 3.5.12
    Git SHA: e7b3bb6cc
    Go Version: go1.20.13
    Go OS/Arch: linux/amd64
    
    ps -aux | grep kube-api | grep "encryption-provider-config"
    # 아직 활성화 되지 않아서 아무 결과가 뜨지 않음
    ```
    
- secret 생성
    
    ```bash
    kubectl create secret generic etcd-secret --from-literal=mypwd=k8spass#
    sudo ECTDCTL_API=3 etcdctl --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --key /etc/kubernetes/pki/apiserver-etcd-client.key --cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/default/etcd-secret | hexdump -C
    00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
    00000010  73 2f 64 65 66 61 75 6c  74 2f 65 74 63 64 2d 73  |s/default/etcd-s|
    00000020  65 63 72 65 74 0a 6b 38  73 00 0a 0c 0a 02 76 31  |ecret.k8s.....v1|
    00000030  12 06 53 65 63 72 65 74  12 d1 01 0a b3 01 0a 0b  |..Secret........|
    00000040  65 74 63 64 2d 73 65 63  72 65 74 12 00 1a 07 64  |etcd-secret....d|
    00000050  65 66 61 75 6c 74 22 00  2a 24 38 33 39 64 34 31  |efault".*$839d41|
    00000060  64 32 2d 65 38 32 31 2d  34 38 61 31 2d 61 65 61  |d2-e821-48a1-aea|
    00000070  31 2d 64 38 66 30 30 38  38 32 62 38 66 37 32 00  |1-d8f00882b8f72.|
    00000080  38 00 42 08 08 b5 a3 b9  ae 06 10 00 8a 01 62 0a  |8.B...........b.|
    00000090  0e 6b 75 62 65 63 74 6c  2d 63 72 65 61 74 65 12  |.kubectl-create.|
    000000a0  06 55 70 64 61 74 65 1a  02 76 31 22 08 08 b5 a3  |.Update..v1"....|
    000000b0  b9 ae 06 10 00 32 08 46  69 65 6c 64 73 56 31 3a  |.....2.FieldsV1:|
    000000c0  2e 0a 2c 7b 22 66 3a 64  61 74 61 22 3a 7b 22 2e  |..,{"f:data":{".|
    000000d0  22 3a 7b 7d 2c 22 66 3a  6d 79 70 77 64 22 3a 7b  |":{},"f:mypwd":{|
    000000e0  7d 7d 2c 22 66 3a 74 79  70 65 22 3a 7b 7d 7d 42  |}},"f:type":{}}B|
    000000f0  00 12 11 0a 05 6d 79 70  77 64 12 08 6b 38 73 70  |.....**mypwd..k8sp**|
    00000100  61 73 73 23 1a 06 4f 70  61 71 75 65 1a 00 22 00  |**ass#**..Opaque..".|
    00000110  0a                                                |.|
    00000111
    ```
    
    - etcdctl을 통해서 etcd에 있는 값을 직접 확인 가능
    - etcd를 확인하면 방금 생성한 secret의 키값이 암호화되지 않은 상태로 저장되어 있다.
- 암호화 기능 사용을 위해 /etc/kubernetes/pki/에 암호화 기능을 할 yaml 파일 생성
    
    ```bash
    head -c 32 /dev/urandom | base64  # 32byte의 랜덤 문자열 출력, 암호화키 생성
    x66/FS7cpglkqdUsBNFVggIOQXHxmGwYOF9Oe/HvL4M=
    ```
    
    ```yaml
    kind: EncryptionConfiguration
    apiVersion: apiserver.config.k8s.io/v1
    resources:
      - resources:
        - secrets
        - configmaps
        providers:
        - secretbox:
            keys:
            - name: key1
              secret: x66/FS7cpglkqdUsBNFVggIOQXHxmGwYOF9Oe/HvL4M=
        - identity: {}
    ```
    
    - Encryption provider는 여러 종류가 있다.
        
        
        | Provider
        Name | 암호화
        방식 | 암호화
        강도 | 암호화 속도 | Key
        길이 | 설명 |
        | --- | --- | --- | --- | --- | --- |
        | identity | x | N/A | N/A | N/A | 암호화를 하지 않을 때 사용된다.
        identity가 첫 번째 프로바이더로 설정되면 새 값이 기록될 떄 암호화된 리소스가 복호화. |
        | aescbc | AES-CBC | Weak | Fast | 32 byte | CBC는 패딩 오라클 공격에 취약 |
        | aesgcn | AES-GCM | 20만 번 읽을 때마다 교체 필요 | Fastest | 16, 24, 32
        byte | 자동 키 회전 방식이 구현된 경우를 제외하고는 비 권장 |
        | kms v1 | DEK | Strongest | Slow | 32 byte | 별도의 KMS plugin gRPC server가 필요하다 |
        | kms v2 | DEK | Strongest | Fast | 32 byte | 별도의 KMS plugin gRPC server가
        필요하다 |
        | secretbox
        (1.27 이상) | XSalsa20
        and
        Poly1305 | Strong | Fastest | 32 byte | 일반 수준의 암호화 수준 지원, latest 암호화 기술 임 |
- 암호화 기능이 apiserver에서 동작할 수 있도록 /etc/kubernetes/manifest/kube-apiserver.yaml에 관련 내용을 추가한다.
    
    ```yaml
    ...
    spec:
      containers:
      - command:
        - kube-apiserver
        - --encryption-provider-config=/etc/kubernetes/pki/encryption.yaml
    ...
    ```
    
    - 변경 감지를 통해 재시작하는데 1분 정도 필요
    - apiserver 재시작 후 기능 활성화 확인
        
        ```bash
        ps -aux | grep kube-api | grep "encryption-provider-config"
        root      611531 32.4  7.9 1038068 316288 ?      Ssl  03:21   0:15 kube-apiserver --encryption-provider-config=/etc/kubernetes/pki/encryption.yaml
        ..
        ```
        
        - encryption-provider-config이 기능이 포함된 것을 확인
- secret을 생성 후 암호화 확인
    
    ```bash
    kubectl create secret generic etcd-secret --from-literal=mypwd=k8spass#
    sudo ECTDCTL_API=3 etcdctl --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --key /etc/kubernetes/pki/apiserver-etcd-client.key --cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/default/etcd-secret | hexdump -C
    00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
    00000010  73 2f 64 65 66 61 75 6c  74 2f 65 74 63 64 2d 73  |s/default/etcd-s|
    00000020  65 63 72 65 74 0a 6b 38  73 3a 65 6e 63 3a 73 65  |ecret.k8s:enc:se|
    00000030  63 72 65 74 62 6f 78 3a  76 31 3a 6b 65 79 31 3a  |cretbox:v1:key1:|
    00000040  24 77 4d 63 db 40 75 43  af a4 05 85 e9 7e 40 01  |$wMc.@uC.....~@.|
    00000050  54 0c 61 46 8a 5a af 3f  61 09 8e 71 ab 61 c8 89  |T.aF.Z.?a..q.a..|
    00000060  7b bf a5 69 58 f8 f5 11  b6 1a df 5e e7 4d cd 1e  |{..iX......^.M..|
    00000070  06 76 5e 08 7d 6f ed 1c  20 2c 96 69 54 ef 45 9c  |.v^.}o.. ,.iT.E.|
    00000080  2c 8c e3 81 64 f2 3f 73  b8 85 70 1f 63 cd 6d c2  |,...d.?s..p.c.m.|
    00000090  b7 30 b3 7f 94 f6 70 8e  e7 ab 97 6d 06 d6 ae 76  |.0....p....m...v|
    000000a0  42 6a a8 89 48 58 90 50  05 d2 14 e9 87 20 29 bd  |Bj..HX.P..... ).|
    000000b0  e3 c9 ce c8 6f 29 e9 65  fb f3 89 eb 03 ab 64 f5  |....o).e......d.|
    000000c0  bc 58 3d b7 13 48 5c 78  1b 8a 6c d0 b8 61 90 c5  |.X=..H\x..l..a..|
    000000d0  40 f1 ba fb 66 39 5e 92  42 08 f8 e8 74 2f 31 97  |@...f9^.B...t/1.|
    000000e0  dd e8 ec ed d4 ae 24 fa  46 3d be 2f 4e 13 f5 3d  |......$.F=./N..=|
    000000f0  e6 db 92 95 11 e6 3c 53  d2 3b ce 80 b9 f9 ec 8d  |......<S.;......|
    00000100  47 60 1f 18 9c 7f 80 0a  18 45 e4 fb 40 f5 74 7b  |G`.......E..@.t{|
    00000110  42 20 7a 39 ed b7 8d 20  a5 35 74 0a 62 72 3d 1a  |B z9... .5t.br=.|
    00000120  31 8a 70 b4 58 34 68 8c  1f 00 22 08 c5 90 7f 68  |1.p.X4h..."....h|
    00000130  06 18 12 91 6e 75 1b 43  06 2c a9 64 e1 13 2d e0  |....nu.C.,.d..-.|
    00000140  c0 1b d8 d5 aa 2c a1 1d  ee 53 73 53 6b b1 6d 4b  |.....,...SsSk.mK|
    00000150  34 19 0a                                          |4..|
    00000153
    ```
    
    - 이전과 달리 secret 키값이 암호화되어 식별되지 않는다.

### etcd encryption - etcd 암호화 기능 해제 후

- Secret을 생성 후 내장된 ectd에 접속해서 암호화 확인
    
    ```bash
    kubectl create secret generic january-secret --from-literal=januarykey=k8spass#
    kubectl -n kube-system exec -it etcd-k8s-master -- sh -c "ETCDCTL_API=3 \
    ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
    ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
    ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
    etcdctl --endpoints=https://127.0.0.1:2379 \
    get /registry/secrets/default/january-secret"
    
    /registry/secrets/default/january-secret
    k8s:enc:secretbox:v1:key1:"�yc:�D�&`f�a�;9���K�
    �u͔H���39n�cK�R��b���m'�v���y+acj�8��H��	�kV�a�ݘ{����UA�@(	�:4H�2�gV�ܬmF��q�$��K`!,iR\G h��x�f%|�]�WhѲ������>D:����s���@��TV{7L%Չ��?�n�sM�.U��,<��%�<�i<�T��RP
      �w*��#���}��n���$6����mP����i��p����i�/:�=�ǋO
    q���YH�uw�'g|bi
    ```
    
    - 생성한 secret의 내용이 모두 암호화되어 있다.
- etc/kubernetes/mainfest/kube-apiserver.yaml에서 파라미터를 추가 혹은 제거로 암호화 기능 지원 여부를 선택할 수 있다.
- 암호화 전 secret은 자동으로 암호화 되지 않아 수동으로 적용 시켜야 한다.
    
    ```bash
    kubectl get secrets january-secret -o json | kubectl replace -f -
    kubectl get secrets -A -o json | kubectl replace -f -
    kubectl get secrets -n [namespace] -o json | kubectl replace -f -
    ```
    
    - 이 명령으로 암호화를 수동으로 실행
    - 전체 Secret 암호화나 namespace 별로 암호화 가능
- 암호화된 Secret이 있는 상태에서 암호화 해제 시 apiserver가 해당 secret 해독 불가로 not running이 될 수 있다.

## 참고 자료
Fastcampus - 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online.