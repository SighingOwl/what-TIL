# EBS Volume

- 인스턴스가 실행 중인 동안 연결 가능한 네트워크 드라이브로 인스턴스가 종료 되더라도 데이터를 남길 수 있도록 한다.
- 네트워크 드라이브로 인스턴스와 물리적 연결은 없지만 네트워크로 연결되어 매우 빠르게 서로 다른 인스턴스로 연결을 전환할 수 있다.
- EBS 볼륨은 하나의 가용 영역에서만 연결하여 사용할 수 있다.
- 프리 티어에서 매달 30GB가 범용 SSD나 마그네틱으로 제공된다.
- 볼륨을 프로비저닝 해야하므로 용량, IOPS를 생성할 때 미리 설정해야한다.
    - 용량이나 성능에 따라 요금이 다르며 용량의 경우 생성 이후에 변경할 수 있다.
    
    ### EBS Volume Delete on Termination attribute
    
    - EC2 인스턴스를 종료할 때 EBS를 함께 삭제할 수 있는 속성이 있다.
    - EC2 인스턴스 속성의 스토리지를 확인하면 블록 디바이스에 종료 시 삭제(Delete on Termination) 옵션을 확인할 수 있다. 이 옵션이 활성화되어 있으면 EC2 인스턴스 종료 시 EBS 볼륨도 함께 삭제된다.
    - Root 볼륨은 기본적으로 이 옵션이 활성화되어 있다.
    
    ### Create EBS Volume
    
    <img src="/images/EBS_volume_1.png" width="50%" height="50%" title="ebs volume 1" alt="ebs volume 1">   
    
    - 볼륨 유형, 크기 : 사용할 목적에 따라 적절한 유형과 크기을 선택.
    - 가용 영역 : 연결할 EC2 인스턴스와 동일한 가용 영역을 선택.
    
    <img src="/images/EBS_volume_2.png" width="50%" height="50%" title="ebs volume 2" alt="ebs volume 2">   
    
    - 볼륨 생성 후 연결할 인스턴스에 볼륨을 연결한다.
    - 볼륨을 연결한 이후 인스턴스 내부에서 볼륨을 파일 시스템 설정, 파티션 설정 등을 해주어야 하지만 SAA에서는 다루지 않는다.
    
    ### Delete EBS Volume
    
    - EBS 볼륨이 EC2 인스턴스에 연결되어 있다면 먼저 EC2 인스턴스에서 볼륨을 분리한다.
    - EBS 볼륨 삭제

## EBS Volume Type

- 6 types
    - gp2/gp3 (SSD) : 범용 SSD 볼륨으로 다양한 워크로드에 대해 가격과 성능의 절충안이 되어 준다.
    - io1/io2 (SSD) : 최고 성능으로 동작하는 SSD 볼륨으로 미션 크리티컬이면서 지연 시간이 낮은 대용량 워크로드에 사용
    - st1 (HDD) : 저비용의 HDD 볼륨, 잦은 접근과 처리량이 많은 워크로드에 사용
    - sc1 (HDD) : 가장 적은 비용의 볼륨으로 접근 빈도가 낮은 워크로드를 위해 설계됨
- EBS 볼륨 정의 방식
    - 용량, 처리량, IOPS으로 정의
- EC2 인스턴스의 부팅 볼륨은 gp2/gp3, io1/io2만 사용할 수 있다.
    
    ### General Purpose SSD
    
    - 짧은 지연시간을 가지고 비용 효율적인 스토리지
    - 시스템 부트 볼륨, 가상 데스크톱, 개발, 테스트 환경에서 사용 가능
    - 1 GiB ~ 16TiB
    - gp2
        - 용량이 작은 gp2 볼륨은 최대 3000 IOPS 성능을 가진다.
        - 용량의 크기와 IOPS 성능이 연결되어있어 용량이 1 GIB 증가할 때마다 3 IOPS가 증가하며 최대 16000 IOPS로 제한되어 계산상 최대 용량은 5334 GiB이다.
    - gp3
        - gp2와 동일한 범용 SSD 볼륨이며 gp2보다 더 나은 성능을 가지고 있다.
        - 3000 IOPS와 125 MiB/s의 처리량을 기본 성능으로 가진다.
        - 최대 16000 IOPS, 1000 MiB/s의 처리량까지 증가시킬 수 있다.
    
    ### Provisioned IOPS(PIOPS) SSD
    
    - IOPS 성능을 유지할 필요가 있는 주요 비즈니스 애플리케이션이나 16000 IOPS 이상을 요하는 애플리케이션에 적합.
    - 데이터베이스 워크로드에 적합
    - io1/io2 (4 GiB ~ 16 TiB)
        - 최대 PIOPS : 65000 for Nitro EC2 Instance & 32000 for other
        - 볼륨의 크기와 독립적으로 PIOPS를 증가시킬 수 있다.
        - io2는 io1과 동일한 비용으로 내구성과 기가 당 IOPS가 더 높다.
    - io2 Block Express (4 GiB ~ 64 TiB)
        - 조금 더 고성능의 볼륨이며 밀리초 미만의 지연시간을 가진다.
        - IOPS대 GIB 비율이 1000: 1일 때 최대 256000 PIOPS의 성능을 가진다.
    - EBS 다중 연결 지원
    
    ### Hard Disk Drives (HDD)
    
    - 부트 볼륨으로 사용할 수 없다.
    - 125 MiB ~ 16 TiB
    - st1 - 처리량 최적화 볼륨
        - 빅데이터, 데이터 웨어하우스, 로그 처리
        - 최대 500 MiB/s의 처리량, 500 IOPS
    - Cold HDD (sc1)
        - 접근 빈도가 낮은 데이터에 적합
        - 최저 비용으로 데이터를 저장할 때 사용
        - 최대 250 MiB/s의 처리량과 250 IOPS

## EBS Multi-Attach - io1/io2 family

- EBS 볼륨은 원칙적으로 하나의 EC2 인스턴스에만 연결 가능
- io1/io2 패밀리에 속한 볼륨은 예외적으로 여러 EC2 인스턴스에 연결 가능하지만 같은 가용영역 안에서만 사용할 수 있는 것은 다른 유형과 동일
- 각 인스턴스는 고성능 볼륨에 대한 읽기 및 쓰기 권한을 모두 가져 동시에 읽고 쓰는 것이 가능
- 최대 16개의 EC2 인스턴스를 하나의 볼륨에 연결할 수 있다.
- 클러스터 인식 파일 시스템을 사용
- Use Cases
    - 고가용성 애플리케이션 구동을 위해 Teradata처럼 클러스터링된 Linux 애플리케이션에서 사용
    - 애플리케이션이 동시 쓰기 작업을 관리해야 할 때 사용

## EBS Encryption

- EBS 볼륨을 생성하는 동시에 발생하는 작업
    - 저장 데이터가 볼륨 내부에서 암호화됨
    - 인스턴스와 볼륨 간의 전송 데이터 역시 암호화됨
    - 스냅샷도 암호화 되며 스냅샷으로 생성한 볼륨도 암호화됨
- 암호화 및 복호화는 모두 보이지 않게 처리
- 암호화를 사용하더라도 지연 시간에는 거의 영향이 없다.
- KMS에서 암호화 키를 생성해 AES-256 암호화 표준을 갖는다.
- 스냅샷 복사는 복호화 한 것을 다시 암호화해서 사용한다.
    
    ### Encrypt EBS Volume
    
    1. EBS 스냅샷 생성
    2. 복사 기능을 사용해 EBS 스냅샷을 암호화
    3. 암호화된 스냅샷을 활용해 볼륨을 생성하면 해당 볼륨도 암호화됨
    4. 암호화된 볼륨을 인스턴스 원본에 연결

## EBS Snapshot

- EBS 스냅샷은 EBS 볼륨의 특정 시점에 대한 백업
- EBS 스냅샷을 생성할 때 EC2 인스턴스에 EBS 볼륨을 분리하는 작업이 필요하지는 않지만 권장된다.
- EBS 스냅샷은 다른 가용 영역이나 다른 리전에 복사할 수 있다. 이것을 사용해서 동일한 형태의 인스턴스를 다른 가용 영역이나 리전에 그대로 생성할 수 있다.
- 기능
    - EBS Snapshot Archive
        - 최대 75% 저렴한 아카이브 티어로 스냅샷을 옮길 수 있는 기능
        - 복원에 24시간에서 72시간의 시간이 소요된다.
    - Recycle Bin for EBS Snapshot
        - 스냡샷을 영구 삭제하는 대신에 휴지통에 넣을 수 있다.
        - 보관 기간은 1일에서 1년사이로 설정할 수 있다.
    - FSR - Fast Snapshot Restore
        - 스냅샷을 완전 초기화하여 첫 사용에서의 지연시간을 없애는 기능
        - 스냅샷의 용량이 매우 크거나 EBS 볼륨 혹은 EC2 인스턴스를 빠르게 초기화해야 할 때 유용
        - 비용이 많이 든다.

### Create EBS Volume Snapshot

<img src="/images/EBS_volume_3.png" width="50%" height="50%" title="ebs volume 3" alt="ebs volume 3">   

- 볼륨 탭에서 스냅샷을 생성할 볼륨을 선택하여 작업에서 스냅샷 생성을 한다.
- 스냅샷은 생성할 볼륨의 크기에 따라서 생성하는 시간에 차이가 있다.

### Copy Snapshot

<img src="/images/EBS_volume_4.png" width="50%" height="50%" title="ebs volume 4" alt="ebs volume 4">   

- 다른 리전에 스냅샷을 복사하는 것은 장애 혹은 재해 복수 전략이 필요한 경우 매우 유용

### Create Volume from Snapshot

<img src="/images/EBS_volume_5.png" width="50%" height="50%" title="ebs volume 5" alt="ebs volume 5">   

- 생성한 스냅샷을 만들어서 동일한 정보를 가지고 있는 다른 볼륨을 생성할 수 있다.
- 볼륨의 유형이나 크기, 생성할 가용 영역을 다르게 설정해서 생성 가능.

### Recycle Bin

- EBS 스냅샷이나 AMI를 의도치 않은 삭제에서 보존하는 역할을 한다.   
    <img src="/images/EBS_volume_6.png" width="50%" height="50%" title="ebs volume 6" alt="ebs volume 6">   

- 휴지통에는 휴지통에 담긴 것을 보존할 때 사용하는 규칙을 정할 수 있다.
- 보존할 리소스 유형과 보존 기간을 설정한다.

## AMI

- Amazon Machine Image의 약자로 EC2 인스턴스를 통해 만든 이미지.
- AMI를 사용해서 EC2 인스턴스를 커스텀 할 수 있다.
    - Software, Configuration, OS, monitoring과 같은 기능을 추가 가능.
    - EC2 인스턴스에 설치하고자 하는 모든 소프트웨어를 미리 패키징 하여 부팅이나 설정에 소요되는 시간을 줄일 수 있다.
- AMI는 직접 구성하여 사용해야 하고 특정 리전에서 생성한 다음 다른 리전에서 복사할 수 있다.
- 종류
    - Public AMI : AWS에서 제공
    - 자체 AMI : 사용자가 직접 만들고 관리하는 AMI
    - AWS Marketplace AMI : 다른 사용자가 만든 AMI - 비용이 청구될 수 있고 만들어 판매할 수도 있다.

### AMI Process

1. EC2 인스턴스를 생성한 후 필요한 소프트웨어 설치나 설정 작업을 하는 커스텀 진행
2. 데이터 무결성을 위해 인스턴스를 중지
3. AMI 빌드 - 빌드과정에서 EBS Snapshot이 생성된다.
4. 빌드한 AMI를 사용해서 인스턴스를 시작할 수 있다.

### Create AMI

1. AMI를 생성할 인스턴스를 선택해서 이미지 생성
    
    <img src="/images/EBS_volume_7.png" width="50%" height="50%" title="ebs volume 7" alt="ebs volume 7">   
    
2. 인스턴스 Root 볼륨 용량을 변경하거나 새로운 볼륨을 추가할 수 있다.
3. 생성한 AMI는 AMI 탭에서 확인할 수 있고 상태가 사용 가능일 때 사용할 수 있다.
4. EC2 인스턴스 생성시 내 AMI를 선택하여 생성할 수 있다.