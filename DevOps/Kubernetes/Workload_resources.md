# Workload 리소스
Kubernetes의 workload는 Pod를 중심으로 구동되는 컨테이너 애플리케이션이고, 이 애플리케이션의 원하는 상태와 특성을 정의하는 다양한 구성 요소들이 있습니다. Workload resources는 Pod의 Life cycle 관리를 지원하며, 다양한 object들이 쿠버네티스에 내장되어 있고, Controller를 통해 관리됩니다.    
워크로드 리소스가 존재하는 이유는 Pod를 관리하기 위해서입니다. Pod는 애플리케이션 배포의 기본 단위이지만 단독으로 사용할 때는 쿠버네티스가 가진 컨테이너 오케스트레이션의 이점을 온전히 얻을 수 없습니다. 따라서 워크로드 리소스를 통해 Pod 관리 자동화가 필요합니다.

| 이점 | 설명 |
| --- | --- |
| Scaling | 언제든 Pod 수를 조정 가능하며 이는 복제 기능을 통해 확장성을 갖추게 됨 |
| Rollout - Rolling update | 가용성을 유지하는 새 버전 교체 기능을 통해 가동중지 시간 최소화 |
| Automatic Healing | Pod의 비정상 중단 상태를 지속적으로 관찰(watch)하여 자동으로 교체 |

Pod는 애플리케이션을 배포하는 가장 기본 단위이며 필요에 따라 언제든 새로 생성되거나 제거될 수 있어 특정 애플리케이션을 식별하는데 좋은 지표가 될 수 없지만 워크로드 리소스는 동일한 워크로드를 수행하는 다수의 pod를 관리하므로 애플리케이션을 특정할 수 있는 리소스가 될 수 있습니다. 대부분의 MSA 애플리케이션은 각각의 특징과 고정된 통신 패턴을 가집니다. Frontend는 Backend에게 지속적으로 요청 트래픽을 보내고, Backend는 DB 서버, 로그 서버. 모니터링 서버 등과 다양한 트래픽을 주고 받는데  이러한 구조의 완성도를 높이려면 다양한 workload resources를 활용해야 합니다.    
이라한 Kubernetes의 workload resources는 애플리케이션 정의, 배포,며관리의 주요 역할 수행하며, 운영을 단순화하고 안정성과 확장성을 높여 리소스 관리를 향상 시키는 추상화 수준의 기능 제공합니다.

## Deployment

- Deployments는 Kubernetes 클러스터에서 컨테이너화 된 애플리케이션을 관리하기 위한 기본 빌딩 블록 제공
    - 애플리케이션 컨테이너 → Pod ReplicaSet Deployment
- Deployments는 복제 Pod 집합을 관리, 확장하는 선언적 방법을 제공하는 API resource로 상위수준의 추상화 object
- Deployments에서 "원하는 상태"를 설명하면 Deployments controller가 제어된 속도로 "현재 상태" 를 원하는 상태로 조정
- Deployments 는 왜 사용할까?
    - ReplicaSet rollout을 통해 Pod 복제본 및 lmage 버전 관리
    - Pod의 지속적 상태 관리(Desired state management)
    - WorkIoad(CPU, Memory usage) 기준 수동 및 자동 확장, 축소 기능 구현가능(HPA)
    - Rolling update를 통한 중단없는 업데이트 지원
    - 다양한 배포 전략을 제공하고 이전 Pod에서 새로운 Pod로 전환속도를 제어 가능(strategy)
    - 이전 버전으로의 RoIIback 지원 (revision)
    - 사용하지 않는 ReplicaSet 정리
- Deployments 생성
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: example-deployment
      namespace: example-ns
    spec:
      **replicas**: 3
      **selector**:
        matchLabels:
          app: example
      **strategy:** {}
      **template**:
        metadata:
          labels:
            app: example
        spec:
          containers:
          - name: example-container
            image: example-image
            ports:
            - containerPort: 80
            resources: {}
    ```
    
    - replicas: Pod 복제본 수 지정
    - selector: DepIoyment가 관리하는 Pod의 선택기로 어떤 label의 Pod를 선택하여 관리할 지에 대한 설정
    - template: 새 Pod를 생성하는 데 사용되는 Pod template 정의
    - strategy: 배포 전략 설정 (기본 RoIIingUpdate)
    - template.metadata.lables: Pod에 적용할 값
    - resources: 통해 CPU,  Memory의 사용량 제어
- Deployment의 리비전은 기본적으로 10개까지 가질 수 있고 원하는 값으로 설정 가능

### Deployment 생성

- deployment manifest - smoke test
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-deployment
    labels:
        app: nginx
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx
        spec:
        containers:
        - name: nginx
            image: nginx:1.14.2
            ports:
            - containerPort: 80
    ```

### Deployment image update

- deployment로 배포한 Pod를 모니터링
    
    ```bash
    kubectl get po --watch
    ```
    
    - kubectl get <resource> —watch를 사용하면 상태를 지속적으로 모니터링 가능
- 다른 터미널에서 deployment 배포
    
    ```bash
    kubectl create deployment msa-web --image=nginx:1.17 --port=80 --replicas=2
    ```
    
- deployment update 속도(시간) 조절
    
    ```bash
    kubectl patch deploy msa-web -p '{"spec": {"minReadySeconds": 10}}';
    ```
    
- Deployment 변경 히스토리 조회
    
    ```bash
    kubectl rollout history deploy msa-web
    deployment.apps/msa-web
    REVISION  CHANGE-CAUSE
    1         <none>
    ```
    
    - 아직 수정된 이력이 없는 deployment라서 리비전이 하나만 있다.
- deployment의 이미지 버전 변경
    
    ```bash
    kubectl set image deploy msa-web nginx=nginx:1.19
    kubectl set image deploy msa-web nginx=nginx:1.21
    ```
    
    - Deployment 변경 히스토리 조회
        
        ```bash
        kubectl rollout history deploy msa-web
        deployment.apps/msa-web
        REVISION  CHANGE-CAUSE
        1         <none>
        2         <none>
        3         <none>
        ```
        
        - 두번의 이미지 버전 변경이 있었고 그 때문에 리비전도 2개 추가되었다.
    - 리비전 세부 사항 확인
        
        ```bash
        kubectl rollout history deployment msa-web --revision 1
        deployment.apps/msa-web with revision #1
        Pod Template:
          Labels:	app=msa-web
        	pod-template-hash=8bd6f7bbc
          Containers:
           nginx:
            Image:	nginx:1.17
            Port:	80/TCP
            Host Port:	0/TCP
            Environment:	<none>
            Mounts:	<none>
          Volumes:	<none>
        ```
        
        - rollout history에 —revision 옵션을 사용하면 리비전에 대한 자세한 정보를 확인 가능
    - 특정 리비전으로 롤백
        
        ```bash
        kubectl rollout undo deployment msa-web --to-revision 2
        deployment.apps/msa-web rolled back
        
        kubectl rollout history deployment msa-web
        deployment.apps/msa-web
        REVISION  CHANGE-CAUSE
        1         <none>
        3         <none>
        4         <none>
        ```
        
        - 특정 리비전으로 롤백을 할 때는 rollout undo —to-revision를 사용한다.
        - undo 후 리비전을 확인하면 롤백하려고 했던 리비전이 가장 마지막 리비전으로 변경되었다.
        - 레플리카셋을 확인하면 리비전의 수만큼 레플리카셋이 있다.
            - 현재 적용된 리비전이 아닌 다른 레플리카셋은 pod 최소, 최대, 요구 수가 모두 0으로 되어 있다.
- Deployment image update, rollout 변경에 대한 주석(CHANGE-CAUSE) 남기기
    
    ```bash
    kubectl annotate deployments.app msa-web kubernetes.io/change-cause="new version"
    kubectl rollout history deploy msa-web
    deployment.apps/msa-web
    REVISION  CHANGE-CAUSE
    1         <none>
    3         <none>
    4         new version
    ```
    
    - annotate를 [kubernetes.io/change-cause에](http://kubernetes.io/change-cause에) 사용하면 리비전이 생성된 이유를 주석으로 달수 있다.
- Deployment image update, rollout → 일시 중지(pause) 및 해제(resume)
    - 일시 중지
        
        ```bash
        kubectl rollout pause deployment msa-web
        ```
        
    - pause 된 상태에서는 롤백을 수행할 수 없다.
        
        ```bash
        kubectl rollout undo deploy msa-web --to-revision 4
        error: you cannot rollback a paused deployment; resume it first with 'kubectl rollout resume' and try again
        ```
        
    - deployment 이미지를 변경할 수 있지만 일시 중지 상태여서 실제 적용은 대기
        
        ```bash
        kubectl set image deploy msa-web nginx=nginx:1.23
        kubectl get deploy msa-web -o=jsonpath='{.spec.template.spec.containers[0].image}{" \n"}'
        nginx:1.23
        
        kubectl get deploy,rs,po | grep msa-web
        deployment.apps/msa-web   2/2     0            2           55m
        replicaset.apps/msa-web-65b8dfc5cb   0         0         0       23m
        replicaset.apps/msa-web-6cc68774c7   2         2         2       23m
        replicaset.apps/msa-web-8bd6f7bbc    0         0         0       55m
        pod/msa-web-6cc68774c7-fqkjx   1/1     Running   0          18m
        pod/msa-web-6cc68774c7-mpxrh   1/1     Running   0          17m
        ```
        
        - 기존에 실행 중이던 pod가 업데이트 되지 않고 그대로 있다.
    - 재시작
        
        ```bash
        kubectl rollout resume deploy msa-web
        kubectl get deploy,rs,po | grep msa-web
        deployment.apps/msa-web   2/2     1            2           56m
        replicaset.apps/msa-web-65b8dfc5cb   0         0         0       25m
        replicaset.apps/msa-web-6cc68774c7   2         2         2       25m
        replicaset.apps/msa-web-796d475cfd   1         1         0       5s
        replicaset.apps/msa-web-8bd6f7bbc    0         0         0       56m
        pod/msa-web-6cc68774c7-fqkjx   1/1     Running             0          19m
        pod/msa-web-6cc68774c7-mpxrh   1/1     Running             0          19m
        pod/msa-web-796d475cfd-fs6pp   0/1     ContainerCreating   0          5s
        ```
        
        - Deployment를 재시작 후에 pod가 롤링 업데이트 되기 시작했다.

### Deployment scaling

- deployment의 replicaset의 수를 조정
    
    ```bash
    kubectl scale deployment msa-web --replicas=6
    ```
    
    - scale 을 사용해서 deployment의 replicaset 수를 조정

### Deployment resource limit

- deployment로 배포한 파드의 리소스를 제한
    
    ```yaml
    spec:
      containers:
      - image: nginx:1.25.3
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          limits:
            memory: "512Mi" 
            cpu: "200m" 
          requests:
            memory: "256Mi" 
            cpu: "100m"
    ```
    
    - limits는 최대 허용량을 의미하고 requeset는 초기 허용량을 의미
- Deployment에 리소스 사용량 제한
    
    ```bash
    kubectl edit deployment.apps msa-web
    ```
    
- Deployment 상태 확인 Probe 사용
    
    ```yaml
    resources:
      limits:
        cpu: 200m
        memory: 512Mi
      requests:
        cpu: 200m
        memory: "256"
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 15
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
    ```
    

### Deployment 배포 전략
### 배포 전략

- 일반적으로 배포 전략에 따라 애플리케이션이 이전 버전에서 최신 버전으로 업데이트 되는 방식이 결정
- 일부 전략에는 다운타임이 포함되며, 일부는 테스트 개념을 도입하고 사용자 분석을
가능하게 함
    - Recreate → 다운타임 발생
    - RollingUpdate(default)
- 트래픽 흐름을 다양한 방식으로 제어할 수 있는 "고급 배포 전략" 사용
    - Blue/Green
    - Canary → 테스트 개념 도입
    - A/B test

| 배포전략 | 설명 | 장점 | 단점 | 롤백 |
| --- | --- | --- | --- | --- |
| Blue/Green | 구/신버전이 동일 환경에 있고, 트래픽 라우팅만 신버전으로 교체하는 방법 | 신버전 검증도 향상과 신속한 전환 속도 | 구/신버전 공존으로 비용 증가 | 라우팅을 구버전으로전환 |
| RollingUpdate | 구버전 수는 점차 줄이고 신버전 수를 점차 늘려 교체하는 방법 | 비용 절감 | 신버전 검증 수준 저하와 전환에 보다 시간 소요 | kubectl rollout undo
또는
kubectl set image |
| Canary | 일부 트래픽만 신버전으로 연결하여 신버전 테스트 하는 방법 | 신버전 미리 검증 | 신버전 문제시 서비스 장애 발생 가능 | canary 버전 Pod 제거 |

### Recreate

- 이전 버전이 다운된| 시점부터 새 Pod가 성공적으로 시작되고 사용자 요청을 처리하기 시작할
때까지 애플리케이션이 일정시간의 "다운타임" 있음
- Recreate는 개발 환경이나 사용자가 장기간의 성능 저하 또는 오류보다 짧은 가동 중지 시간을
선호하는 경우나 구, 신 버전을 동시에 사용할 수 없는 경우 사용
- 설정이 간단하고, 애플리케이션의 상태가 완전히 교체되어 완전히 업데이트 됨.

```yaml
spec:
	replicas: 10
	strategy:
		type: Recreate
```

### RollingUpdate

- RollingUpdate는 실행중인 Pod를 새 버전으로 업데이트 하는 배포전략으로 가동중지 시간을 줄이기 위해 설계된 전략이고, 모든 노드는 새 버전으로 점진적 업데이트 수행
- 장점은 상대적으로 를백하기 쉽고 배포를 재생성하는 것보다 덜 위험하며 구현하기 쉬움
- 단점은 속도가 느릴 수 있고 문제가 발생할 경우 이전 버전으로 를백 됨. 또한, 애플리케이션에 여러 버전이 동시에 병렬로 실행될 수 있음을 의미.
- 정해진 비율 만큼의 Pod만 점진적으로 배포하는 방식
- maxSurge
    - 롤링 업데이트를 위해 최대로 생성할수 있는 Pod 수.% / 개수 단위로 지정
    - maxSurge를 높게 설정하면 롤링 배포를 빠르게 적용가능
    - 기본값 25%
- maxUnavailable
    - 롤링 업데이트 중 최대로 삭제할 Pod 수. %/ 개수 단위로 지정
    - maxUnavaiIabIe를 높게 설정하면 롤링 배포를 빠르게 적용 가능
    - 단, 높게 설정 시 트래픽이 남은 소수의 Pod로 집중되어 성능 문제 유발 가능
    - 기본값 25%

```yaml
spec:
  replicas : 10
  strategy :
    type: RollingUpdate
    rollingUpdate :
      maxSurge: 3
      maxUnavai1ab1e: 1
```

### 10개의 replicaset, 한 번에 3개 생성, Rollout 중에 1개를 사용할 수 없도록 지정

- replicaset이 1개인 deployment 생성
    
    ```bash
    kubectl create deployment msa-web --image=nginx:1.21 --port=80 --replicas=1
    ```
    
- deployment의 replicaset을 10개로 늘이고 maxSurge: 3, maxUnabailable: 1로 변경
- deployment의 image를 변경 후 변경 진행 사항 확인
    
    ```bash
    kubectl set image deploy msa-web nginx=nginx:1.23
    kubectl rollout status deployment msa-web
    kubectl rollout status deployment msa-web
    Waiting for deployment "msa-web" rollout to finish: 8 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 8 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 8 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 8 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 9 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 9 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 9 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 9 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 9 out of 10 new replicas have been updated...
    Waiting for deployment "msa-web" rollout to finish: 3 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 3 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 3 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 3 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 2 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 2 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 1 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 1 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 1 old replicas are pending termination...
    Waiting for deployment "msa-web" rollout to finish: 9 of 10 updated replicas are available...
    deployment "msa-web" successfully rolled out
    ```
    
    - Deployment가 update를 진행할 때 kubectl rollout status를 사용하면 진행사항을 실시간으로 확인할 수 있다.

### Recreate

- 이전에 실습한 deployment의 strategy를 recreate로 변경
- Deployment의 이미지 변경 후 업데이트 진행사항 확인
    
    ```bash
    kubectl set image deploy msa-web nginx=nginx:1.25
    kubectl rollout statue deployment msa-web
    Waiting for deployment "msa-web" rollout to finish: 0 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 1 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 2 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 3 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 4 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 5 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 6 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 7 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 8 of 10 updated replicas are available...
    Waiting for deployment "msa-web" rollout to finish: 9 of 10 updated replicas are available...
    deployment "msa-web" successfully rolled out
    ```
    
    - 한번에 모든 이전 파드를 버리고 새로운 파드를 올린다.

## StatefulSet

- statefuISet은 애플리케이션의 StatefuISet(상태저장)을 관리하는 workload 리소스
- StateIess 애플리케이션은 DepIoyment로 배포하고, Stateful인 DB같은 경우는 StatefulSet으로 배포
    - StatefuISet로 실행시킨 Pod는 Deployment(deploy-nginx-5s8km)와 다르게 pod-0, pod-1과 같은 순서값과 안정적인 네트워크 ID를 Pod에 할당
    - Pod는 오름차순으로 생성되고, 다음 Pod는 이전 Pod가 준비되고 실행 상태가 된 후에만 생성 된다. 삭제는 반대로 큰 수의 Pod부터 삭제됨 → "OrderedReady"
    - spec.podManagemetPoIicy: "ParaIIeI" → Pod를 순서없이 병렬로 실행되거나 종료
        - kubectl explain statefuIset.spec.podManagementPoIicy
- 특징
    - StatefulSet의 각 Pod들은 동일 spec으로 생성되지만 서로 교체는 불가능
        - re-scheduling이 되도 지속적으로 동일 식별자 유지.
        - sts-0-pod 를 지워도 sts-0-pod 이름으로 재생성 됨
    - statefulSet 애플리케이션은 서버, 클라이언트, 애플리케이션에서 사용할 수 있도록 영구 볼륨에 데이터저장. → StorageClass를 사용한 PVC 사용(동적볼륨)
        - statefuISet의 복제본 증가시 자동으로 PV/PVC가 생성
        - statefuISet의 복제본 축소시 사용했던 PV/PVC는 삭제되지 않고 유지
        - PV/PVC가 StatefulSet 복제본 증가시 자동 생성되고 축소시 유지되는 이유
            - 스토리지가 고유한 상태를 유지해야 하는데 scale로 인해 삭제되면 상태 데이터도
            같이 삭제되기 때문이다.
    - StatefuISet은 HeadIess service를 사용해야 한다.
        - ClusterIP를 사용하지 않는 것
    - StatefuISet에 연결된 Service들은 Service를 통하지 않고, 논리적으로 Pod 집합을 만들어 주는 Service만 있으면 되므로 HeadIess Service를 사용 한다.
    - IP가 없어도 DNS에 등록되므로 nslookup을 통해 Pod의 주소를 확인하여 연결 가능하다.

### StatefulSet 생성

- statefulSet을 만들기 위해 statefulset, pvc, headless service 생성이 필요하다.
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    name: nginx
    labels:
        app: nginx
    spec:
    ports:
    - port: 80
        name: web
    clusterIP: None
    selector:
        app: nginx
    ---
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
    name: web
    spec:
    selector:
        matchLabels:
        app: nginx # .spec.template.metadata.labels 와 일치해야 한다
    serviceName: "nginx"
    replicas: 3 # 기본값은 1
    minReadySeconds: 10 # 기본값은 0
    template:
        metadata:
        labels:
            app: nginx # .spec.selector.matchLabels 와 일치해야 한다
        spec:
        terminationGracePeriodSeconds: 10
        containers:
        - name: nginx
            image: registry.k8s.io/nginx-slim:0.8
            ports:
            - containerPort: 80
            name: web
            volumeMounts:
            - name: www
            mountPath: /usr/share/nginx/html
    volumeClaimTemplates:
    - metadata:
        name: www
        spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "my-storage-class"
        resources:
            requests:
            storage: 1Gi  
    ```
    
    - service는 headless로 생성해야 하므로 clusterIP: None을 사용
- Statefulset Scale out
    
    ```bash
    kubectl scale sts nginx --replicas 4
    ```
    - Scaling을 수행하면 한번에 필요한 pod가 추가되지 않고 하나씩 점진적으로 추가됨
    
- StatefulSet scale in
    
    ```bash
    kubectl scale sts nginx --replicas 2
    ```
       
    - StatefulSet scale in을 수행하면 마지막에 추가된 pod부터 역순으로 pod가 삭제된다.
    - pod를 삭제되었지만 상태를 유지하기 위해 pv, pvc는 지워지지 않고 유지한다.

## DaemonSet
### DaemonSet

- DaemonSet 은 Deployment와 유사하지만 RepIicas 옵션은 없다.
- 이는 DaemonSet 이 노드 단위 배포이고, 노드의 백그라운드에서 항상 Pod를 데몬으로 실행할 수 있게 해주는 workload resources 라는 것이다.
- DaemonSet은 모드 노드에서 백그라운드로 항상 실행되어야 업무를 수행에 적합
    - 모니터링 시스템, 로그 수집 에이전트, 노드 데이터 백업 과 같은 장기간 지속되는 작업
    - Prometheus와 같은 모니터링 도구 사용시 exporter를 DaemonSet으로 배치
- DaemonSet 업데이트 전략의 기본은 RoIIingUpdate (또는 OnDeIete)
    - “OnDeIete": 이전 데몬이 종료된 경우에만 교체
        - 데몬을 교체할 때 기존의 데몬을 삭제해야만 신규 데몬으로 교체 가능
    - "RollingUpdate": 롤링을 사용하여 이전 데몬을 신규 데몬으로 교체
- Taint와 toleration 옵션을 사용하여 특정 노드에 배포 선택 가능
- 대규모 클러스터인 경우 nodeSeIector를 사용하여 특정 노드 선택 가능
    - using node label → kubectl label node k8s-node2 log-collect-node=true
    - nodeSelector:
      log-collect-node: “true”

### DaemonSet 생성 - kubernetes docs 예시

- kubernetes Daamonset 예시 Manifest
    
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd-elasticsearch
      namespace: kube-system
      labels:
        k8s-app: fluentd-logging
    spec:
      selector:
        matchLabels:
          name: fluentd-elasticsearch
      template:
        metadata:
          labels:
            name: fluentd-elasticsearch
        spec:
          tolerations:
          # these tolerations are to have the daemonset runnable on control plane nodes
          # remove them if your control plane nodes should not run pods
          - key: node-role.kubernetes.io/control-plane
            operator: Exists
            effect: NoSchedule
          - key: node-role.kubernetes.io/master
            operator: Exists
            effect: NoSchedule
          containers:
          - name: fluentd-elasticsearch
            image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
            resources:
              limits:
                memory: 200Mi
              requests:
                cpu: 100m
                memory: 200Mi
            volumeMounts:
            - name: varlog
              mountPath: /var/log
          # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
          # preempts running Pods
          # priorityClassName: important
          terminationGracePeriodSeconds: 30
          volumes:
          - name: varlog
            hostPath:
              path: /var/log
    ```
    
    - toleration를 사용해서 master node나 control-plane이 노드 역할인 경우에도 daemonset이 스케줄 되도록 한다.
    - 배포한 daemonset 확인
        
        ```bash
        kubectl -n kube-system get ds,po -o wide | grep fluentd
        daemonset.apps/fluentd-elasticsearch   4         4         4       4            4           <none>                   35s   fluentd-elasticsearch   quay.io/fluentd_elasticsearch/fluentd:v2.5.2   name=fluentd-elasticsearch
        pod/fluentd-elasticsearch-6wxn6                  1/1     Running   0              34s    10.111.156.68    k8s-node1    <none>           <none>
        pod/fluentd-elasticsearch-ndsp9                  1/1     Running   0              34s    10.111.218.73    k8s-node3    <none>           <none>
        pod/fluentd-elasticsearch-p4845                  1/1     Running   0              35s    10.108.82.220    k8s-master   <none>           <none>
        pod/fluentd-elasticsearch-tcs7j                  1/1     Running   0              34s    10.109.131.20    k8s-node2    <none>           <none>
        ```
        
        - 마스터 노드를 포함한 모든 노드에 pod가 한개씩 생성된 것을 확인 가능
- 위 Manifest에서 toleration을 삭제하면 taint가 있는 node에는 daemonset이 스케줄 되지 않는다.

### Node Label을 사용한 Node 선택

- 워커 노드 2개에 식별 가능한 label을 붙인다.
    
    ```bash
    kubectl label nodes k8s-node1 az=ap-northeast-2
    kubectl label nodes k8s-node3 az=ap-northeast-2
    ```
    
- daemonset mainfest에서 nodeselector를 추가
    
    ```yaml
    spec:
      selector:
        matchLabels:
          name: fluentd-elasticsearch
      template:
        metadata:
          labels:
            name: fluentd-elasticsearch
        spec:
          nodeSelector:
            az: ap-northeast-2
    ```
    
- daemonset 확인
    
    ```bash
    kubectl -n kube-system get ds,po -o wide | grep fluentd
    daemonset.apps/fluentd-elasticsearch   2         2         2       2            2           az=ap-northeast-2           21s   fluentd-elasticsearch   quay.io/fluentd_elasticsearch/fluentd:v2.5.2   name=fluentd-elasticsearch
    pod/fluentd-elasticsearch-9pn8w                  1/1     Running   0              21s    10.111.156.74    k8s-node1    <none>           <none>
    pod/fluentd-elasticsearch-tgcj9                  1/1     Running   0              21s    10.111.218.76    k8s-node3    <none>           <none>
    ```
    
    - nodeSelctor에 지정한 node만 선택해서 daemonset을 배포했다.

## Job & CronJob

- Job은 Pod와 같은 종류지만, Pod와 다르게 특정 작업만 수행하고 종료된다.
    - Job 은 일회성 및 일괄 작업 실행에 적합한 반면, CronJob은 주로 반복 작업을 예약하는 데 사용
    - Job 은 작업을 정의하고 Job이 완료될 때까지 실행되도록 보장, 하나 이상의 Pod의 수명 주기를 관리하여 원하는 성공 횟수가 도달할 때 까지 관리
- CronJob 리소스는 사전 정의된 일정(일일,주간 또는 월간)에 따라Job 생성
    - CronJob에 의해 생성된 각 Job 은 특정 작업 또는 작업 집합을 Pod로 실행
    - CronJobs는 백업, 보고서 생성 등과 같은 정기적인 예약 작업을 수행하기 위해 사용.
    - 반복적 Job이 요구되는 경우 (예: 하루/주/월에 한 번)
    - 작업을 시작해야하는 해당 간격 내 특정 시점을 정의
    - 리눅스의 cronjob과 동일한 스케줄 작성(분.시.일.월.요일)
        
        ```bash
        # ┌───────────── minute (0 - 59)
        # │ ┌───────────── hour (0 - 23)
        # │ │ ┌───────────── day of the month (1 - 31)
        # │ │ │ ┌───────────── month (1 - 12)
        # │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
        # │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
        # │ │ │ │ │ 
        # │ │ │ │ │
        # * * * * *
        ```
        

### Job으로 원주율 2000자리까지 계산해서 출력

- job 생성
    
    ```bash
    kubectl create job pi --image=perl:5.34.0 --dry-run=client -o yaml >> job.yaml
    ```
    
    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
    name: pi
    spec:
    template:
        spec:
        containers:
        - name: pi
            image: perl:5.34.0
            command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
        restartPolicy: Never
    backoffLimit: 4
    ```
    - command
        - Job이 실행할 작업 지정
    - restartPolicy : Never
        - 컨테이너가 실패하거나 종료되는 경우 컨테이너를 다시 시작하지 않도록 지정

### Job completion 옵션

- job을 실행할 Pod의 수를 지정 - 모든 pod가 하나씩 완료될 때까지 job이 Pod를 관리
- Job parallelism 옵션을 사용하면 여러 pod가 한번에 작업을 실행한다.
- Job manifest에 completion 지정
    
    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hello-job
    spec:
      completions: 10
      template:
        metadata:
        spec:
          containers:
          - command:
            - echo
            - Hello I'm running job
            image: busybox
            name: hello-job
          restartPolicy: Never
    ```
    
    - job 작업 진행 확인
        
        ```bash
        kubectl get jobs.batch -w
        NAME          COMPLETIONS   DURATION   AGE
        hello-job     2/10          14s        14s
        nodeversion   1/1           54s        11m
        hello-job     2/10          20s        20s
        hello-job     3/10          20s        20s
        hello-job     3/10          26s        26s
        hello-job     4/10          26s        26s
        hello-job     4/10          32s        32s
        hello-job     5/10          32s        32s
        hello-job     5/10          39s        39s
        hello-job     6/10          39s        39s
        hello-job     6/10          46s        46s
        hello-job     7/10          46s        46s
        hello-job     7/10          52s        52s
        hello-job     8/10          52s        52s
        hello-job     8/10          58s        58s
        hello-job     9/10          58s        58s
        hello-job     9/10          65s        65s
        hello-job     10/10         65s        65s
        ```
        
        - 작업이 하나씩 순차적으로 작업을 완료한다.
- Job parallelism 옵션
    
    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hello-job
    spec:
      parallelism: 5
      completions: 10
      template:
        metadata:
        spec:
          containers:
          - command:
            - echo
            - Hello I'm running job
            image: busybox
            name: hello-job
          restartPolicy: Never
    ```
    
    - job 작업 진행 확인
        
        ```bash
        kubectl apply -f hellojob.yaml
        kubectl get job.batch -w
        job.batch/hello-job created
        NAME        COMPLETIONS   DURATION   AGE
        hello-job   0/10          1s         1s
        hello-job   0/10          10s        10s
        hello-job   1/10          10s        10s
        hello-job   1/10          11s        11s
        hello-job   1/10          12s        12s
        hello-job   3/10          12s        12s
        hello-job   3/10          13s        13s
        hello-job   4/10          13s        13s
        hello-job   4/10          14s        14s
        hello-job   5/10          14s        14s
        hello-job   5/10          17s        17s
        hello-job   6/10          17s        17s
        hello-job   6/10          19s        19s
        hello-job   9/10          20s        20s
        hello-job   9/10          20s        20s
        hello-job   10/10         20s        20s
        ```
        
        - COMPLETIONS를 확인하면 6/10 다음 바로 9/10이 출력되는 것을 확인할 수 있다.
        - 병렬처리로 한번에 작업을 완료하기 때문에 여러 작업이 함께 완료되기 때문


### Cronjob 생성

- Linux Cronjob과 거의 동일하다.
- Cronjob 생성
        
    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
    name: hello
    spec:
    schedule: "* * * * *"
    jobTemplate:
        spec:
        template:
            spec:
            containers:
            - name: hello
                image: busybox:1.28
                imagePullPolicy: IfNotPresent
                command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
            restartPolicy: OnFailure
    ```
- Cronjob 옵션
    - successfulJobsHistoryLimit / failedJobsHistoryLimit : 기본값 3
        - CronJob 컨트롤러가 마지막 3개의 성공 및 실패한 작업 기록을 각각 유지한다는 의미
    - ConcurrencyPolicy는 CronJob이 동시 작업 실행을 처리하는 방법을 지정
        - Allow(기본값)는 동일한 작업의 동시 실행 허용,
        - Forbid는 동일한 작업이 순차적으로 실행되는 경우의 동시 실행 방지
        - Replace 는 현재 실행 중인 작업을 취소하고 새작업으로 대체

## 참고 자료
Fastcampus - 실무까지 한 번에 끝내는 DevOps를 위한 Docker & Kubernetes feat. aws EKS 초격차 패키지 Online.      
[Kubernetes Docs - Workload Management](https://kubernetes.io/docs/concepts/workloads/controllers/)     
[Kubernetes Docs - 디플리로이먼트](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)        
[Kubernetes Docs - 스테이트풀셋](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)     
[Kubernetes Docs - 데몬셋](https://kubernetes.io/ko/docs/concepts/workloads/controllers/daemonset/)     
[Kubernetes Docs - 잡](https://kubernetes.io/ko/docs/concepts/workloads/controllers/job/)       
[Kubernetes Docs - 크론잡](https://kubernetes.io/ko/docs/concepts/workloads/controllers/cron-jobs/)     