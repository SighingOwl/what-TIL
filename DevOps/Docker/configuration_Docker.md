# Docker 플랫폼 환경 구성
## Oracle VirtualBox에 Linux Ubuntu 환경 구성
### Mac OS에 Oracle VirtualBox 설치
> BIOS에서 가상화 기술 - VT 활성화
    
Oracle VirtualBoxs는 호스트 OS가 하드웨어 수준의 가상화로 하드웨어 자원을 VM에 나눠줄 수 있어야 한다. 따라서 BIOS에서 가상화 기능 활성화가 필요하다. 만일 Windows나 Linux를 OS로 사용한다면 메인보드 제조사의 지침에 따라 BIOS에서 가상화 기술(VT)를 활성화 해야한다. Mac의 경우 BIOS 대신 EFI를 사용해서 일반 PC처럼 BIOS에 집입할 수 없고 최신 Mac은 이미 가상화 기술 사용이 활성화되어 있어 별도의 설정이 필요하지 않다.
> <img src="/images/asus_bios.jpeg" width="50%" height="50%" title="asus_bios" alt=">
> 출처 : asus 한국
   
#### Oracle Virtual Box 설치

먼저 VirtualBox 사이트에서 가장 최신 버전의 VirtualBox 다운로드한다.    

<img src="/images/virtualbox_main_site.png" width="30%" height="30%" title="VirtualBox main page" alt="main page"></img>    
공식 사이트에 접속하면 최신 버전의 VirtualBox 다운로드 버튼이 보인다.

<img src="/images/virtualbox_site.png" width="30%" height="30%" title="VirtualBox download page" alt="download page"></img>     
다운로드 링크를 따라가면 VirtualBox를 설치할 플랫폼에 맞는 패키지를 다운로드 한다. Intel Macbook에 설치할 예정이므로 macOS/Intel hosts를 설치한다.
만일 Apple Silicon 계열 CPU를 사용하는 Mac에서 VirtualBox를 설치하고 싶다면 이전 빌드 버전에서 Developer preview for macOS / ARM64 (M1/M2) hosts를 사용할 수는 있으나 VirtualBox는 ARM 아키텍처 기반 CPU를 지원하지 않아 권장하는 방법은 아니다. 만일 Apple Silicon를 사용하는 Mac에서 VM을 사용을 희망한다면 VMware Fusion을 사용하는 것을 사용하는 것을 권장한다.

패키지를 다운로드 받았으면 이후에는 패키지를 실행시켜 설치를 수행한다.

### Ubuntu 설치 및 환경 구성

Ubuntu 머신을 생성하기 위해서 Ubuntu ISO 파일을 다운로드 한다. 원하는 버전을 선택해서 다운로드해도 되지만 LTS 버전을 사용하기를 권장한다.   
[Ubuntu 22.04 LTS - Jammy Jellyfish](https://releases.ubuntu.com/jammy/)    
<img src="/images/ubuntu_download_page.png" width="30%" height="30%" title="Ubuntu download page" alt="download page"></img>  

VirtualBox를 실행하여 새로 만들기로 VM을 생성한다.

<img src="/images/VM_create_1.png" width="30%" height="30%" title="VM_create_1" alt="VM_create_1"></img>   

먼저 VM의 이름과 OS를 선택한다. VM의 이름에 특정 OS의 이름이 포함되면 자동으로 OS가 선택된다. 다음으로 VM이 사용할 디렉토리를 설정해야하는데 이때 PC에 2개 이상 디스크가 설치되어 있다면 가급적 OS와 간섭을 피하기 위해 OS가 설치되어 있지 않은 디스크의 디렉토리를 선택하는 것을 권장한다. 여기서 사용할 OS의 ISO 이미지를 넣고 자동으로 설치되도록 할 수 있지만 VM에 Ubuntu를 설치할 때 디스크의 파티션을 나눌 예정이므로 선택하지 않았다.    

다음으로 VM의 기본적인 하드웨어 성능 설정이 필요하다. Docker를 사용할 때 실행하는 애플리케이션에 따라 시스템 리소스 사용량이 많을 수 있어 어느정도 여유를 두고 설정했다.    
<img src="/images/VM_create_2.png" width="30%" height="30%" title="VM_create_2" alt="VM_create_2"></img>   
<img src="/images/VM_create_3.png" width="30%" height="30%" title="VM_create_3" alt="VM_create_3"></img>   
<img src="/images/VM_create_4.png" width="30%" height="30%" title="VM_create_4" alt="VM_create_4"></img>   
> CPU : 4 cores, RAM : 8GB, DISK : 100 GB

머신을 생성한 다음에는 머신을 실행하기 전 설정이 필요하다. Ubuntu ISO 이미지를 넣고 설치가 필요하므로 [디스플레이 설정] 그림과 같이 광 디스크가 가장 먼저 부팅되고 하드디스크가 그 다음에 부팅이 되도록 순서를 조정한다. 프로세스에서 [프로세스 설정] 그림과 같이 PAE/NX 활성화를 해야하며 그 이유는 CPU의 물리적 주소 확장 기능을 VM에서도 사용하기 위해서다. 디스플레이 설정에서 별도로 수행할 작업은 없지만 VM 실행시 화면이 보이지 않을 경우에는 그래픽 컨트롤러를 VBoxVGA로 변경해야한다.  
<img src="/images/VM_config_1.png" width="30%" height="30%" title="VM_config_1" alt="VM_config_1"></img>    
<img src="/images/VM_config_2.png" width="30%" height="30%" title="VM_config_2" alt="VM_config_2"></img>    

저장소에서 IDE는 광 디스크가 연결되어 있어 Ubuntu 설치를 위한 ISO 이미지를 삽입한다. SATA에는 VM의 하드디스크가 연결되어 있는데 Docker가 사용할 별도의 디스크를 추가하고 연결한다.
 
<img src="/images/VM_Storage_config_1.png" width="30%" height="30%" title="VM_storage_1" alt="VM_storage_1"></img>      
<img src="/images/VM_Storage_config_2.png" width="30%" height="30%" title="VM_storage_2" alt="VM_storage_2"></img>      
<img src="/images/VM_Storage_config_3.png" width="30%" height="30%" title="VM_storage_3" alt="VM_storage_3"></img>      

네트워크 설정에서 어댑터 1은 이미 NAT로 설정되어 있다. 이유는 VirtualBox를 설치하면 VirtualBox 호스트 이더넷 어댑터가 생성되기 때문이다. 이 어댑터는 게이트웨이 역할을 수행하고 SSH 접속 주소로 사용할 수 있다. 어댑터 2를 호스트 전용 네트워크로 설정한다. 이 어댑터를 추가하면 VirtualBox에 생성된 VM들이 내부 네트워크를 사용해서 다른 VM에 호스트 이름으로 접속이 가능해진다.
> 원래 호스트 전용 어댑터가 있었으나 지원 종료 예정이므로 비슷한 기능을 수행하는 호스트 전용 네트워크를 사용해야 한다.

<img src="/images/VM_Network_config.png" width="30%" height="30%" title="VM_network" alt="VM_network"></img>    

설정이 끝났으면 VM을 실행하여 Ubuntu 설치를 진행한다. 먼저 VM을 실행하면 광 디스크에 넣은 ISO 이미지가 실행되며 Ubuntu 설치화면이 출력된다. Ubuntu 사용 언어는 영어를 사용하는 것을 권장한다. 한글로 사용하면 디렉토리의 이름이 한글로 설정되어서 추후 스크립트 실행에 오류가 발생할 수 있다.   

<img src="/images/Ubuntu_Install_1.png" width="30%" height="30%" title="ubuntu_install_1" alt="Ubuntu_install_1"></img>   
<img src="/images/Ubuntu_Install_3.png" width="30%" height="30%" title="ubuntu_install_3" alt="Ubuntu_install_3"></img>    
<img src="/images/Ubuntu_Install_4.png" width="30%" height="30%" title="ubuntu_install_4" alt="Ubuntu_install_4"></img>     

'Installation type' 단계에서 보통 디스크를 모두 비우고 설치를 수행하지만 파티션을 따로 설정하려면 something else를 선택하여 진행한다. 파티션은 다음과 아래와 같이 나누었다.
|파티션|용량|형식|마운트|
|-|-|-|-|
|/dev/sda1|70000MB|XFS|/|
|/dev/sda1|8192MB|swap|-|
|/dev/sda3|15000MB|Ext4|/DATA|
|/dev/sda4|남은 용량|Ext4|/BACKUP|
|/dev/sdb1|남은 용량|XFS|/var/lib/docker|

<img src="/images/Ubuntu_Install_5.png" width="30%" height="30%" title="ubuntu_install_5" alt="Ubuntu_install_5"></img>     

파티션을 나누었으면 설치를 진행하고 계정을 생성한다.

<img src="/images/Ubuntu_Install_6.png" width="30%" height="30%" title="ubuntu_install_6" alt="Ubuntu_install_6"></img>     

### Ubuntu Linux 환경 구성
호스트 OS에서 VM으로 SSH 접속이 가능하도록 수동으로 고정 IP를 설정한다.
|속성|값|
|-|-|
|IP|192.168.56.101|
|subnet|255.255.255.0|
|gateway|192.168.0.2|
|dns|8.8.8.8|

네트워크 테스트를 위해 net-tools 패키지를 설치하고 ping 테스트를 실행 및 SSH 접속 테스
- openssh-server, vim, net-tools 설치
    
    ```bash
    sudo apt-get install openssh-server vim net-tools -y
    ```
    
- 네트워크 테스트
    
    ```bash
    ping -c 2 <local pc ip>
    pint -c 2 8.8.8.8
    ```
    
    - Windows 사용시 8.8.8.8로 ping 테스트가 되지 않는 경우 방화벽을 해제할 필요가 있다.
        - Mac은 불필요
- ssh 설정
    - mac은 터미널을 사용해서 바로 ssh 접속이 가능하다.
        
        ```bash
        ssh jongeun@192.168.56.101
        ```
        
        - ubuntu는 패스워드 접속이 불가능하므로 ssh 키를 생성하여 공개키를 로컬로 전달해야한다.
        - /etc/ssh/sshd_config에서 임시로 패스워드 접속이 가능하도록 PASSWORDAUTHENTICATION yes 활성화 후 ssh 접속하여 공개키를 로컬로 복사한다.

## Docker 엔진 설치와 구성
리눅스에 Docker 엔진 설치는 아래 Docker docs의 설치 지침에 따라서 설치를 진행했다.  
[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

1. Docker 설치에 필요한 도구 설치
    
    ```bash
    sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
    ```

|패키지|패키지 설명|    
|-|-|
|apt-transport-https|https 링크에서 파일을 전달 받을 수 있도록 하는 패키지|
|ca-certificates|https를 위한 인증서를 사용할 수 있는 패키지|
|curl|api 통신에 사용하는 패키지|
|gnupg-agent|docker가 사용할 gnu 패키지 가드|
|software-properties-common|docker 레포지토리에서 등록, 다운로드 등의 관리 기능 제공|
2. Docker 레포지토리 등록
    
    ```bash
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```
    
    - Docker의 경우 EE(Enterprise)버전과 CE(Comunity) 버전으로 나뉜다.
    - CE 버전은 Edge 버전과 Stable 버전으로 나뉜다.
        - Edge는 베타 버전
        - Stable은 안정화된 버전
3. Docker 설치
    1. apt에 있는 docker 버전 검증
        
        ```bash
        sudo apt-cache policy docker-ce
        docker-ce:
          Installed: (none)
          Candidate: 5:24.0.7-1~ubuntu.22.04~jammy
          Version table:
             5:24.0.7-1~ubuntu.22.04~jammy 500
                500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
        ```
        
        - 현재 최신 버전이 Candidate에서 확인할 수 있는 24.0.7이다.
    2. Docker 설치
        
        ```bash
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        ```
        
4. 로그인된 사용자가 Docker를 사용할 수 있도록 docker 그룹에 사용자 추가
    
    ```bash
    sudo usermod -aG docker jongeun
    sudo systemctl daemon-reload
    sudo systemctl enable docker
    sudo systemctl restart docker
    sudo reboot
    ```

## 간단한 컨테이너 서비스 구현
- Docker Hub 사용할 때 제약 사항
    - 익명의 사용자의 경우 6시간동안 100개의 이미지만 다운로드 가능
    - 무료 사용자의 경우 6시간동안 200개의 이미지만 다운로드 가능
    - 회사 사무실에서 사용 중일 때 하나의 라우터 혹은 허브를 사용할 경우 IP에도 제한된다.
- Docker Hub에서 이미지 고르는 요령
    - Docker Hub에서 원하는 이미지를 검색
    - 필터링을 통해서 적합한 이미지를 선택
    - 되도록이면 slim, alpine 태그가 붙은 경량 이미지를 사용

### Centos7

- docker로 centos:7 이미지 pull
    
    ```bash
    docker pull centos:7
    7: Pulling from library/centos
    2d473b07cdd5: Pull complete
    Digest: sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
    Status: Downloaded newer image for centos:7
    docker.io/library/centos:7
    
    docker images
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    centos       7         eeb6ee3f44bd   2 years ago   204MB
    ```
    
    - Centos7은 레이어 하나로 되어 있다.

### Ubuntu 16.04

- docker로 ubuntu:16.04 이미지 pull
    
    ```bash
    docker pull ubuntu:16.04
    16.04: Pulling from library/ubuntu
    58690f9b18fc: Pull complete
    b51569e7c507: Pull complete
    da8ef40b9eca: Pull complete
    fb15d46c38dc: Pull complete
    Digest: sha256:1f1a2d56de1d604801a9671f301190704c25d604a416f59e03c04f5c6ffee0d6
    Status: Downloaded newer image for ubuntu:16.04
    docker.io/library/ubuntu:16.04
    
    docker images
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    ubuntu       16.04     b6f507652425   2 years ago   135MB
    ```
    
    - Centos7과 달리 ubuntu:16.04는 4개의 레이어로 되어 있다.

### 컨테이너 실행

- docker run
    
    ```bash
    docker run -it --name=sys-container-1 centos:7 echo 'welcome fastcampus!'
    welcome fastcampus!
    ```
    
- docker ps -a
    
    ```bash
    docker ps -a
    CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS                      PORTS     NAMES
    9f7224c95c7d   centos:7   "echo 'welcome fastc…"   23 seconds ago   Exited (0) 22 seconds ago             sys-container-1
    ```
    
    - docker 컨테이너에 echo 다음에 별도의 실행 명령을 내리지 않아 echo 명령 후 자동으로 종료
- docker rm
    
    ```bash
    docker rm sys-container-1
    ```
    
    - 컨테이너 삭제
- docker exec
    
    ```bash
    docker exec sys-container-1
    ```
    
    - 실행 중인 컨테이너에 다시 명령을 내리고 싶을 때 사용
    - 중지 된 컨테이너에서는 동작하지 않는다.
- docker start
    
    ```bash
    docker start sys-container-1
    ```
    
    - 중지된 컨테이너를 다시 실행
- ctrl+p+q
    - 실행 중인 컨테이너를 중지시키지 않고 빠져나올 때 사용

### Nginx

- nginx 이미지 pull
    
    ```bash
    docker pull nginx:1.25.3-alpine
    ```
    
- nginx 이미지 실행
    
    ```bash
    docker run -d -p 8001:80 --name=webserver1 nginx:1.25.3-alpine
    ```
    
    - nginx는 80번 포트를 사용하므로 80번포트로 매핑해준다.
- 새로운 index.html을 nginx 컨테이너에 전달
    
    ```bash
    docker cp index.html webserver1:/usr/share/nginx/html/index.html
    ```
    
    - docker cp를 사용해서 파일을 컨테이너로 복사

### 이미지 빌드

- Dockerfile 작성
    
    ```docker
    FROM nginx:1.25.0-alpine
    COPY index_html_sample2.html /usr/share/nginx/html/index.html
    COPY docker_logo.png /usr/share/nginx/html/docker_logo.png
    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]
    ```
    
- 이미지 빌드
    
    ```bash
    docker build -t myweb:v1.0 .
    ```
    
    - docker build할 때는 Dockerfile과 소스, 데이터가 같은 디렉토리에 있어야 한다.
- 이미지 테스트
    
    ```bash
    docker run -d --name=webserver2 -p 8002:80 myweb:v1.0
    ```
    
    - 이미지를 실행하여 테스트한다.

### MySQL

- MySQL 이미지 pull
    
    ```bash
    docker pull mysql:5.7-debian
    ```
    
- MySQL 컨테이너 실행
    
    ```bash
    docker run -it -e MYSQL_ROOT_PASSWORD=pass123# mysql:5.7-debian /bin/bash
    ```
    
- MySQL 서비스 실행 및 접속
    
    ```bash
    /etc/init.d/mysql start
    mysql -uroot -p
    ```
    

### MariaDB

- MariaDB와 MySQL을 워크밴치 연동 - 컨테이너도 동일하게 연동 가능
- MariaDB 컨테이너 실행 - 이미지를 자동으로 pull
    
    ```bash
    docker run --name mariabd -e MYSQL_ROOT_PASSWORD=mkevin -d \
    > -e MARIADB_DATABASE=item -p 3306:3306 mariadb:10.2
    ```
    
- MySQL 워크벤치 설정
    
    <img src="/images/MySQL_workbench.png" width="30%" height="30%" title="MySQL_workbench" alt="MySQL_workbench"></img>     
    
- docker 컨테이너에서 동작 중인 mariadb와 연동

## Docker GUI 관리도구
### Portainer

- 웹 GUI 기반 docker 컨테이너 관리 도구
- Portainer CE 버전은 docker, kubernete 및 ACI 환경을 관리하는데 사용할 수 있는 컨테이너화된 애플리케이션을 위한 경량 서비스 제공 플랫폼
- ‘Smart’ GUI 및 광범위한 API를 통해 docker에서 사용되는 대부분의 리소스를 관리

### Potainer 컨테이너 생성

```bash
docker pull portainer/portainer-ce
docker volume create portainer_data
docker run -d -p 9000:9000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
--restart=always \
portainer/portainer-ce
```

- Portainer를 사용하기 위해 portainer가 사용할 볼륨을 생성하고 공유 설정 해야한다.
- 공유 설정에서 runc를 통해 커널 기술을 공유하고 컨테이너를 관리하므로 runc의 런타임 소켓을 공유해야 한다. - Docker가 가진 정보를 띄우기 위해
- Portiainer가 오류가 생겼을 때 자동으로 재시작하도록 설정
- 9000포트로 접속하여 실행 확인
    
    <img src="/images/Portainer.png" width="30%" height="30%" title="Portainer" alt="Portainer"></img>     
    

### Portainer APP Template으로 특정 컨테이너 배포

- 브라우저 콘솔 환경에서 이미지를 선택하고 명령어 없이 콘솔 환경에서 컨테이너 배포 가능
- nginx
    - app templates에서 nginx 선택
    - deploy
        
        <img src="/images/Potainer_nginx.png" width="30%" height="30%" title="Portainer_nginx" alt="Portainer_nginx"></img>     