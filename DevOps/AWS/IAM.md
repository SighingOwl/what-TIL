# IAM이란?
IAM은 Identity and Access Management의 약자이며 AWS 리소스에 대한 액세스를 안전하게 제어할 수 있는 웹 서비스. 이를 활용해 사용자가 액세스 할 수 있는 AWS 리소스에 대한 제어 권한을 중앙에서 관리할 수 있다.     
[IAM이란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/introduction.html)

## IAM Users & Groups

- Identity and Access Management, Global Service
- AWS 계정을 생성하면 가장 먼저 Root 계정이 기본으로 만들어진다. 보안을 위해 Root 계정은 계정을 생성할 때만 사용, 그 외에는 사용자를 생성해서 AWS 서비스를 사용해야한다.
- 사용자는 개인 사용자 혹은 조직 내 한 사람에 해당하며 사용자는 사용자 그룹으로 묶을 수 있다. 이때 사용자는 여러 그룹에 속할 수 있다.
    
    ### Group을 사용하는 이유
    
    - 같은 업무를 담당하는 사용자들에게 동일한 AWS 권한을 부여하기 위해서 사용
    - 하나의 정책은 사용자의 권한을 정의한다.
    - 사용자가 많은 비용이나 보안 문제를 발생시키는 것을 방지하기 위해 AWS는 사용자에게 모든 권한을 허용하지 않으며 최소 권한의 원칙을 적용한다.
    - 권한 부여는 JSON 문서를 사용한다.
        
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "ecr:*",
                        "cloudtrail:LookupEvents"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "iam:CreateServiceLinkedRole"
                    ],
                    "Resource": "*",
                    "Condition": {
                        "StringEquals": {
                            "iam:AWSServiceName": [
                                "replication.ecr.amazonaws.com"
                            ]
                        }
                    }
                }
            ]
        }
        ```
        
        → IAM 권한 예시 - AmazonEC2ContainerRegistryFullAccess
        
    
    ### Create IAM User
    
    1. AWS 콘솔 액세스가 가능한 IAM 사용자를 만들기 위해서 IAM 사용자를 직접 생성하거나 기존 사용자를 지정할 수 있다.
        - 사용자 이름 설정
        - 콘솔 암호 설정
        
        <img src="/images/AWS_IAM_1.png" width="75%" height="75%" title="aws iam 1" alt="aws iam 1">    
        
    2. 사용자 그룹 생성
        - 사용자 그룹을 생성 후 그룹에 사용자를 추가하면 그룹이 가지고 있는 권한을 사용자가 그대로 승계한다.
        
        <img src="/images/AWS_IAM_2.png" width="75%" height="75%" title="aws iam 2" alt="aws iam 2">    
        
    3. Tag 설정
        - 사용자의 접근을 추적, 조직, 제어할 수 있도록 도와주는 정보
    4. 자격 증명 정보가 있는 csv 파일 다운로드
    
    ### Login with IAM User
    
    1. IAM 대시보드에서 AWS 계정 ID를 확인
        - 보통 AWS 계정 ID는 12자리 숫자로 되어있다.
        - 계정 별칭 혹은 Account Alias를 변경하여 사용할 수 있다.
    2. IAM 사용자로 로그인을 한다. 보통 MFA를 설정해서 사용한다.

## IAM Policies

- IAM 정책은 사용자의 권한을 정의하며 그룹 정책 혹은 인라인 정책을 사용해서 사용자에게 권한을 부여할 수 있다.
    
    ### IAM Policies Structrue
    
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ecr:*",
                    "cloudtrail:LookupEvents"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:CreateServiceLinkedRole"
                ],
                "Resource": "*",
                "Condition": {
                    "StringEquals": {
                        "iam:AWSServiceName": [
                            "replication.ecr.amazonaws.com"
                        ]
                    }
                }
            }
        ]
    }
    ```
    
    | 요소 | 설명 |
    | --- | --- |
    | Version | 정책 언어 버전으로 보통 “2012-10-17”을 사용한다. |
    | ID (Optional) | 정책을 식별하는 ID |
    | Statement(Required) | 하나 혹은 여러 문장 <br> Sid (Optional) : 문장의 식별자 <br> Effect : 특정 API에 접근 허용 여부 설정 <br> Principle : 특정 정책이 적용될 사용자, 계정 혹은 역할 <br> Action : Effect에 따라 허용 혹은 거부되는 API 호출 목록 <br> Resource : Action이 적용될 리소스의 목록 <br> Condition : Statement가 언제 적용될지를 결정 |
    
    ### Create Policies
    
    정책 생성은 JSON 문서를 직업 작성하는 방법도 있지만 Visual editor를 사용해서 생성할 수도 있다.
    
    ### Visual Editor
    
    1. 권한을 부여할 서비스 선택
        
        <img src="/images/AWS_IAM_3.png" width="75%" height="75%" title="aws iam 3" alt="aws iam 3">    
        
    2. 선택한 서비스에서 수행할 작업 선택
        
        <img src="/images/AWS_IAM_4.png" width="75%" height="75%" title="aws iam 4" alt="aws iam 4">    
        
    3. 작업을 수행할 리소스 선택
        1. 모든 작업
        2. 특정 작업 - ARN 지정
    4. Visual editor에서 설정한 내용은 JSON 편집기로 확인가능하다.

## IAM MFA

### Password Policy

- IAM 사용자를 생성할 때 비밀번호 정책을 설정할 수 있다.
    - 최소 길이 설정
    - 특정 문자 포함 설정 : 대소문자, 숫자, 특수 문자
    - IAM 사용자들이 비밀번호 변경 허용/금지 설정
    - 일정 기간마다 비밀번호 변경 요청 설정
    - 이전 비밀번호 사용 금지 설정

### Set Password Policy

1. 계정 설정에서 암호 정책 편집
    
    <img src="/images/AWS_IAM_5.png" width="75%" height="75%" title="aws iam 5" alt="aws iam 5">    
    
2. 사용자 지정 암호 정책 설정 및 저장
    
    <img src="/images/AWS_IAM_6.png" width="75%" height="75%" title="aws iam 6" alt="aws iam 6">    
    

### MFA

- Multi Factor Authentication - MFA 설정
    - OTP나 2단계 인증처럼 MFA는 비밀번호와 함께 추가적인 보안 장치를 사용해서 계정을 보호하는 수단
    - 계정 사용자가 소유한 물리적인 보안 장치가 필요하므로 계정이 침해당할 가능성이 현저히 낮아진다.
    - 가상 MFA 장치
        - Google Authenticatior
        - MS Authenticatior
        - Authy
    - Universal 2nd Factor - U2F Security Key
        - 보안 키를 담고 있는 USB 저장소 형태의 보안 장치
    - Hardward Key Fob MFA Deivce
        - OTP 장치
    - Hardward Key Fob MFA Deivce for AWS GovCloud

### Set MFA

1. 오른쪽 상단 내 계정을 클릭 후 보안 자격 증명 들어간다.
2. MFA 디바이스 할당 - 사용 중인 AWS 계정이어서 이미 설정되어 있다.
    
    <img src="/images/AWS_MFA_1.png" width="75%" height="75%" title="aws mfa 1" alt="aws mfa 1">    
    
3. 사용할 MFA 장치 선택 - Authenticator app
    
    <img src="/images/AWS_MFA_2.png" width="75%" height="75%" title="aws mfa 2" alt="aws mfa 2">    
    
4. 자격 증명 앱을 사용할 경우 QR를 촬영하여 장치에 계정을 추가하고 연속된 MFA 코드 2개를 입력하여 동기화한다. 
    
    <img src="/images/AWS_MFA_3.png" width="75%" height="75%" title="aws mfa 3" alt="aws mfa 3">    
    

## IAM Role

사용자의 계정에서 서비스 실행이 필요할 때 해당 서비스도 어떠한 권한이 필요하다. 이 경우에 IAM Role을 사용해서 서비스에 권한을 부여할 수 있다.

- 공통적으로 많이 사용하는 역할
    - EC2 Instance Roles
    - Lambda Function Roles
    - Roles for CloudFormation

### Create Roles

1. 엔터티 유형과 사용 사례를 선택
    
    <img src="/images/AWS_IAM_7.png" width="75%" height="75%" title="aws iam 7" alt="aws iam 7">    
    
2. 역할에 부여할 정책과 권한을 선택
3. 역할 이름 설정 후 생성

## IAM Security Tools

- IAM Credential Report (자격 증명 보고서) - 계정 수준
    - 계정에 있는 사용자와 사용자의 자격 증명 상태 포함
- IAM Access Advisor (액세스 관리자) - 사용자 수준
    - 사용자에게 부여된 서비스의 권한과 해당 서비스에 마지막으로 액세스한 시간을 확인
    - 최소 권한 원칙을 따랐을 때 도움이 되는 정보
    - 사용자가 어떠한 도구를 사용할 때 사용되지 않는 권한을 확인하여 권한을 줄일 수 있다.

### Credential Report

IAM 콘솔에서 자격 증명 보고서를 클릭하면 보고서를 다운로드 받을 수 있다.
    
    [자격 증명 보고서](./status_reports_Fri%20May%2031%202024%2017_04_47%20GMT+0900%20(한국%20표준시).csv)
    

### Access Advisor

1. 사용자를 선택한 후 액세스 관리자 탭 확인
    
    <img src="/images/AWS_IAM_8.png" width="75%" height="75%" title="aws iam 8" alt="aws iam 8">    
    
2. 가장 최근 사용한 서비스부터 확인할 수 있으며 사용된 권한과 액세스 날짜를 확인할 수 있다.

## IAM Guidelines & Best Practices

- 루트 계정은 AWS 계정을 설정할 때를 제외하고 사용하지 않는다.
- 한 개의 AWS 사용자 = 한 명의 실제 사용자
- 그룹에 권한을 부여한 후 사용자를 그룹에 추가
- 강력한 암호 정책 생성
- MFA 활성화
- AWS 서비스에 권한을 주기 위해서 역할을 사용한다.
- AWS CLI 혹은 SDK를 사용할 때 반드시 액세스 키를 만들어 사용한다.
- 계정 권한을 감사할 때는 IAM 자격 증명 보고서와 IAM 액세스 관리자를 사용한다.
- IAM 사용자와 액세스 키는 절대로 공유하지 않는다.