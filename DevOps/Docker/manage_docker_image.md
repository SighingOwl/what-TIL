#  Docker 이미지 관리
## Docker 이미지 이해와 구조 확인
### Docker 이미지

- 컨테이너 런타임에 필요한 바이너리, 라이브러리, 설정 값 등을 포함
- 변경되는 상태 값은 포함하지 않아 immutable 하다
- 기존의 이미지를 변경할 수 없지만 기존 이미지를 사용해서 새로운 이미지 생성은 가능

### Docker Command

- 이미지 내려 받기
    
    ```bash
    docker [image] pull [options] name:[tag]
    ```
    
    - tag를 붙이지 않으면 최신 버전으로 받는다.
    - name에는 이미지 이름이 아닌 이미지의 경로를 사용하면 경로에 있는 이미지를 받을 수 있다.
        - 이미지 이름만 사용할 수 있는 것은 docker hub의 기본 경로가 설정되어 있기 때문
        - 다른 레지스트리를 사용하고 싶다면 전체 경로를 입력한다.
- 이미지 구조 확인
    
    ```bash
    docker image inspect [image]:[tag]
    ```
    
    - 다운로드 받은 이미지의 구조 파악을 위해 사용하며 JSON 형태로 내용을 출력한다.
    - 기본적으로 이미지는 여러 레이어 구조를 가지고 있다.
        - 레이어를 사용하는 이유
            - 어떤 이미지를 다운로드 받을 때 이미 받은 이미지를 사용하는 경우 재사용이 가능하기 때문
            - 언제든지 레이어를 재조합해서 사용할 수 있으므로.
    - Digest
        - 실제로 이미지가 저장이 될 때 /var/lib/docker 영역을 사용한다.
        - 이미지 구분을 할 때 이 값을 사용해서 구분한다.
    - Container
        - Docker는 이미지를 빌드할 때 이미지를 불변의 상태로 만들기 위해 컨테이너 상태로 만든다.
        - Docker commit을 하면 빌드한 컨테이너를 신규 이미지로 만든 후 다시 컨테이너로 빌드하는 것을 반복한다.
        - 이미지는 read only 형태로 만들어지고 docker run 명령으로 컨테이너를 생성하연 read write가 추가된다.
            - docker run을 사용하면 이미지의 스냅샷이 컨테이너에 저장된다.
            - commit을 하면 컨테이너 내부에서 변경되거나 추가된 데이터를 포함한 새로운 이미지가 생성된다.
            - 컨테이너는 여러 레이어가 하나의 fs를 사용할 수 있도록 UFS를 사용한다.
        - Hostname이 최종적으로 만들어진 컨테이너의 ID가 된다.
        - MergedDir : 레이어를 합친 이미지의 경로
        - UpperDir : 변경 사항이 발생했을 때 경로
    
    ```bash
    docker image history [image]:[tag]
    ```
    
    - 이미지 빌드에 사용된 Dockerfile에 대한 정보를 확인할 수 있다.
    
    ```bash
    2f44b7a888fa: Pull complete
    5abb3599da34: Pull complete
    4f4fb700ef54: Pull complete
    fa608a886227: Pull complete
    afe6bbf00437: Pull complete
    fd0ef2a49677: Pull complete
    ```
    
    - 이미지는 여러 레이어들로 이루어져 있고 각 레이어는 서비스나, 소스, OS계층 정보 등으로 이루어져 있다.
    
    ```bash
    /var/lib/docker/image/overlay2/distribution/diffid-by-digest/sha256
    ```
    
    - 레이어 구조 저장 장소 - mac에서는 어디에 있는지 확인 필요 

## Docker hub에 이미지 push
### 이미지 올리기

- Dockerfile을 사용해 생성한 이미지나 docker commit을 통해 생성된 이미지를 저장하는 곳을 registry라 한다.
- 레포지토리는 public/private로 설정 가능
- docker hub의 레포지토리에 넣기 위한 태그가 필요하다.
    - 자신의 계정 이름이 포함된 태그를 사용
    - 버전, os를 포함하는 것이 일반적
    
    ```bash
    docker image tab myweb:v1.0 sighingowl/myweb:v1.0
    ```
    
    - 태그를 추가하면 기존의 이미지의 ID와 동일한 이미지의 태그가 추가가 된다. → 이미지가 변경되지 않는다.
- docker push로 이미지 업로드
    
    ```bash
    docker push sighingowl/myweb:v1.0
    ```
    
- docker hub를 확인하면 자동으로 레포지토리를 생성하여 이미지가 업로드 된 것을 볼 수 있다.
    
    <img src="/images/Docker_hub_1.png" width="50%" height="50%" title="docker hub" alt="docker hub">
    
- 업로드 된 이미지를 다른 머신에서 다운받아 실행 테스트를 해본다.

### 로컬에서 docker hub 로그인

- 로컬에서 docker 로그인을 해야 docker hub로 이미지 push가 가능하다.
    
    ```bash
    docker login
    Username : [본인 계정]
    Password : [본인 암호]
    ```
    
    - 이미 자격 증명이 있는 경우 자동으로 로그인 되기도 한다.
    - 토큰을 사용한 로그인 방법
        1. Docker hub에서 토큰 발급
        2. access_token 파일을 사용하여 로그인
            
            ```bash
            vi .access_token
            cat .access_token | docker login --username [본인 계정] --password-stdin
            ```
            

### 이미지 공유 방법

1. Registry에 push
2. Dockerfile을 Github에 올려 공유
3. 이미지를 docker save를 통해 파일로 백업하여 전달 후 docker load를 통해 서버에 등록

### Docker 이미지 백업 및 이전

- docker save 명령을 통해 레이어로 구성된 이미지를 *.tar 파일로 묶어 파일로 저장
- 해당 파일을 전달 받은 서버에서 docker load를 통해 이미지를 등록
- hostos1
    
    ```bash
    docker image save [image]:[tag] > [image].tar
    docker image save [image]:[tag] | gzip > [image].tar.gz
    docker image save [image]:[tag] | bzip > [image].tar.bz2
    
    scp [image].tar.gz jongeun@hostos2:/home/jongeun/backup/[image].tar.gz
    ```
    
    - tar 그대로 보내도 되고 gzip이나 bzip을 사용해 압축 후 보내도 된다.
    - scp를 사용해 2번 서버에 파일 전달
- hostos2
    
    ```bash
    docker image load < [image].tar.gz
    ```
    
    - 2번 서버에서는 파일을 로드만 수행하여 이미지를 등록

### 이미지 삭제

- 로컬에 사용하지 않은 이미지를 계속 두면 용량 부족 문제가 발생할 수 있어 불필요한 이미지는 삭제한다.

```bash
docker image rm [image:tag | imageID]
docker rmi [option] [image:tag | imageID]

# 이미지 전체 삭제
docker rmi $(docker images -q)

# 특정 이미지명이 포한된 것만 삭제
docker rmi $(docker images | grep debian)

# 특정 이미지명이 포함되지 않은 것만 삭제
docker rmi $(docker images | grep -v centos)

# 자수 사용하는 명령을 전역 alias로 적용하여 활용
vi .bashrc
alias cexrm='docker rm $(docker ps --filter 'status=exited' -a -q)'
source .bashrc
```

## Docker 레지스트리 관리
### Docker container registry

- 기업 내부에서 생성한 프로젝트용 이미지를 public에 올리는 경우는 없다.
- image에 네트워크나 OS 및 미들웨어 설정 등의 정보가 포함되어 있으므로 보안상 불특정 다수에게 공개되는 곳에 올릴 수 없는 경우 private registry를 구축
- 회사 인프라내에 private docker registry를 구축하기 위해서는 Docker hub에 공개되어 있는 공식 image인 registry를 사용한다.
- 적은 용량의 container service로 사용하기에 적합

### 로컬 Docker registry 설치

```bash
docker pull registry
docker run -d \
-v /home/jongeun/registry_data:/var/lib/registry \
-p 5000:5000 \
--restart=always \
--name=local-registry \
registry
```

- 레지스트리 데이터를 보존하기 위한 볼륨 연결이 필요하다.
- 컨테이너 레지스트리는 기본적으로 5000번 포드에 노출된다.

### 로컬 레지스트리에 업로드

```bash
docker image tag sighingowl/myweb:v1.0 192.168.56.101:5000/myweb:v1.0

sudo vi /etc/inti.d/docker
# DOCKER_OPTS=--insecure-registry 192.168.56.101:5000

sudo vi /etc/docker/daemon.json
# { "insecure-registries": ["192.168.56.101:5000"] }

sudo systemctl restart docker.service

curl -X GET http://192.168.56.101:5000/v2/_catalog

curl -X GET http://192.168.56.101:5000/v2/myweb/tags/list
```

- Docker hub에 업로드 할 때 태그에 계정 이름이 포함되어야 하는 것처럼 로컬 레지스트리에 업로드하기 위해서 로컬 ip:5000이 태그에 포함되어야 한다.
- Docker daemon은 업로드 주소가 docker hub로 되어있어 로컬 주소로 수정해야 로컬 레지스트리에 업로드 가능
- curl 명령을 사용해서 레지스트리에 있는 이미지나 이미지의 태그를 확인 가능

### 다른 서버에서 로컬 레포지토리로 이미지 pull & push
- 2번 서버에서 1번 서버의 레지스트리의 이미지를 pull
- 1번 서버에서 설정한 것처럼 로컬 레지스트리를 사용 설정을 동일하게 해주어야 한다.

```bash
docker pull 192.168.56.101:5000/myweb:v1.0
```

- 이미지를 받을 수 있는 것을 확인할 수 있다.