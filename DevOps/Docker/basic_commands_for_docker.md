# Docker 컨테이너를 위한 기본적인 CLI 명령
## 컨테이너 격리 기술

### Chroot & Pivot_root

```bash
root@24567ba4ca01:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

- Docker 컨테이너에 들어오면 / 영역은 일반적인 OS 구조와 유사하다.
- 이 기술을 chroot&pivot_root라고 한다. → LXC 기술
- 최상위 경로를 만들어 독립적인 파일 시스템을 구축

### Mount namespace

```bash
root@24567ba4ca01:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         100G  2.5G   98G   3% /
tmpfs            64M     0   64M   0% /dev
shm              64M     0   64M   0% /dev/shm
/dev/sdb1       100G  2.5G   98G   3% /etc/hosts
tmpfs           3.9G     0  3.9G   0% /proc/asound
tmpfs           3.9G     0  3.9G   0% /proc/acpi
tmpfs           3.9G     0  3.9G   0% /proc/scsi
tmpfs           3.9G     0  3.9G   0% /sys/firmware
```

- /dev/sdb1이 공유 되는 부분이다.
    - host 머신의 디바이스를 마운트 및 정보를 복제해서 사용가능
    - 운영이 가능한 OS에 pivot 영역이 자리잡는다.
- mount namespace
    - 디바이스를 마운트하는 네임스페이스

### UTS namespace

```bash
root@24567ba4ca01:/# hostname
24567ba4ca01
```

- 호스트네임 정보를 제공하는 것이 UTS namespace이다.
- 컨테이너 ID가 hostname이 된다.

### PID / IPC namespace

```bash
root@24567ba4ca01:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 08:37 pts/0    00:00:00 bash
root          12       1  0 08:47 pts/0    00:00:00 ps -ef
```

- 예시의 컨테이너가 시스템 컨테이너여서 root가 PID 1인 것을 확인 가능
- PID / IPC 프로세스를 배치하는 것이 PID / IPC namespace

### Network namespace

```bash
root@24567ba4ca01:/# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:03  txqueuelen 0  (Ethernet)
        RX packets 1631  bytes 29316449 (29.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1209  bytes 69837 (69.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

- 네트워크와 관련된 정보를 배치하는 것이 Network namespace

### 정리

| 격리 기술 | 설명 |
| --- | --- |
| chroot | 프로세스의 루트 디렉토리를 변경, 격리하여 가상의 루트 디렉토리를 생성 |
| pivot_root | 루트 파일시스템 자체를 바꿔, 컨테이너가 전용 루트 파일시스템을 가지도록 함 - chroot 보완 |
| Mount namespace | namespace 내에 파일 시스템 트리를 구성 |
| UTS namespace | 컨테이너에 대한 hostname 격리를 수행하여 고유한 hostname 보유 가능 |
| PID namespace | PID와 프로세스를 분리 - systemd와 분리 |
| Network namespace | 네트워크 리소스 (IP, Port, route table, ethernet, …) 할당 |
| IPC namespace | 전용 process table 보유 |

### Docker 컨테이너 Lifecycle

- docker 컨테이너는 docker create 명령을 통해 image의 snapshot으로 /var/lib/docker 영역에 생성
- docker start 명령은 읽고 쓰기가 가능한 Process 영역 즉, container layer를 생성하여 동적 컨테이너를 구성, docker stop 명령을 container layer를 삭제
- docker rm은 생성된 snapshot을 삭제하는 과정을 통해 docker container lifecycle을 알 수 있다.

## 컨테이너 관리를 위한 docker CLI

### docker [container] run [option] docker_image [command]

```bash
docker run -itd -p 6060:6060 --name=node-run -h node-run noderun:1.0
```
| 옵션 | 설명 |
| --- | --- |
| -i, —interactive | 대화식 모드 열기 |
| -t | TTY (단말 디바이스) 할당 |
| -d, —detach=true | 백그라운드에서 컨테이너를 실행하고 컨테이너 ID 등록 |
| —name | 실행되는 컨테이너에 이름 부여, 사용하지 않으면 자동으로 부여된다. |
| —rm | 컨테이너 종료 시 자동으로 컨테이너 제거 |
| —restart | 컨테이너 종료시 적용할 재시작 정책 지정, no, on-failure, on-failure:횟수, always |
| —env | 컨테이너의 환경변수 지정, —env-file은 여러 환경변수를 파일로 생성하여 지정|
| -v, —volume=호스트경로:컨테이너경로 | 호스트 경로와 컨테이너 경로의 공유 볼륨 설정 - Bind mount |
| -h | 컨테이너의 호스트 네임을 지정, 미지정시 컨테이너 ID가 호스트명으로 등록 |
| -p 호스트포트:컨테이너포트, —publish | 호스트 포트와 컨테이너 포트 연결 |
| -P, —publish-all=true|false | 컨테이너 내부의 노출된 포트를 호스트 임의의 포트에 개시 |
| —workdir, -w | 컨테이너 내부의 작업 경로 |

### docker top | port | stats

```bash
docker top node-run
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                3794                3772                0                   18:19               pts/0               00:00:00            /sbin/tini -- node runapp.js
root                3814                3794                0                   18:19               pts/0               00:00:00            node runapp.js
```
- 컨테이너에서 실행 중인 프로세스 정보를 확인할 때 사용

```bash
docker port node-run
6060/tcp -> 0.0.0.0:6060
6060/tcp -> [::]:6060
```
- 컨테이너에 매핑 된 포트 정보를 확인할 때 사용

```bash
docker stats node-run
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O   PIDS
089f9bb3ae35   node-run   0.01%     9.281MiB / 7.747GiB   0.12%     4.08kB / 517B   0B / 0B     11
```
- 실행 중인 컨테이너의 리소스 사용 통계에 대한 실시간 스트림을 보여준다.
    
    #### cadvisor
    - docker stats와 유사한 통계, metric 정보를 수집

### docker logs

```bash
docker logs -f node-run
```

- -f : 실시간으로 발생하는 로그를 지속적으로 보여줌
- 컨테이너의 로그를 수집
- docker 컨테이너의 로그는 지속적으로 쌓이기 때문에 디스크 용량 문제의 원인이 된다.
    - truncate를 사용해서 주기적으로 로그 삭제
    - 컨테이너에서 발생하는 로그 크기 제한 → /etc/docker/daemon.json에 관련 내용 추가
        
        ```json
        { "log-driver": "json-file",
        	"log-opts": {
        		"max-size": "30m",
        		"max-file": "10"
        	}
        }
        ```
        
    - 컨테이너 실행할 때 로그 크기 제한
        
        ```bash
        docker run -itd -p 6062:6060 --name=node-run2 \
        -h node-run --log-driver json-file --log-opt max-size=30m --log-opt max-file=10 \
        nodrun:v1.0
        ```
        

### docker [container] inspect

```bash
docker container inspect node-run
```

- 컨테이너 내부 구조 정보 확인
- image inspect와 달리 컨테이너 내부 정보를 확인할 수 있으며 네트워크 정보와 같은 동적인 정보가 포함되어 있다.

### docker cp | restart

```bash
docker cp [복사할 파일 경로] [container]:[복사 경로]
```

- 호스트 머신과 컨테이너 내부 사이에서 파일을 복사할 때 사용 - 양방향 가능

```bash
docker restart [container]
```

- 컨테이너를 재시작할 때 사용
- docker cp를 사용하고 변경점을 적용하기 위해 컨테이너를 재시작 할 필요가 있을 때 사용

### docker stop | start | pause | unpause

- 컨테이너 정지, 시작, 일시정지, 일시정시 해제

### docker kill

- linux의 kill 명령과 유사하며 컨테이너 종료와 함께 종료 코드를 반환

### docker attach | exec

```bash
docker attach [container]
```

- 실행 중인 컨테이너에 stdout, stdin, stderr 스트림 연결

```bash
docker exec [option] [container] [cmd]
```

- 실행 중인 container 내부에 접속하지 않고 컨테이너에 명령을 실행할 때 사용

### docker diff

```bash
docker diff [container]
```

- 컨테이너의 변경점을 확인
- symbol
    
    
    | Symbol | 설명 |
    | --- | --- |
    | A | 파일이나 디렉토리가 추가됨 |
    | B | 파일이나 디렉토리가 삭제됨 |
    | C | 파일이나 디렉토리가 변경됨 |

### docker commit

```bash
docker commit [실행 중인 컨테이너] [생성할 이미지 이름 : 태그]
```

- 실행 중인 컨테이너의 변경점을 포함한 새로운 이미지 생성

### docker export | import

```bash
docker export [container] > [container.tar]
```

- 실행 중인 컨테이너의 파일 시스템을 tar archive로 내보내기 → 백업, 마이그레이션
- image save와 다르게 이미지의 레이어에 대한 내용은 포함되지 않는다.
- 컨테이너 내 정보는 하나의 레이어로 통합된다.

```bash
cat [container.tar] | docker import - [image:tag]
```

- 이 방법으로 이미지를 가져온 경우 이미지에 cmd가 포함되어 있지 않아 import 시킨 이미지를 Dockerfile에 CMD를 추가하여 새로 빌드해야 이미지로 컨테이너를 실행시킬 수 있다.

```bash
docker import --change 'CMD ["command"]' [container.tar] [image:tag] 
```

- import 할 때부터 이미지에 커멘드를 추가하면 추가적인 조치없이 해당 이미지로 컨테이너를 실행 시킬 수 있다.