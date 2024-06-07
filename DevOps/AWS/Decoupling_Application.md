# Decoupling Application
애플리케이션을 개발할 때 애플리케이션 간 결합도를 낮추는 것은 애플리케이션의 확장성, 유연성, 유지보수성에 유리하기 때문에 중요합니다. 이러한 점은 MSA에서 잘 드러나는데 그 이유는 MSA에서는 애플리케이션이 작은 서비스로 독립되어 동작해서 어떤 마이크로 서비스의 코드가 변경이 되더라도 다른 마이크로 서비스에 영향이 없고 새로운 서비스가 추가될 때 서비스간 적절한 통신 구조만 구성이 된다면 확장이 용이하기 때문입니다. MSA에서는 독립된 애플리케이션간 통신을 위해 EDM(Event Driven Microsoft)를 적용한 아키텍처를 설계하는데 이 때 비동기 메시지 큐 방식이 적합합니다. AWS에서는 SQS, SNS, Kinesis와 같은 메시지나 데이터를 전달하는 클라우드 네이티브 기술을 제공하며 기본 MQTT, AMQT 프토토콜을 사용하는 애플리케이션을 위한 Amazon MQ와 같은 서비스도 제공합니다.

## Messaging

- 애플리케이션을 여러개 배포할 때 정보와 데이터를 공유하기 위해 애플리케이션 간 소통이 필요하다.
- 소통 패턴
    - Synchronous Communication - 동기 커뮤니케이션
        - 애플리케이션과 애플리케이션을 직접 연결
    - Asynchronous / Event based - 비동기 혹은 이벤트 기반
        - ‘큐’라는 미들웨어가 애플리케이션들을 연결
- 애플리케이션 동기화는 트래픽이 갑자기 증가하거나 한 서비스가 다른 서비스를 압도하는 경우 문제가 발생한다.
- 트래픽이 갑자기 증가하는 일이 빈번하거나 예측할 수 없을 때 일반적으로 애플리케이션을 분리하고 분리 계층을 확장하는 것이 좋다.
    - SQS : 대기열 모델
    - SNS : pub/sub 모델
    - Kinesis : 실시간 스트리밍 모델 - 대용량 데이터를 다룰 때
- 분리 애플리케이션은 다른 서비스와 독립적으로 확장할 수 있다.

## Amazon SQS

- 간단한 대기열 서비스
- 메시지 포함
- 메시지를 생성하여 대기열에 보내는 주체를 생산자 - producer라고 하며 하나 이상 존재한다.
- 대기열에서 메시지를 받는 주체를 소비자 - consumer라고 하며 대기열에서 자신 앞으로 온 메시지를 확인하여 메시지를 받고 메시지를 처리하면 대기열에서 메시지를 삭제한다. 여러 소비자가 메시지를 소비할 수 있도록 할 수 있다.
- 대기열은 생산자와 소비자를 분리하는 버퍼역할을 수행

### Standard Queue

- AWS의 첫 번째 서비스 중 하나
- 완전 관리형 서비스이며 애플리케이션을 분리하는데 사용
- 속성
    - 무제한 처리량을 제공 - 원하는 만큼 메시지를 보낼 수 있고 원하는 만큼 대기열에 메시지를 포함시킬 수 있다.
    - 메시지는 수명이 짧다.
        - 기본적으로 4일 최대 14일까지 대기열에서 유지
    - 저지연 시간
        - 메시지를 보내거나 읽는데 10ms 미만의 지연 시간 가능
    - 전송되는 메시지당 256KB 미만
- 중복 메시지가 있을 수 있다.
- 최선의 오더라는 뜻으로 품절 메시지를 보낼 수 있다. - 이 제한을 처리할 수 있는 SQS의 다른 제품 유형이 있다.

### Producing Message

- 최대 256KB 크기의 메시지가 생산자에 의해 SQS로 전송
- 생산자는 SDK를 사용하여 SQS에 메시지를 전송할 수 있다.
    - API : SendMessage
- 소비자가 메시지를 읽고 처리할 때까지 메시지는 SQS 대기열에 유지
- 메시지 보존
    - 4일 ~ 14일
- 예시 : 처리를 위한 명령 - 메시지에 포함되는 항목
    - Order ID
    - Customer ID
    - 원하는 속성

### Cosuming Messages

- 소비자는 일부 코드로 작성해야하는 애플리케이션이고 어떠한 플랫폼에서 실행 중이다.
    - EC2 인스턴스
    - 서버
    - AWS Lambda
- 소비자는 대기열에서 메시지를 풀링하며 최대 10개까지 가능하다.
- 메시지를 처리해야하는 의무가 있다.
- 메시지를 처리하면 메시지를 대기열에서 삭제

### Multiple Consumers

- 소비자는 대기열에서 병렬로 메시지를 처리할 수 있다.
- 메시지가 소비자에 의해 충분히 빠르게 처리되지 않으면 다른 소비자가 같은 메시지를 받을 수 있다.
    - 최선의 노력으로 메시지 순서 지정
- 메시지가 처리되면 대기열에서 삭제
- 대기열에 메시지가 많아서 처리량을 늘려야 한다면 소비자를 수평확장해서 처리량 개선 가능
    - ASG와 연동해서 사용가능하다는 의미
    - 이 경우에 ASG가 사용하는 지표는 CloudWatch 지표 중 대기열 길이를 사용한다.
    - 대기열의 길이가 일정 수준을 넘어서면 CloudWatch 경보를 발생시키고 해당 경보를 트리거로 오토 스케일링 그룹의 용량을 증가시킨다.

### Security

- Encryption
    - HTTPS API를 사용하여 전송 중 암호화
    - KMS 키를 사용하여 미사용 암호화를 얻는다.
    - 원하는 경우 클라이언트 측 암호화 가능 - 클라이언트가 자체적으로 암호화 및 암호 해독을 수행
- Access Control
    - 액세스 제어를 위한 IAM 정책은 SQS API에 대한 액세스를 규제
- SQS Access Policies
    - S3 버킷 정책과 유사
    - SQS 대기열에 대한 교차 계정 액세스 수행하려는 경우
    - Amazon SNS 혹은 Amazon S3 같은 다른 서비스가 SQS 대기열에 S3 이벤트 같은 것을 쓸 수 있도록 허용

### Message Visibility Timeout

- 소비자가 메시지를 풀링하면 해당 메시지는 다른 소비자들에게 보이지 않는다.
- 기본 메시지 가시성 시간 초과는 30초이다.
    - 30초 안에 메시지가 처리되어야 한다.
    - 이 시간 내에 다른 소비자가 해당 메시지를 요청하면 메시지가 대기열에 없어 반환되지 않는다.
    - 이 시간이 지난 후에도 메시지가 삭제되지 않는다면 메시지를 다시 대기열에 넣고 다른 소비자가 받을 수 있게 한다.
- ChangeMessageVisibility
    - 소비자가 메시지를 적극적으로 처리하고 있지만 시간이 더 필요할 경우에 사용
    - 소비자가 가시성 시간 초과를 넘어 해당 메시지를 두 번 처리하고 싶지 않을 때 소비자는 이 API를 호출하여 SQS에 알려야한다.
- 시간설정
    - 너무 오래 처리하도록 시간을 설정하면 소비자가 충돌했을 때 충돌한 메시지가 다시 나타날때까지 몇시간이 걸릴 수 도 있다.
    - 너무 낮은 값을 사용하면 메시지를 처리할 시간이 충분하지 않아 다른 소비자가 메세지를 여러 번 읽을 것이며 중복 처리될 수 있다.
    - 애플리케이션에 합당한 것으로 설정

### Long Polling

- 소비자가 메시지를 요청하는데 대기열에 메시지가 없을 때 대기하는 것을 “Long Polling”이라고 한다.
- 사용 이유
    - 지연 시간을 줄이기 위해
    - SQS로 보내는 API 호출 숫자를 줄이기 위해
    - 애플리케이션의 효율성을 증가시키고 지연 시간을 줄일 수 있다.
- 1~20 사이의 대기 시간을 설정할 수 있다.
- Long Polling을 Short Polling보다 더 선호할 것이다.
- 구성 방법
    1. 대기열 수준에서 구성하여 풀링하는 아무 소비자로부터 Long Polling을 활성화
    2. WaitTimeSeconds를 지정하여 소비자가 스스로 Long Polling 하도록 선택할 수 있다.

### FIFO Queue

- 먼저 들어온 메시지가 먼저 나갈 수 있도록 정렬된 대기열
- 표준 대기열보다 순서가 더 확실히 보장되는 대기열
- 소비자가 대기열에 들어온 순서대로 메시지를 받도록 보장
- 대기열 처리량에 제한이 있다.
    - 300msg/s - 묶음이 아닐 경우
    - 3000msg/s - 묶음을 사용하는 경우
- 중복 제거가 가능한 FIFO 대기열의 기능으로 인해 메시지는 소비자에게 정확히 한 번만 보낼 수 있다.
- 메시지가 소비자에 의해 순서대로 처리
- 생성할 때 “Content-based deduplication”을 활성화하면 5분 이내에 같은 메시지가 들어왔을 때 중복을 방지한다.

### SQS + ASG

- ASG와 SQS를 함께 사용할 때 소비자는 ASG에 속한 EC2 인스턴스들이 되며 이들이 SQS 대기열에서 메시지를 풀링한다.
- CloudWatch - 대기열 길이 지표를 사용해서 오토 스케일링을 한다.
    - AppoximateNumberOfMessages
    - 일정 대기 메시지량을 넘어서면 경보를 발생시키고 경보를 트리거로 스케일링한다.
    
    ### SQS를 DB의 버퍼로 사용
    
    - ASG에서 DB로 향하는 트랜잭션이 너무 많아질 경우에 DB에 직접 연결해서 사용하는 것은 효율성이 떨어진다.
    - SQS는 무한 확장이 가능하므로 트랜잭션을 SQS 대기열에 먼저 쓰는 방법을 사용한다.
    - 트랜잭션을 SQS 메시지로 전송
    - SQS 뒤에 Dequeue역할을 하는 ASG를 추가로 생성하여 DB에 삽입하도록 구성하여 Enqueue를 담당하는 ASG가 생산자 Dequeue를 담당하는 ASG가 소비자 역할을 하게 한다.
    - 이 패턴은 클라이언트에게 따로 DB에 쓰기 작업 확인을 전송할 필요가 없을 때만 사용 가능
    
    ### SQS를 프론트앤드와 백앤드의 분리에 사용
    
    - 프론트앤드 ASG에서 사용자 요청을 받아 SQS 대기열에 메시지를 저장
    - 백앤드 ASG에서 SQS 대기열에서 메시지를 받아 처리

## Amazon SNS
Simple Notification Service

**Direct Integration**

- 메시지 하나를 여러 수신자에게 보낼 때 Direct Integration을 사용할 수 있다.
- 새로운 수신 서비스를 추가할 때마다 통합을 생성하고 작성해야하므로 번거로울 수 있다.

**Pub / Sub**

- 메시지를 SNS 주제로 전송할 수 있다.
- 주제에는 여러 구독자를 가지고 있을 수 있다.
- SNS 주제에서 메시지를 수신하고 보관할 수 있다.
- SNS에서 이벤트 생산자는 한 SNS 주제에만 메시지를 보낸다.
- 이벤트 수신자 혹은 구독자는 구독 중인 주제와 관련한 SNS 알림을 받으려고 한다.
    - 구독자는 구독 중인 주제의 메시지를 모두 받게 된다.
    - 메시지 필터링 가능
- 주제별로 최대 1250만 구독자를 가질 수 있다. - 변경될 수 있다.
- 계정당 가질 수 있는 주제 수 : 10만개 - 변경될 수 있다.
- 게시 가능한 메시지
    - 직접 Email 전송
    - SMS 및 모바일 알림 전송
    - 지정된 HTTP / HTTPS 엔드 포인트로 직접 데이터를 전송
    - SQS와 같은 특정 AWS 서비스와 통합하여 메시지를 대기열로 직접 전송
    - 메시지 수신 후 Lambda 함수가 코드를 수행하도록 전송
    - Firehose를 통해 데이터가 Amazon S3나 Redshift로 보낼 수 있다.

### Integration with AWS Services

- SNS로 데이터 전송
    - CloudWatch
    - ASG
    - CloudFormation
    - AWS Budgets
    - S3 Bucket
    - AWS DMS
    - Lambda
    - DynamoDB
    - RDS Events
    - …
- AWS에서 알림이 발생하면 서비스들이 지정된 SNS 주제로 알림을 전송

### How to Publish

- Topic Publish - SDK 사용
    - 주제 생성
    - 하나 이상의 구독 생성
    - SNS 주제에 게시하면 구독자들이 자동으로 메시지를 받는다.
- Direct Publish - 모바일 SDK 전용
    - 플랫폼 애플리케이션 생성
    - 플랫폼 엔드 포인트 생성
    - 플랫폼 엔드 포인트에 게시
    - Google GCM, Apple APNS, Amazon ADM에 사용 가능
    - 모두 모바일 애플리케이션으로 알림을 수신

### Security

- Encryption
    - 전송 중 암호화 : HTTPS API
    - 저장 데이터 암호화 : KMS 키
    - 클라이언트 측 암호화 : 클라이언트가 SNS에 암호화된 메시지를 보내려는 경우 사용, 암호화 및 복호화는 클라이언트에 책임이 있다.
- Access Control
    - IAM 정책으로 SNS API를 규제
- SNS Access Policy
    - S3 버킷 정책
    - SNS 주제에 교차 계정 액세스 권한 부여
    - S3 이벤트와 같은 서비스가 SNS 주제에 작성할 수 있도록 허용

### SNS + SQS : Fan Out

- 메시지를 여러 SQS 대기열에 보내고 싶은데 모든 SQS 대기열에 개별적으로 메시지를 보내면 애플리케이션이 비정상 종료되는 경우나 전달에 실패하는 경우, SQS 대기열이 더 추가되는 경우에 문제가 발생할 수 있다.
- 이러한 경우 Fan Out 패턴을 사용한다.
- SNS 주제에 메시지를 전송한 후 원하는 수의 SQS 대기열이 SNS 주제를 구독하게 하는 방식
- 완전 분리된 모델이며 데이터 손실이 발생하지 않는다.
- SQS로 작업을 다시 시도할 수 있을 뿐 아니라 데이터 지속성, 지연 처리도 수행가능
- 필요할 경우 SNS 주제를 구독하는 SQS 대기열을 더 추가할 수 있다.
- SQS 액세스 정책에서 SNS 주체가 대기열에 쓰기 작업을 할 수 있도록 허용
- 교차 리전 전달 가능 : 보안상 가능하다면 한 리전의 SNS 주제에서 다른 리전의 SQS 대기열로 메시지를 보낼 수 있다.
    
    ### Application : S3 Events to Multiple Queues
    
    - S3 이벤트 규칙 제한 조건
        - 객체 생성과 같은 이벤트 유형과 /image와 같은 접두사 조합이 동일하다면 S3 이벤트 규칙은 한 가지여야 한다.
        - 여러 대기열에 동일한 S3 이벤트 알림을 보내고 싶을 때 Fan Out 패턴을 사용한다.
    
    ### Application : SNS to Amazon S3 through Kinesis Data Firehose
    
    - KDF를 통해 SNS에서 Amazon S3로 직접 데이터를 전송 가능
    - KDF 서비스가 SNS 주제를 구독하여 특정한 KDF 목적지 어디든 전달 가능
    
    ### Amazon SNS - FIFO Topic
    
    - SQS FIFO와 유사하다.
        - 메시지 그룹 ID에 따라 정렬
        - 중복 제거 ID나 내용을 비교하여 중복 데이터 제거
    - SQS FIFO 대기열을 FIFO SNS 주제의 구독자로 설정
    - 처리량을 제한적이며 SQS FIFO와 동일
    - 사용 이유
        - SQS FIFO를 활용한 Fan Out을 사용하기 위해서 팬아웃, 순서화, 중복 제거가 필요하다.

### SNS - Message Filtering

- JSON 정책을 사용하여 SNS 주제를 구독할 때 전송되는 메시지를 필터링
- 구독에 아무런 필터링 정책이 없다면 모든 메시지를 받는다.

## Amazon Kinesis

- 실시간 스트리밍 데이터를 손쉽게 수집하고 처리하여 분석 가능
- 실시간 데이터
    - 애플리케이션 로그
    - 지표
    - 웹 사이트 클릭스트림
    - IoT 원격 측정 데이터
    - 실시간으로 빠르게 생성되는 데이터
- 서비스 종류
    - Kinesis Data Streams : 데이터를 스트림 수집하여 처리, 저장
    - Kinesis Data Firehose : AWS 내부나 외부의 데이터 저장소로 데이터 스트림을 읽어 들인다.
    - Kinesis Data Analytics : SQL 언어나 Apache Flink를 활용하여 데이터 스트림을 분석
    - Kinesis Video Streams : 비디오 스트림을 수집하고 처리하여 저장

### Kinesis Data Streams

- 시스템에서 큰 규모의 데이터 흐름을 다루는 서비스
    - 여러 개의 shard로 구성
    - 각 shard는 순서대로 번호가 부여되며 사전에 사용자가 프로비저닝 해야한다.
- 데이터는 모든 shard에 분배
- Shard는 데이터 수집률이나 소비율 측면에서 스트림의 용량을 결정
- 데이터 보존 기간 1일 ~ 365일
    - 데이터를 다시 처리하거나 확인 가능
- 데이터가 Kinesis에 들어오면 삭제 불가능
- 데이터 스트림으로 메시지를 전송하면 파티션 키가 추가되며 같은 파티션 키를 가진 스트림들은 같은 shard로 들어가게 되어 키를 기반으로 데이터 정렬 가능

#### 생산자

- 생산자는 매우 낮은 수준의 SDK에 의존하며 Kinesis Data Stream에 레코드를 전달
    - 데이터를 스트림으로 보낼 때 shard마다 초당 1MB를 전송하거나 1000msg/s를 전송할 수 있다.
- 생산자 레코드
    - Partition Key : 레코드가 사용할 shard 결정에 사용
    - Data Blob : 최대 1MB 크기
- AWS SDK, KPL - Kinesis Producer Library, Kinesis Agent를 사용하여 데이터를 전송 가능

#### 소비자

- 데이터가 스트림에 들어가면 많은 소비자가 이 데이터를 사용한다.
    - SDK에 의존하거나 높은 수준에서는 KCL - Kinesis Client Library에 의존하는 애플리케이션
    - Kinesis 스트림에서 서버리스로 처리하려는 경우 Lambda 함수 사용
    - Kinesis Data Firehose
    - Kinesis Data Analytics
- 소비자 레코드
    - Partition Key
    - Sequence no. : Shard에서 레코드의 위치를 나타낸다.
    - Data Blob
- 메시지를 받는 방법
    - Shard마다 초당 2MB의 처리량을 모든 소비자가 공유
    - 소비자마다 shard당 2MB/s씩 받을 수 있다.
        - 효율성을 높인 소비 유형 - 팬아웃 방식
- KCL, AWS SDK를 활용하여 데이터를 직접 작성가능
- AWS Lambda, Kinesis Data Firehose, Kinesis Data Analytics를 활용

#### 용량 모드

- 프로비저닝 모드
    - 프로비저닝할 샤드 수를 정하고 API를 활용하거나 수동으로 조정
    - 각 shard는 초당 1MB나 1천 개의 레코드를 받아들임
    - 각 shard는 초당 2MB를 출력
    - shard를 프로비저닝할 때마다 시간당 비용이 부과
- 온디맨드 모드 - New
    - 프로비저닝하거나 용량을 관리할 필요가 없다.
    - 기본적으로 4MB/s 또는 4000개의 레코드를 처리
    - 지난 30일 동안 관측한 최대 처리량에 기반하여 자동으로 조정된다.
    - 시간당, 스트림당 송수신 데이터양(GB 단위)에 따라 비용이 부과
    - 사전에 사용량을 예측할 수 없다면 온디맨드 유형을 사용

#### 보안

- IAM 정책을 사용하여 shard를 생성하거나 shard에서 읽어 들이는 접근 권한을 제어
- 전송 중 암호화 : HTTPS
- 저장 중 암호화 : KMS
- 클라이언트 측 암호화 및 복호화 가능 - 어렵지만 보안이 더 좋다.
- VPC 엔드 포인트 사용가능
    - 인터넷을 거치지 않고 프라이빗 서브넷의 인스턴스에서 직접 손쉽게 접근 가능
- 모든 API 요청은 CloudTrail로 감시 가능

### Kinesis Data Firehose - 2024년 기준 Amazon Data Firehose로 이름이 변경
경
- 생산자에서 데이터를 가져올 수 있는 유용한 서비스
    - 주로 Kinesis Data Stream에서 가져온다.
- 생산자에게 레코드를 가져오면 배치 쓰기로 수신자에게 쓸 수 있으며 옵션으로 Lambda 함수를 사용하여 데이터를 변환할 수도 있다.
- 완전 관리형 서비스이며 자동으로 용량이 조정되고 서버리스이므로 관리할 서버가 없다.
    - AWS Redshift, Amazon S3, ElasticSearch나 서드파티, HTTP 엔드 포인터로 데이터 전송 가능
- Firehose를 통한 데이터에 대해서만 비용을 지불
- 근 실시간 모델 - 수신처로 데이터를 배치로 쓰기 때문
    - 배치 - 데이터 모아서 한번 작성
    - 전체 배치가 아닌 경우 최소 60초의 지연시간이 발생
    - 한 번에 적어도 1MB의 데이터가 있을 때까지 기다려야한다.
- 여러 데이터 형식과, 데이터의 전환, 변환, 압축을 지원 혹은 lambda를 활용하여 자체적인 데이터 변환 사용 가능
- 실패 혹은 모든 데이터를 백업 S3 버킷에 전송

#### 수신처

- Amazon S3
    - 모든 데이터를 S3에 작성 가능
- Amazon Redshift
    - 데이터 웨어하우스
    - 먼저 S3에 데이터를 작성한 후 복사
- Amazon ElasticSearch
- 서드 파티
    - Datadog
    - Splunk
    - New Relic
    - MongoDB
- 사용자 지정
    - HTTP 엔드 포인트
- 수신처에서 데이터를 수신받은 후 작업 - 택 1
    - 모든 데이터를 백업으로 S3버킷에 전송
    - 수신처에 쓰이지 못한 데이터를 실패 S3 버킷에 전송

#### Kinesis Data Stream vs Firehose

Kinesis Data Stream

- 데이터를 대규모로 수집할 때 사용하는 스트리밍 서비스
- 생산자와 소비자에 대해 커스텀 코드를 작성 가능
- 200ms 미만의 실시간
- 용량을 직접 조정할 수 있어 shard의 분할이나 샤드 병합을 통해 용량이나 처리량을 늘릴 수 있다.
- 1일 ~ 365일동안 데이터 저장 가능
- 데이터 재사용 및 확인 가능

Kinesis Data Firehose

- 데이터 수집 서비스로 S3나 Redshift, ElasticSearch, 서드 파티 파트너나 사용자 지정 HTTP 엔드포인트로 스트리밍 한다.
- 완전 관리형 서비스, 서버리스
- 근 실시간
- 오토 스케일링
- firehose를 통과하는 데이터에 대해서 비용 청구
- 데이터 저장 기능 없음

### Ordering Data into Kinesis

- SQS FIFO와 Kinesis 모두 데이터를 정렬할 수 있지만 사용하는 기술을 전혀 다른 기술을 사용한다.
    
    ### Kinesis
    
    - Kinesis는 파티션 키를 활용한다
    - 같은 파티션 키를 사용하면 항상 동일한 shard로 전달되며 파티션 키가 같은 데이터는 항상 같은 shard에 정렬된다.
    - shard 수준에서 정렬된 데이터를 얻을 수 있다.
    - 100개의 데이터와 5개의 shard가 있는 경우
        - 평균적으로 20개의 데이터가 각 shard에 분배된다.
        - 데이터는 각 shard에 순서대로 정렬
        - 최대 소비자 개수는 shard의 개수와 같은 5개로 제한된다.
        - 최대 5MB/s의 데이터를 처리할 수 있다.
    - 많은 데이터를 전송하고 Kinesis Data Stream에 shard 당 데이터를 정렬할 때 유용
    
    ### SQS FIFO
    
    - SQS 자체로는 데이터를 정렬하는 기능이 없으므로 SQS FIFO를 사용해 데이터를 정렬한다.
    - SQS FIFO는 메시지가 그룹 ID를 사용하지 않으면 보내진 순서를 따르며 소비자는 하나만 존재
    - 소비자를 스케일링하고 서로 연관된 메시지를 그룹화하려는 경우 그룹 ID를 사용할 수 있다. - Kinesis의 파티션 키와 개념이 비슷하다.
        - 정의한 그룹마다 각각 소비자를 가질 수 있다.
    - 100개의 데이터와 1개의 SQS FIFO가 있는 경우
        - 100개 상응하는 그룹 ID를 100개 생성
        - 소비자의 개수는 그룹 ID의 개수와 같은 최대 100개까지 될 수 있다.
        - 초당 300 메시지 혹은 배치를 사용하여 초당 3000 메시지를 처리 가능
    - 그룹 ID 숫자에 따른 동적 소비자 수를 원할 때 유용

### SQS vs SNS vs Kinesis

**SQS**

- 소비자가 대기열에서 데이터를 가져오는 모델
- 데이터를 처리한 후 소비자가 대기열에서 삭제
- 작업자나 소비자 수는 제한이 없다.
- 관리형 서비스로 처리량을 프로비저닝할 필요가 없고 빠르게 수백 수천개의 메시지로 확장 가능
- 순서를 보장하려면 FIFO 대기열을 활성화
- 각 메시지에 지연시간이 있어 30초 혹은 지정한 시간 뒤에 대기열에서 보이도록 할 수 있다.

**SNS**

- Pub/Sub 모델로 다수의 구독자에게 데이터를 푸시하면 메시지의 복사본을 받는다.
- 주제별로 1250만 구독자까지 보유 가능
- 데이터가 한 번 SNS에 전송되면 영구적으로 보관되지는 않는다. 제대로 전달되지 않으며 데이터 손실 가능성이 있다.
- 계정당 10만개 주제로 확장 가능
- 처리량을 프로비저닝하지 않아도 된다.
- Fan Out 패턴으로 SQS와 결합 가능
- SNS FIFO 주제를 SQS FIFO 대기열과 결합 가능

**Kinesis**

- Standard
    - 데이터를 pull
    - 2MB per shard
- Enhanced-fan out
    - 데이터를 소비자에게 push
    - Shard 하나에 소비자당 2MB/s 속도
- 데이터가 유지되므로 데이터를 재처리하거나 확인 가능
- 실시간 빅데이터 분석, ETL 등에 활용
- Shard 수준에서 정렬이 가능하므로 Kinesis Data Stream마다 원하는 shard 양을 지정해야한다.
- 데이터 만료 기간 지정
- 용량을 미리 프로비저닝하거나 온디맨드로 사용할 수 있다.

## Amazon MQ

- SQS, SNS는 AWS 독점 기술로 클라우드 네이티브 서비스이다.
- On-premise에서 기존 애플리케이션을 실행하는 경우 개방형 프로토콜인 MQTT, AMQP, STOMP, Openwire, WSS 등을 사용
- 애플리케이션을 마이그레이션할 때 SQS나 SNS을 사용한 리팩토링을 원하지 않고 기존 MQTT나 AMQP 프로토콜을 사용하고 싶을 때 Amazon MQ를 사용하면 된다.
- RabbitMQ와 ActiveMQ 두 가지 기술을 위한 관리형 메시지 브로커 서비스
- SQS나 SNS처럼 확장성이 크지 않다.
    - Amazon MQ는 서버에서 실행되므로 서버 문제가 발생할 수 있기 때문
- 고가용성을 위해 장애 조치와 함께 다중 AZ 설정을 실행할 수 있다.
- SQS처럼 보이는 대기열 기능과 SNS처럼 보이는 주제 기능을 단일 브로커의 일부로 제공
    
    ### High Avaiability
    
    - 2개의 가용영역이 있을 때 한 영역은 활성 상태, 다른 하나는 대기 상태로 Amazon MQ 브로커를 추가
    - 장애 조치를 위해 백앤드 스토리지로 EFS를 정의
        - EFS는 활성 상태인 브로커와 대기 상태의 브로커에 모두 마운트되어 있으므로 장애 조치가 가능

## 참고 자료
[AWS Docs - Amazon 심플 큐 서비스란?](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)   
[AWS Docs - Amazon SNS란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/sns/latest/dg/welcome.html)   
[AWS - Amazon Kinesis](https://aws.amazon.com/ko/kinesis/)      
[AWS Docs - Amazon Kinesis Data Streams란?](https://docs.aws.amazon.com/ko_kr/streams/latest/dev/introduction.html)     
[AWS Docs - Amazon 데이터 파이어호스란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/firehose/latest/dev/what-is-this-service.html)    
[AWS Docs - Amazon MQ란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html)   