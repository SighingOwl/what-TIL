# Container Services
일반적으로 컨테이너 실행 환경을 떠올리면 Docker나 Kubernetes가 생각납니다. 이 둘을 클라우드에서 실행하려면 어떻게 구성해야될까 생각해보면 가장 먼저 EC2 인스턴스에 Docker나 Kubernetes 환경 구성을 고려해볼 것입니다. 먼저 Docker를 EC2 환경에 구성하다면 보통 VM 환경에서 Docker를 구성하는 것과 동일할 것입니다. 따라서 Docker 엔진 업데이트나 리소스 관리, 보안, 필요에 따라서는 EC2 인스턴스 오토스케일링을 설정을 사용자가 직접 수행 해야합니다. Kubernetes의 경우 Kops를 활용해 AWS에 kubernetes 클러스터를 구성할 수 있습니다. 하지만 이를 위해 API Server와 통신을 위한 별도의 도메인과 Route53 설정이 필요하며 처음 클러스터 구성 난이도가 높고 kubernetes 버전 업데이트와 컨트롤 플레인 관리를 직접 수행해야합니다.   
이 문제들은 애플리케이션이나 서비스를 운영하는데 필요한 인적 자원과 비용을 증가시키는 요인이 될 수 있습니다. AWS에서는 컨테이너 실행과 관련된 관리형 서비스를 제공하여 운영 오버헤드를 줄일 수 있는 솔루션을 제공하고 있습니다. Amazon ECS, EKS가 대표적으로 ECS를 사용하면 ECS가 클러스터에 사용자가 원하는 성능의 EC2 인스턴스를 프로비저닝하고 유지관리 수행하며 사용자가 정의한 테스크를 자동으로 실행하거나 정지하게 됩니다. Fargate를 사용한다면 EC2 인스턴스를 프로비저닝하지 않아 서버리스 서비스 형태로 컨테이너를 실행할 수도 있습니다. EKS는 관리형 Kubernetes 클러스터 서비스이며 AWS는 컨트롤 플레인을 직접 관리하게 됩니다. 따라서 사용자는 워커노드에서 워크로드를 실행하는 것에 집중할 수 있습니다. EKS를 사용하면 AWS는 물론 온프레이스나 Azure, GCP와 같은 다른 클라우드에서 실행되는 클러스터도 관리할 수 있습니다. 

## Amazon ECS

- AWS에서 컨테이너를 실행하면 ECS 클러스터에 ECS 태스크를 실행
- EC2 시작 유형
    
    <img src="/images/AWS_container_1.png" width="75%" height="75%" title="aws container 1" alt="aws contaienr 1">      
    
    - 인프라를 직접 프로비저닝 및 유지관리
    - 각 EC2 인스턴스는 ECS 에이전트를 실행
        - ECS 서비스가 각 EC2 인스턴스를 ECS 클러스터에 등록
    - ECS 태스크를 수행하기 시작하면 AWS가 컨테이너의 시작과 정지를 관리
        - 컨테이너가 생기면 시간에 따라 EC2 인스턴스에 지정
        - ECS 태스크를 시작하거나 멈추면 자동으로 위치가 지정
- Fargate 시작 유형
    
    <img src="/images/AWS_container_2.png" width="75%" height="75%" title="aws container 2" alt="aws contaienr 2">      
    
    - EC2 시작 유형처럼 AWS에서 컨테이너를 시작하는 유형
    - 인프라를 프로비저닝하지 않아 관리할 EC2 인스턴스가 없다 - 서버리스
        - 관리할 서버가 없는 것이지 서버가 없는건 아님
    - ECS 태스크 정의만 생성하면 필요한 CPU, RAM에 따라 ECS 태스크를 AWS가 대신 실행
    - 새 Docker 컨테이너가 실행되면 어디서 실행되는지 알리지 않고 그냥 실행
        - 작업을 위해 백앤드에 EC2 인스턴스가 생성될 필요가 없다.
    - 확장을 위해 간단하게 태스크 수만 늘리면 된다.

### IAM Role

<img src="/images/AWS_container_3.png" width="75%" height="75%" title="aws container 3" alt="aws contaienr 3">      

- EC2 인스턴스 프로파일
    1. ECS 에이전트가 사용
    2. ECS 서비스로 API 호출
    3. 인스턴스에 저장된 ECS 서비스가 CloudWatch 로그에 API 호출해서 컨테이너 로그를 전송
    4. ECR로부터 도커 이미지를 가져온다.
    5. Secrets Manager나 SSM Parameter Store에서 민감 데이터를 참고하기도 한다.
- ECSTask Role
    1. 두 개의 태스크가 있다면 각자에 특정 역할을 만들 수 있다.
    2. 역할이 각자 다른 ECS 서비스에 연결할 수 있기 때문
    3. 태스크 정의에서 태스크의 역할을 정의

### Load Balancer Integration

- EC2 시작 유형과 Fargate 시작 유형 동일하게 로드 밸런서와 통합할 수 있다.
- HTTP/HTTPS 엔드포인트로 태스크를 활용하기 위해 ALB를 ECS 클러스터 앞에서 실행
    - 모든 사용자가 ALB 및 백앤드의 ECS 테스크에 직접 연결된다.
- ALB
    - 대부분의 use case를 지원
- NLB
    - 처리량이 매우 많거나 높은 성능을 요구될 때만 권장
    - AWS Private Link와 함께 사용할 때 권장
- ELB - CLB
    - 구세대 ELB는 사용할 수 있지만 권장하지 않는다.
    - Fargate와는 연결할 수 없다.

### EFS

- ECS의 데이터 볼륨으로 자주 사용된다.
- ECS 태스크에 EFS 파일 시스템을 마운트해서 데이터를 공유
- EC2 인스턴스와 Fargate와 호환 및 ECS 태스크에 파일 시스템을 직접 마운트 가능
- ECS 태스크가 어느 가용영역에 있든 EFS와 연결되어 있다면 데이터 공유 가능
- Fargate + EFS = Serverless
    - 데이터 지속 가능한 서버리스
- Use case
    - 다중 AZ가 공유하는 컨테이너의 영구 스토리지
- Note
    - Amazon S3는 파일 시스템으로 마운트 될 수 없다.

### Auto Scaling

- 오토 스케일링 지표
    - ECS 서비스의 평균 CPU 사용률
    - ECS 서버스의 평균 RAM 사용률
    - 대상마다 발생하는 ALB 요청 수 - ALB 지표 사용
- Target Tracking - 특정한 CloudWatch 지표의 대상 값을 기반으로 스케일링
- Step Scaling - 특정한 CloudWatch 경보를 기반으로 스케일링
- Scheduled Scaling - 특정 시간이나 날짜를 기반으로 스케일링
- ECS 오토 스케일링은 태스크 수준에서 오토 스케일링이 되는 것, EC2 인스턴스 수준인 EC2 오토 스케일링과 다르다
- EC2 오토 스케일링이 필요없는 상황이면 Fargate를 사용하는 것이 서비스 오토 스케일링에 도움이 된다.
    
    #### EC2 Launch Type
    
    - ASG Scaling
        - CPU 사용률에 따라 스케일링
        - EC2 인스턴스 추가
    - ECS Cluster Capacity Provider - 클러스터 용량 공급자
        - 새 태스크를 실행할 용량이 부족하면 자동으로 ASG를 확장
        - ASG와 함께 사용
        - RAM이나 CPU가 부족할 대 EC2 인스턴스를 추가
        - ECS 사용시에 ASG보다 사용이 권장

### Solution Architect
#### ECS tasks invoked by Event Bridge

<img src="/images/AWS_container_4.png" width="75%" height="75%" title="aws container 4" alt="aws contaienr 4">      

- 사용자가 S3 버킷에 객체를 업로드 및 S3 버킷이 Amazon EventBridge와 통합되어 모든 이벤트를 전송
- Amazon EventBridge는 항상 ECS 태스크를 실행하기 위한 규칙을 가질 수 있다.
- ECS 태스크 생성 및 역할에 따라 태스크가 실행
- S3 버킷에서 객체를 받아 처리 후 DynamoDB에 저장
- 이미지나 객체를 처리할 수 있는 서버리스 아키텍처

#### ECS tasks invoked by Event Bridge Schedule

<img src="/images/AWS_container_5.png" width="75%" height="75%" title="aws container 5" alt="aws contaienr 5">      

- Amazon EventBridge에서 1시간마다 트리거되는 규칙을 스케줄링
- 스케줄링 규칙에 따라 새로운 ECS 태스크를 생성
- 모든 아키텍처는 서버리스

#### SQS Queue

<img src="/images/AWS_container_6.png" width="75%" height="75%" title="aws container 6" alt="aws contaienr 6">      

- 2개의 ECS 태스크가 있는 서비스가 있을 때 이 서비스는 SQS 대기열에서 메시지를 가져와 처리
- SQS 대기열에 메시지가 많을 경우 ECS 서비스 오토 스케일링을 활성화하여 더 많은 태스크가 메시지를 처리할 수 있도록 한다.

#### Intercept Stopped Tasks using EventBridge

<img src="/images/AWS_container_7.png" width="75%" height="75%" title="aws container 7" alt="aws contaienr 7">      

- 태스크가 종료되거나 시작하는 것은 EventBridge에서 이벤트로서 트리거될 수 있다.
- ECS 상태가 변할 때 발생
- SNS 주제로 경보하거나 관리자에게 이메일을 전송
- EventBridge가 최소한 사용자가 ECS 클러스터에 있는 컨테이너의 수명 주기를 이해할 수 있게 한다.

## Amazon ECR

- AWS에 docker 이미지를 저장하고 관리하는데 사용
- Options
    - 하나 이상 계정에 한해 이미지를 비공개로 저장
    - Amazon ECR Public Gallery에 퍼블릭으로 이미지를 게시
- ECR은 Amazon ECS와 완전 통합되어 있다.
- 이미지는 백그라운드에서 S3에 저장
- ECS 클러스터의 EC2 인스턴스에 이미지를 pull하기 위해서 EC2 인스턴스에 IAM 역할을 지정
    - ECR에 대한 모든 접근은 IAM이 보호
    - ECR에서 이미지를 pull한 후에 컨테이너를 실행할 수 있다.
- 단순히 이미지 저장 뿐 아니라 이미지의 취약점 스캐닝, 버저닝, 태그 및 수명 주기 확인을 지원

## Amazon EKS

- AWS에서 관리형 Kubernetes 클러스터를 실행할 수 있는 서비스
- Kubernetes
    - 오픈 소스 시스템으로 Docker로 컨테이너화한 애플리케이션의 자동 배포, 확장, 관리를 지원
- 컨테이너를 실행한다는 목적은 ECS와 비슷하지만 사용하는 API가 다르다.
    - 오픈 소스이며 여러 클라우드 서비스가 사용하므로 표준화를 기대할 수 있다.
- Launch Option
    - EC2 Launch Type
        - 워커 노드를 배포하고 싶을 때 사용
    - Fargate Launch Type
        - 서버리스 컨테이너를 배포할 때 사용
- Use case
    - 회사가 on-premise나 클라우드에서 kubernetes나 kubernetes API를 사용 중일 때 kubernetes 클러스터를 관리하기 위해 사용
- Cloud-agnostic이므로 Azure, GCP등 모든 클라우드에서 지원
    - 클라우드 또는 컨테이너 간 마이그레이션을 실행하는 경우 유용

### Diagram

<img src="/images/AWS_container_8.png" width="75%" height="75%" title="aws container 8" alt="aws contaienr 8">      

- EKS 워커 노드를 생성하면 EC2 인스턴스가 구성
    - 각 노드는 EKS Pod를 실행
    - EKS 노드는 ASG으로 관리 가능
- EKS 서비스나 Kubernetes 서비스를 노출
    - 프라이빗 로드밸런서
    - 퍼블릭 로드밸런서를 사용해 웹에 연결

### Node Types

- Managed Node Groups
    - AWS로 노드(EC2 인스턴스)를 생성하고 관리
    - 노드는 EKS 서비스로 관리되는 오토 스케일링 그룹의 일부
    - 온디맨드와 스팟 인스턴스 지원
- Self-Managed Nodes
    - 사용자 지정 사항이 많고 제어 대상이 많은 경우 사용
    - 사용자가 직접 노드를 생성 EKS 클러스터에 등록 ASG의 일부로 관리
    - 사전 빌드 된 AMI 중 Amazon EKS에 최적화 AMI를 사용하면 시간을 절약할 수 있다.
    - 온디맨드 및 스팟 인스턴스 지원
- AWS Fargate
    - 노드 사용을 원하지 않을 경우 사용
    - 노드 관리 같은 유지관리가 필요없다.

### Data Volume

- EKS 클러스터에 스토리지 클래스 매니패스트를 지정
- CSI - Container Storage Interface 규격의 드라이버 사용
- Amazon EBS
- Amazon EFS
- Amazon FSx for Lustre
- Amazon FSx for NetApp ONTAP

## App Runner

- 완전 관리형 서비스
- 규모에 따라 웹 애플리케이션, API 배포를 돕는다.
- 인프라 경험을 요구하지 않는다.
- 소스 코드나 docker 이미지로 원하는 구성을 설정
    - vCPU수나 컨테이너 메모리의 크기
    - 오토 스케일링 여부
    - 상태 확인
    - 웹 애플리케이션이나 API에 들어갈 기본 설정
- 웹 애플리케이션을 자동으로 빌드 및 배포
- 웹 애플리키이션 배포 후 URL을 통해 바로 액세스 가능
- 오토 스케일링, 고가용성, 로드 밸런싱, 암호화 기능 지원
- 컨테이너의 VPC 액세스 지원
- DB, 캐시, 메시지 대기열 서비스 연결
- Use case
    - 빨리 배포해야 하는 웹 애플리케이션, API, 마이크로서비스

## 참고 자료
[AWS Docs - Amazon Elastic Container Service란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/Welcome.html)     
[AWS Docs - Amazon Elastic Container Registry란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AmazonECR/latest/userguide/what-is-ecr.html)     
[AWS Docs - Amazon EKS란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/what-is-eks.html)      
[AWS Docs - AWS App Runner은 무엇입니까?](https://docs.aws.amazon.com/ko_kr/apprunner/latest/dg/what-is-apprunner.html)     