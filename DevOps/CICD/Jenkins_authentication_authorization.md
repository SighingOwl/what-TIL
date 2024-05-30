# Jenkins 인증과 권한 부여
Jenkins에는 사용자마다 로그인 허용과 권한을 부여할 수 있다.

## Securing Jenkins

- User Login
    - Jenkins own database
        - Sign UP
    - LDAP Integration
- Permissions on Jenkins
    - Admin
    - Read
    - Jobs
    - Credential
    - Plugins
    - etc
- Permissions on Jobs
    - View
    - Build
    - Delete
    - Configure
    - etc

## Setup Authentication

1. Jenkins 관리의 Security에서 진행
2. Authentication에서 “Security Realm”를 변경
    1. Delegate to servlet container
    2. Jenkins’ own user database
        1. 사용자 가입 허용 - 사용자가 계정을 생성할 수 있다.
    3. LDAP
        1. Jenkins을 조직에서 사용하고 있을 때 유용하다.
        2. 이미 다른 서버에서 인증을 사용하고 있다면 서버를 연동할 수 있다.
    4. Unix user/group database
        1. OS의 사용자와 그룹을 사용해서 로그인
        2. 권장하지 않는다.

## Setup Authorization

1. Jenkins 관리의 Security에서 진행
2. Authorization 설정을 변경
    1. Anyone can do anything
    2. Legacy mode - 익명 액세스
    3. Logged-in uses can do anything - 기본값
        1. Allow anonymous read access
    4. Matrix-based securtiy
        
        <img src="/images/jenkins_authentication_authorization.png" width="75%" height="75%" title="jenkins authentication authorization" alt="jenkins authentication authorization">
        
        1. Jenkins 수준에서 할 수 있는 권한 부여 방식
        2. 사용자나 그룹을 추가하여 권한을 부여할 수 있다.
    5. Project-based Matrix Authorization Strategy