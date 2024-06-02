# Amazon EFS

<img src="/images/EFS_1.png" width="50%" height="50%" title="efs 1" alt="efs 1">    

- Elastic File System의 약자로 amazon에서 제공하는 관리형 NFS(Network File System)이다.
- EBS와 달리 하나의 EFS에 서로 다른 위치의 가용 영역에 있는 여러 EC2 인스턴스가 마운트 가능
- 고가용성이면서 확장성이 뛰어나지만 비용이 매우 비싸다. - gp2 볼륨의 3배, 사용량에 따라 비용을 지불하므로 미리 프로비저닝할 필요는 없다.
- 내부적으로 NFS 프로토콜을 사용하며 EFS 액세스에 대한 보안그룹을 설정해야 한다.
- EFS는 Linux 기반 AMI와만 호환되며 Windows는 사용할 수 없다.
- POSIX 파일 시스템에 기반한 표준 파일 API를 사용
- KMS를 사용하여 암호화
- Use Case
    - 콘텐츠 관리
    - 웹 서빙
    - 데이터 공유
    - Wordpress

### Performance & Storage Classes

| 클래스 | 설명 |
| --- | --- |
| EFS Scale | 동시에 수천개의 NFS 클라이언트와 10GB/s 이상의 처리량을 확보 가능 <br> PB 규모의 네트워크 파일 시스템으로 자동 확장 가능 |
| Performance Mode | General Purpose (default) : 웹 서버나 CMS와 같이 지연 시간에 민감한 서비스에 사용 <br> MAX I/O : 처리량을 최대화하는 옵션, 지연시간이 다소 길지만 처리량이 높고 병렬성이 높아 빅데이터 애플리케이션이나 미디어 처리에 유용 |
| Throughput Mode | Bursting : 1 TB당 50MiB/s 처리량, 스토리지의 용량에 따라 처리량이 확장 <br> Provisioned : 스토리지 크기에 관계없이 처리량을 설정할 때 사용 <br> Elastic : 워크로드에 따라 처리량을 자동을 조절 <br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - 최대 3GiB/s 읽기 성능, 1GiB/s 쓰기 성능 <br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - 워크로드를 예측하기 어려울 때 유용|

[아마존 EFS 성능 - Amazon Elastic File System](https://docs.aws.amazon.com/ko_kr/efs/latest/ug/performance.html#bursting)


### Storage Classes

- Storage Tiers : 수명 주기 관리 기능이며 파일이 마지막으로 액세스 된 시간을 기준으로 일정 시간이 지나면 다른 계층으로 옮기는 기능
    - Standard : 자주 액세스하는 파일의 계층
    - Infrequent access (EFS-IA) : 자주 액세스하지 않는 파일의 계층, 파일을 검색할 때 비용이 발생하지만 데이터 저장에 사용되는 비용은 더 적다.
- Availability & Durability
    - Standard : 다중 가용 영역 사용 설정, 프로덕션 사용에 적합
    - One Zone : 한 가용 영역 사용 설정, 개발용으로 적합, 백업이 기본적으로 활성화 되어있지만 액세스 빈도가 낮은 스토리지 계층과 호환되지 않는다. 90% 정도 할인

### Create EFS

1. 파일 시스템 설정
    
    <img src="/images/EFS_2.png" width="50%" height="50%" title="efs 2" alt="efs 2">    

    - 스토리지 클래스, 백업, 수명 주기 관리 설정
    - 성능 모드에서 처리량 모드만 나와있지만 추가 설정에서 성능 모드도 설정할 수 있다.
2. 네트워크 액세스
    - EFS를 연결할 EC2 인스턴스가 있는 VPC를 선택
    - 가용 영역 별로 서브넷과 EFS 액세스가 가능한 보안 그룹을 연결
3. 파일 시스템 정책
    - 파일 시스템 액세스에 대한 정책 설정
4. 검토 및 생성

### Attach EFS to EC2

- EC2 인스턴스 생성
    1. EC2 인스턴스를 생성할 때 스토리지에 파일 시스템 편집
    2. 공유 파일 시스템 추가 및 EFS를 선택 - 네트워크에서 서브넷이 설정되어 있어야 선택 활성화됨
    3. 생성한 EFS 선택
    4. 보안 그룹을 자동으로 생성 및 연결 활성화 - 보안 그룹 설정을 자동으로 수행한다.
    5. 필요한 사용자 데이터 스크립트를 연결하여 공유 파일 시스템을 자동으로 탑재 - 예전에는 수동으로 마운드 했지만 자동으로 가능하게 되었다.

## EBS vs EFS

### EBS

- 하나의 볼륨에 하나의 인스턴스 연결
- 가용 영역 수준에서 벗어날 수 없다.
- 다른 가용 영역으로 마이그레이션
    - 볼륨 스냅샷 생성
    - 다른 가용 영역에서 스냅샷 복원
    - EBS 볼륨 복원은 IO를 사용하므로 애플리케이션이 많은 트래픽을 처리하는 경우에는 EC2 인스턴스의 성능에 영향을 미치므로 사용하지 않는다.
- Root EBS 볼륨은 EC2 인스턴스가 종료되면 기본적으로 종료된다. 비활성화 가능

### EFS

- 여러 가용 영역에 걸쳐 수백개의 인스턴스와 연결될 수 있다.
- EBS에 비해 가격이 높다.
- EFS-IA를 사용하여 비용을 절감할 수 있다.