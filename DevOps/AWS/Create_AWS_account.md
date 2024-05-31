# AWS 계정 생성
AWS를 사용하기 위해서는 AWS 계정이 필요하다. 이 계정을 통해 AWS 서비스에 액세스하고 AWS에서 실행되는 컴퓨팅 자원을 관리할 수 있다. AWS에 계정을 생성하면 12개월 동안 프리티어가 적용되며 일부 서비스를 제한된 성능으로 무료로 사용이 가능하다. 프리티어 계정도 유료 서비스를 사용할 수 있지만 일반 계정과 동일하게 요금이 발생한다.   
[AWS 프리티어 가능한 서비스 확인](https://aws.amazon.com/ko/free/?gclid=CjwKCAjwx-CyBhAqEiwAeOcTdemfzUmfhgiBVpJaZLayvPAFYeurfdudoOCl1slgWG2Ab1C0c8aWdxoCCWAQAvD_BwE&trk=2e777eb1-7c1a-4acc-ae47-724e1cd50096&sc_channel=ps&ef_id=CjwKCAjwx-CyBhAqEiwAeOcTdemfzUmfhgiBVpJaZLayvPAFYeurfdudoOCl1slgWG2Ab1C0c8aWdxoCCWAQAvD_BwE:G:s&s_kwcid=AL!4422!3!444218215904!e!!g!!aws!10287751092!99328587341&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)   

## 계정 생성 절차
1. AWS 메인 사이트에 접속하면 무료로 AWS 시작하기 링크로 이동한다.  
    <img src="/images/AWS_login_1.png" width="50%" height="50%" title="AWS_login 1" alt="AWS login 1">  
2. AWS 루트 계정으로 사용할 이메일 주소와 사용자 이름을 입력한다. 이때 이메일 주소는 AWS에서 보내는 요금서나 알림을 받을 수 있는 이메일을 사용해야 한다.    
    <img src="/images/AWS_login_2.png" width="50%" height="50%" title="AWS_login 2" alt="AWS login 2">  
3. AWS가 루트 계정 이메일로 보낸 확인코드를 입력한다. 최대 5분 정도 소요될 수 있으나 코드를 받지 못했다면 코드 재전송을 하거나 이메일 계정 확인, 다른 이메일 계정을 사용해야한다.   
    <img src="/images/AWS_login_3.png" width="50%" height="50%" title="AWS_login 3" alt="AWS login 3">  
4. 루트 이메일 주소가 확인되면 루트 사용자 암호를 입력한다. 이 암호는 반드시 안전한 곳에 보관하여 유출되지 않도록 해야한다. 루트 계정은 AWS의 모든 권한을 가지고 있으므로 유출되면 사용자가 원치 않는 비용이나 보안 문제가 발생할 수 있다.      
    <img src="/images/AWS_login_4.png" width="50%" height="50%" title="AWS_login 4" alt="AWS login 4">  
5. 다음 단계에서는 AWS 사용 계획과 사용자의 연락처를 입력한다.      
    <img src="/images/AWS_login_5.png" width="50%" height="50%" title="AWS_login 5" alt="AWS login 5">  
6. 연락처 정보를 입력 후 AWS 요금 결제를 위한 신용카드 정보을 입력한다. 이때 체크카드는 해외원화결제가 제한되어 있는 경우 사용이 불가능할 수 있다.      
    <img src="/images/AWS_login_6.png" width="50%" height="50%" title="AWS_login 6" alt="AWS login 6">  
7. 전화번호 인증 수행   
8. 계정을 생성하면 지원 플랜을 선택할 수 있다. 일반 무료 플랜과, 개발자 플랜, 비즈니스 플랜이 있으며 개인 용도로 사용할 때는 프리티어 사용을 위해 무료 플랜을 선택한다.     

## MFA 설정
AWS 계정에 안전한 사용을 위해서 MFA 설정은 필수이다. MFA는 AWS 로그인을 수행할 때 비밀번호 외에 외부인증수단을 통한 추가 인증을 의미한다. 사용자가 실제로 사용하는 스마트폰 인증 앱이나 OTP와 같은 장치를 사용하여 인증하므로 더 안전한 로그인이 가능하다. 루트 계정 뿐만 아니라 IAM에서 생성한 사용자에게도 MFA를 적용할 수 있다.

### 절차
1. AWS 콘솔 오른쪽 상단에 내 계정을 클릭하면 아래 보안 자격 증명이 보이는데 그 링크에서 MFA를 추가할 수 있다.
2. MFA 디바이스 할당으로 MFA 추가를 진행한다.   
    <img src="/images/AWS_MFA_1.png" width="50%" height="50%" title="aws mfa 1" alt="aws mfa 1">
3. MFA 디바이스 유형을 선택한다. 스마트폰 인증 앱, USB 타입의 보안 키, 하드웨어 OTP 토큰 중 하나를 선택할 수 있다.      
    <img src="/images/AWS_MFA_2.png" width="50%" height="50%" title="aws mfa 2" alt="aws mfa 2">
4. 스마트폰 인증 엡을 선택하고 Google Authenticator이나 Microsoft Authenticator를 사용하여 MFA 연동을 진행한다.     
    <img src="/images/AWS_MFA_2.png" width="50%" height="50%" title="aws mfa 3" alt="aws mfa 3">