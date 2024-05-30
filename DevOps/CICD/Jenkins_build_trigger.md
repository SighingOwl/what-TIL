# Jenkins 빌드 트리거

지금까지 만든 Jenkins CICD Pipeline은 코드 빌드부터 애플리케이션 호스팅까지 모두 자동화되었지만 빌드 시작은 수동으로 해야하는 문제가 있다. 레포지토리에 코드가 커밋될 때마다 CI를 진행하기 위해서는 코드의 변경 사항이 발생할 때마다 자동으로 빌드를 수행할 수 있도록 트리거를 생성해야한다.

## 종류

- Git Webhook
    - 가장 유명한 빌드 트리거
    - 레포지토리에 커밋을 할 때마다 github 레포지토리가 jenkins 작업을 수행하도록 하는 트리거 역할을 한다.
- Poll SCM
    - Git Webhook의 작동 방식과 반대 방향으로 작동한다.
    - Jenkins가 github 레포지토리를 사용자가 설정한 일정 시간마다 체크한다. 그 시간 사이에 새로운 커밋이 발생하면 빌드 작업을 수행한다.
- Scheduled jobs
    - 간단한 형태의 트리거
    - Cronjob처럼 특정 날짜나 특정 시간 또는 특정 간격으로 jenkins 작업이 실행되도록 예약한다.
- Remote triggers
    - 다소 복잡한 트리거이며 대부분의 사람들이 굳이 사용하지는 않는다.
    - Devops 엔지니어나 아키텍트에게는 매우 유용한 트리거다.
    - 스크립트나 ansible playbook과 같이 어디에서든 jenkins 작업을 실행시킬 수 있다.
    - 토큰이나 비밀 그리고 다양한 요소들을 필요로 한다.
    - API call를 활용하여 jenkins 작업을 사용한다.
- Build after other projects are built
    - 간단한 형태의 트리거
    - 이전 작업이 완료되면 트리거에 의해 작업이 시작된다.

## Git Webhook

### Steps

1. Github에 git 레포지토리 생성
2. Git SSH 자격 증명 생성
3. Git 레포지토리에 Jenkinsfile을 생성하고 커밋
4. Git 레포지토리에서 Jenkinsfile에 액세스하기 위한 Jenkins 작업 생성
5. 트리거 테스트

### Jenkins Setup

1. Jenkins 관리의 “Security”에서 “Git Host Key Verification Configuration” 설정을 “Accept first connection”으로 변경
2. 새로운 파이프라인 생성
    1. Pipeline script를 git에서 가져오도록 설정
    2. Github 레포지토리 SSH 주소를 URL로 입력
    3. 자격 증명 생성
        1. SSH Username with private key 설정
        2. Username은 GitHub 사용자 이름을 입력
        3. Private Key는 Enter directly 후 GitHub SSH 키의 private key를 입력
    4. 생성한 자격 증명 선택 - 자격 증명 선택 후 연결 실패 메시지가 출력되면 안 된다.
        
    <img src="/images/Jenkins_build_trigger_1.png" width="75%" height="75%" title="jenkins build trigger 1" alt="jenkins build trigger 1">    
        
    5. 브랜치 설정 - Jenkinsfile이 있는 브랜치를 선택해야한다.
3. 작업 구성 후 빌드 테스트
    
    <img src="/images/Jenkins_build_trigger_2.png" width="75%" height="75%" title="jenkins build trigger 2" alt="jenkins build trigger 2">    
    
4. [Github Setup](https://www.notion.so/CI-Jenkins-c4b20b85efd14ed6aa0d421c5966192c?pvs=21)을 마친 후 작업 구성에서 Build Triggers에서 “GitHub hook trigger for GITScm polling”을 활성화

### Github Setup

1. 트리거 역할을 할 레포지토리의 세팅의 Webhooks 탭에서 webhook을 추가
    
    <img src="/images/Jenkins_build_trigger_3.png" width="75%" height="75%" title="jenkins build trigger 3" alt="jenkins build trigger 3">    
    
    1. Payload URL에 Jenkins이 실행 중인 “URL + /github-webhook/”을 입력 - EC2 인스턴스가 동적 IP인 경우에는 부팅마다 바꿔줘야 함.
    2. Content type - application/json
    3. 트리거를 유발할 이벤트 설정
        1. Just the push event.
        2. Send me everything
        3. Let me select individual events.
    
    <img src="/images/Jenkins_build_trigger_4.png" width="75%" height="75%" title="jenkins build trigger 4" alt="jenkins build trigger 4">    
    
    → 생성한 Webhook 왼쪽에 녹색 체크 표시가 생기면 성공한 것이다.
    
    → 실패한 경우 Jenkins URL과 Jenkins EC2 인스턴스 보안그룹이 8080번 포트의 모든 트래픽을 허용 여부를 확인해야한다.
    

### 실행 결과

<img src="/images/Jenkins_build_trigger_5.png" width="75%" height="75%" title="jenkins build trigger 5" alt="jenkins build trigger 5">    

→ GitHub 레포지토리에 새로운 파일을 commit 후 push 하면 Jenkins에서 자동으로 빌드하는 것을 확인

→ Change를 확인하면 1 commit이 있음을 확인할 수 있다.

## Poll SCM

Poll SCM은 Cronjob 형식으로 시간 정책을 설정하여 특정 날짜, 시간 혹은 일정 간격으로 GitHub 레포지토리를 확인해서 commit 된 것이 있으면 빌드하는 방법이다.

Jenkins 작업 구성에서 빌드 트리거를 Poll SCM으로 설정

```bash
* * * * *
```

→ Cronjob 형식으로 매분 확인하도록 설정

<img src="/images/Jenkins_build_trigger_6.png" width="75%" height="75%" title="jenkins build trigger 6" alt="jenkins build trigger 6">    

→ Git Polling Log에서 매분 확인할 때 변경 사항이 발견되지 않으면 No Change 메시지가 가장 마지막 줄에 출력된다.

<img src="/images/Jenkins_build_trigger_7.png" width="75%" height="75%" title="jenkins build trigger 7" alt="jenkins build trigger 7">    

→ GitHub 레포지토리에 변경 사항이 발견되어 Changes found 메시지가 출력되고 빌드를 시작한다.

## Schedualed jobs

Poll SCM과 유사하지만 특정 시간에만 작동하도록 설정

Jenkins 작업 구성의 빌드 트리거에서 “Build periodically”를 선택

```bash
45 16 * * 1-5
```

→ Cronjob 형식으로 특정 시각을 설정

## Remote triggers

어디에서든 아무때나 Jenkins 서버에 접속할 수 있으면 원격으로 빌드를 수행할 수 있도록 하는 트리거이다.

1. Jenkins 작업 구성의 Build Triggers에서 “빌드를 원격으로 유발”을 선택  후 자격 증명 토큰 입력
2. 계정 설정에서 API Token을 생성한다.
3. 아래 커맨드로 CRUMB을 다운한다.
    
    ```bash
    wget -q --auth-no-challenge --user admin --password JenkinsPW --output-document - 'http://43.201.248.106:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)'
    Jenkins-Crumb:f5cc78be0c8e2a28deb37cda314a9ffbfc6030159100e0348836f6e541c0c870%
    ```
    
4. 아래 형식의 커멘드에 API Token, CRUMB을 기입해서 shell에서 실행하면 원격으로 빌드를 실행한다.
    
    ```bash
    curl -I -X POST http://username:APItoken @Jenkins_IP:8080/job/JOB_NAME/build?token=TOKENNAME-H "Jenkins-Crumb:CRUMB"
    curl -I -X POST http://admin:1108275426ce7af42daa01f6de8f6dd673@43.201.248.106:8080//job/build/build?token=mybuildtoken -H "Jenkins-Crumb:f5cc78be0c8e2a28deb37cda314a9ffbfc6030159100e0348836f6e541c0c870%"
    ```
    

<img src="/images/Jenkins_build_trigger_8.png" width="75%" height="75%" title="jenkins build trigger 8" alt="jenkins build trigger 8">    

→ Shell에서 빌드를  실행하는 것을 확인할 수 있다.

## Build after other projects are built

이전 빌드 작업이 완료되면 자동으로 빌드를 시작한다.

Jenkins 작업 구성의 빌드 트리거에서 “Build after other projects are built”활성화 후 어떤 작업 이후 실행할 것인지 지정

<img src="/images/Jenkins_build_trigger_9.png" width="75%" height="75%" title="jenkins build trigger 9" alt="jenkins build trigger 9">    

→ 작업 이후 빌드를 진행한 것을 확인할 수 있다.