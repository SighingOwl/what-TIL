# Jenkins 작업
## Jenkins 작업의 종류
### Freestyle Jobs

- GUI를 사용하는 작업
- Jenkins을 학습할 때나 기능들을 살펴볼 때 사용
- 실시간 작업에는 권장하지 않는다.

### Pipeline As a Code

- 파이프라인은 Groovy 언어로 생성된다.
- Jenkins를 실제 업무에 사용할 때 권장된다.

## Jenkins에 도구 설치
1. Jenkins 관리의 “Tools”를 선택
    1. Tools에서는 설치한 플러그인을 관리할 수 있다.
2. 가장 먼저 JDK 설정을 한다.
    1. Add JDK
    2. JDK의 이름을 설정 - 아무이름이나 붙여도 되지만 가급적 작업에서 사용하는 것과 같은 이름을 사용
    3. JAVA_HOME 경로 설정 or Install automatically
        1. JAVA_HOME 경로는 java가 설치된 경로를 의미
        2. “/usr/lib/jvm/java-1.11.0-openjdk-amd64”
3. 빌드 도구로 maven을 사용하기 위해 maven을 추가한다.
    1. Add Maven
    2. Maven 이름 설정
    3. MAVEN_HOME 경로 설정 or Install automatically
        1. “/usr/share/maven/bin/mvn”
4. 저장

## 작업 구성 및 실행    
1. Jenkins Dashboard에서 “Create Jobs”을 선택
2. 첫번째 단계에서 작업의 이름과 작업 형식을 선택
    
    <img src="/images/jenkins_job_1.png" width="75%" height="75%" title="jenkins job 1" alt="jenkins job 1">    
    
    학습을 목적으로 하므로 Freestyle을 선택
    
3. 작업 구성을 진행한다.
    
    <img src="/images/jenkins_job_2.png" width="75%" height="75%" title="jenkins job 2" alt="jenkins job 2">    
    
    1. General
        1. 작업 설명
        2. JDK 사용 여부, 버전 선택
    2. 소스 코드 관리 - 하지 않거나 git을 선택
    3. 빌드 유발 설정
        1. 빌드를 원격으로 유발 - 예 : 스크립트 사용
        2. Build after other projects are built
        3. Build periodically
        4. GitHub hook trigger for GITScm polling
        5. Poll SCM
    4. 빌드 환경
        1. Delete workspace before build starts
        2. Use secret text(s) or file(s)
        3. Provide Configuration files
        4. Add timestamps to the Console Output
        5. Inspect build log for published build scans
        6. Provide Node & npm bin/folder to PATH
        7. Terminate a build if it’s stuck
    5. Build Steps
        1. 코드를 빌드하거나 실행하는 작업을 진행할 수 있다.
        2. 아래와 같이 command를 입력하거나 선택한 작업에 맞는 스크립트를 작성하여 빌드할 수 있다.
            
            <img src="/images/jenkins_job_3.png" width="75%" height="75%" title="jenkins job 3" alt="jenkins job 3">    
            
        
        c. Build Steps는 여러 작업을 추가하여 실행할 수 있다.
        
4. 작업 구성이 완료되면 프로젝트 대시보드 화면이 표시된다.
    
    <img src="/images/jenkins_job_4.png" width="75%" height="75%" title="jenkins job 4" alt="jenkins job 4">    
    
5. 지금 빌드를 선택하면 프로젝트가 빌드된다.
    
    <img src="/images/jenkins_job_5.png" width="75%" height="75%" title="jenkins job 5" alt="jenkins job 5">    
    
    → 빌드가 성공하면 좌측에 Build ID와 함께 체크표시가 출력된다.
    
    <img src="/images/jenkins_job_6.png" width="75%" height="75%" title="jenkins job 6" alt="jenkins job 6">    
    
    → 빌드가 실패하면 Build ID와 함께 X표시가 출력된다.
    
6. Build ID를 클릭하면 빌드 상태와 변경 사항, 콘솔 출력을 확인할 수 있다.
    
    <img src="/images/jenkins_job_7.png" width="75%" height="75%" title="jenkins job 7" alt="jenkins job 7">    
    
    Console Output - 마지막 줄에 SUCCESS 메시지가 출력된다.
    
    <img src="/images/jenkins_job_8.png" width="75%" height="75%" title="jenkins job 8" alt="jenkins job 8">    
    
    Colsole Output - 마지막 줄에 FAILURE 메시지가 출력된다.
    
7. Workspace
    1. Workspace는 작업의 데이터를 가지고 있는 공간이다. 빌드 과정에서 만들어진다.
        
        <img src="/images/jenkins_job_9.png" width="75%" height="75%" title="jenkins job 9" alt="jenkins job 9">    
        
8. Project 삭제
    1. Project 삭제를 클릭하면 확인메시지를 확인하고 삭제 가능.

## 소스 코드 빌드 작업
소스코드를 빌드 작업을 생성한다. 작업 생성 과정은 동일하지만 작업 구성에서 소스 코드를 불어오는 과정이 추가된다.

1. 소스 코드 관리에서 Git을 선택
    
    <img src="/images/Jenkins_build_job_1.png" width="75%" height="75%" title="jenkins build job 1" alt="jenkins build job 1">       
    
    소스코드 정보 입력란
    
    1. 빌드할 소스코드가 있는 레포지토리의 URL을 입력한다.
    2. 자격증명설정
        1. 퍼블릭 레포지토리인 경우 자격증명이 필요하지 않다.
        2. 프라이빗 레포지토리인 경우 자격증명이 필요하며 자격증명을 등록한 후 사용할 수 있다.
    3. 브랜치 설정 
        1. 빌드할 소스코드가 있는 브랜치 경로를 입력.
    4. Build Steps에서 소스코드를 빌드할 플러그인을 선택 - Maven
        1. 플러그인 버전 선택
        2. 플러그인의 세부 명령을 입력 - Maven이 빌드할 때 사용한 명령을 사용, “install” or “clean install”
    5. Maven인 경우 고급선택에서 pom.xml 경로를 따로 지정할 수 있다.
        1. pom.xml이 레포지토리의 최상위 디렉토리에 있으면 비워두면 된다. 
2. 빌드 구성이 완료된 이후 빌드를 시작한다.
    1. t2.micro 인스턴스의 경우 RAM 용량부족으로 인스턴스가 동작하지 않을 수 있어 이 경우 t2.small 이상으로 유형을 변경한다.
    
    <img src="/images/Jenkins_build_job_2.png" width="75%" height="75%" title="jenkins build job 2" alt="jenkins build job 2">       
    
    → 빌드가 완료되면 로컬에서 동작한 것과 동일한 메시지가 콘솔에 출력된다.
    
3. Workspace로 가면 빌드 결과물들을 확인할 수 있다.
    1. 로컬에서 진행한 것과 동일하게 빌드 결과물을 패키징한 파일을 찾아 다운로드한다.
- 구성의 가장 마지막 항목인 빌드 후 조치를 설정하면 빌드 이후에 진행할 작업을 자동으로 진행한다.
    - Aggregate downstream test results
    - Archive the artifacts
    - Build other projects
    - Publish JUnit test result report
    - Record fingerprints of files to track usage
    - Git Publisher
    - E-mail Notification
    - Editable Email Notification
    - Set GitHub commit status (universal)
    - Set build status on GitHub commit [deprecated]
    - Delete workspace when build is done
    
    <img src="/images/Jenkins_build_job_3.png" width="75%" height="75%" title="jenkins build job 3" alt="jenkins build job 3">       
    
    빌드 이후에 결과물을 기록하도록 조치 - “.war”형식의 파일을 찾아서 기록. 이 작업을 하면 workspace를 초기화 후에도 기록으로 남긴 것은 삭제되지 않는다.
    

### 기존의 빌드 구성을 사용해서 새로운 작업 생성

- 새로운 작업 생성을 할 때 가장 아래 항목에서 복사할 작업을 선택한 후 생성하면 선택한 빌드 구성을 그대로 가져온다.
    
    <img src="/images/Jenkins_build_job_4.png" width="75%" height="75%" title="jenkins build job 4" alt="jenkins build job 4">       

## 버전 관리
작업에 빌드를 할 때마다 workspace의 데이터들은 모두 새로운 데이터로 대체된다. 따라서 이전 빌드 결과물을 다시 가져올 수 없다. 만일 이전 결과물을 유지하고 싶다면 결과물마다 버전을 지정해주어야 한다. Jenkins에서는 2가지 방법이 있다.

### 작업 구성의 Build Steps 사용

- 패키징 한 결과물의 이름에 빌드 ID를 추가한 파일을 별도의 디렉토리에 복사하도록 shell command 지정
    
    ```bash
    mkdir -p versions
    cp target/vprofile-v2.war versions/vprofile-V$BUILD_ID.war
    ```
    
    → $BUILD_ID는 Jenkins의 환경 변수를 사용하는 것이다. 자세한 내용은 아래 링크를 참조
    
    [Using a Jenkinsfile](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#using-environment-variables)
    
- 빌드를 할 때마다 versions 폴더에 빌드 결과물이 하나씩 생성되는 것을 확인할 수 있다.
    
    <img src="/images/Jenkins_versioning_1.png" width="75%" height="75%" title="jenkins versioning 1" alt="jenkins versioning 1">    
    

### 작업 구성의 General에서 매개변수 사용

Build Steps를 사용하는 것은 동일하지만 버전으로 사용하는 변수가 Jenkins 환경변수가 아닌 빌드할 때의 매개변수를 사용하는 방법이다.

1. 작업 구성의 General 탭에서 “이 빌드는 매개변수가 있습니다”를 활성화 한 후 매개변수 설정을 한다.
    
    <img src="/images/Jenkins_versioning_2.png" width="75%" height="75%" title="jenkins versioning 2" alt="jenkins versioning 2">    
    
2. 작업 구성을 저장하면 “지금 빌드”가 “파라미터와 함께 빌드”로 변경되고 빌드를 할 때 매개변수를 입력해야한다.
    
    <img src="/images/Jenkins_versioning_3.png" width="75%" height="75%" title="jenkins versioning 3" alt="jenkins versioning 3">    
    
    매개변수 입력 페이지
    
    <img src="/images/Jenkins_versioning_4.png" width="75%" height="75%" title="jenkins versioning 4" alt="jenkins versioning 4">    
    
    빨간색 박스 안에 입력한 매개변수를 버전으로 이름이 지정된 패키지를 확인할 수 있다.
    

이 방법은 빌드마다 매개변수를 입력해주어야 하므로 여러 작업을 동시에 실행하거나 자동화하기에는 적합하지 않다. 자동화할 수 있으나 사용자의 매개변수를 사용하는 것은 휴먼 에러와 같은 여러 문제가 발생할 수 있다.

### 플러그인 사용

Jenkins에는 버전을 사용할 수 있도록 도와주는 플러그인이 있다.

#### Add Plugins

1. Jenkins 관리에서 “Manage Plugins”를 선택
2. “Avilable Plugins”에서 설치가능한 플러그인을 찾을 수 있다.
    1. 버전 지정을 timestamp를 활용하여 진행
    2. timestamp를 검색하여 원하는 플러그인을 설치 - Zentimestamp
    
    <img src="/images/Jenkins_versioning_6.png" width="75%" height="75%" title="jenkins versioning 6" alt="jenkins versioning 6">    
    
    플러그인 검색 페이지
    
    c. 플러그인 설치를 시작하면 플러그인에 필요한 종속성과 함께 설치를 진행한다.
    
    <img src="/images/Jenkins_versioning_7.png" width="75%" height="75%" title="jenkins versioning 7" alt="jenkins versioning 7">    
    
    플러그인 설치 페이지
    
1. 플러그인 설치 완료 후 작업 구성의 General에서 “Change date pattern for the BUILD_TIMESTAMP (build timestamp) variable**”**를 선택하고 날짜 패턴을 지정한다.
2. Build Steps에서 버전 매개변수를 “BUILD_TIMESTAMP”로 변경한다.
3. 빌드를 진행하면 아래와 같이 time stamp와 함께 이름이 지정된 패키지가 생성된다.
    
     <img src="/images/Jenkins_versioning_5.png" width="75%" height="75%" title="jenkins versioning 5" alt="jenkins versioning 5">    