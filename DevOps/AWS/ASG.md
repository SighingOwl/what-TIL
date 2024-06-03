# ASG
## Scalability

- 조정을 통해 더 많은 양을 처리할 수 있다는 의미
- 스케일링성은 고가용성과 연관되어 있지만 의미는 다르다.

### Vertical Scalability

- 단일 인스턴스의 크기를 확장하는 것을 의미
- t2.micro를 t2.large로 업그레이드하는 방식
- 데이터베이스와 같이 분산되지 않는 시스템에서 사용
- RDS, ElasticCache
- 수직 확장은 확장할 수 있는 정도에 한계가 있다.
- Scale Up : 성능 확장
- Scale Down : 성능 축소

### Horizontal Scalability (Elasticity)

- 애플리케이션에서 인스턴스나 시스템의 수를 늘리는 것을 의미
- t2.micro 인스턴스의 수를 늘려 처리량을 늘리는 방식
- 수평 확장을 한다는 것은 분산 시스템이 있다는 것을 알 수 있다.
- 웹 애플리케이션이나 현대적인 애플리케이션은 보통 분산 시스템을 사용한다.
- AWS와 같은 클라우드 서비스 덕분에 수평확장이 더 수월해짐
- Scale out : 인스턴스 수를 확장
- Scale in : 인스턴스 수를 축소
- 오토 스케일링 그룹, 로드 밸런서를 사용

## Auto Scaling Group

- 웹 사이트나 애플리케이션은 사용자가 늘어날 수록 로드가 변할 수 있다.
- AWS는 EC2 인스턴스를 활용해서 서버를 매우 빠르게 증설하거나 감설할 수 있는데 이것을 자동화하는 것이 ASG다.
- 목표
    - 증가하는 로드에 맞춰 Scale out
    - 감소하는 로드에 맞춰 Scale in
    - 실행 중인 EC2 인스턴스의 최소 개수와 최대 개수를 매개변수로 정의할 수 있다.
    - ASG를 로드 밸런서에 페어링하면 ASG에 속한 모든 EC2 인스턴스가 로드 밸런서와 연결된다.
    - 장애가 발생한 EC2 인스턴스를 자동으로 종료하고 새로운 EC2 인스턴스를 생성할 수 있다.
- ASG 크기 설정
    
    <img src="/images/ASG_1.png" width="50%" height="50%" title="asg 1" alt="asg 1">    
    
- Launch Template
    - ASG를 사용하기 위해 시작 템플릿을 만들어야 한다.
    - 시작 템플릿은 EC2 인스턴스를 시작하는 방법에 대한 정보가 포함되어 있다.
    - AMI + Instance Type
    - EC2 User Data
    - EBS Volumes
    - Security Groups
    - SSH Key Pair
    - IAM Roles for EC2 Instances
    - Network + Subnets Information
    - Load Balancer Information
- CloudWatch Alarms & Scaling
    - ASG는 CloudWatch 알람을 사용해서 스케일 조정이 가능하다.
    - 알람은 지표를 모니터링 해서 조건에 맞으면 알람을 발생시킨다.
    - 경보 기반
        - Scale-out 정책 생성
        - Scale-in 정책 생성

### Create ASG

1. EC2 콘솔의 Auto Scaling Group을 생성
    
    <img src="/images/ASG_2.png" width="50%" height="50%" title="asg 2" alt="asg 2">    
    
2. ASG를 생성하기 위해서 시작 템플릿이 필요하므로 ASG를 적용할 EC2 인스턴스의 시작 템플릿을 생성 및 버전 선택
    - 시작 템플릿을 만들기 위해 AMI가 필요하며 커스텀한 AMI나 AWS 제공 AMI를 사용한다.
    - 인스턴스 유형 선택
    - 네트워크 설정
        - 서브넷
        - 보안 그룹
    - 인스턴스 용량 설정
    - 필요한 경우 사용자 데이터 입력
3. 인스턴스 시작 옵션
    - 네트워크
        - VPC와 서브넷을 선택
4. 고급 옵션 구성
    - 로드 밸런싱
        - 새로 로드밸런서를 생성하거나 기존 로드밸런서와 연결
        - 로드밸런서를 사용하지 않아도 되지만 그러한 사례는 찾기 어렵다.
    - 상태확인
        - EC2 인스턴스 상태 확인은 항상 활성화되어 있다.
        - ELB 상태확인을 켜면 로드 밸런서가 EC2 인스턴스의 장애를 감지했을 때 ASG가 해당 인스턴스를 종료시킬 수 있다.
5. 그룹 크기 및 크기 조정 정책 구성
    - ASG내 인스턴스의 최소, 최대, 원하는 용량을 설정할 수 있다.
    - 크기 조정 정책 설정
        - 조건에 따라 크기를 조정 여부를 결정하도록 설정
        - 설정을 하지 않을 수도 있다.
6. 알람 설정

### Dynamic Scaling Polices

- Target Tracking Scaling
    - 가장 단순하고 설정하기 쉬운 정책
    - 예시) ASG의 평균 CPU 사용률을 추적하여 수치가 일정 수준 이하 혹은 이상, 유지할 수 있도록 할 때 사용
- Simple / Step Scaling
    - CloudWatch 경보가 ASG 크기 조정에 트리거 역할을 한다.
    - 추가하거나 제거할 인스턴스의 수를 단계적으로 설정할 필요가 있다.
- Scheduled Actions
    - 파악된 사용 패턴을 바탕으로 스케일링을 예상
    - 특정 시점이나 특정 날짜에 사용량이 증가 혹은 감소할 것을 대비해 ASG 최소 크기를 해당 시간에 늘리도록 할 수 있다.
- Predictive Scaling
    - AWS 내 오토 스케일링 서비스를 활용하여 지속적으로 예측을 생성할 수 있다.
    - 로드 상태를 확인하여 다음 스케일링을 예측한다.

### 크기 조정을 위한 지표

- CPUUtilization: ASG에 속한 인스턴스의 평균 CPU 사용률
- RequestCountPerTarget : EC2 인스턴스가 안정적으로 처리할 수 있는 요청 수
- Average Network In / Out : 평균 네트워크 입출력량을 기반으로 스케일링
- CloudWatch 지표

### Scaling Cooldowns

- 인스턴스 추가 혹은 제거 후 스케일링이 실행되지 않는 시간을 의미한다.
- 기본적으로 스케일링 되고 300초 동안 스케일링을 하지 않도록 설정되어 있다.
- 새로운 인스턴스가 안정화 될 시간을 확보하고 지표의 변화 양상을 확인하기 위해 사용

### Create Scaling Polices

1. 생성한 ASG의 Auto Scaling 탭에서 크기 조정 정책을 생성할 수 있다.
    
    <img src="/images/ASG_3.png" width="50%" height="50%" title="asg 3" alt="asg 3">    
    
- 예약된 작업 생성
    1. 특정 시간에 크기 조정을 하도록 작업을 생성
    2. 최소, 최대, 원하는 용량과 시작 시간, 반복 횟수 등을 설정하여 생성
- 예측 크기 조정 정책 생성
    - 예측을 바탕으로 크기 조정 정책을 생성
    - 과거의 크기 조정을 바탕으로 머신러닝 기반 예측
    - 지표와 목표 사용률 설정
    - 예측 생성을 위해서 1주일 정도 시간이 필요하다.
- 동적 크기 조정 정책
    - 대상 크기 추적 조정, 단계 크기 조정, 단순 크기 조정의 세가지 옵션이 있다.
    - 단순 크기 조정 정책은 CloudWatch 경보가 트리거가 되며 인스턴스의 추가 및 제거가 가능
    - 단계 크기 조정 정책은 CloudWatch 경보가 트리거가 되며 단계별로 인스턴스의 추가 및 제거가 가능하다
    - 대상 추적 조정 정책은 지표를 직접 선택할 수 있고 선택한 정책으로 CloudWatch 경보를 생성할 수 있다.