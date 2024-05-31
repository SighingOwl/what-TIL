# AWS CLI

AWS 서비스에 접근하는 방법은 콘솔, CLI, SDK를 사용하는 세가지가 있고 각각의 방법은 보호되는 방식이 다르다.

- AWS Management Console : 패스워드 + MFA
- AWS CLI : Access Key
    - Command-line shell에서 AWS 서비스를 커맨드로 상호작용 할 수 있도록 하는 도구이다.
- AWS SDK : Access Key
    - 개발자가 개발에 사용하는 개발자 키트이며 애프리케이션 소스 코드에 포함된다.
    - AWS 서비스를 API로 상호작용할 수 있도록 제공되는 라이브러리
    - Javascript, Python, PHP, .NET, Ruby, Java, GO, Node.js, C++ 언어 지원
    - Mobile SDKs - Android, iOS
    - IoT Device SDK - Embeded C, Arduino
- 사용자의 권한이 바뀌면 접근방식에 관계없이 바뀐 권한이 모두 동일하게 적용된다.

### Install AWS CLI

- Windows
    1. Installer를 다운받아 설치
    2. cmd나 powershell에서 aws —version 커맨드로 설치 확인
- Mac OS
    1. 아래 커맨드를 터미널에서 실행
        
        ```bash
        curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
        sudo installer -pkg AWSCLIV2.pkg -target /
        ```
        
    2. Windows와 동일하게 터미널에서 aws —version 커맨드로 설치 확인
- Linux
    1. 아래 커맨드를 터미널에 실행
        
        ```bash
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install
        ```
        
    2. aws —version 커맨드로 설치 확인

### Access Key for AWS CLI

AWS CLI를 사용하기 위해서는 사용자의 액세스 키를 발급한 후 AWS CLI에 액세스 키와 비밀 액세스 키를 설정해야한다.

1. 사용자 액세스 키 발급
    1. 사용자의 보안 자격 증명에서 액세스 키를 발급
    2. 액세스 키 사용 사례 선택 - CLI
        
        <img src="/images/AWS_CLI_1.png" width="50%" height="50%" title="aws cli 1" alt="aws cli 1">    
        
2. AWS CLI에 액세스 키와 비밀 액세스 키 입력
    
    ```bash
    aws configure
    AWS Access Key ID [None]:
    AWS Secret Access Key [None]:
    Default region name [None]:
    Default output formet [None]:
    ```
    
    → AWS Access Key ID : 발급 받은 액세스 키의 ID를 입력
    
    → AWS Secret Access Key ID : 다운로드한 csv파일에서 Secret Access Key를 확인하여 입력
    
    → Default region name : AWS 서비스를 사용할 리전을 입력
    
    → Default output format : AWS 서비스 실행 결과 출력 형식 설정 - 기본값 JSON
    

## AWS CloudShell

- AWS 서비스를 로컬에서 터미널을 사용해 접근하는 방법의 대안으로 사용할 수 있도록 도와주는 도구이다.
- CloudShell은 사용할 수 있는 리전이 한정되어있으므로 확인 후 사용 - 서울 리전 사용 가능
    
     <img src="/images/AWS_CLI_2.png" width="50%" height="50%" title="aws cli 2" alt="aws cli 2">    
    
- CloudShell에는 별도의 저장소가 있어 CloudShell에 생성한 파일은 CloudShell을 재시작해도 남아있다.
- Shell의 설정을 변경할 수 있고 파일을 업로드하거나 다운로드 할 수 있으며 필요할 경우 여러 shell을 동시에 사용할 수 있다.