# Amazon S3 Security
## Basic
S3 객체 암호화는 4가지 방법으로 할 수 있습니다.

### SSE - Server-Side Encryption

- Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3)
    - Amazon S3에서 관리하는 키를 이용한 서버 측 암호화
    - 버킷과 객체에 대해 기본적으로 활성화되어 있다.
- Server-Side Encryption with KMS Keys stored in AWS KMS (SSE-KMS)
    - KMS 키를 이용해서 암호화 키를 관리
- Server-Side Encryption with Customer-Provided Keys (SSE-C)
    - 사용자가 가지고 있는 암호키를 사용하여 암호화
- DSSE-KMS
    - KMS를 기반으로 하는 이중 암호화 - 시험에 출제되지는 않음
    - 2023년 6월에 출시

### Client-Side Encryption

- 클라이언트 측에서 모든 것을 암호화하여 S3에 업로드

### SSE-S3

- AWS가 처리 및 관리, 소유한 키를 사용해서 암호화, 사용자가 절대 액세스할 수 없다.
- 객체는 서버 측에서 암호화
- AES-256
- Amazon S3가 SSE-S3 메커니즘을 이용해서 객체를 암호화하도록 요청하기 위해 헤더를 “x-amz-server-side-encryption”:”AES256”으로 설정
- 새로운 버킷과 새로운 객체에 대해 기본값으로 활성화

### SSE-KMS

- AWS와 S3 서비스가 보유한 키에 의존하지 않고 사용자가 KMS 서비스로 직접 키를 관리
- 사용자가 키를 통제하고 CloudTrail을 사용해서 키 사용을 검사할 수 있는 장점이 있다.
    - 누군가가 KMS에서 키를 사용할 때마다 CloudTrail에 로그가 남는다.
- 헤더를 “x-amz-server-side-encryption”:”aws:kms”로 설정
- 제한 사항
    - KMS 키에는 GenerateDataKey같은 자체 API가 있고 사용자는 S3에서 다운로드 할 때 Decrypt API를 사용해서 복호화를 진행해야한다.
    - 따라서 다운로드를 사용할 때도 KMS에 API 호출을 해야하며 이 호출 건은 KMA API 초당 호출 쿼터에 합산된다.
    - S3 버킷의 처리량이 매우 많은 경우 KMS로 인해 스로틀링이 발생할 수 있다.

### SSE-C

- 키 자체는 AWS 외부에서 관리되지만 키를 AWS에 전송하여 사용하기 때문에 서버측 암호화로 사용할 수 있다.
- S3는 사용자가 제공한 암호화 키를 절대 저장하지 않으며 사용 후 폐기한다.
- HTTPS 프로토콜을 사용해야한다.
- 모든 요청에 HTTP 헤더의 일부로서 키를 전달
- 다운로드해서 복호화를 하려고 하면 사용자가 제공한 키가 있어야 한다.

### Client-Side Encryption

- 클라이언트가 직접 데이터 암호화를 한 후에 S3에 전송하는 개념
- 데이터를 받아 복호화하는 작업도 동일하게 클라이언트 측에서 이루어진다.

### Encryption in transit - SSL/TLS

- SSL/TLS를 사용한 전송 중 암호화
- Amazon S3는 기본적으로 2개의 엔드포인트가 있다.
    - HTTP Endpoint - 암호화 하지 않음
    - HTTPS Endpoint - 전송 중 암호화 지원
- HTTPS 사용이 권장된다.
- SSE-C 타입의 암호화를 사용하면 HTTPS 프로토콜을 사용해야 한다.

### 전송 중 암호화 강제

- 버킷 정책을 사용해서 전송 중 암호화를 강제할 수 있다.
- 보안 전송을 사용하지 않으면 “GetObject” 작업을 거부 정책을 사용

### 기본 암호화 vs 버킷 정책

- 기본적으로 모든 버킷은 SSE-S3 암호화가 되어 있다.
- 선택적으로 버킷 정책을 사용해서 암호화가 강제되고 올바른 암호화 헤더가 없는 경우 S3 객체를 PUT하는 API 호출을 거절할 수 있다. SSE-KMS, SSE-C에 적용가능

## CORS

- CORS - Cross-Origin Resource Sharing은 교차 오리진 리소스 공유
- Origin : Scheme(Protocol) + Host(Domain) + Port로 구성
    - 예시) https://www.example.com → HTTPS : 443 port
- Web Broswer : CORS는 웹 브라우저 기반 보안 메커니즘으로 메인 오리진을 방문하는 동안 다른 오리진에 대한 요청을 허용하거나 거부
- Same Origin : 체계, 호스트, 포트가 동일할 때 오리진이 같다고 말한다.
    - http://example.com/app1 & http://example.com/app2
- Different Origin
    - http://www.example.com & http://other.example.com
- 웹 브라우저가 한 웹 사이트를 방문하는 동안 요청 체계의 일부로 다른 웹사이트에 요청을 보내야 할 때 다른 오리진이 CORS 헤더를 사용해서 요청을 허용하지 않는 한 해당 요청은 이행되지 않는다.
    - Access-Control-Allow-Origin

### S3

- 클라이언트가 S3 버킷에서 교차 오리진 요청을 하면 정확한 CORS 헤더를 활성화해야한다.
- 이 작업을 빠르게 수행하려면 특정 오리진을 허용하거나 모든 오리진을 허용한다.

## MFA Delete

- S3에 MFA 사용 설정을 하면 S3에서 중요한 작업을 하기 전에 MFA 인증을 해야한다.
- MFA가 필요한 시점
    - 객체 버전을 영구적으로 삭제할 때 필요
    - 버킷에서 버전 관리를 중단할 때 필요
- MFA가 필요하지 않은 작업
    - 버전 관리 활성화
    - 삭제한 버전 목록 나열
- MFA Delete를 사용하려면 버킷에서 버전 관리를 활성화 해야한다.
- 버킷 소유자 - Root 계정만이 MFA Delete를 활성화하거나 비활성화 할 수 있다.

### Enable MFA Delete

- AWS 콘솔에서는 MFA Delete를 활성화할 수 없다. 대신 AWS CLI를 활용해서 가 능하다.
- 루트 계정만이 MFA Delete를 활성화할 수 있으므로 AWS CLI 사용을 위해 루트 계정의 액세스 키를 생성 - 권장하지 않지만 방법이 이것밖에 없다. 작업 후 삭제 요망

```bash
# generate root access keys
aws configure --profile root-mfa-delete-demo
AWS Access Key ID [None] : # 생성한 Root 계정 액세스 키 ID
AWS Secret Access Key [None] : # Root 계정 비밀 액세스 키
Default region name [None] : # 사용할 기본 리전
Default output format [None] : JSON

# enable mfa delete
aws s3api put-bucket-versioning --bucket "버전 이름" --versioning-configuration Status=Enabled,MFADelte=Enabled --mfa "MFA 장치 ARN + MFA 인증 코드" --profile "생성한 프로파일 이름"
```

## Access Logs

- 감사 목적으로 S3 버킷에 대한 모든 액세스를 기록
- 어떤 계정에서든 S3로 보낸 모든 요청은 승인 또는 거부 여부와 상관없이 다른 S3 버킷에 파일로 기록
- 로그는 Amazon Athena와 같은 데이터 분석 도구로 분석될 수 있다.
- 대상 로깅 버킷은 같은 AWS 리전에 있어야 한다.
- S3 버킷에 요청이 발생하면 액세스 로그를 활성화해서 모든 요청이 로깅 버킷에 기록되도록 설정
- 로그 포맷 확인 링크
    
    https://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html
    
- 주의사항
    - 절대로 로깅 버킷과 모니터링하는 버킷을 동일하게 설정하지 않는다.
    - 동일한 버킷으로 설정하면 로깅 루프가 발생하여 로그를 무한 반복 기록하며 버킷의 크기가 기하급수적으로 증가

## Pre-Signed URLS

- S3 콘솔, CLI, SDK를 사용하여 생성할 수 있는 URL
- URL 만료 기한
    - S3 콘솔 : 1분 ~ 12시간
    - AWS CLI : 기본 3600초, 최대 168시간
- 미리 서명된 URL을 생성할 때 URL을 받는 사용자는 URL을 생성한 사용자의 GET 또는 PUT에 대한 권한을 상속
- Use Case
    - 프라이빗 S3 버킷이 있을 때 AWS 외부 사용자에게 한 파일에 대한 액세스 권한 부여가 필요한 상황에서 해당 파일을 퍼블릭 액세스 설정을 원하지 않으면 사용할 수 있다.
        - S3 버킷이 미리 서명된 URL을 제공하고 URL이 자격 증명을 이어받아 해당 파일에 액세스할 수 있는 권한을 부여
        - 제한 시간 내에 파일 액세스 권한을 부여할 대상 사용자에게 보내면 사용자가 URL을 사용해서 S3 버킷의 파일에 액세스
    - 로그인한 사용자만 S3 버킷에서 프리미엄 비디오를 다운로드할 수 있도록 허용하거나 사용자 목록이 계속 변하는 경우 URL을 동적으로 생성해서 파일을 다운로드할 수 있게 한다.
    - 일시적으로 사용자가 S3 버킷의 특정한 위치에 파일을 업로드하도록 허용할 수 있다.

## S3 Lock Policies & Glacier Vault Lock

### Glacier Vault Lock

- WORM - Write Once Read Many 모델을 채용하기 위해 Glacier 볼트를 잠그는 것
- 객체를 Glacier Vault에 넣은 다음 수정하거나 삭제할 수 없도록 잠그는 것
- Glacier 위에 볼트 잠금 정책을 생성
- 향후 편집을 위해 정책을 잠근다. - 정책을 생성 후 잠그면 누구도 변경하거나 삭제할 수 없다.
- 규정 준수와 데이터 보존에 아주 유용

### S3 Object Lock

- Glacier Vault Lock과 유사하지만 조금 더 복잡하다.
- S3 객체 잠금을 활성화하려면 먼저 버저닝을 활성화한다.
- WORM 모델 채택가능
- S3 버킷 전체 잠금이 아닌 객체 수준에서 설정할 수 있는 잠금
- 특정 객체 버전이 특정 시간 동안 삭제 되는 걸 차단할 수 있다.
- Retention Mode
    - Compliance
        - 규정 준수 모드는 S3 Glacier Vault Lock과 매우 유사
        - 사용자를 포함한 그 누구도 객체 버전을 덮어쓰거나 삭제할 수 없다.
        - 보존 모드 자체와 보존 기한을 변경할 수 없다.
    - Governance
        - 규정 준수 모드보다 조금 더 유연하다.
        - 대부분의 사용자는 객체 버전을 덮어쓰기나 삭제, 로그 설정을 변경할 수 없다.
        - 관리자 같은 일부 사용자는 IAM을 통해 부여받은 특별 권한으로 보존 기간을 변경하거나 객체를 바로 삭제할 수 있다.
    - 두 가지 모드 모두 보존 기간을 설정
        - 고정된 기간만큼 객체를 보존할 수 있다.
        - 원하는만큼 기간을 연장 가능
    - Legal Hold
        - S3 버킷 내 모든 객체를 무기한으로 보호
        - 대개 재판에서 사용될 수 있는 중요한 객체를 법적 보존
        - 보존 모드과 기간에 관계없이 영구적으로 보호
        - s3:PutObjectLegalHold IAM 권한을 가진 사용자는 어떤 객체든 법적 보존을 설정하거나 제거할 수 있다.

## Access Point

- 액세스 포인트 정책을 사용해서 특정 데이터가 특정 접두어와 연결되도록 할 수 있다.
    - 액세스 포인트 정책은 S3 버킷 정책과 동일한 형태이다.
- 데이터가 많아짐에 따라 데이터를 그룹으로 관리하기 위해서 사용할 수 있다.
- S3 보안 정책으로 꺼네 액세스 포인트에 넣으므로써 모든 액세스 포인트는 각자의 보안을 가지고 있다.
- 아주 간단하게 보안을 관리할 수 있는 방법이 된다.
- S3 버킷에 대한 액세스를 스케일링 할 수 있다.
- 액세스 포인트가 가지게 되는것
    - 액세스 포인트 소유의 DNS 네임
    - 액세스 포인트가 인터넷 오리진에 연결되거나 프라이빗 트래픽의 경우 VPC 오리진에 연결되도록 할 수 있다.
    - 버킷 정책과 유사한 액세스 포인트 정책 첨부 - 보안 관리를 스케일링 가능

### VPC Origin

- S3 액세스 포인트의 VPC 오리진을 프라이빗 액세스가 가능하도록 정의 가능
- VPC 엔드포인트를 반드시 생성해야한다.
- VPC 엔드포인트에는 정책이 있고 정책에는 타겟 버킷과 액세스 포인트에 대한 액세스를 허용해야한다.

### S3 Object Lambda

- S3 버킷이 호출자 애플리케이션이 객체를 받기 직전에 그 객체를 수정하려할 때 사용
- 버킷을 복제해서 버킷에 각 객체의 다른 버전을 갖는 대신에 S3 객체 람다를 사용할 수 있다.
- 이것을 사용하기 위해 S3 액세스 포인트가 필요하다.
- Use case
    - 분석기나 비프로덕션 환경을 위해 PII - 개인식별정보 데이터을 삭제하는 경우
    - 데이터 형식을 XML에서 JSON으로 변환하는 경우
    - 즉석에서 이미지 크기를 조정하거나 워터마크를 추가하는 경우