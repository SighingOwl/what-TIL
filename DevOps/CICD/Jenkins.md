# CI - Continuous Integration

## 기존 프로세스와 CI 프로세스 비교
### Continuous Process
<img src="/images/continuous Process.png" width="75%" height="75%" title="continuous process" alt="continuous process">


Continuous Process은 위 다이어그램처럼 코드를 작성, 빌드, 테스트, 푸시를 반복하는 것을 의미한다. 이렇게 만들어진 결과물은 VCS에 지속적으로 병합된다. 이러한 점이 문제가 된다.

- 여러 개발자들이 서로 다른 코드를 개발하고 VCS로 병합할 때 다양한 충돌과 버그가 발생할 수 있다. 하지만 이러한 사실이 통합을 하기 전까지 알 수 없을 수 있다.
- 이러한 충돌과 버그를 해결하기 위해 개발자들은 많은 재작업이 필요하고 수정에 많은 시간을 투자해야한다.

수 많은 커밋과 병합 이후에 통합을 진행할 때는 수많은 충돌과 버그를 동반하므로 이를 해결하기 위해서 커밋할 때마다 코드를 빌드하고 테스트하는 것이 자동화될 필요가 있다. 그럼 코드가 수정될 때마다 오류나 충돌을 확인할 수 있으므로 개발자의 대응이 더 빨라진다. 이러한 프로세스가 Continuous Integration이다.

### CI Process
<img src="/images/CI_process.png" width="75%" height="75%" title="ci process" alt="ci process">

## Jenkins
Jenkins는 CI 도구 중 가장 많이 사용하는 도구이다.
<img src="/images/jenkins_ci.png" width="75%" height="75%" title="jenkins ci process" alt="jenkins ci process">
> Jenkins의 간략화한 flow   

### 확장성이 높은 오픈소스 도구
Jenkins는 오픈소스 도구이며 확장성이 좋아 플러그인으로 많은 기능을 제공하고 있다. 그래서 CI Server로 활용하거나 CD 도구 혹은 스크립트 실행, 소프트웨어 테스트 케이스 실행, 클라우드 자동화 실행, devops 도구나 개발자 도구, 테스트 도구 통합 등으로 사용할 수 있다.     
- 플러그인
    - VCS 플러그인
    - 빌드 플러그인
        - Java
        - .net
        - Nodejs
    - Cloud 플러그인
    - 빌드 플러그인

### Jenkins 설치
Java, JRE, JDK가 설치되어있는 어떤 OS에서도 설치하여 사용할 수 있다.    
[Installing Jenkins](https://www.jenkins.io/doc/book/installing/)


1. [Jenkins.io](http://Jenkins.io)의 설치 가이드에 따라 원하는 OS나 플랫폼에 적합하게 설치할 수 있다. 단 Jenkins를 설치하기 건에 JDK를 먼저 설치해야한다.
2. Ubuntu로 동작하는 EC2 인스턴스에 Jenkins를 설치하여 EC2 인스턴스의 퍼블릭 IP:8080으로 접속하면 Jenkins 화면을 확인할 수 있다.
    
    <img src="/images/Ubuntu_Install_1.png" width="75%" height="75%" title="jenkins installation 1" alt="jenkins installation 1">    
    
    Jenkins에 첫 접속했을 때 표시화는 화면
    
3. 화면에 출력된 경로의 Jenkins 패스워드를 페이지에 입력하면 로그인할 수 있다.
4. Jenkins에 접속하면 사용할 플러그인 설치에 관한 화면이 표시된다.
    
    <img src="/images/Ubuntu_Install_2.png" width="75%" height="75%" title="jenkins installation 2" alt="jenkins installation 2">    
    
    권장 설치 혹은 사용자 정의 설치를 선택
    
    <img src="/images/Ubuntu_Install_3.png" width="75%" height="75%%" title="jenkins installation 3" alt="jenkins installation 3">    
    
    사용자 정의 설치에서 사용할 플러그인 선택
    
    <img src="/images/Ubuntu_Install_4.png" width="75%" height="75%" title="jenkins installation 4" alt="jenkins installation 4">    
    
    플러그인 설치 중인 화면
    
5. 플러그인 설치 완료 후 관리자 계정 생성
    
    <img src="/images/Ubuntu_Install_5.png" width="75%" height="75%" title="jenkins installation 5" alt="jenkins installation 5">    
    
    관리자 계정 생성 화면
    
6. 다음 화면에서 Jenkins URL을 확인할 수 있다. 로컬에 설치했을 경우에는 정적 IP를 사용해서 IP가 변하지 않지만 EC2 인스턴스는 동적 IP를 사용하므로 기억해둘 필요는 없다.
7. 설치가 완료되면 대시보드 화면을 확인할 수 있다.
    
    <img src="/images/Ubuntu_Install_6.png" width="75%" height="75%" title="jenkins installation 6" alt="jenkins installation 6">    