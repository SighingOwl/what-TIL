# Docker 이미지 CI/CD
<img src="/images/Jenkins_docker_cicd_1.png" width="75%" height="75%" title="jenkins docker cicd 1" alt="jenkins docker cicd 1">    

Pipeline을 사용해서 코드를 빌드, 테스트, 코드 분석, docker 이미지 빌드 및 업로드까지 진행했다. 이제 Amazon ECS를 사용해서 docker화 된 애플리케이션을 호스팅하는 것이 필요하다.

## Container Hosting Platforms

- Docker Engine
    - Docker engine에서 직접 컨테이너를 실행시키는 것은 로컬에서 테스팅하는 용도로는 충분하지만 실제 프로덕션으로 사용하기에는 불편함이 많다. Docker engine을 직접 세팅과 관리를 해야하고 EC2 인스턴스나 가상 머신, 물리 머신에서 동작 중인 플랫폼 역시 직접 관리해야 한다. 또한 docker enigne은 고가용성, 자가복구와 같은 기능을 제공하지 않는다.
- Kubernetes
    - Standalone, EKS, AKS, GKE, OpenShift 등의 형태로 사용할 수 있다.
- AWS ECS
    - Amazon에서 제공하는 완전관리형 컨테이너 서비스이다. 컨테이너를 쉽게 실행, 중지 및 관리할 수 있는 컨테이너 관리 서비스.

### Create & Setup ECS Cluster

1. VPC 및 subnet 선택 - ap-northeast-2d 제외
2. 인프라 설정
    1. AWS Fargate(서버리스) - 기본 활성화가 되어있다.
        1. AWS가 리소스를 모두 관리하므로 가장 사용하기 편하다.
    2. Amazon EC2 인스턴스
3. 모니터링
    1. CloudWatch가 컨테이너가 사용하는 리소스에 대한 지표를 수집 설정
4. 테스크 정의
    1. 클러스터 생성 후 진행
    2. 컨테이너에 대한 정보를 가지고 있다.
        1. 레지스트리 위치
        2. Docker 이미지 위치
        3. 시스템 리소스 용량
    3. 새 테스크 정의
    4. 인프라 요구사항
        1. 시작 유형
        2. OS, 아키텍처, 네트워크
        3. CPU
        4. 테스크 역할
            1. CloudWatch를 사용하므로 역할에 CloudWatch 정책이 포함되어야 한다.
    5. 컨테이너
        1. 이미지 URL에 컨테이너가 있는 URL을 입력 - ECR의 URL
        2. 로그 수집
            1. 수집할 로그 설정
    6. 테스크 역할 수정
        1. “CloudWatchLogsFullAccess” 정책 추가

### Create Service

ECS 클러스터에서 직접 테스크를 실행할 수 있지만 이 경우 컨테이너를 직접 관리해야한다. 하지만 서비스를 생성해 컨테이너를 테스크를 실행하면 로드밸런서 생성 옵션이나 컨테이너 정보, 컨테이너를 관리하기 위한 서비스를 제공한다.

1. 환경
2. 배포 구성
    1. 서비스 - 어떠한 작업이 실행, 중지를 반복할 수 있는 것이면 서비스를 선택한다.
    2. 테스크 - 어떠한 작업이 실행 후 종료되는 것이면 테스크를 선택한다.
    3. 테스크 정의
        1. 패밀리 - 사전에 생성한 테스크 선택
        2. 서비스 유형 - 복제본
        3. 원하는 테스크 수 설정
3. 네트워킹
    1. VPC 및 서브넷 선택
    2. 보안그룹 선택 혹은 생성
        1. HTTP 모든 트래픽 허용
        2. TCP 8080번 포트 모든 트래픽 허용
4. 로드 밸런싱
    1. Application Load Balancer 사용
    2. 대상 그룹 설정

<img src="/images/Jenkins_docker_cicd_2.png" width="75%" height="75%" title="jenkins docker cicd 2" alt="jenkins docker cicd 2">    

서비스 생성 완료 후 상태 페이지

### Pipeline

Jenkins에 “Pipeline: AWS Steps”를 먼저 설치한다.

```bash
def COLOR_MAP = [					// Slack message color map
	'SUCCESS': 'good',
	'FAILURE': 'danger',
]
pipeline {					// Pipeline start
	agent any 				// Agent execute anywhere
	environment {			// AWS ECR repository, ECS info -> global variables
		registryCredential = 'ecr:ap-northeast-2:awscreds'
		appRegistry = "862279563460.dkr.ecr.ap-northeast-2.amazonaws.com/vprofileappimg"	// ECR registry
		vprofileRegistry = "https://862279563460.dkr.ecr.ap-northeast-2.amazonaws.com"
		cluster = "vprofile" 
		service = "vprofileappsvc"
	}
	tools {					// build tools
	  	maven "MAVEN3"
	  	jdk "OracleJDK11"
	}

	stages {				// execute build
		stage('Fetch Code') {
			steps {
				git branch: 'docker', url: 'https://github.com/hkhcoder/vprofile-project.git'
			}
		}

		stage('UNIT TESTS') {
			steps {
				sh 'mvn test'						// Test with maven
			}
		}

		stage('Checkstyle Analysis') {				// Code analysis with maven
			steps {	
				sh 'mvn checkstyle:checkstyle'
			}
		}

		stage('Sonar Analysis') {					// Code analysis & Upload result to SonarQube server
			environment {
				scannerHome = tool 'sonar4.7'
			}
			steps {
				withSonarQubeEnv('Sonar') {
					sh '''${scannerHome}/bin/sonar-scanner \
						-Dsonar.projectKey=vprofile \
						-Dsonar.projectName=vprofile \
						-Dsonar.projectVersion=1.0 \
						-Dsonar.sources=src/ \
						-Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
						-Dsonar.junit.reportsPath=target/surefire-reports/ \
						-Dsonar.jacoco.reportsPath=target/jacoco.exec \
						-Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
				}
			}
		}

		stage('Quality Gate') {						// Wait for quality result from SonarQube server
			steps {
				timeout(time: 1, unit: 'HOURS') {
					waitForQualityGate abortPipeline: true
				}
			}
		}

		stage('Build App Image') {
       		steps {
         		script {
                	dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
            	}
    		}
    
   		}

		stage('Upload App Image') {		// Upload Image to Amazon ECR
			steps {
				script {		// Docker command
					docker.withRegistry( vprofileRegistry, registryCredential ) {
                		dockerImage.push("$BUILD_NUMBER")
                		dockerImage.push('latest')
              		}
				}
			}
		}

		stage('Deploy to ecs') {
			steps {
				withAWS(credentials: 'awscreds', region: 'ap-northeast-2') {
					sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
				}
			}
		}
	}
	post {				// Slack Notification after stages
		always {
			echo 'Slack Notifications.'
			slackSend channel: '#jenkinscicd',
				color: COLOR_MAP[currentBuild.currentResult],
				message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
		}
	}
}
```

- 이 코드를 Jenkins에서 빌드하면 ECS 클러스터의 서비스에서 새로운 테스크가 실행된다. 이때 이전의 테스크는 정지되고 자동으로 새로운 테스크로 전환된다.
- 이전 테스크 ID : d4fc52e420454320bbe4c43345d47e4c
    → 새로운 테스크 ID : 6ba67d2b99274d059b13e0fde43ac8dd