# ELB
ELB는 Elastic Load Balancing의 약자로 하나 이상의 가용 영역에 있는 여러 대상에 들어오는 트래픽을 분산하는 기능을 수행하는 로드밸런싱 서비스입니다. 지원하는 로드벨런서는 ALB, NLB, GLB 세가지 유형과 사용이 권장되지 않지만 이전 세대인 CLB이 있고 각 로드 밸런서는 OSI의 3, 4, 7계층 중 한곳에서 동작합니다. ELB는 AWS에서 동작하는 애플리케이션의 내결함성 보장을 위해 고가용성과 부하분산, 강력한 보안 기능을 제공합니다.

## High Availability

- 보통 스케일링성과 함께 다루긴 하지만 늘 그렇지는 않다.
- 애플리케이션 또는 시스템을 적어도 둘 이상의 AWS의 AZ나 데이터 센터에서 가동 중이라는 것을 의미
- 고가용성의 목표는 데이터 센터에서 발생하는 손실에서 살아남는 것
- 고가용성이 수동적인 경우는 다중 가용 영역에 미리 서비스가 실행되는 방식이다.
- 고가용성이 능동적인 경우는 수평 확장과 같은 방식과 함께 사용되는 방식이다.
- 수평 확장과 유사하게 오토 스케일링 그룹과 로드 밸런서를 사용하지만 범위가 다중 가용 영역으로 확장된다.

## Load Balance

<img src="/images/ELB_1.png" width="50%" height="50%" title="elb 1" alt="elb 1">    

### 로드밸런싱 특징
- 로드 밸런서는 트래픽을 백앤드 서버나 EC2 인스턴스로 전달하는 역할을 한다.
- 사용자는 로드밸런서에 연결되면 하나의 엔드포인스에 연결되는 것만 인지할 수 있다.
- 로드밸런서 필요성
    - 트래픽 부하를 다수의 다운스트림 인스턴스로 분산하기 위해 사용
    - 어플리케이션에 모든 인스턴스의 앤드포인트가 아닌 DNS만 노출시킬 수 있다.
    - 다운스트림 인스턴스의 장애를 원활히 처리할 수 있다.
    - 인스턴스의 상태 체크 가능
    - SSL 종료 지원으로 웹 사이트에 암호화된 HTTPS 트래픽 전송 가능
    - 쿠키 고정성을 강화
    - 영역에 걸쳐 고가용성 제공
    - 클라우드 내에서 사설 트래픽과 공용 트래픽을 분리할 수 있다.

### 로드밸런싱 알고리즘

| 알고리즘 | 설명 |
| --- | --- |
| Round Robin | 실제 서버로의 세션 연결을 순차적으로 맺는 방식. 연결되어있는 세션 수에 상관 없이 순차적으로 연결시켜 세션에 대한 보장을 제공하지 않음 <br> 서버가 동일한 스펙을 가지고 있거나, 세션을 오래 지속하지 않는 서비스에 적합 |
| Hash | Hash 알고리즘을 사용하는 로드밸런싱으로 클라이언트와 서버 간에 연결된 세션을 유지하는 방식 <br> 클라이언트가 특정 서버와 연결을 지속해야 할 때 세션 보장을 제공 <br> 접속자 수가 많을 수록 분산 효율이 뛰어남 |
| Least Connection | 최소 연결 방식으로 가장 적은 세션 수를 보유한 서버로 세션을 맺는 방식 <br> 트래픽 분산이 서버마다 일정하지 않을 때 적합 |
| Least Responce time | 최소 응답시간 방식으로 연결 상태와 응답 시간이 차이가 있는 환경에서 가장 짧은 응답시간을 보이는 서버로 트래픽을 할당 <br> 서버 가용할 수 있는 리소스와 성능, 처리 중인 데이터의 양이 다를 때 적합 |

## Elastic Load Balancer

- AWS가 제공하는 관리형 로드밸런서
    - AWS가 어떠한 경우에도 로드밸런서가 동작하는 것을 보장
    - AWS가 업그레이드와 유지 관리 및 고가용성을 책임
    - 로그밸런서의 작동 방식을 수정할 수 있게끔 구성 놉을 제공
- 자체 로드밸런서를 사용하는 것보다 저렴하고 확장성이 좋아 Elastic Load Balancer를 사용하는 것이 이득이다.
- Elastic Load Balancer는 여러 AWS 서비스와 통합되어 있다.
    - EC2, EC2 Auto Scaling Group, Amazon ECS
    - AWS Certification Manager (ACM), CloudWatch
    - Route 53, AWS WAF, AWS Global Accelerator
    - 더 늘어날 예정

### Health Check

- 로드 밸런서가 EC2 인스턴스의 올바른 작동 여부를 확인하기 위해 사용
- 상태 확인은 포트와 라우트에서 이뤄진다.
- HTTP 응답 코드로 200이 응답되지 않으면 문제가 있는 것으로 판단

### 종류

- Classic Load Balancer (v1 - 2009, CLB)
    - HTTP, HTTPS, TCP, SSL (Secure TCP) 프로토콜 지원
    - AWS에서 사용을 권장하지 않고 곧 지원 종료 예정
- Application Load Balancer (v2 - 2016, ALB)
    - HTTP, HTTPS, WebSocket 프로토콜 지원
- Network Load Balancer (v2 - 2017, NLB)
    - TCP, TLS (Secure TCP), UDP 프로토콜 지원
- Gateway Load Balancer (2020, GWLB)
    - OSI 3계층인 네트워크 계층에서 작동 - IP 프로토콜 지원
- 몇몇 로드 밸런서는 사설망이나 공용 네트워크에서 사용할 수 있다.

### 보안 설정

- HTTP, HTTPS 프로토콜을 사용하는 로드밸런서가 있을 때 80, 443번 포트의 트래픽을 관리하게 된다.
- 로드 밸런서와 연결된 EC2 인스턴스는 80, 443번 포트의 트래픽 중 소스가 로드밸런서인 트래픽만 허용하도록 보안 그룹 규칙을 만들어야 한다.

### Sticky Session - Session Affinity

- 고정 세션은 클라이언트가 애플리케이션에 요청을 보낼 때 동일한 인스턴스로 트래픽을 보낼 수 있도록 하는 것이다.
- CLB와 ALB에서 설정할 수 있다.
- 쿠키를 사용하여 세션을 고정시키며 쿠키가 만료되면 다른 인스턴스로 리다이렉션 될 수 있다.
- 세션을 고정하면 백앤드 EC2 인스턴스 부하에 불균형이 발생할 수 있다.
- Use case
    - 사용자 로그인과 같이 중요한 정보를 취급하는 세션의 데이터를 잃지 않기 위해 사용자가 동일한 백앤드 인스턴스에 연결
    
    ### 쿠키 종류
    
    - Application-based Cookie
        - Custom Cookie
            - 대상에 의해 생성됨
            - 애플리케이션에 필요한 모든 사용자 정의 속성을 포함할 수 있다.
            - 쿠키 이름은 각 대상 그룹별로 개별적으로 지정
        - Application Cookie
            - 로드 밸런서에서 생성됨
            - 쿠키 이름은 ‘AWSALBAPP’이다.
    - Duration-based Cookie
        - 로드 밸런서에서 생성됨
        - 쿠키 이름은 ‘AWBALB’ for ALB, ‘AWSCLB’ for CLB이다.
    
    ### Stick Session 활성화
    
    - 대상 그룹의 속성에서 ‘Stickyness’ 혹은 ‘세션 고정성’ 활성화
    - 로드밸런서에서 생성되는 쿠키와 애플리케이션 기반 쿠키를 정할 수 있다.

### Cross-Zone Load Balancing

- With Cross Zone Load Balancing
<img src="/images/ELB_2.png" width="50%" height="50%" title="elb 2" alt="elb 2">    

- 교차 영역 로드 밸런싱을 사용하면 각 가용 영역의 로드 밸런서에 등록된 모든 인스턴스에게 부하가 고르게 분산된다.

- Without Cross Zone Load Balancing
<img src="/images/ELB_3.png" width="50%" height="50%" title="elb 3" alt="elb 3">    

- 교차 영역 로드 밸런싱을 사용하지 않으면 각 로드 밸런서가 할당받은 트래픽의 양을 각 로드밸런서에 등록된 인스턴스의 수만큼 나눠 가지게 된다.
- ALB
    - ALB는 기본적으로 교차 영역 로드 밸런싱이 활성화되어 있으며 비활성화도 가능하다.
    - 데이터를 다른 가용 영역으로 옮기는데 비용이 청구되지 않는다. 일반적으로는 다른 가용영역으로 데이터를 옮길 때 비용이 청구된다.
- NLB & GWLB
    - NLB와 GWLB는 교차 영역 로드 밸런싱이 기본 활성화가 되지 않으며 활성화가 가능하지만 비용이 청구된다.
- CLB
    - CLB는 교차 영역 로드 밸런싱이 기본 활성화가 되어 있지 않지만 활성화를 하더라도 비용이 발생하지 않는다.

### SSL/TLS - Basic

- SSL 인증서는 클라이언트와 로드 밸런서 사이에서 트래픽이 이동하는 동안 트래픽을 암호화하고 송신자와 수신자 측에서만 복호화  - 전송 중 암호화 (in-flight encryption)
- SSL은 Secure Socket Layer의 약자로 연결을 암호화하는데 사용
- TLS는 Transfer Layer Security의 약자로 최신 버전의 암호화 기술이다.
- TLS를 최근에 주로 사용하지만 아직 많은 사람들이 SSL이라고 부른다.
- 공용 SSL 인증서는 CA(Certificate Authorities)에서 발급
    - Comodo, Symantec, Go Daddy, GlobalSign, Digisert, Let’s Encypt 등
- 공용 SSL 인증서를 로드 밸런서에 추가하면, 클라이언트와 로드 밸런서 사이의 연결을 암호화한다.
    
    ### 작동 방식
    
    1. 사용자와 로드밸런서 사이의 통신은 HTTPS 프로토콜로 암호화된다.
    2. 로드 밸런서에 트래픽이 도착하면 SSL 종료가 되며 트래픽이 복호화된다.
    3. 복호화 된 트래픽은 로드 밸런서와 EC2 인스턴스 사이 통신에 사용된다.
        1. 로드 밸런서와 EC2 인스턴스 사이의 통신은 사설 IP를 사용하므로 복호화되지 않아도 안전하게 보호된다.
    - 로드 밸런서는 X.509 인증서를 사용하며 이를 SSL 혹은 TLS 서버 인증서라고 한다.
    - ACM - Amazon Certificate Manager를 사용하여 인증서를 관리할 수 있다.
    - HTTPS Listener
        - 기본 인증서를 지정
        - 다중 도메인을 지원하기 위해 다른 인증서를 추가할 수 있다.
        - 클라이언트는 SNI - Server Name Indication을 사용해서 호스트의 이름을 알릴 수 있다.
    
    ### SNI
    
    - 여러 개의 SSL 인증서를 하나의 웹 서버에 로드하여 하나의 웹 서버가 여러 개의 웹 사이트를 지원할 수 있게 한다.
    - 확장된 프로토콜로 최초 SSL 핸드셰이크 단계에서 클라이언트가 대상 서버의 호스트 이름을 지정하도록 한다. 이러한 방식으로 웹 서버가 어떠한 인증서를 사용해야하는지 알 수 있다.
    - 확장 기능이므로 모든 클라이언트가 지원하지는 않는다.
    - ALB & NLB, CloudFront에서만 동작한다.
    
    ### Enable SSL
    
    - ALB & CLB
        1. 로드 밸런서에 리스너를 추가
        2. HTTPS 프로토콜, 443번 포트로 트래픽이 들어오면 대상 그룹으로 전달되도록 설정
        3. 보안 리스너 설정의 보안 정책 선택
        4. ACM이나 IAM에서 인증서를 가져오거나 직접 인증서를 추가할 수 있다.
    - NLB
        1. ALB와 과정은 비슷하나 TLS 프로토콜에 443번 포트의 트래픽이 대상 그룹으로 전달되도록 설정
        2. ALPN 정책 설정 가능

### Connection Draining

- CLB에서 Connection Draining으로 불리고 ALB나 NLB에서 Deregistration Delay라고 불린다.
- 인스턴스가 등록 취소, 혹은 비정상 상태에 있을 때 인스턴스에 어느 정도 시간을 주어 인-플라이트 요청, 즉 활성 요청을 완료할 수 있도록 하는 기능
- 드레이닝이 완료되면 새로운 요청을 해당 인스턴스로 더이상 보내지 않는다.

## Application Load Balancer

<img src="/images/ELB_4.png" width="50%" height="50%" title="elb 4" alt="elb 4">    

- OSI 7계층인 응용(Application) 계층 전용 로드 밸런서로 머신간 HTTP 애플리케이션 라우팅에 사용
- HTTP 애플리케이션을 사용하는 머신들은 대상 그룹이라는 그룹에 묶이게 된다.
- 동일한 EC2 인스턴스 상의 여러 애플리케이션에 부하를 분산
- HTTP/2와 WebSocket과 리다이렉트 지원, 리다이렉트의 경우 로드 밸런서 수준에서 가능함.
- 서로 다른 대상 그룹으로 라우팅 가능
    - URL에 포함된 경로 기반의 라우팅 가능 - [example.com/users](http://example.com/users) & example.com/posts
    - URL에 포함된 호스트네임 기반의 라우팅 가능 - [one.example.com](http://one.example.com) & other.example.com
    - 쿼리 문자열과 헤더에 기반한 라우팅 가능 - example.com/users?id=123&order=false
- 마이크로서비스나 컨테이너 기반 애플리케이션에 가장 좋은 로드 벨런서 - Docker & Amazon ECS
- 포트 매핑 기능이 있어 동적 포트로 리다이렉션이 가능
- CLB의 경우 애플리케이션 마다 로드밸런서가 있어야 하지만 ALB는 하나로 다수의 애플리케이션을 처리할 수 있다.
- 고정 호스트네임이 부여된다.
- 애플리케이션 서버는 클라이언트의 IP를 직접 보지 못하며 클라이언트의 실제 IP는 X-Forwarded-For라고 불리는 헤더에 삽입된다. 포트와 프로토콜은 각각 X-Forwarded-Port, X-Forwarded-Proto에서 확인할 수 있다.
    - 클라이언트의 IP가 로드밸런서와 직접 통신해 연결 종료를 수행하고 로드 밸런서가 EC2 인스턴스와 통신할 때는 사설 IP인 로드 밸런서 IP를 사용해 EC2 인스턴스로 들어간다.

### Target Group

<img src="/images/ELB_5.png" width="50%" height="50%" title="elb 5" alt="elb 5">    

- EC2 인스턴스나 ECS 작업, 람다 함수들이 대상 그룹에 포함될 수 있다.
- ALB는 IP 주소 앞에 위치할 수 있는데 이때 IP 주소는 사설 IP이여야 한다.
- ALB는 여러 대상 그룹으로 라우팅 할 수 있으며 상태 확인은 대상 그룹 수준에서 이루어진다.

### Create ALB

1. 생성할 로드 밸런서 유형 선택
    
    <img src="/images/ELB_6.png" width="50%" height="50%" title="elb 6" alt="elb 6">    
    
2. ALB 설정
    1. 기본 구성
        - 로드밸런서 이름
        - 체계
            - 인터넷 경계
            - 내부
        - IP 주소 유형
            - IPv4
            - 듀얼 스택
    2. 네트워크 매핑
        - VPC 및 가용 영역 선택 - 로드 밸런서를 배포 위치
    3. 보안 그룹
        - HTTP 프로토콜 트래픽을 허용하는 보안 그룹 연결
    4. 리스너 및 라우팅
        - 대상 그룹 생성 후 선택
            - 대상 유형과 사용할 프로토콜, VPC 선택
            - 상태 검사 프로토콜 및 경로 설정
            - 대상 그룹에 대상 등록 - 인스턴스
3. 로드 밸런서가 정상적으로 생성되었으면 로드밸런서의 DNS를 브라우저에서 띄운 후 새로고침 할 때마다 연결되는 인스턴스가 바뀌는 것을 확인할 수 있다.

### Security Group for EC2 Instances

- 위 실습에서 사용한 EC2 인스턴스 보안그룹의 인바운드 규칙을 확인하면 모든 트래픽이 허용되어있어 EC2 인스턴스 공용 IP주소와 로드밸런서 모두를 통해 접속할 수 있다.
- 로드밸런서를 사용하는 경우 EC2 인스턴스의 접속은 로드밸런서만을 통해 접속하는 것이 바람직하므로 EC2 인스턴스 보안그룹의 인바운드 규칙을 수정하여 소스가 로드밸런서인 트래픽만 허용하도록 한다.

### Listener Rules for ALB

- 로드 밸런서는 리스너 규칙을 사용해서 특정 트래픽에 대한 작업을 지정할 수 있다.
    
    <img src="/images/ELB_7.png" width="50%" height="50%" title="elb 7" alt="elb 7">    
    
- 지정한 조건에 부합하는 트래픽이 취할 작업을 설정
    - 대상 그룹으로 전달, URL로 리디렉션, 고정 응답 반환을 설정할 수 있다.
    
    <img src="/images/ELB_8.png" width="50%" height="50%" title="elb 8" alt="elb 8">    
    
- 규칙을 적용할 우선 순위를 설정하여 트래픽에 규칙을 적용할 수 있다.
    - 트래픽이 우선 순위가 더 높은 규칙에 부합될 경우 우선순위가 낮은 규칙은 무시된다.
    
    <img src="/images/ELB_9.png" width="50%" height="50%" title="elb 9" alt="elb 9">    
    

## Network Load Balancer

<img src="/images/ELB_10.png" width="50%" height="50%" title="elb 10" alt="elb 10">    

- L4 로드밸런서로 TCP와 UDP 트래픽을 다룰 수 있다.
- 초당 수백만 건의 request를 처리할 수 있고 ALB에 비해 지연 시간이 짧다.
    - ALB : 400ms
    - NLB : 100ms
- 가용 영역별로 하나의 고정 IP를 가진다. → Elastic IP를 가용영역마다 하나를 할당받는다.
    - 여러 고정 IP를 가진 애플리케이션을 노출할 때 유용
- AWS 프리 티어에 포함되지 않는다.

### Target Group

<img src="/images/ELB_11.png" width="50%" height="50%" title="elb 11" alt="elb 11">    

- EC2 인스턴스
- IP 주소 - 사설
- ALB
    - NLB으로 고정 IP를 할당받을 수 있고 ALB로 HTTP 유형의 트래픽을 처리하는 규칙을 생성할 수 있어 유효한 설정 조합이다.
- NLB 대상그룹은 TCP, HTTP, HTTPS 프로토콜을 지원한다.

### Create NLB

1. 생성할 로드 밸런서 유형 중 네트워크 로드 밸런서를 선택
2. 로드 밸런서를 배포할 VPC와 가용영역, 서브넷을 선택
    - 각 서브넷에 IPv4 주소를 할당할 수 있으며 AWS에서 할당하게 하거나 미리 할당받은 Elastic IP를 사용하면 된다.
3. 대상 그룹 생성 및 선택
    - NLB에 사용할 대상그룹이므로 사용할 프로토콜은 TCP로 설정
    - 상태검사는 사용할 애플리케이션에 맞춰 설정
    - 대상그룹에 포함할 인스턴스를 등록
4. ALB와 달리 NLB는 별도의 보안그룹을 설정하지 않았다. 이 의미는 트래픽 허용은 EC2 인스턴스의 보안그룹에서 담당한다는 뜻이다.
5. NLB도 ALB와 동일하게 DNS로 접속 후 페이지를 새로고침하면 접속한 인스턴스가 바뀌는 것을 확인할 수 있다.

## Gateway Load Balancer

- 배포 및 확장, 서드파티 네트워크 가상 어플라이언스의 플릿 관리에 사용
- 네트워크의 모든 트래픽이 방화벽을 통과하게 하거나 침입 탐지 및 방지 시스템에 사용
- IDPS나 심층 패킷 분석 시스템 혹은 일부 페이로드를 네트워크 수준에서 수정할 수 있다.
- GWLB를 사용하는 이유와 과정
    
    <img src="/images/ELB_12.png" width="50%" height="50%" title="elb 12" alt="elb 12">    
    
    - 애플리케이션에 액세스하려는 사용자가 있을 때 트래픽이 애플리케이션으로 이동하기 전에 모든 트래픽을 검사하려고 하면 다소 복잡한 과정이 필요했지만 GWLB를 사용하면 간단하게 해결된다.
    - GWLB를 생성하면 VPC에서 사용자의 트래픽이 GWLB를 통과하도록 라우팅 테이블이 업데이트되된다.
    - GWLB는 가상 어플라이언스의 대상 그룹 전반으로 트래픽을 확산한다. 이 과정을 거쳐 모든 트래픽이 가장 어플라이언스에 도달하고 어플라이언스가 트래픽을 분석 및 처리한다.
    - 트래픽이 정상이면 다시 GWLB로 반환하고 이상이 있으면 해당 트래픽을 드롭한다.
    - 정상 트래픽은 GWLB를 통과하여 애플리케이션으로 보내진다.
- GWLB는 NLB보다 낮은 OSI 3계층에서 작동한다.
    - Transparent Network Gateway : VPC의 모든 트래픽이 GWLB이라는 단일 출입구를 통과
    - Load Balancer : 대상 그룹의 가상 어플라이언스 집합에 트래픽을 분산
- GENEVE 프로토콜의 6081번 포트

### Target Group

<img src="/images/ELB_13.png" width="50%" height="50%" title="elb 13" alt="elb 13">    

- EC2 Instances
- IP 주소 - 사설

## Classic Load Balancer
CLB는 AWS의 1세대 로드밸런서이며 OSI 3, 4계층에서 동작하는 로드밸런서입니다. 현재는 3, 4계층 로드밸런싱은 각각 GLB와 NLB로 대체되었고 기존 CLB를 사용하던 애플리케이션이 아니면 AWS에서 사용을 권장하지 않습니다. 다른 유형의 ELB들은 대상 그룹으로 트래픽을 라우팅해서 대상 그룹에 속한 대우(EC2, Lambda 함수 등)에 트래픽을 전달할 수 있지만 CLB은 대상 그룹에 트래픽을 라우팅할 수 없어 EC2 인스턴스에만 기본적인 로드밸런싱을 수행합니다.

## 참고 자어
[AWS Elastic Load Balancing](https://aws.amazon.com/ko/elasticloadbalancing/)   
[AWS Docs - Application Load Balancer는 무엇입니까?](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/introduction.html)      
[AWS Docs - Network Load Balancer는 무엇입니까?](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/network/introduction.html)      
[AWS Docs - Gateway Load Balancer는 무엇입니까?](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/gateway/introduction.html)      
[AWS Docs - Classic Load Balancer는 무엇입니까?](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/classic/introduction.html)      
[정보 문화사 - 아마존 웹 서비스](https://www.yes24.com/Product/Goods/69304366)      