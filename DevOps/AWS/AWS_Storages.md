# AWS Storages
AWS는 S3나 EBS, EFS외에도 다른 스토리지 서비스를 제공하고 있습니다. 대용량 데이터를 AWS로 안전하고 빠르게 마이그레이션이 필요하거나 인터넷 연결이 힘든 환경에서도 AWS 서비스를 제공하는 스토리지 서비스, HPC에 사용할 수 있는 고성능 파일 시스템을 지원하는 서비스, 하이브리드 클라우드를 위한 스토리지 서비스, 스토리지 파일 전송 및 동기화 서비스 등 부가적인 스토리지 서비스를 제공하여 AWS로 마이그레이션이나 하이브리드 클라우드 구성을 돕습니다.

## AWS Snow Family
- 엣지에서 데이터를 모으고 처리하고 AWS 외부에서 AWS로 데이터를 마이그레이션하기 위한 매우 안전한 휴대기기를 의미한다.

### Data Migration

- 종류
    - Snowcone
    - Snowball Edge
    - Snowmobile
- 네트워크를 통해 대량의 데이터를 전송하는데 걸리는 시간은 꽤 오래 걸릴 수 있다.
- 대용량 데이터 마이그레이션에 발생하는 문제점
    - 연결 제한
    - 대역폭 제한
    - 데이터 전송을 위한 높은 네트워크 비용
    - 대역폭 공유 - 전체 대역폭을 사용할 수 없다.
    - 연결 불안정 - 다시 전송을 시작해야하는 경우가 발생할 수 있다.
- Snow Family는 데이터 마이그레이션을 할 수 있게 해주는 오프라인 기기이다.
    - AWS는 우편으로 실제 물리장치를 배송해준다.
    - 사용자는 데이터를 물리장치에 로딩 후 AWS에 반송한다.
- 네트워크를 사용한 데이터 마이그레이션에 일주일 이상이 소요된다면 Snowball 기기를 사용해야한다.

### Snowball Edge

- 거대한 박스 형태
- 테라바이트나 페타바이트 용량의 데이터를 AWS로 물리적으로 전송하기 위해 사용
- 네트워크를 통하지 않고 장치에 저장 후 옮긴다.
- 전송 작업 당 비용이 청구
- 블록 스토리지나 Amazon S3와 호환 가능한 객체 스토리지를 제공
- 유형
    - Storage Optimized
        - 80TB 용량의 HDD가 제공
        - 블록 볼륨이나 S3 호환 객체 스토리지
    - Compute Optimized
        - 42TB HDD or 28TB NVMe가 제공
        - 블록 볼륨이나 S3 호환 객체 스토리지
- Use case
    - 대규모 데이터의 클라우드 마이그레이션
    - 데이터센터 폐지 또는 재해 복구를 위해 AWS로 데이터 백업

### Snowcone

- 아주 작은 휴대 기기이며 열악한 환경에서도 견딜만큼 견고하고 안전하다.
    - 2.1kg이며 원한다면 드론에 탑재 가능
- 데이터의 양이 적은 환경에서 사용
- 에지 컴퓨팅, 스토리지, 데이터 전송에 사용
- 유형
    - 8TB HDD
    - 14TB SSD
- Snowball이 적합하지 않은 경우에 Snowcone을 사용
    - 공간 제약이 있는 환경
    - 베터리와 케이블을 제공할 수 있는 환경
- AWS 전송 방법
    - 오프라인으로 데이터를 발송
    - 기기가 인터넷에 연결가능할 때 데이터 센터에 연결하는 방법
        - AWS DataSync 서비스를 사용해서 데이터를 다시 AWS에 전송

### Snowmobile

- 실제 트럭이다. 개꿀
- 엑사바이트급 데이터를 옮길 수 있다.
- Snowmobile 한대당 100PB 용량을 옮길 수 있다.
- 보안이 뛰어나다.
    - 공조장치
    - GPS
    - 24시간 비디오 감시
- 10PB 이상의 데이터를 옮길 경우에 Snowball보다 더 좋다.

### Usage Process

1. AWS 콘솔에서 기기 배송 요청
2. Snowball 클라이언트 또는 AWS OpsHub 서버 설치
3. Snowball을 서버에 연결 후 클라이언트 안에서 파일 복사
4. 장치가 준비되면 기기를 AWS로 반송
    1. 전자 마커가 있어 올바른 AWS 시설로 기기가 곧바로 배송
5. S3 버킷에 데이터가 로딩
6. 최고 수준의 보안 조치에 따라 Snowball에서 데이터를 완전히 제거

### Edge computing

- 본래 목적은 데이터 마이그레이션이 전부이지만 엣지 로케이션에서 컴퓨팅 성능을 제공할 수도 있다.
- 종류
    - Snowcone
        - 2개 CPU, 4GB RAM, 유무선 액세스
        - USB-C 전원 혹은 베터리 옵션
    - Snowball Edge - Compute Optimized
        - 104개 vCPU, 416GiB RAM
        - GPU 선택 - 영상 작업이나 머신러닝 용
        - 28TB NVMe or 42TB HDD
        - 스토리지 클러스터링이 16개 노드로 전체 스토리지 용량을 늘릴 수 있다.
    - Snowball Edge - Storage Optimized
        - 40개 vCPU, 80GiB RAM, 80TB 스토리지
    - 모든 기기들은 EC2 인스턴스와 람다 함수를 실행할 수 있다.
        - 람다 함수를 위해 AWS IoT Greengrass 서비스를 활용
- 엣지 컴퓨팅은 엣지 로케이션에서 데이터를 생성하는 중에 그 데이터를 처리하는 것을 의미
- 엣치 로케이션은 인터넷이 없거나 클라우드에서 멀리 떨어져 있는 모든 위치를 의미
    - 트럭이나 바다에 있는 배
    - 이러한 위치는 데이터가 생산되지만 인터넷 연결이 제한되거나 컴퓨팅 능력에 액세스 할 수 없는 곳
- 엣지 로케이션에서 엣지 컴퓨팅을 위해 Snowball Edge나 Snowcone을 주문하여 사용가능
- Use case
    - 데이터 전처리
    - 엣지에서 머신러닝
    - 미디어 스트림의 사전 트랜스코딩
- 데이터를 AWS로 보내야한다면 장치를 AWS로 반송한다.
- 데이터가 생성되는 곳 가까이서 데이터를 처리하고 AWS로 반송
- 장기간 사용할 수 있으므로 렌트할 수 있고 1년 또는 3년까지 할인된 가격으로 사용할 수 있다.

### OpsHub

- 과거에 엣지 장치를 사용하기 위해서는 CLI를 사용 해야했지만 OpsHub를 사용하면 GUI 환경에서 엣지 장치를 사용할 수 있다.
- 클라우드가 아닌 로컬 장치에 설치해서 사용
- 싱글 혹은 클러스터 기기를 열어서 설정
- 파일 전송
- 인스턴스 시작 및 관리
- 기기와 메트릭을 관리
- 호환되는 AWS 서비스 사용

### Snowball into Glacier

- Snowball을 바로 Glacier로 불러올 수 없다.
- S3의 수명 주기 정책을 생성하여 Amazon Glacier로 객체를 전환할 수 있다.

## Amazon FSx

- AWS에서 제공하는 완전 관리형 서비스
- 타사 고성능 파일 시스템을 실행
- 종류
    - FSx for Lustre
    - FSx for Windows File Server
    - FSx for NetApp ONTAP
    - FSx for OpenZFS
    - etc

### FSx for Windows File Server

- 완전 관리형 Windows 파일 서버 공유 드라이버
- SMB 프로토콜과 Windows NTFS 지원
- Microsoft Active Directory를 지원하므로 사용자 보안을 추가할 수 있고 ACL로 사용자 할당량을 추가해 액세스를 제어
- Windows뿐만 아니라 Linux EC2 인스턴스에도 마운트할 수 있다.
- 기존 On-premise Windows 파일 서버가 있는 경우 Microsoft DFS - Distributed File System 기능을 사용해서 파일 시스템을 그룹화할 수 있다.
    - 이 기능으로 On-premise Windows 파일 서버와 FSx for Windows File Server를 결합할 수 있다.
- 10s of GB/s, millions of IOPS, 100s PB of Data까지 확장될 수 있다.
- Storage Options
    - SSD - 지연시간이 짧아야 하는 워크로드를 저장 - DB, 미디어 처리, 데이터 분석
    - HDD - 넓은 스펙트럼의 워크로드를 저장 - home 디렉토리, CMS
- VPN이나 Direct Connect 같은 프라이빗 연결로 On-premise 인프라에 액세스 가능
- 고가용성 다중 AZ에 대해 구성 가능
- 모든 데이터는 재해 복구 목적으로 Amazon S3에 매일 백업

### FSx for Lustre

- Lustre = Linux + Cluster
- Lustre는 원래 분산 파일 시스템으로 대규모 연산에 사용
- 머신 러닝, HPC에 사용
- 동영상 처리, 금융 모델링, 전자 설계 자동화 등의 애플리케이션에 사용
- 100s GB/s, millions of IOPS, sub-ms 지연시간까지 확장할 수 있다.
- Storage Options
    - SSD - 낮은 지연 시간, IOPS 특화 워크로드, 크기가 작고 무작위 파일 작업, HDD보다 비쌈
    - HDD - 처리량 특화 워크로드, 크고 시퀸셜 파일 작업
- S3와 무결정성 통합 가능
    - FSx로 S3를 파일 시스템처럼 읽어들일 수 있다.
    - FSx의 연산 출력값을 다시 S3에 쓸 수 있다.
- VPN이나 Direct Connect 같은 프라이빗 연결로 On-premise 인프라에 액세스 가능

### Deployment Options

- 스크래치 파일 시스템
    
    <img src="/images/AWS_Storage_1.png" width="75%" height="75%" title="aws storage 1" alt="aws storage 1">    
    
    - 임시 스토리지로 데이터가 복제되지 않는다.
    - 서버가 오작동하면 파일이 모두 유실
    - 최적화로 초과 버스트를 사용 가능
        - 영구 파일 시스템보다 성능을 6배 높일 수 있다.
        - 200MBps per TiB
    - 단기 처리 데이터에 사용, 데이터 복제가 없어 비용 최적화 가능
- 영구 파일 시스템
    
    <img src="/images/AWS_Storage_2.png" width="75%" height="75%" title="aws storage 2" alt="aws storage 2">    
    
    - 장기 스토리지 동일한 가용 영역에 데이터가 복제
    - 서버가 오작동하면 몇 분내에 해당 파일이 대체된다.
    - 민감한 데이터의 장기 처리 및 스토리지

### FSx for NetApp ONTAP

- AWS 관리형 NetApp ONTAP
- NFS, SMB, iSCSI 프로토콜과 호환 가능
- On-premise 시스템의 ONTAP이나 NAS에서 실행 중인 워크로드를 AWS로 옮길 수 있다.
- 지원 플랫폼
    - Linux
    - Windows
    - MacOS
    - VMware Cloud on AWS
    - Amazon Workspaces & AppStream 2.0
    - Amazon EC2, ECS and EKS
- 스토리지는 자동으로 스케일링 된다.
- 복제와 스냅샷 기능 지원
    - 비용이 적게 든다.
    - 데이터 압축이나 중복제거 가능
- 지정 시간 복제 기능
    - 새 워크로드를 테스트할 때 유용

### FSx for OpenZFS

- AWS 관리형 OpneZFS
- 여러 버전의 NFS 프로토콜과 호환 가능
- 주로 ZFS에서 실행되는 워크로드를 내부적으로 AWS로 옮길 때 사용
- 지원 플랫폼
    - Linux
    - Windows
    - MacOS
    - VMware Cloud on AWS
    - Amazon Workspaces & AppStream 2.0
    - Amazon EC2, ECS and EKS
- 백만 IOPS까지 확장 가능, 0.5ms이하 지연시간
- 스냅샷, 압축 지원
    - 비용이 적게 든다
    - 데이터 중복 제거 기능은 지원하지 않음
- 지정 시간 동시 복제 기능
    - 새 워크로드 테스트에 유용

### Create FSx

1. 사용할 파일 시스템 유형 선택 - Lustre
    
    <img src="/images/AWS_Storage_3.png" width="75%" height="75%" title="aws storage 3" alt="aws storage 3">    
    
2. 파일 시스템 세부 정보 지정
    
    <img src="/images/AWS_Storage_4.png" width="75%" height="75%" title="aws storage 4" alt="aws storage 4">    
    
    1. 배포 및 스토리지 유형 선택
        - 영구, SSD
        - 영구, HDD
        - 스크래치, SSD
    2. 스토리지 단위당 처리량, 용량 선택
    3. VPC 및 VPC 보안 그룹 선택
    4. 암호화 수준 설정

## Storage Gateway
### Hybrid Cloud

- AWS는 하이브리드 클라우드를 권장한다.
- 일부 인프라는 AWS 클라우드에 있고 나머지는 on-premise에 두는 방식
- 사용 이유
    - 클라우드 마이그레이션이 오래 걸린다.
    - 보안 또는 규정 준수 요건이 있는 경우
    - 전략에 따라 엘라스틱 워크로드에만 클라우드를 적용
- S3는 독점 스토리지 기술로 NFS 규정 준수 파일 시스템인 EFS와 다르다.
- S3 스토리지에 있는 데이터를 on-premise에 두려면 AWS Storage Gateway를 사용한다.

### AWS Storage Cloud Native Options

- Block
    - Amazon EBS
    - EC2 Instance Store
- File
    - Amazon EFS
    - Amazon FSx
- Odject
    - Amazon S3
    - Amazon Glacier

### AWS Storage Gateway

- On-premise와 클라우드 데이터 간의 가교 역할을 한다.
- Use case
    - 재해 복구 목적으로 on-premise 데이터를 클라우드로 백업
    - 백업 및 복구 목적으로 클라우드 마이그레이션 혹은 온프레미스에서 클라우드 간 스토리지 확장
    - 콜드 데이터는 클라우드에 웜 데이터는 on-premise에 둔다.
    - 대부분의 데이터를 AWS에 저장하고 파일 액세스 지연 시간을 줄이기 위해 storage gateway를 on-premise의 캐시로 사용
- 유형
    - S3 File Gateway
    - FSx File Gateway
    - Volume Gateway
    - Tape Gateway

### S3 File Gateway

- S3 Glacier를 제외한 모든 클래스를 연결할 수 있다.
- 사용 절차
    1. 애플리케이션 서버가 동작 중인 데이터센터에 S3 File Gateway를 생성
    2. 애플리케이션 서버와 S3 File Gateway간 프로토콜은 NFS나 SMB를 사용하도록 설정
    3. S3 File Gateway는 NFS나 SMB 프로토콜을 HTTPS 요청으로 변환시켜 S3로 전송
    4. 애플리케이션 서버에서 보기에는 일반적인 파일 공유 액세스로 보이지만 실제로는 Amazon S3를 사용하는 것이다.
    5. 데이터를 아카이브 하기 위해서는 AWS에서 수명 주기 정책을 활용하여 Glacier로 객체를 이동시킨다.
- S3 버킷의 데이터는 NFS나 SMB 프로토콜을 사용해서 액세스 가능
- 사용된 데이터는 신속한 액세스를 위해 file gateway에 캐시된다.
    - 전체 데이터가 아닌 최근에 사용한 데이터만 있다.
- 버킷에 액세스하기 위해 각 file gateway마다 IAM 역할을 생성
- Windows 파일 시스템 네이티브인 SMB 프로토콜을 사용하는 경우 사용자 인증을 위해 AD와 통합해야한다.

### FSx File Gateway

- FSx for Windows File Server에 네이티브 액세스가 가능하다.
- 사용절차 - FSx for Windows File System이 Amazon FSx 파일 시스템에 배포되어 있고 회사 데이터 센터의 SMB 클라이언트가 액세스 하려는 경우
    1. 
- FSx for Windows File System에 액세스하려는 경우 File Gateway를 생성하지 않아도 되지만 로컬 캐시를 확보하여 더 빠른 액세스가 가능하기 때문
- SMB, NTFS, AD과 호환 가능
- 그룹 파일 공유나 on-premise를 연결할 홈 디렉토리로 사용 가능

### Volume Gateway

- 블록 스토리지로 S3가 백업하는 iSCSI 프로토콜을 사용
- 볼륨이 EBS 스냅샷으로 저장하여 필요에 따라 on-premise 볼륨을 복구할 수 있다.
- 유형
    - Cached volume : 데이터 액세스 시 지연시간이 낮다.
    - Stored volume : 전체 데이터 세트가 온프레미스에 있어 주기적인 Amazon S3 백업 진행

### Tape Gateway

- 물리 테이프를 사용한 백업 프로세스를 회사가 클라우드를 활용하여 데이터를 백업할 수 있도록 한다.
- VTL - Virtual Tape Library는 백업을 Amazon S3나 Glacier에 사용
- 테이프 기반 프로세스의 기본 백업 데이터를 iSCSI 인터페이스로 백업
- 업계를 선도하는 백업 소프트웨어 벤더가 사용하는 서비스

### Storage Gateway - Hardware appliance

- 모든 Gateway는 회사의 데이터 센터에 설치되어야 한다.
- 하지만 gateway를 설치할 가상 서버가 없는 경우가 종종 있는데 이 경우 AWS의 하드웨어를 사용할 수 있다. - Amazon.com에서 주문
- 장치가 작동하기 위한 CPU, 메모리, 네트워크 SSD 캐시 리소스가 필요하다.
- 소규모 데이터 센터의 일일 NFS 백업처럼 가상화가 없는 경우 상당히 유용

## Transfer Family

- Amazon S3 또는 EFS의 안팎으로 데이터를 전송하려고 할 때 FTP 프로토콜을 사용하려고 할 때 사용
- 지원 프로토콜
    - AWS Transfer for FTP
    - AWS Transfer for FTPS - SSL을 사용하는 암호화된 전송
    - AWS Transfer for SFTP - 보안 파일 전송
- 완전 관리형 인프라이며 확장성, 안정성, 가용성이 높다.
- 가격 : 시간당 프로비저닝된 엔드 포인트 비용에 전송 제품군 안팎으로 전송된 데이터의 GB당 요금
- 서비스 내에서 사용자 자격 증명을 저장 및 관리 가능
- 아래의 인증 시스템과 통합 가능
    - Microsoft Active Directory
    - LDAP
    - Okta
    - Amazon Cognito
    - Custom
- Usage
    - Amazon S3나 EFS의 FTP 인터페이스를 갖기 위해서 사용
    - 파일 공유 및 공개 데이터셋 공유
    - CRM
    - ERP

## DataSync

- 데이터를 동기화를 통해 대용량의 데이터를 한 곳에서 다른 곳으로 옮길 수 있다.
    - On-premise나 AWS외 다른 클라우드로 데이터를 옮길 수 있다.
        - 서버를 NFS, SMB, HDFS 또는 다른 프로토콜로 연결
        - 옮길 위치인 on-premise나 연결할 다른 클라우드에 에이전트가 있어야 한다.
    - 한 AWS 서비스에서 다른 AWS 서비스로 데이터를 옮길 때 사용
        - 에이전트는 필요없다.
- 동기화 가능 리소스
    - Amazon S3 - Glacier 포함
    - Amazon EFS
    - Amazon FSx
- 복제 작업은 스케줄에 따라 매 시간, 매일 혹은 매주 실행되도록 할 수 있다. - 지연이 발생할 수 있다.
- 파일 권한과 메타데이터 저장 기능
    - NFS POSIX 파일 시스템
    - SMB 권한을 준수
- 에이전트 하나의 테스크가 초당 10GB까지 사용 가능하며 네트워크 성능 초과를 방지하기 위해 대역폭에 제한을 걸 수 있다.

### AWS 스토리지 서비스 간 동기화

- AWS DataSync를 사용하여 데이터 복사본을 만든다.
- 서로 다른 스토리지 서비스 간 메타데이터 유지

## Storage Comparison
### S3

- 객체 스토리지
- 구체적 API
- 대부분 AWS 서비스와 연결 가능

### S3 Glacier

- 객체 아카이브

### EBS Volume

- 한 번에 한 개의 EC2 인스턴스에만 스토리지를 연결
- io1/io2는 다중 연결 지원

### Instance Storage

- EC2 인스턴스에 높은 IOPS를 가지는 고성능 물리 스토리지가 필요할 때 사용

### EFS

- 인스턴스가 네트워크 파일 시스템을 필요로 할 때 사용
- 다중 가용 영역 마운트를 사용해야 하면서 POSIX 파일 시스템을 사용해야할 때 사용

### FSx for Windows

- Windows 서버 파일 시스템을 필요로 하는 경우

### FSx for Lustre

- 고성능 연산 Linux 파일 시스템이며 Lustre 클라이언트와 호환 가능해야하는 경우 사용

### FSx for NetApp ONTAP

- 높은 OS 호환성과 네트워크 파일 시스템이 필요할 때 사용

### FSx for OpenZFS

- 관리형 ZFS 파일 시스템이 필요할 때 사용

### Storage Gateway

- On-premise와 AWS 간 스토리지 연결 방법

### Transfer Family

- S3나 EFS외에 FTP, FTPS, SFTP 인터페이스가 필요할 때 사용

### DataSync

- 온프레미스에서 AWS, AWS에서 AWS로 일정에 따라 데이터를 동기화할 때 DataSync를 사용

### Snow Family

- 데이터를 옮기는데 쓸 네트워크 용량이 없어 물리적으로 대용량의 데이터를 옮겨야 할 때는 Snowcone / Snowball / Snowmobile 장치를 주문해서 온프레미스에 설치한 다음 클라우드로 이전
- Snowcone은 DataSync 에이전트가 사전 설치되어 온다

### Database

- 데이터 저장은 가능하나 인덱스와 쿼리 작업을 필요로 하는 특수한 워크로드가 있다.

## 참고 자료    
[AWS Docs - AWS Snow 패밀리](https://docs.aws.amazon.com/ko_kr/whitepapers/latest/how-aws-pricing-works/aws-snow-family.html)      
[AWS Docs - AWS Snowball 엣지는 무엇인가요?](https://docs.aws.amazon.com/ko_kr/snowball/latest/developer-guide/whatisedge.html)    
[AWS Docs - AWS Snowcone는 무엇인가요?](https://docs.aws.amazon.com/ko_kr/snowball/latest/snowcone-guide/snowcone-what-is-snowcone.html)   
[AWS 한국 블로그 - AWS Snowmobile – 엑사 바이트(Exabyte) 데이터를 몇 주 만에 클라우드로](https://aws.amazon.com/ko/blogs/korea/aws-snowmobile-move-exabytes-of-data-to-the-cloud-in-weeks/)     
[AWS Docs - FSx for Windows File Server란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/fsx/latest/WindowsGuide/what-is.html)      
[AWS Docs - Amazon FSx for Lustre란?](https://docs.aws.amazon.com/ko_kr/fsx/latest/LustreGuide/what-is.html)        
[AWS Docs - ONTAP용 아마존 NetApp FSx란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html)        
[AWS Docs - What is Amazon FSx for OpenZFS?](https://docs.aws.amazon.com/ko_kr/fsx/latest/OpenZFSGuide/what-is-fsx.html)        
[AWS Docs - AWS Storage Gateway 설명서](https://docs.aws.amazon.com/ko_kr/storagegateway/)      
[AWS Docs - AWS Transfer Family 무엇입니까?](https://docs.aws.amazon.com/ko_kr/transfer/latest/userguide/what-is-aws-transfer-family.html)      
[AWS Docs - AWS DataSync란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/datasync/latest/userguide/what-is-datasync.html)  