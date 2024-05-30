# Jenkins Master & Slave
지금까지는 Jenkins Master에서 모든 작업을 수행했다. 하지만 다른 머신에서 작업을 수행해야하는 유스케이스가 생길 수 있다. 이 다른 머신이 Slave이다.

## Use Cases

- Load Distribution
    - Jenkins을 조직에서 사용하고 자동으로 유발되는 많은 작업들이 존재할 때 master에서 이를 모두 수행할 수 없게 된다. 만일 Jenkins에 노드를 추가하면 Jenkins는 master 혹은 slave가 작업을 수행할 수 있도록 분배한다.
- Cross Platform Builds
    - 서로 다른 운영체제를 위한 패키지를 빌드하기 위해 사용할 수 있다.
    - Linux 머신에서 windows 패키지를 빌드하거나 windows 머신에서 mac os 패키지를 빌드할 수 있도록 할 수 있다.
- Software Testing
    - Jenkins에서 CD를 수행하려면 소프트웨어 테스트 단계를 포함해야한다.
    - 테스트를 수행할 때 개발 환경의 OS와 사용자에게 제공되는 OS가 다를 수 있고 그래픽 환경에서 테스트 케이스를 실행해야 할 수도 있다.
    - 이 경우에 slave에서 테스트 케이스를 실행하도록 할 수 있다.

## 준비 사항

- Any OS
- Network access from Master
    - Firewall rules (include third party firewall)
    - Security Group
    - SQL
- Java, JRE, JDK
- User
    - Jenkins가 접속할 수 있는 사용자
- Directory with User ownership
- Tools as required by the Jenkins job
    - Maven
    - Ant
    - Git
    - etc

### Setup Jenkins

1. Jenkins 관리의 “Nodes and Clouds”에서 노드 추가
2. 노드가 실행할 수 있는 작업 개수 설정
3. “Remote root directory”에는 다른 머신에 생성한 Jenkins 사용자 소유의 디렉토리 주소를 입력”
4. “Usage”선택
    1. Use this node as much as possible - load distribution에 사용
    2. Only build jobs with label expressions matching this node - 다른 OS 환경에서 빌드에 사용, 사용할 노드를 반드시 지정
5. “Launch method” - 시작 에이전트를 실행할 방법 선택 → ssh
    1. Host
        1. 노드의 IP 주소 - 같은 VPC의 EC2 인스턴스인 경우 프라이빗 IP를 사용
        2. Slave node의 보안그룹 인바운드 규칙에 Jeknins에서 들어오는 SSH 프로토콜 트래픽 허용 추가
    2. Credential
        1. Slave node를 만들 때 Jenkins 사용자는 패스워드를 사용해서 로그인하도록 설정을 했으므로 사용자의 이름과 패스워드를 추가한 자격 증명 생성
        2. 만일 ssh 키로 로그인을 하도록 설정했으면 키를 사용한 자격 증명을 생성하면 된다.
    3. Host Key Verification Strategy - SSH로 처음 로그인할 때 yes/no로 답하는 것, Jenkins는 사람이 아니라서 답할 수 없으므로 “Non verifying Verification Strategy”를 선택
6. 설정이 모두 올바르게 완료되면 노드의 로그에 연결 성공 메시지가 출력된다.

### 실행 결과

<img src="/images/Jenkins_master_slave_1.png" width="75%" height="75%" title="jenkins master slave 1" alt="jenkins master slave 1">     

→ 자격 증명에 작성되어 있는 ID와 slave 노드의 Jenkins root 디렉토리에서 Jenkinsd 이 실행된 것을 확인할 수 있다.

<img src="/images/Jenkins_master_slave_2.png" width="75%" height="75%" title="jenkins master slave 2" alt="jenkins master slave 2">     

→ Slave node에 접속한 후 Jenkins root 디렉토리를 확인하면 에이전트(remoting)과 workspace를 확인할 수 있다.

### 작업 실행 노드 지정

- 작업 구성을 할 때 이 작업을 실행할 노드를 레이블로 지정할 수 있다.
- 작업 구성의 General에서 “Restrict where this project can be run”을 활성화 한 후 사용할 노드의 레이블을 입력.
- Jenkins에 추가한 노드 설정의 Usage 설정을 “Only build jobs with label expressions matching this node”로 변경