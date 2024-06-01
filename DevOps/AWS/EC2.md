# EC2란?
Elastic Compute Cloud의 약자로 AWS에서 IaaS로 제공하는 컴퓨팅 자원이고 AWS에서 가장 많이 사용하는 서비스 중 하나이다. EC2는 EC2 인스턴스를 의미하기도 하지만 EC2에는 아래와 같은 서비스가 포함되어 있다.
| 서비스 | 설명 |
| --- | --- |
| EC2 | 임대형 VM 서비스 |
| EBS | Elastic Block Storage의 약자로 가상 드라이브에 데이터를 저장하는 서비스 |
| ELB | Elastic Load Balancer의 약자로 로드벨런서 서비스 |
| ASG | Auto Scaling Group의 약자로 EC2 인스턴스의 수를 오토스케일링 정책에 따라 스케일링 하는 서비스 | 

EC2 인스턴스를 구성할 때는 OS, 인스턴스 유형, 스토리지 유형, 네트워크, 보안 설정이 필요하다. OS는 Linux, Windows, Mac OS 중 선택할 수 있고 인스턴스 유형을 선택해여 CPU 성능과 RAM 용량을 선택할 수 있다. 스토리지는 네트워크로 연결된 EBS나 EFS 혹은 EC2와 하드웨어적으로 연결된 EC2 인스턴스 스토어를 선택하고 필요한 용량과 성능을 설정 가능하다. 네트워크는 EC2 인스턴스가 생성될 VPC와 서브넷, 퍼블릭 IP 할당을 설정하고 보안그룹을 생성해 방화벽으로 활용한다.

### EC2 사용자 데이터
만일 EC2 인스턴스를 생성할 때 미들웨어 설치, 환경 변수 설정 같은 프로비저닝이 필요하다면 사용자 데이터에 스크립트를 작성하여 자동화할 수도 있다. 사용자 데이터는 EC2 인스턴스를 생성할 때 단 한번만 실행되며 다음 부팅부터는 실행되지 않는다. 사용자 데이터로 실행되는 명령은 모두 루트 권한으로 실행된다.

## EC2 인스턴스 생성
AWS 콘솔에서 아래의 절차대로 EC2 인스턴스를 생성할 수 있다.     
    <img src="/images/EC2_creation_1.png" width="50%" height="50%" title="create ec2 instance 1" alt="create ec2 instance 1">   
1. 이름 및 태그 설정
이름만 설정해도 상관없지만 태그를 설정하면 EC2 인스턴스 관리가 용이해진다.      
    <img src="/images/EC2_creation_2.png" width="50%" height="50%" title="create ec2 instance 2" alt="create ec2 instance 2">   
2. 애플리케이션 및 OS 이미지
EC2 인스턴스에 사용할 OS를 선택한다. OS는 AMI라고 하는 이미지로 제공되며 AMI는 아무설정이 되어 있지 않는 OS 이미지부터 웹 서버나 DB 서버, 딥러닝 용도 등으로 커스텀 된 이미지를 선택가능하다. 또한 필요에 따라서 x86이나 arm과 같은 아키텍처를 선택할 수도 있다.        
    <img src="/images/EC2_creation_3.png" width="50%" height="50%" title="create ec2 instance 3" alt="create ec2 instance 3">   
3. 인스턴스 유형을 선택한다. EC2 인스턴스의 성능을 선택하는 것으로 유형에 따라 제공되는 CPU 코어 수와 RAM의 용량이 다르고 같은 유형의 인스턴스를 사용해도 사용하는 OS에 따라 요금이 달라지기도 한다.        
    <img src="/images/EC2_creation_4.png" width="50%" height="50%" title="create ec2 instance 4" alt="create ec2 instance 4">   
4. 키 페어 설정을 수행한다. 반드시 할 필요는 없지만 SSH를 통한 인스턴스 액세스를 위해 필요하다. Ubuntu의 경우 SSH의 기본 설정이 Key pair를 사용한 로그인만 허용하므로 설정을 바꿀 것이 아니라면 반드시 필요하다. 기존에 사용하는 키 페어를 선택하거나 새로 생성하여 사용한다.       
    <img src="/images/EC2_creation_5.png" width="50%" height="50%" title="create ec2 instance 5" alt="create ec2 instance 5">   
    <img src="/images/EC2_creation_6.png" width="50%" height="50%" title="create ec2 instance 6" alt="create ec2 instance 6">   
5. 네트워크 설정에서 EC2 인스턴스가 생성될 VPC와 서브넷을 설정한다. 또한 보안 그룹이라는 방화벽을 선택하거나 생성하여 인바운드, 아웃바운드 트래픽 제어가 가능하다.      
    <img src="/images/EC2_creation_7.png" width="50%" height="50%" title="create ec2 instance 7" alt="create ec2 instance 7">   
6. 스토리지 구성에서 볼륨의 크기와 유형을 선택하거나 새로운 볼륨을 추가한다. 고급 설정에서 IOPS나 암호화, 처리량을 설정할 수 있다. 특이하게 종료시 삭제라는 항목이 있는데 EC2 인스턴스가 삭제될 때 함께 삭제되는 여부를 선택하는 것     
    <img src="/images/EC2_creation_8.png" width="50%" height="50%" title="create ec2 instance 8" alt="create ec2 instance 8">   
    <img src="/images/EC2_creation_9.png" width="50%" height="50%" title="create ec2 instance 9" alt="create ec2 instance 9">   
7. 고급 세부 설정의 가장 아래에 사용자 데이터 항목에 EC2 인스턴스가 생성될 때 실행할 스크립트를 입력한다. 반드시 넣을 필요는 없다.      
    <img src="/images/EC2_creation_10.png" width="50%" height="50%" title="create ec2 instance 10" alt="create ec2 instance 10">   

EC2 인스턴스를 시작하면 AWS 콘솔에 생성한 인스턴를 확인할 수 있다. EC2 인스턴스는 실행 중일 때만 요금이 부과된다. 따라서 중지된 EC2에는 요금이 부과되지 않는다. EC2 인스턴스를 삭제하고 싶다면 EC2 인스턴스를 종료한다.     
    <img src="/images/EC2_creation_11.png" width="50%" height="50%" title="create ec2 instance 11" alt="create ec2 instance 11">   
    <img src="/images/EC2_creation_12.png" width="50%" height="50%" title="create ec2 instance 12" alt="create ec2 instance 12">   

## EC2 인스턴스 유형
EC2 인스턴스의 성능을 인스턴스 유형에의해 결정된다. 인스턴스 유형은 인스턴스 사용 목적에 따라 여러 유형으로 나뉘어져 있고 인스턴스 유형 이름에서 대략적인 클래스와 세대, 성능을 확인할 수 있다.
> t2.micro  
t : 인스턴스 클래스     
2 : 세대    
micro : 인스턴스 클래스의 크기 (혹은 성능)  

인스턴스의 클래스는 인스턴스가 사용되는 목적으로 분류한 것으로 사용자는 이를 보고 적절한 인스턴스 클래스를 결정할 수 있다.
| 유형 | 설명 | 클래스 종류 |
| --- | --- | --- |
| 범용 | 균형 있는 컴퓨팅, 메모리 및 네트워킹 리소스를 제공하며, 다양한 여러 워크로드에 사용할 수 있습니다. 이 인스턴스는 웹 서버 및 코드 리포지토리와 같이 이러한 리소스를 동등한 비율로 사용하는 애플리케이션에 적합 | M, T |
| 컴퓨팅 최적화 | 고성능 프로세서를 활용하는 컴퓨팅 집약적인 애플리케이션에 적합하며 이 범주에 속하는 인스턴스는 배치 처리 워크로드, 미디어 트랜스코딩, 고성능 웹 서버, 고성능 컴퓨팅(HPC), 과학적 모델링, 전용 게임 서버 및 광고 서버 엔진, 기계 학습 추론 및 기타 컴퓨팅 집약적인 애플리케이션에 매우 적합. | C |
| 메모리 최적화 | 메모리에서 대규모 데이터 세트를 처리하는 워크로드를 위한 빠른 성능을 제공하기 위해 설계 | R, X, z, u- |
| 가속 컴퓨팅 | 하드웨어 액셀러레이터 또는 코프로세서를 사용하여 부동 소수점 수 계산이나 그래픽 처리, 데이터 패턴 일치 등의 기능을 CPU에서 실행되는 소프트웨어보다 훨씬 효율적으로 수행 | P, G, Trn, Inf, DL, F, VT |
| 스토리지 최적화 | 로컬 스토리지에서 매우 큰 데이터 세트에 대해 많은 순차적 읽기 및 쓰기 액세스를 요구하는 워크로드를 위해 설계됨. 이러한 인스턴스는 애플리케이션에 대해 대기 시간이 짧은, 수만 단위의 무작위 초당 I/O 작업 수(IOPS)를 지원하도록 최적화됨 | I, D, H |
| HPC 최적화 | AWS에서 HPC 워크로드를 대규모로 실행할 때 최고의 가격 대비 성능을 제공, HPC 인스턴스는 대규모의 복잡한 시뮬레이션 및 딥 러닝 워크로드와 같이 고성능 프로세서가 유용한 애플리케이션에 적합 | Hpc |

## 보안 그룹
보안 그룹은 AWS에서 네트워크 보안을 실행하는데 핵심이 되며 EC2 인스턴스에서 오고가는 트래픽을 제어하는 방화벽 역할을 수행한다. 특이한 점은 로드밸런서와 달리 거부 조건 없이 허용 조건만 사용한다. 규칙을 생성할 때는 IP 주소나 다른 보안 그룹을 참조할 수 있다.
### 보안 그룹이 제어하는 요소
- 포트 액세스
- 특정 대역의 IP 주소
- 인바운드 트래픽
- 아웃바운드 트래픽 

    <img src="/images/EC2_securtiy_group_1.png" width="50%" height="50%" title="ec2 security group 1" alt="ec2 security group 1">   

보안 그룹은 여러 인스턴스에 연결될 수 있고 인스턴스 외부에 위치하여 보안 그룹이 트래픽을 거부해도 EC2는 알 수 없다. 따라서 EC2 인스턴스에 접속시 타임아웃이 발생하면 EC2 내부의 문제보단 보안 그룹에 문제가 있을 가능성이 높아 보안 그룹을 우선적으로 확인해야한다. 보안 그룹을 생성하면 기본적으로 모든 인바운드 트래픽은 허용되어 있지 않고 모든 아웃바운드 트래픽은 허용되어 있다. 보안 그룹은 VPC 내부에 묶여있으므로 다른 리전이나 VPC에서 사용할 수 없다.

### 다른 보안 그룹 참역
보안 그룹은 IP 대역을 참조하여 트래픽을 허용하도록 되어있지만 다른 보안 그룹을 참조해서 트래픽을 허용할 수 있다. 쉽게 말하면 다른 보안 그룹에서 이미 허용한 아웃바운드 트래픽을 허용하도록 참조하는 것이다. 이를 사용하면 보안 그룹을 생성할 때 매번 IP 주소를 고려하지 않아도 되는 장점이 있다.
    <img src="/images/_referencing_other_security_groups.drawio.png" width="50%" height="50%" title="referencing other sg" alt="referencing other sg">   

### 보안 그룹 생성
보안 그룹을 생성하는 방법은 2가지이다. 첫번째는 EC2 인스턴스를 생성할 때 새로 생성하는 방법이고 두번째는 보안 그룹 탭에서 새로운 보안그룹을 생성하는 방법이다.
EC2 생성할 때 이미 보안 그룹 생성을 했기 때문에 이번에는 보안 그룹 탭에서 보안 그룹을 새로 생성하는 절차를 아래에 설명한다.
1. 보안 그룹 기본 정보
보안 그룹의 이름과 보안 그룹 설명을 작성하고 생성할 보안 그룹을 생성할 VPC를 선택한다. 보안 그룹은 VPC에 종속적이기 때문에 반드시 VPC를 선택해야 한다.
    <img src="/images/SG_creation_1.png" width="50%" height="50%" title="create sg 1" alt="create sg 1">   
2. 인바운드 규칙 설정
EC2 인스턴스로 들어가는 트래픽을 허용하기 위한 규칙을 설정한다. 기본적으로 모든 트래픽이 허용되어 있지 않다.
    <img src="/images/SG_creation_2.png" width="50%" height="50%" title="create sg 2" alt="create sg 2">   
3. 아웃바운드 규칙 설정
EC2 인스턴스에서 외부로 나가는 트래픽을 허용하기 위한 규칙을 설정한다. 기본적으로 모든 트래픽이 허용되어 있다.
    <img src="/images/SG_creation_3.png" width="50%" height="50%" title="create sg 3" alt="create sg 3">   

## EC2 인스턴스 연결
EC2 인스턴스는 보통 SSH 프로토콜로 Linux, Mac OS에서 terminal을 사용하거나 windows에서 putty를 사용해서 접속할 수 있다. 또 다른 방법으로는 EC2 Instance Connect를 사용하여 브라우저에서 연결할 수 있다. EC2 대시보드 오른쪽 상단에 "연결" 버튼을 클릭하면 아래와 같은 화면이 출력된다.      
    <img src="/images/Connect_to_instance_1.png" width="50%" height="50%" title="ec2 instance connect 1" alt="ec2 instance connect 1">   
여기서 EC2 인스턴스 연결 탭에서 "EC2 Instance Connect를 사용하여 연결" 유형으로 연결하면 브라우저에 terminal 화면이 출력된다. 이때 EC2 인스턴스에 SSH로 연결하기 때문에 보안 그룹의 인바운드 규칙에 SSH 프로토콜이 허용되어 있지 않거나 특정 대역에서만 허용되어 있다면 연결이 되지 않아 보안 그룹 인바운드 규칙 허용이 필요하다.       
    <img src="/images/Connect_to_instance_2.png" width="50%" height="50%" title="ec2 instance connect 2" alt="ec2 instance connect 2">   
    <img src="/images/Connect_to_instance_3.png" width="50%" height="50%" title="ec2 instance connect 3" alt="ec2 instance connect 3">   

### EC2 IAM Role
EC2가 AWS 내부에서 실행되는 서비스이지만 다른 AWS 서비스를 사용하기 위해 권한이 필요하다. 만일 우리가 로컬의 PC에서 AWS 서비스에 액세스하기 위해서는 IAM 사용자에서 액세스키를 발급해 aws 액세스 정보를 구성하지만 EC2 인스턴스에 그렇게 한다면 EC2에 접속할 수 있는 누구나 액세스 정보를 볼 수 있어 위험한 방법이고 EC2마다 aws 액세스 정보를 구성하는 것도 비효율적인 방법이다. 대신 EC2에 IAM Role을 설정하여 EC2가 AWS 서비스에 액세스할 수 있도록 한다.

EC2 인스턴스를 생성되면 연결된 IAM 역할이 없다. 따라서 이 상태에서는 다른 AWS 서비스에 액세스할 수 있는 권한이 없다.        
    <img src="/images/IAM_role_for_ec2_1.png" width="50%" height="50%" title="iam role for ec2 1" alt="iam role for ec2  1">   
```bash
ec2-user@ip-172-31-5-251 ~]$ aws iam list-users
Unable to locate credentials. You can configure credentials by running "aws configure". 
``` 

아래 절차에 따라서 IAM Role을 EC2에 연결할 수 있다.
1. IAM Role을 연결할 EC2 인스턴스를 선택 후 작업 -> 보안 -> IAM 역할 수정을 선택
2. EC2 인스턴스와 연결할 IAM Role을 선택 후 업데이트
    <img src="/images/IAM_role_for_ec2_2.png" width="50%" height="50%" title="iam role for ec2 2" alt="iam role for ec2 2">   
3. 터미널로 돌아가 동일한 명령을 실행하면 오류가 발생하지 않고 정상적으로 결과가 출력된다.
```bash
[ec2-user@ip-172-31-5-251 ~]$ aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "admin",
            "UserId": "AIDA4RQ657DCKIW5WAN4B",
            "Arn": "arn:aws:iam::xxxxxxxxxxxx:user/xx",
            "CreateDate": "xxxx-xx-xxTxx:xx:xx+xx:xx",
            "PasswordLastUsed": "xx-xx-xxTxx:xx:xx+xx:xx"
        },
```
면
## EC2 인스턴스 요금 유형
EC2 인스턴스는 온디멘드 외에도 보다 더 저렴한 구매 모델이 있다. 요금 구조를 살펴보면 단위 시간당 요금은 온디멘드가 가장 비싸다.

| 구매 모델 | 설명 |
| --- | --- |
| 온디멘드 | 단기 워크로드와 애플리케이션의 동작이 예측이 되지 않을 때 적합 <br> Linux와 Windows는 초당 요금으로 다른 OS는 시간당 요금으로 계산한다. 사용한 만큼 요금이 발생해서 요금 예상이 가능하다. <br> 장기 약정이 없다. |
| 예약 | 1년 또는 3년 동안 인스턴스 사용을 예약하는 유형이 인스턴스 유형, 리전, OS를 예약한다. <br> 예약 인스턴스 - 장기간 워크로드에 적합하지만 인스턴스 유형 변경이 불가는 하다. <br> 전환형 예약 인스턴스 - 장기간 워크로드에 적합하며 필요에 따라 인스턴스의 유형을 변경할 수 있다. 예약 인스턴스에 비해 할인률이 낮다. <br> 온디멘드에 비해 최대 72% 저렴하다. <br> 선결제, 부분 선결제, 선결제 없음 중 하나를 선택해 요금을 지불할 수 있고 전체 선결제가 가장 높은 할인률을 보인다. <br> 독특한 점은 예약 인스턴스를 마켓플레이스에서 구매하거나 팔 수 있다.|
| 절감형 플랜 | 1년 또는 3년 동안 사용량을 약정하는 방식, 인스턴스를 예약하는 예약 방식과 차이가 있다. 장기간 워크로드에 적합 <br> 온디멘드에 비해 최대 72% 더 저렴하다. <br> 약정 사용량까지는 절감형 플랜 요금이 부과되지만 초과된 부분에 대해서는 온디멘드 요금이 부과된다.|
| 스팟 인스턴스 | 아주 짧은 워크로드에 적합한 독특한 유형이다. 데이터 센터에서 잉여 컴퓨팅 자원을 활용하여 제공하는 인스턴스로 매우 저렴하지만 언제든지 회수될 수 있어 데이터 저장이나 장기간 워크로드에는 부적절하다. <br> 온디멘드에 비해 최대 90% 저렴하다. <br> 스팟 인스턴스에 지불할 최대 가격을 설정하고 사용 요금이 이를 넘어서면 스팟 인스턴스는 회수된다. <br> 배치 작업이나 데이터 분석, 이미지 프로세싱, 분산 워크로드에 적합하다.|
| 전용 호스트 | 물리적 서버 전체를 예약해서 인스턴스 배치를 제어할 수 있다. <br> 실제 서버를 임대하기 때문에 가장 비싼 옵션이다. EC2 인스턴스와 유사하기 온디멘드와 예약 요금으로 나뉘어져 있다. <br> 사용사례로는 라이센스 모델과 함꼐 제공되는 소프트웨어를 사용하거나 규정이나 법규를 반드시 준수해야하는 기업이 가지고 있는 경우가 있다.|
| 전용 인스턴스 | 단일 AWS 계정 전용 하드웨어에서 실행되는 인스턴스이며 다른 AWS 계정의 인스턴스와 물리적으로 격리된다. 하지만 같은 계정에 있는 전용 인스턴스가 아닌 다른 인스턴스의 하드웨어는 공유가 가능하다. <br> 전용 호스트와 달리 EC2 인스턴스 배치를 제어할 수 없다.|
| 용량 예약 | 원하는 기간 동안 특정한 AZ에 용량을 예약 가능 <br> 용량을 예약해서 기간 약정이 따로 없다. 따라서 요금 할인이 따로 존재하지는 않지만 예약 인스턴스이나 절감형 플랜과 결합하여 요금 할인이 가능하다. <br> 인스턴스 실행 여부에 상관없이 온디멘드 요금이 부과된다. 특정한 AZ에 있어야 하는 단기 워크로드에 적합 |

> 출처 : [Amazon EC2 요금](https://aws.amazon.com/ko/ec2/pricing/)

### EC2 스팟 인스턴스
스팟 인스턴스틑 온디멘드에 비해 최대 90% 저렴한 인스턴스이다. 스팟 인스턴스 사용을 위해 최대 스팟 요금을 정의하는데 현재 스팟 가격이 정의한 가격보다 낮다면 인스턴스를 유지하지만 높다면 2분 이내 인스턴스 중지 혹은 종료 중 하나를 골라야 한다. 만일 중지를 하면 추후 스팟 가격이 내려갈 때 중지된 작업을 지속할 수 있다. 스팟 요금은 AWS 콘솔에서 스팟 요청 -> 요금 내역을 확인하면 스팟 요금 변화를 볼 수 있다. 스팟 요금은 AZ에 따라 다르다.


<img src="/images/spot_lifecycle.png" width="50%" height="50%" title="spot lifecycle" alt="spot lifecycle">     

> 출처 : [스팟 인스턴스 작업](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/spot-requests.html)   
스팟 생명 주기

스팟 요청은 2가지 유형으로 나뉜다.
| 유형 | 설명 |
| --- | --- |
| 일회성 요청 | 스팟 요청이 완료되는 즉시 인스턴스가 시작되고 요청이 이행되거나 인스턴스가 중지되는 조건이 발생하면 인스턴스가 바로 종료된다. |
| 영구 인스턴스 요청 | 스팟 요청이 유효한 기간 동안 인스턴스의 수가 유지된다. 어떠한 이유로 인스턴스가 중지되더라도 요청이 만료되지 않았으면 스팟 인스턴스를 다시 시작할 수 있을 때 인스턴스를 실행한다. |


스팟 요청을 취소하기 위해서는 활성 상태이거나 비활성 상태여야 한다. 만일 스팟 요청이 실패했거나 닫혔거나 취소된 경우에는 취소가 불가능 하다.    
<img src="/images/spot_request_states.png" width="50%" height="50%" title="spot request state" alt="spot request state">
> 출처 : [스팟 인스턴스 작업](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/spot-requests.html)   
스팟 요청 상태  

### 스팟 플
스팟 플릿은 스팟 인스턴스 세트를 정의하는 방법이고 선택적으로 온디멘드 인스턴스도 포함할 수 있다. 사용자의 요구 사항을 충족하는 스팟 용량 풀을 선택해 목표 용량을 충족하는 스팟 인스턴스를 시작한다. 플릿에서 스팟이 종료되더라도 목표 용량을 유지하기 위해 다른 스팟 인스턴스를 자동으로 실행한다. 스팟 플릿은 AWS CLI로 실행할 수 있으나 현재 spotfleet 관련 API에 지원 계획이 없어 ASG나 EC2 플릿 사용이 권장 된다.

> 출처 : [스팟 플릿](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/spot-fleet.html)

- 스팟 인스턴스 할당 전략
| 전략 | 설명 |
| --- | --- |
| lowestPrice | 가장 낮은 가격의 풀에서 시작하므로 비용 최적화된 전략이다. 매우 짧은 워크로드에 적합, 권장하지 않는다.|
| diversified | 스팟 인스턴스는 모든 풀에 두루 분산된다. 가용성 보장과 장기간 워크로드에 적합 - 한 풀이 사라져도 다른 풀이 활성화 되어 있어서 |
| capacityOptimized | 원하는 인스턴스 수에 맞는 최적의 용량을 가진 풀을 가진다. |
| priceCapacityOptimized | AWS에서 권장하는 방법, 사용 가능한 풀 중 가장 용량이 큰 풀 중에서 가장 낮은 가격을 선택하는 전략 |


> 출처 : [스팟 인스턴스 할당 전략](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/spot-fleet-allocation-strategy.html)

- 스팟 플릿 요청 예시   

1. 스팟 플릿을 요청하기 위해 먼저 JSON 형식의 요청 파일을 생성해야 한다. 요청 사항에는 스팟 가격, 목표 용량, 인스턴스의 AMI, 유형, 가중치, 서브넷 ID와 같은 내용이 작성되어야 한다.

    ```json
    {
    "SpotPrice": "0.70",
    "TargetCapacity": 20,
    "IamFleetRole": "arn:aws:iam::123456789012:role/aws-ec2-spot-fleet-tagging-role",
    "LaunchSpecifications": [
        {
        "ImageId": "ami-1a2b3c4d",
        "InstanceType": "r3.2xlarge",
        "SubnetId": "subnet-482e4972",
        "WeightedCapacity": 1
        },
        {
        "ImageId": "ami-1a2b3c4d",
        "InstanceType": "r3.4xlarge",
        "SubnetId": "subnet-482e4972",
        "WeightedCapacity": 2
        },
        {
        "ImageId": "ami-1a2b3c4d",
        "InstanceType": "r3.8xlarge",
        "SubnetId": "subnet-482e4972",
        "SpotPrice": "0.90",
        "WeightedCapacity": 4
        }
    ]
    }
    ```
2. 요청 사항 파일을 포함한 aws 명령으로 spot fleet 요청을 실행한다.
    ```bash
    aws ec2 request-spot-fleet --spot-fleet-request-config file://config.json
    ```

> 출처 : [자습서: 인스턴스 가중치를 부여한 스팟 플릿 사용](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/instance-weighting-walkthrough.html)