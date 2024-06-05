# Amazon Route 53
Route 53은 AWS에서 사용할 수 있는 가용성과 확장성이 뛰어난 DNS 웹 서비스이며 AWS 인프라뿐만 아니라 AWS 외부 인프라에도 효과적으로 연결할 수 있습니다. 따라서 개발자와 기업은 웹 서비스나 웹 애플리케이션에 사용자를 안정적이고 비용효율적으로 연결할 수 있습니다. Route 53은 Health Check 기능으로 애플리케이션의 앤드 포인트를 모니터링하여 유사시 자동으로 장애 조치가 가능해 고가용성을 보장합니다.

## DNS - Domain Name System

- 사람이 알아볼 수 있는 호스트 이름을 서버 IP 주소로 번역한다.
    - 예) [www.google.com](http://www.google.com) → 172.217.18.36
- Backbone of Internet
- 계층적 이름 구조
    - .com - TLD
    - [example.com](http://example.com) - SLD
    - [www.example.com](http://www.example.com) - Sub Domain
    - [api.example.com](http://api.example.com) - Domain Name
- Terminologies - 용어
    - Domain registrar : 도메인 이름을 등록하는 곳
        - Amazon Route 53, Go Daddy
    - DNS Recodes
        - A
        - AAAA
        - CNAME
        - NS
    - Zone File : 모든 DNS 레코드를 포함, 호스트 이름과 IP 주소를 일치시키는 방법
    - Name Server : DNS 쿼리를 실제로 처리하는 서버
    - Top Level Domain (TLD) : .com, .us, .in, .gov, .org …
    - Second Level Domain (SLD) : amazon.com, google.com
    - Sub Domain : www.example.com
    - Domain Name : api.www.example.com
    - Protocol : 연결에 사용되는 프로토콜 이름, http://, https://
    - FQDN - Fully Qualified Domain Name : 전체 도메인 이름 - http://api.www.example.com

### DNS 작동방식

<img src="/images/Route53_1.png" width="75%" height="75%" title="route 53 1" alt="route 53 1">      

1. 웹 브라우저가 example.com에 접근하기 위해 로컬 DNS 서버에 물어본다.
2. 로컬 DNS 서버는 보통 회사에 의해 할당 및 관리되거나 ISP에 의해 동적으로 할당
3. 로컬 DNS 서버가 해당 쿼리를 이전에 본적이 없다면 ICANN에 의해 관리된 DNS 서버의 루트에 물어본다.
4. Root DNS 서버는 가장 먼저 로컬 DNS서버로부터 요청을 받고 본 적은 없지만 .com 도메인을 알고 있어 TLD DNS 서버로 가볼 것을 .com NS 레코드로 IP 주소를 알려줌
5. ICANN에 의해 관리되는 TLD DNS 서버는 [example.com](http://example.com)의 정확한 레코드를 알지 못해 쿼리에 당장 답할 수는 없지만 example.com의 서버를 알고 있어 example.com NS 레코드로 IP를 알려줌
6. 마지막으로 도메인 이름 레지스트라에 의해 관리되는 Sub Domain의 DNS 서버에 물어본다. 해당 DNS 서버에는 example.com이 무엇인지 알고 있고 레코드를 확인하여 IP 주소를 답변한다. 
    1. 레지스트라 : Amazon Route 53, Go Daddy
7. DNS 서버는 반복적으로 DNS에 대한 질문하고 가장 구체적인 답변을 얻어내고 이후 동일한 요청이 발생하면 바로 답변을 줄수 있다. DNS 서버가 받은 답변을 브라우저로 보내고 브라우저는 IP 주소를 활용해서 웹 서버에 액세스할 수 있다.

## Basic

- 고가용성 및 확장성을 갖추고 완전히 관리되며 권한있는 DNS
    - Authoritative - 권한있는
        - 사용자가 DNS 레코드를 업데이트할 수 있다.
- Route 53은 도메인 레지스트라로 도메인 이름을 등록할 수 있다.
- Route 53 리소스 상태확인 가능
- 100% SLA 가용성을 제공하는 유일한 AWS 서비스
    
    [SLA란 무엇인가요? - 서비스 수준 계약 설명 - AWS](https://aws.amazon.com/ko/what-is/service-level-agreement/)
    
- Route 53의미
    - 53은 전통적으로 DNS 포트로 사용하는 포트 번호이다.
- Records
    - Route 53은 레코드를 통해 특정 도메인으로 라우팅하는 방법을 정의
    - 레코드 정보
        - Domain/subdomain Name : example.com
        - Record Type : A, AAAA, CNAME, NS
        - Value : 123.456.789.123
        - Routing Policy : Route 53이 쿼리에 응답하는 방식
        - TTL - Time to Live : DNS Resolver에서 레코드가 캐싱되는 시간
    - 레코드 종류
        - 필수 : A / AAAA / CNAME / NS
        - 고급 옵션 : CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV

        > ❓<br> A - 호스트 이름과 IPv4를 매핑 <br>
        AAAA - 호스트 이름과 IPv6를 매핑 <br>
        CNAME - 호스트 이름을 다른 호스트 이름과 매핑, 대상 호스트 이름은 A나 AAAA 레코드가 될 수도 있다. 상위 노드에 대한 CNAME을 생성할 수 없다. <br>
        NS - 호스팅 존의 네임 서버, 서버의 DNS 이름 또는 IP 주소로 호스팅 존에 대한 DNS 쿼리에 응답할 수 있다. 트래픽이 도메인으로 라우팅되는 방식을 제어
        
        
- Hosted Zones
    - 호스팅 존은 레코드의 컨테이너이며 도메인과 서브도메인으로 가는 트래픽의 라우팅 방식을 정의
    - Public Hosted Zone
        
        
        <img src="/images/Route53_2.png" width="75%" height="75%" title="route 53 2" alt="route 53 2">      
        
        - 퍼블릭 도메인을 살 때마다 퍼블릭 호스팅 존을 만들 수 있다.
        - 퍼블릭 존은 쿼리에 도메인 이름의 IP가 무엇인지 알 수 있다.
    - Private Hosted Zone
        
        
        <img src="/images/Route53_3.png" width="75%" height="75%" title="route 53 3" alt="route 53 3">      
        
        - 공개되지 않은 도메인 이름을 지원
        - VPC만이 URL을 리졸브할 수 있다.
        - 회사 네트워크 혹은 인트라넷에서만 접근할 수 있는 도메인 존
    - AWS에서 만드는 어떤 호스팅 존이든 월 50센트를 지불

## Register Domain

1. 도메인 이름 설정 및 가용성 확인 - 도메인은 고유한 이름을 사용해야한다.
    
    <img src="/images/Route53_4.png" width="75%" height="75%" title="route 53 4" alt="route 53 4">      
    
2. 사용할 최상위 도메인을 선택 후 사용 기간을 설정 및 자동 갱신 활성화 여부 설정
3. 사용자 연락처 정보 입력
    1. 개인 정보 보호 옵션을 활성화하는 것을 권장한다.
4. 검토 및 생성을 하면 등록한 결제수단으로 비용이 지불되며 도메인이 생성된다. 생성되는 시간은 몇 분에서 몇 시간 정도가 걸릴 수 있다.
5. 호스팅 존에 가면 등록한 도메인 이름을 확인할 수 있고 도메인 이름을 클릭하면 도메인에 대한 레코드를 확인할 수 있다.

### Create Domain

- 생성한 도메인에서 새로운 레코드를 생성할 수 있다.
    - 레코드 이름 설정
    - 레코드 타입 설정
    - 레코드 타입에 알맞은 값을 입력
    - TTL 설정
    - 라우팅 정책 설정
- 테스트
    - 생성한 레코드를 사용해서 브라우저로 접속을 시도하면 어떤 일이 발생한다
    - A 레코드로 생성하고 값을 무작위 IP로 넣었을 때 해당 IP 주소와 연결된 서버가 있다면 접속이 될 수도 있지만 대게는 되지 않을 것이다.
    - 터미널에서 간단하게 테스트하는 방법이 있다.
        - AWS CloudShell에 bind-utils를 설치하면 nslookup과 dig가 설치된다.
        - ‘nslookup 레코드’ 커맨드를 실행하면 레코드가 연결된 IP 주소로 보내지는 것을 확인할 수 있다.
        - ‘dig 레코드’ 커맨드도 동일하게 실행할 수 있지만 nslookup에 비해 더 자세한 결과를 확인할 수 있다.
            - 도메인의 레코드, 값, TTL등을 질문 세션, 응답 세션을 나누어 보여준다.

## TTL

- TTL - Time to Live
- 클라이언드가 DNS 요청을 보내면 DNS으로부터 회신을 받는다.
- 회신 내용 중에는 TTL 정보가 포함되어있으며 TTL 동안 회신 내용을 클라이언트가 캐시하도록 요청한다.
- TTL 시간 동안에는 클라이언트가 재요청을 하거나 같은 호스트 이름으로 접속할 경우 DNS에 쿼리를 보내지 않아도 된다.
- 예시
    - High TTL → 24hr
        - Route 53의 트래픽이 현저히 줄어든다.
        - 클라이언트가 오래된 레코드르 받을 가능성이 있다.
        - 레코드를 수정하고자 하면 클라이언트 모두가 새 레코드를 받을 때까지 24시간이 걸린다는 것을 의미
    - Low TTL → 60sec
        - DNS 트래픽 양이 많아져서 요금이 증가한다. - Route 53은 들어오는 요청의 양에 따라 요금이 책정
        - 이전 레코드의 보관 시간이 짧아져 레코드 변경이 빨라지고 레코드 변경이 편리해진다.
- 적합한 TTL을 설정하는 것은 상황에 따라 달라진다.
    - 레코드를 변경하고자 할 때 TTL을 낮춘 다음 클라이언트가 변경된 TTL을 가진 것이 확인이 되면 레코드 값을 변경 및 클라이언트 업데이트 완료 후 다시 TTL을 올린다.
- TTL은 Alias 레코드를 제외한 모든 레코드에 필수다.

## CNAME vs Alias
### CNAME

- CNAME 레코드는 호스트 이름이 다른 호스트 이름으로 향할 수 있도록 한다.
    - [app.example.com](http://app.example.com) → something.new.com
- Root Domain 이름이 아닌 경우에만 가능
    - something.example.com인 경우에만 가능

### Alias

- AWS 리소스에만 한정적으로 사용할 수 있다.
- 호스트 이름을 특정 AWS 리소스로 향할 수 있다.
    - [app.example.com](http://app.example.com) → something.amazonaws.com
- Root Domain과 non Root Domain 모두 사용할 수 있다.
- 무료
- 자체적인 상태 확인 가능
- DNS의 확장 기능으로 모든 DNS에 사용할 수 있다.
- ALB에서 IP가 변경되면 Alias는 자동으로 인식한다.
- CNAME과 달리 Zone Apex이라는 DNS 네임스페이스의 상위 노드로 사용될 수 있다.
    - example.com에도 Alias를 사용할 수 있다.
- AWS 리소스를 위한 Alias의 레코드 타입은 항상 A / AAAA이다.
- Alias를 설정하면 Route 53이 TTL을 설정하므로 사용자는 TTL을 설정할 수 없다.

### Alias Records Target

- Elastic Load Balancers
- CloudFront Distributions
- API Gateway
- Elastic Beanstalk environments
- S3 Websites
- VPC Interface Endpoints
- Global Accelerator accelerator
- Route 53 record in the same hosted zone
- EC2 인스턴스의 DNS에 대해서는 Alias를 사용할 수 없다.

## Routing Policies

- 라우팅 정책은 Route 53이 DNS 쿼리에 응답하는 것을 돕는다.
- Route 53에서 사용하는 라우팅 정책은 트래픽을 라우팅하는 것이 아닌 DNS 쿼리를 라우팅하는 것이다.
- 라우팅 정책을 통해 클라이언트들은 이를 통해 HTTP 쿼리 등을 처리하는 방법을 알게 된다.
- DNS는 호스트 이름들을 클라이언트가 실제 사용 가능한 엔드포인트로 변환하는 것을 돕는다.
- 정책 종류
    - Simple - 단순
    - Weighted - 가중치 기반
    - Failover - 장애 조치
    - Latency based - 지연 시간 기반
    - Geolocation - 지리적
    - Multi-ValueAnswer - 다중 값 응답
    - Geoproximity (using Route 53 Traffic Flow feature) - 지리 근접

### Simple

- 트래픽을 단일 리소스로 보내는 방식
- 동일한 레코드에 여러 개의 값을 지정하는 것이 가능
- DNS에 의해 다중 값을 받은 경우 클라이언트에서 무작위로 하나를 선택
- Alias를 함께 사용하면 하나의 AWS 리소스만을 대상으로 지정
- 상태 확인은 사용할 수 없다.

### Weighted

- 가중치를 활용해 요청의 일부 비율을 특정 리소스로 보내는 방식의 제어가 가능하다.
    - *traffic(%) = Weight for a specific record / Sum of all the weights for all records*
- DNS 레코드들은 동일한 이름과 유형을 가져야 한다.
- 상태확인을 사용할 수 있다.
- 가중치 0의 값을 보내면 트래픽 보내는 것을 중단해 가중치를 바꿀 수 있다.
- 모든 리소스 레코드의 가중치가 0이면 모두 같은 가중치를 가진다.
- Use case
    - 서로 다른 지역들에 걸쳐 로드 밸런싱을 할 때
    - 적은 양의 트래픽을 보내 새 애플리케이션을 테스트하는 경우에 사용

### Latency-based

- 지연 시간이 가장 짧은, 가장 가까운 리소스로 리다이렉팅하는 정책
- 지연 시간에 민감한 웹사이트나 애플리케이션에 유용
- 지연 시간은 사용자가 레코드로 가장 가까이 있는 것으로 식별된 AWS 리전에 연결하기까지 걸리는 시간을 기반으로 측정
- 상태 확인과 연결이 가능

### Failover (Active-Passive)

- 기본 EC2 인스턴스에 연결할 때 반드시 상태확인도 함께 연결해야한다.
- 기본 인스턴스에 장애가 발생하면 Route 53은 자동으로 재해 복구용 EC2 인스턴스로 연결되며 이때 상태확인은 선택사항이고 상태확인 기본, 보조 각각 하나씩만 사용가능
- 클라이언트에서는 정상으로 생각되는 리소스를 자동으로 얻게 된다.

### Geolocation

- 지연 시간과 다르게 사용자의 실제 위치를 기반으로 한다.
- 사용자의 대륙이나 국가, 더 정확하게는 주, 도시 등 가장 정확하게 파악할 수 있는 위치를 기반으로 가장 가까운 곳의 IP로 라우팅된다.
- 일치하는 위치가 없는 경우 Default 레코드를 생성해야한다.
- 상태확인 연결 가능
- Use case
    - 콘텐츠 분산을 제한하고 로드밸런싱을 실행하는 웹사이트 현지화
    - 서비스가 지원하는 언어별로 서비스, 사용하려는 지역의 언어가 지원되지 않을 때는 기본 IP로 라우팅

### Geoproximity Routing Policy

- 사용자와 리소스의 지리적 위치를 기반으로 트래픽을 리소스로 라우팅할 때 사용
- 편향값을 사용하여 특정 위치의 리소스로 더 많은 트래픽을 이동시키는 것
- 지리적 위치를 변경하기 위해서 편향값을 지정
    - 특정 리소스에 더 많은 트래픽을 보내려면 편향값을 증가
    - 트래픽을 줄이려면 편향값을 음수값으로 축소
- 리소스
    - AWS 리소스 - 특정 리전의 리소스를 의미
    - Non-AWS 리소스 - 위도와 경도를 지정해서 AWS가 위치를 파악하도록 한다.
- 편향값 활용을 위해 Route 53 Traffic Flow를 사용해야한다.

### IP-based

- 클라이언트 IP를 기반으로 라우팅을 정의
- Route 53에서 CIDR 목록을 정의 - 클라이언트의 IP 범위
- CIDR에 따라 트래픽을 어느 위치에 보내야 하는지를 정함
- IP를 미리 알고 있어 성능을 최적화할 수 있고 비용을 절약할 수 있다.
- 예시
    - 특정 ISP가 특정 IP 주소 셋을 사용하는 것을 안다면 특정 엔드포인트로 라우팅할 수 있다.

### Multi-Value

- 트래픽을 다중 리소스로 라우팅할 때 사용
- Route 53은 다중 값과 리소스를 반환
- 상태확인과 연결하면 다중 값 정책에서 반환되는 유일한 리소스는 정상 상태 확인과 관련이 있다.
- 각 다중 값 쿼리에는 최대 8개의 정상 레코드가 반환된다.
- ELB와 유사해 보이지만 ELB를 대체할 수 없다.

## Health Check

- 공용 리소스와 개인 리소스 모두 상태 확인을 사용할 수 있다.
- 상태 확인을 하는 이유는 DNS의 장애 조치를 자동화하기 위함
    - 공용 엔드포인트 모니터링 - Application, Server, Other AWS Resources
    - 다른 상태 확인을 모니터링 - Calculated Health Checks
    - CloudWatch 경보 상태를 모니터링 - 개인 리소스를 모니터링에 유용
- 상태 확인은 각 서비스의 지표를 활용하고 원한다면 CloudWatch 지표에서도 확인 가능하다.

### Monitor an Endpoint

- 전세계 15개의 health checker가 엔드포인트의 상태를 확인.
    - Healthy/Unhealthy Threshold - 3 (default)
    - Interval - 30sec (10초 간격도 가능하지만 더 높은 비용이 든다.)
    - HTTP, HTTPS, TCP 등의 프로토콜을 지원
    - 18%의 health checker가 엔드포인트를 정상으로 판단하면 Route 53도 정상으로 간주
    - 상태 확인에 사용될 위치를 선택할 수 있다.
- 상태확인은 로드 밸런서로부터 2xx 혹은 3xx의 코드를 받아야 통과가 된다.
- 텍스트 기반 응답일 경우 응답의 처음 5120바이트를 확인
    - 텍스트 응답에서 HTTP 응답 코드가 있는지 확인하기 위해
- 상태 확인이 애플리케이션 밸런서나 엔드포인트에 액세스 가능해야한다.
    - Route 53의 상태 확인 IP 주소 범위에서 들어오는 모든 요청을 허용

### Calculated Health Checks

- 여러 개의 상태 확인 결과를 하나로 합쳐주는 기능
- 상태 확인을 모두 합치기 위한 조건은 OR, AND, NOT을 사용한다.
- 하위 상태 확인을 256개까지 모니터링할 수 있다.
- 상위 상태 확인이 통과하기 위해 몇 개의 상태 확인을 통과해야하는지 설정 가능
- Usage
    - 상태 확인이 실패하는 일 없이 상위 상태 확인이 웹사이트를 유지관리함

### Private Hosted Zone

- 일반적으로 Route53 상태 확인은 VPC 외부에 있으므로 사설 엔드포인트로 액세스할 수 없다.
- CloudWatch 지표를 생성 및 CloudWatch 경보 연결을 통해 상태 확인을 만들 수 있다.
- 경보가 발생하면 상태 확인은 자동으로 비정상이 되게 한다.

## Domain Registrar vs DNS Service
### Domain Registrar

- 도메인 레지스트라에서 도메인을 구매할 수 있다.
- Amazon Route 53, Go Daddy, Google Domain
- 도메인 레지스트라에 도메인을 등록하면 DNS 레코드 관리를 위한 DNS 서비스를 제공

### DNS Service

- 도메인 레지스트라를 사용하지 않아도 DNS 서비스로 DNS 레코드를 관리할 수 있다.
- 예시
    - GoDaddy에서 도매인을 구매해서 Route 53에서 DNS 레코드를 관리해도 된다.

### GoDaddy as Registrar & Route 53 as a DNS Service

- GoDaddy에 DNS를 등록하면 네임 서버 옵션이 생기는데 사용자 정의 네임 서버를 지정할 수 있다
- 먼저 Amazon Route 53에 원하는 도메인의 공용 호스팅 영역을 생성
- 호스팅 영역 상세에서 네임 서버를 확인하고 이것을 GoDaddy 웹사이트에서 변경한다.

### 3rd Party Registrar with Amazon Route 53

1. Route 53에서 호스팅 영역을 생성
2. 서드 파티 웹사이트에서 NS 레코드나 네임 서버를 업데이트
3. 서드 파티에서 구매한 도메인이 Route 53을 가리키게 된다.

## 참고 자료
[Route 53](https://aws.amazon.com/ko/route53/)
[정보 문화사 - 아마존 웹 서비스](https://www.yes24.com/Product/Goods/69304366)      