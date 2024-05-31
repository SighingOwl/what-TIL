# AWS란?
AWS는 클라우드 서비스 제공자이며 Microsoft Azure, Google GCP와 함께 클라우드 시장에서 가장 많은 사용자 비중을 차지하고 있다. 전세계에 200개 이상의 데이터 센터에서 서비스를 제공한다. 클라우드에서 컴퓨팅, 스토리지, DB과 같은 인프라 기술과 AI/ML와 같은 고급 기능도 제공한다. 또한 사용자에게 뛰어난 보안을 제공하여 IT 인프라를 사용함에 있어 보안에 대한 부담을 줄일 수 있다. AWS는 지속적으로 새로운 기능을 클라우드에 추가하고 있어 현재 300개 정도의 서비스를 제공한다.       
> 출처 : [AWS란?](https://aws.amazon.com/ko/what-is-aws/?nc1=f_cc)

## 클라우드 컴퓨팅이란?
클라우드 컴퓨팅은 IT 인프라나 리소스를 인터넷을 통해 사용하는 것을 의미하며 사용한만큼 비용을 지불하는 것이 가장 큰 특징이다. 따라서 클라우드 컴퓨팅 사용자는 클라우드에 접속할 장치를 인터넷에 연결하면 언제든 클라우드에 접근하여 컴퓨팅 작업을 수행할 수 있다.       
AWS에서 말하는 클라우드 컴퓨티의 이점은 다음과 같다.
- 민첩성
- 탄력성
- 비용 절감
- 몇 분만에 전세계에 배포   

이러한 클라우드 컴퓨팅은 사용자에게 3가지 유형으로 제공된다.
| 유형 | 설명 |
| --- | --- |
| IaaS - Infrastructure as a Service | IT 인프라를 임대해주는 서비스를 의미하며 가장 기본적인 클라우드 서비스이다. AWS에서 EC2와 S3와 같은 서비스를 의미한다. |
| PasS - Platform as a Service | 서비스나 애플리케이션 개발 및 실행에 필요한 플랫폼을 제공하는 서비스이다. 이를 활용하여 개발 환경을 구축하는 부담을 줄이고 빠른 서비스 개발을 돕는다. |
| SaaS - Service as a Service | 애플리케이션을 클라우드 환경에서 동작시켜 사용자에게 제공하는 서비스이다. 서비스 공급자에 의해 관리되므로 사용자가 서비스 유지 관리를 하지 않아도 된다. 기존 패키지 소프트웨어 처럼 구매하는 방식이 아닌 필요한 기능을 필요한 기간 동안만 사용하여 비용을 지불하는 형태. Gmail이나 MS office 365와 같은 서비스가 SaaS이다. | 

<img src="/images/_cloud_computing_type.drawio.png" width="75%" height="75%" title="types of cloud computing" alt="types of cloud computing">   

> 출처 : [클라우드 컴퓨팅이란 무엇입니까?](https://aws.amazon.com/ko/what-is-cloud-computing/?nc1=f_cc)

## AWS 글로벌 인프라
AWS는 전세계에 인프라를 제공할 수 있다. 이것이 가능한 이유는 전세계에 리전을 두고 이를 AWS 사설 네트워크로 연결하였기 때문이다. 리전은 최소 3개의 가용 영역(AZ)로 구성되어 사용자에게 고가용성을 제공할 수 있다     
이와 별개로 엣지 로케이션을 두어 사용자에게 더 빠르게 컨텐츠를 전달할 수 있는 POP을 보유하고 있다.      
    <img src="/images/AWS_region_1.png" width="50%" height="50%" title="AWS region" alt="AWS region">   



### 리전
AWS 리전은 전세계에 있는 데이터센터를 클러스터링하는 물리적 위치를 의미한다. 각 AWS 리전은 서울 리전, 버지니아 북부 리전, 도쿄 리전 처럼 지리적 영역 내에서 격리된다.   
AWS애서 리전을 선택할 때는 지연시간이나 요금, AWS 서비스 제공 여부, 법률 준수를 고려해야한다. AWS 사설 네트워크를 사용하긴 하지만 애플리케이션이 사용자와 가까이에서 서비스될 때 지연시간과 네트워크 요금을 줄일 수 있다. 그리고 AWS의 일부 서비스는 특정 리전에서 제공되지 않을 수 있어 서비스가 제공되는 리전을 선택해야 할 때도 있다. 법률 준수는 한국과 연관성이 매우 큰데 예를 들어 만일 한국에서 지도 서비스를 제공하고 싶을 때 한국의 지도 데이터는 한국 외에 보관해서는 안된다는 법률을 준수해야하는 점이 있다.

### 가용 영역 - AZ
가용 영역은 각 리전에 3개 이상으로 구성되어 있는 물리적으로 독립된 하나 이상의 개별 데이터 센터를 의미한다. 하나의 가용 영역에 하나의 데이터 센터가 있을 수도 있고 여러 개의 데이터 센터가 있을 수도 있다. 이러한 가용 영역은 재난 발생에 대비해 서로 나뉘어져 있다. a, b, c 3개의 가용 영역이 있을 때 a 가용 영역에 재난이 발생하더라도 남은 b, c 가용 영역은 아무런 영향이 없도록 설계되어 있다.      
> 출처 : [AWS 글로벌 인프라](https://aws.amazon.com/ko/about-aws/global-infrastructure/regions_az/)     
출처 : [Amazon CloudFront](https://aws.amazon.com/ko/cloudfront/features/?p=ugi&l=ap&whats-new-cloudfront.sort-by=item.additionalFields.postDateTime&whats-new-cloudfront.sort-order=desc)      