# CloudFront
Amazon CloudFront는 AWS에서 제공하는 CDN 서비스로 글로벌 사용자에게 정적 및 동적 컨텐츠를 더 짧은 지연시간에 배포하기 위한 서비스입니다. AWS가 보유한 글로벌 엣지 로케이션으로 사용자의 요청이 라우팅되어 사용자는 컨텐츠를 얻기 위해 복잡한 네트워크 경로를 사용하지 않고 빠르게 컨텐츠를 받을 수 있고 컨텐츠가 엣지 로케이션에 분산되어 있어 DDoS 공격에서 보호받을 수 있습니다.

## CDN이란?
CDN은 Contents Delivery Network 혹은 Contents Distribution Network의 약자로 컨텐츠를 효율적으로 전달하기 위해 여러 노드를 가진 네트워크에 데이터를 저장하여 제공하는 서비스를 의미합니다. 원본을 가진 서버에서 직접 컨텐츠를 전달하는 것보다 저렴하고 빠르게 전송가능하고 CDN 서버에 컨텐츠를 분산시켜 사용자의 네트워크 경로 상 가장 가까운 곳의 서버로부터 컨텐츠를 전송하도록 하여 트래픽이 특정 서버에 집중되지 않고 여러 서버에 분산되도록 하는 기술입니다.


## CloudFront 기본 정보
### 원본 제공 방법

- S3 버킷
    - CloudFront를 통해 엣지 로케이션으로 파일을 분산하고 캐싱할 수 있게 한다.
    - OAC - Origin Access Control : 버킷에는 CloudFront만 접근할 수 있게 보장
        - OAC는 OAI - Origin Access Identity를 대체한다.
    - CloudFront를 사용해서 S3 버킷으로 데이터를 보낼 수도 있다 - Ingress
- Custom Origin (HTTP)
    - ALB
    - EC2 인스턴스
    - S3 Website - 버킷을 활성화해서 정적 웹사이트로 설정
    - HTTP 백앤드

### CloudFront at a High Level

- 클라이언트가 엣지 로케이션에 HTTP 요청을 보내면, 엣지는 요청의 결과가 캐싱되어 있는지 확인
- 엣지에 캐싱되어 있지 않으면 원본으로 가서 요청 결과를 가져온다.
- 로컬 캐시에 결과를 저장하고 다른 클라이언트가 동일한 요청을 하면 캐싱된 결과를 사용한다.

### S3 as Origin

- 엣지 로케이션은 내부망을 통해 S3 버킷에서 원본을 받아온다.
- S3 버킷은 OAC로 보호받으며, S3 버킷 정책에 의해서만 수정될 수 있다.

### CloudFront vs S3 Cross Region Replication

CloudFront

- 글로벌 엣지 네트워크
- 파일은 TTL동안 캐시된다.
- 전세계를 대상으로 한 정적 컨텐츠를 사용하고자 할 때 용이

CRR

- 복제를 원하는 각 리전에 CRR 설정이 되어있어야 한다.
- 파일은 거의 실시간으로 갱신된다.
- 캐싱되지 않는다.
- 읽기 전용으로만 설정 가능
- 일부 리전을 대상으로 동적 컨텐츠를 낮은 지연 시간으로 제공

## ALB or EC2 as an origin
### EC2

- 사용자가 엣지 로케이션을 통해 EC2 인스턴스에 접근하려고 할 때 EC2 인스턴스는 반드시 퍼블릭으로 설정되어있어야 한다.
- 엣지 로케이션의 모든 공용 IP가 EC2 인스턴스에 접근할 수 있도록 하는 보안그룹 설정이 필요

### ALB

- 로드 밸런서를 사용해서 EC2 인스턴스에 접근하는 것
- ALB는 퍼블릭 설정과 엣지 로케이션의 모든 공용 IP가 접근 가능해야한다.
- ALB와 EC2 인스턴스 사이의 통신은 프라이빗으로 사용가능하다.
- EC2 인스턴스의 보안 그룹이 로드 밸런서를 허용하도록만 설정하면 된다.

## Geo Restriction

- 사용자들의 지역에 따라 배포 객체 접근을 제한할 수 있다.
    - Allowlist : 접근 가능한 국가 목록 설정
    - Blocklist : 접근 불가능한 국가 목록 설정
- 사용자의 IP가 어떤 국가에 해당하는지 확인 가능
- Use case:
    - 컨텐츠 저작권법으로 인한 제한

## Price Classes

- CloudFront 엣지 로케이션은 전세계에 퍼져있으므로 데이터 전송비용이 엣지 로케이션마다 다르다.
- 가격 등급
    - Price Class All : 모든 리전, 최고 성능, 최고 비용
    - Price Class 200 : 가장 비싼 리전을 제외한 대부분의 리전
    - Price Class 100 : 가장 저렴한 리전만 사용

## Cache Invalidations

---

- CloudFront는 엣지 로케이션에 오리진의 내용을 캐시한다.
- 만일 오리진에서 원본의 내용이 변경되더라도 CloudFront는 알지 못한다.
- TTL이 만료된 뒤에 업데이트 된 원본을 캐싱한다.
- 업데이트 된 내용을 빠르게 받고 싶을 때 CloudFront 무효화를 사용해서 TTL이 만료되기 전에 일부 혹은 전체 캐시를 강제로 새로고침 할 수 있다.

## Global Accelerator

- 한 리전에서 글로벌 애플리케이션을 서비스할 때 공용 인터넷을 사용해서 app에 접근하면 여러 라우트를 거치는 동안 수많은 홉이 발생하여 지연시간이 길어지거나 연결이 끊어질 수 있다.
- 지연 시간을 최소화하기 위해 퍼블릭 인터넷을 사용하는 것보다는 AWS 네트워크를 사용하는 것이 좋다.

### Unicast vs Anycast IP

Unicast IP

- 하나의 서버가 하나의 IP 주소를 가진다.

Anycast IP

- 모든 서버가 동일한 IP 주소를 가지며 클라이언트는 가장 가까운 서버로 라우팅된다.
- Global Accelerator는 Anycast IP 개념을 사용한다.
- 사용자는 바로 서버로 접근하는 것이 아니라 가장 가까운 엣지 로케이션을 통해 서버로 라우팅된다.
- 애플리케이션에 사용하기 위해 2개의 Anycast IP가 생성되면 모두 글로벌하다.
- Anycast IP는 사용자와 가장 가까운 엣지 로케이션으로 트래픽을 직접 전송
- 함께 사용되는 서비스
    - Elastic IP
    - EC2 Instances
    - ALB
    - NLB
    - 위 서비스가 공용일 수도 있고 사설일 수도 있다.
- 안정적인 성능
    - 지능형 라우팅으로 지연 시간이 가장 짧은 엣지 로케이션으로 연결
    - 장애가 발생한 경우 신속한 장애초치가 이루어진다.
    - 캐시하지 않기에 클라이언트 캐시와도 문제없다.
- 상태 확인
    - Global Accelartor가 애플리케이션에 대해 상태확인을 실행
    - 한 리전에 있는 한 ALB에 대해 상태확인을 실패하면 자동화된 장애 조치가 1분 안에 정상 엔드 포인트로 실행

### Security

- 단 2개의 외부 IP만 존재하므로 보안 측명에서 매우 안전
- DDoS 방어 및 보호
- Global Accelerator & CloudFront

### CloudFront vs Global Accelerator

CloudFront

- 이미지나 비디오처럼 캐시 가능한 내용과 API 가속 및 동적 사이트 전달 같은 동적 내용에 대해 성능을 향상시킨다.
- 콘텐츠는 엣지 로케이션에서 제공
- 가끔 한 번씩 출처로부터 내용을 가져온다.
- 캐시된 내용을 엣지로부터 가져와서 전달

Global Accelerator

- TCP나 UDP상의 다양한 애플리케이션 성능을 향상
- 패킷은 엣지 로케이션으로부터 하나 이상의 AWS 리전에서 실행되는 애플리케이션으로 프록시된다.
- 모든 요청이 애플리케이션으로 전달되며 캐싱은 불가능하다.
- 게임이나 IoT 또는 Voiceover IP 같은 비 HTTP를 사용할 경우에 매우 적합
- 글로벌하게 고정 IP를 요구하는 HTTP에도 적합
- 결정적이고 신속한 리전 장애 조치에 적합

## 참고 자료
[AWS Docs - Amazon CloudFront란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
[정보 문화사 - 아마존 웹 서비스](https://www.yes24.com/Product/Goods/69304366)      