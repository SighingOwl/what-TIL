# ENI란?
ENI는 VPC에서 가상 네트워크 카드를 나타내는 논리적 네트워킹 구성요소입니다. EC2 인스턴스가 네트워크에 액세스할 수 있도록 도와주며 EC2 인스턴스를 생성하면 기본 ENI가 eth0에 연결되어 네트워크 연결이 가능해진다. ENI는 EC2와 독립적인 리소스이므로 사용자가 별도의 ENI를 생성하여 EC2에 연결하거나 장애 조치를 위해 장애 인스턴스에서 정상 인스턴스로 ENI를 옮길 수 있다. 예를 들어 EC2 인스턴스가 2개의 ENI에 연결되어 있을 때 해당 인스턴스에 장애가 발생할 경우 ENI를 그대로 정상 인스턴스로 이동시켜 장애조치를 할 수 있다.     

<img src="/images/ENI_1.png", width="50%" height="50%" title="eni 1" alt="eni 1">   

### ENI 속성
- 주 사설 IPv4와 하나 이상의 보조 IPv4를 가질 수 있다.
- 하나의 사설 IPv4에 하나의 탄력적 IP 혹은 임의 공용 IP를 가질 수 있다.
- 하나 이상의 보안 그룹을 연결할 수 있다.
- MAC 주소 연결
- 특정 AZ에 바인딩 됨

### Create ENI

<img src="/images/ENI_2.png", width="50%" height="50%" title="eni 2" alt="eni 2">   

- ENI를 생성할 서브넷를 선택 및 사용할 보안 그룹 선택 후 생성
- ENI는 이미 인스턴스에 연결되어 있으면 ‘In-use’ 상태이고 아닌 경우에는 ‘Available’ 상태이다.

<img src="/images/ENI_3.png", width="50%" height="50%" title="eni 3" alt="eni 3">   

- 생성한 ENI는 원하는 인스턴스에 연결할 수 있으며 필요할 경우에는 분리도 가능하다.