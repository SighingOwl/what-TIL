# Docker Network 관리
## 컨테이너 네트워크 이해

### Docker network

- Docker network는 기본적으로 Linux network를 따른다.
- 커널 네트워크 스택의 하위, 상위에는 네트워크 드라이버를 생성
- CNM(Container Networking Model)이라고 한다.
    - Docker 네트워크 전체를 구현할 수 있는 집합체
    - 실체로 구현하는 구현체는 libnetwork라고 있다.
        [libnetwork](https://github.com/moby/libnetwork)
        
    - OS 및 인프라에 구애 받지 않고 애플리케이션이 동일한 환경을 가질 수 있다.
- 리눅스 네트워킹 빌딩 블록
    - 리눅스 브릿지 네트워크
        - docker0 : 브릿지 네트워크, OSI 2 계층 - MAC 주소 사용
    - 네트워크 네임스페이스
        - 컨테이너 내부에 네트워크를 심기 위한 것
        - Sandbox 안에 ethernet, ip, mac 주소를 세팅
    - veth pair 및 iptable
        - veth
            - 호스트 브릿지 네트워크와 1:1 연결되기 위해 virtual ethernet이 필요
            - 컨테이너 내부에 virtual ethernet과 ethernet이 페어 되는 것이 veth pair
                
                ```bash
                brctl show
                bridge name	 bridge id		      STP enabled	 interfaces
                docker0		   8000.0242629d9af9	no		       vethc4efb3d
                ```
                
            - 두 네트워크 네임스페이스 사이의 연결선으로 동작하는 리눅스 네트워킹 인터페이스
            - 각 네임스페이스에 단일 인터페이스가 있는 전이중 링크
            - 도커 네트워크를 만들 때 도커 네트워크 드라이버는 veth를 사용하여 네임스페이스 간에 명시적인 연결 제공
            - 컨테이너가 도커 네트워크에 연결되면 veth 한쪽 끝은 컨테이너 내부에 배치되며(ethN) 다른 쪽은 도커 네트워크에 연결
            - 컨테이너의 네트워크(eth0)는 vethxxxxxx의 숫자보다 하나 작은 값을 가진다.
        - iptables
            - 컨테이너 내부에 기본적인 방화벽 역할을 iptable이 수행
            
            ```bash
            sudo iptables -t nat -L -n
            Chain DOCKER (2 references)
            target     prot opt source               destination
            RETURN     all  --  0.0.0.0/0            0.0.0.0/0
            DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:5000 to:172.17.0.2:5000
            ```
            
- Linux Bridge
    - 커널 내부의 물리적 스위치를 가상으로 구현한 OSI 2계층 장치
    - 트래픽을 검사하여 동적으로 학습되는 MAC 주소를 기반으로 트래픽을 전달
    - 기본 대역
        - 172.{17-31}.0.0/16
        - 192.168.{0-240}.0/20
- Network namespace
    - 커널에 격리된 네트워크 스택으로 자체 인터페이스, 라우트 및 방화벽 규칙 보유
    - 컨테이너와 리눅스의 보안적인 측면으로, 컨테이너를 격리하는데 사용
    - 네트워크 네임스페이스는 도커 네트워크를 통해 구성된 경우가 아니면 동일한 호스트의 두 컨테이너가 서로 통신하거나 호스트 자체와 통신할 수 없음을 보장
    - 일반적으로 CNM 네트워크 드라이버는 각 컨테이너에 대해 별도의 네임스페이스 구현
- Docker 컨테이너 네트워크 옵션 

    | 옵션 | 설명 |
    | --- | --- |
    | —add-host=[Host명:IP Address] | 컨테이너의 /etc/hosts에 Host 명과 IP Address를 설정 |
    | —dns=[IP Address] | DNS 서버의 IP Address를 설정 (/etc/resolv.conf) (168.126.63.1~3 / 8.8.8.8) |
    | —mac-address=[MAC Address] | 컨테이너의 MAC Address 설정 |
    | —expose=[포트 번호] | 컨테이너 내부에서 Host로 노출할 포트 번호 지정 |
    | —net=[bridge | none | host] | 컨테이너의 네트워크 설정 (bridge = docker0) |
    | -h, —hostname=”Host명” | 컨테이너의 Host명 설정 (default, container ID가 호스트명) |
    | -P, —publish-all=[true | false] | 컨테이너 내부의 노출된 포트를 호스트 임의 (32758~) 포트를 호스트와 연결 (명시적) |
    | —link=[container:container_id] | 동일 Host의 다른 컨테이너에서 액세스 시 이름 설정 → IP가 아닌 컨테이너의 이름을 사용해 통신 가능 |

- docker-proxy
    - 커널이 아닌 사용자 환경에서 host가 받은 패킷을 그대로 컨테이너의 port로 전달
    - Port를 외부로 노출하도록 설정하게 되면, docker host에는 docker-proxy라는 프로세스가 자동으로 생성
- overlay
    - 서로 다른 Host에서 서비스되는 컨테이너를 네트워크로 연결하는데 사용 되고, 이런 네트워크 생성을 위해 overlay network driver를 사용
    - 네트워크로 연결된 여러 docker host안에 있는 docker daemon 간의 통신을 관리하는 가상 네트워크
    - 컨테이너는 overlay network의 서브넷에 해당하는 IP 대역을 할당 받고, 받은 IP를 통해 상호간의 내부 통신을 수행
    - Docker swarm을 통해 구현 가능
    - docker network inspecr [network-id]로 조회 가능

## 사용자 정의 네트워크 생성과 조회

### 사용자 정의 docker network

- docker는 기본적으로 Host OS와 브릿지 연결을 하며, —net 옵션을 통해 네트워크 설정 가능
    | 옵션 항목 | 설명 |
    | --- | --- |
    | bridge | bridge 접속, 172.168.0.2~ |
    | none | 네트워크에 접속하지 않음 |
    | container:[container_name | id] | 다른 컨테이너 네트워크 사용 |
    | host | 컨테이너가 host os의 네트워크 사용 |
    | macvlan | 물리적 네트워크에 컨테이너 mac 주소를 통한 직접적 연결 구현 시 사용 |
    | NETWORK(사용자 정의 네트워크 이름) | 사용자 정의 네트워크 사용 |
- docker network create로 “사용자 정의 bridge network” 생성
- 사용자 정의 네트워크에 연결하면 Container는 Container의 이름이나 IP주소로 서로 통신 가능
- Overlay network나 커스텀 플러그인 사용시 mulit-host 연결 가능

### 사용자 정의 네트워크 생성

```bash
docker network create mynet
d31568620d0262c245f928604884f2181d4e0d7340bdb4f548aed4d76da8d858

docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
...
d31568620d02   mynet     bridge    local
...

route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
...
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-d31568620d02
...

ifconfig
br-d31568620d02: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:9e:b2:f6:18  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

- 사용자 지정 네트워크를 생성하면 네트워크를 확인할 수 있는 여러 명령에서 생성된 네트워크를 확인할 수 있다.

```bash
docker run --net=mynet -it --name=net-check1 ubuntu:22.04
```

- 생성한 사용자 네트워크를 사용하여 컨테이너 생성

```bash
"Containers": {
            "5b95e165b1bb8df3faaa072646fb6e4ecf68af630394c2039d755ff644498d42": {
                "Name": "net-check2",
                "EndpointID": "035d91b6e9abcc73ad62ec076d5b21415dd68ada3e7570567ad2f3c0dea1c2cb",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "b1629d556fc453dc4cdcb86e79e85b156d5518864911da130ba9c5678dcefd57": {
                "Name": "net-check1",
                "EndpointID": "b2919614e686717132a5f913e55a852d061583689c997f4641163b3b3b37e066",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
```

- docker network inspect로 사용자 생성 네트워크를 확인하면 이 네트워크를 사용하는 컨테이너를 확인 할 수 있다.

### 사용자 정의 네트워크 커스텀

```bash
docker network create \
--driver bridge \
--subnet 172.30.1.0/24 \    # CIDR 표기만 설정 가능
--ip-range 172.30.1.0/24 \   # subnet 이하, IP 범위 조정 가능
--gateway 172.30.1.1 \
vswitch-net
```

- —subnet, —ip-range, —gateway는 필수로 사용해야 한다.

### docker network connect | disconnect

- docker connect
    - 실행 중인 컨테이너에 새로운 네트워크 대역의 docker network를 연결
    
    ```bash
    # hostos
    docker network connect fc-net2 net-check2
    
    # net-check2 container
    ifconfig
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 172.18.0.3  netmask 255.255.0.0  broadcast 172.18.255.255
            ether 02:42:ac:12:00:03  txqueuelen 0  (Ethernet)
            RX packets 1859  bytes 29416992 (29.4 MB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 1334  bytes 77049 (77.0 KB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 172.19.0.2  netmask 255.255.0.0  broadcast 172.19.255.255
            ether 02:42:ac:13:00:02  txqueuelen 0  (Ethernet)
            RX packets 33  bytes 4476 (4.4 KB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 10  bytes 1130 (1.1 KB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 10  bytes 1130 (1.1 KB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    ```
    
    - 호스트 머신에서 컨테이너에 새로운 네트워크를 연결하면 컨테이너에서 새로운 네트워크를 함께 사용하는 것을 확인할 수 있다.
- docker disconnect
    - 실행 중인 컨테이너에 연결된 네트워크를 연결해제
    - 연결된 네트워크가 연결 해제가 되지 않은채 네트워크를 삭제하려고 하면 삭제가 되지 않는다.

## Docker DNS

### Docker DNS

- Docker 컨테이너는 IP를 사용자 정의 네트워크의 컨테이너 이름으로 자동 확인하는 DNS 서버가 Docker 호스트에 생성
    - Docker 기본 docker0 bridge driver에는 DNS가 포함되어 있지 않으므로 DNS는 내장된 docker0 bridge driver에서 작동하지 않는다.
- 동일 네트워크 alias 할당을 통해 하나의 타겟 그룹을 만들어 요청에 Round Robin 방식으로 응답
- 컨테이너 생성 시 호스트 시스템에서 다음 세 파일을 복사하여 컨테이너 내부에 적용하여 컨테이너 간에 이름으로 찾기 가능
    - /etc/hostname
    - /etc/hosts
    - /etc/resolv.conf

### libnetwork

- 핵심 네트워크 뿐만 아니라 서비스 검색 기능 제공을 통해 모든 컨테이너가 이름으로 서로를 찾을 수 있게 한다. —name or -net-alias 사용시 DNS 등록

### 네트워크 확인 실습

- 컨테이너 생성
    
    ```bash
    docker run -d --name=es1 --net=fc-net --net-alias=esnet-tg -p 9201:9200 -p 9301:9300 -e "discovery.type=single-node" elasticsearch:7.17.10
    docker run -d --name=es2 --net=fc-net --net-alias=esnet-tg -p 9202:9200 -p 9302:9300 -e "discovery.type=single-node" elasticsearch:7.17.10
    docker run -d --name=es3 --net=fc-net --net-alias=esnet-tg -p 9203:9200 -p 9303:9300 -e "discovery.type=single-node" elasticsearch:7.17.10
    ```
    
    - fc-net 사용자 정의 네트워크 사용
    - esnet-tg net alias 타겟 그룹 사용
- nslookup으로 컨테이너 확인
    
    ```bash
    docker run -it --rm --name=request-container --net=fc-net busybox nslookup esnet-tg
    Server:		127.0.0.11
    Address:	127.0.0.11:53
    
    Non-authoritative answer:
    
    Non-authoritative answer:
    Name:	esnet-tg
    Address: 172.20.0.2
    Name:	esnet-tg
    Address: 172.20.0.3
    ```
    
    - 타겟 그룹 안에 등록된 ip 확인 가능
    
    ```bash
    docker run -it --rm --name=request-container --net=fc-net busybox nslookup 172.20.0.2
    Server:		127.0.0.11
    Address:	127.0.0.11:53
    
    Non-authoritative answer:
    2.0.20.172.in-addr.arpa	name = es1.fc-net
    
    docker run -it --rm --name=request-container --net=fc-net busybox nslookup 172.20.0.3
    Server:		127.0.0.11
    Address:	127.0.0.11:53
    
    Non-authoritative answer:
    3.0.20.172.in-addr.arpa	name = es2.fc-net
    ```
    
    - 두 컨테이너가 같은 네트워크에서 동작 중 인것을 알 수 있다.
- 이것을 통해서 docker 자체가 docker DNS를 가지고 있다는 것을 확인할 수 있다.

### Round-Robin 실습

- centos 8에서 curl를 사용하여 네트워크 확인
    
    ```bash
    curl -s esnet-tg:9200
    {
      "name" : "7361d22a84e0",
      "cluster_name" : "docker-cluster",
      "cluster_uuid" : "OYXNcUljQj-kv1tI9RiADQ",
      "version" : {
        "number" : "7.17.10",
        "build_flavor" : "default",
        "build_type" : "docker",
        "build_hash" : "fecd68e3150eda0c307ab9a9d7557f5d5fd71349",
        "build_date" : "2023-04-23T05:33:18.138275597Z",
        "build_snapshot" : false,
        "lucene_version" : "8.11.1",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"
    }
    
    curl -s esnet-tg:9200
    {
      "name" : "7da69d24bd8b",
      "cluster_name" : "docker-cluster",
      "cluster_uuid" : "P6X1IEibS1abt0hqdjMGUw",
      "version" : {
        "number" : "7.17.10",
        "build_flavor" : "default",
        "build_type" : "docker",
        "build_hash" : "fecd68e3150eda0c307ab9a9d7557f5d5fd71349",
        "build_date" : "2023-04-23T05:33:18.138275597Z",
        "build_snapshot" : false,
        "lucene_version" : "8.11.1",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"
    }
    
    curl -s esnet-tg:9200
    {
      "name" : "88936737f9a6",
      "cluster_name" : "docker-cluster",
      "cluster_uuid" : "KBjHydydR62TwbIhmmSW4A",
      "version" : {
        "number" : "7.17.10",
        "build_flavor" : "default",
        "build_type" : "docker",
        "build_hash" : "fecd68e3150eda0c307ab9a9d7557f5d5fd71349",
        "build_date" : "2023-04-23T05:33:18.138275597Z",
        "build_snapshot" : false,
        "lucene_version" : "8.11.1",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"
    }
    ```
    
    - 타겟 그룹에 여러번 요청 하면 같은 타겟 그룹에 있는 컨테이너들이 round-robin 형식으로 응답한다.
        - 여러번 요청했을 때 서로 다른 컨테이너가 응답하는 것을 확인 할 수 있다.
        - 로드밸런서와 비슷한 효과

### docker DNS를 활용한 docker proxy 실습

1. 사용자 정의 브릿지 네트워크 생성
    
    ```bash
    docker network create \
    --driver bridge \
    --subnet 172.200.1.0/24 \
    --ip-range 172.200.1.0/24 \
    --gateway 172.200.1.1 \
    netlb
    ```
    
2. —net-alias를 이용한 타겟 그룹 생성
    
    ```bash
    docker run -itd --name=nettest1 --net=netlb --net-alias tg-net ubuntu:22.04
    docker run -itd --name=nettest2 --net=netlb --net-alias tg-net ubuntu:22.04
    docker run -itd --name=nettest3 --net=netlb --net-alias tg-net ubuntu:22.04
    ```
    
3. 등록된 DNS 등록 확인 - “dig (domain infomation grouping”  tool
    - dig를 사용하기 위해 dnsutils 패키지 설치가 필요
        
        ```bash
        apt-get install dnsutils
        ```
        
    - dig
        
        ```bash
        dig tg-net
        
        ; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> tg-net
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41563
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0
        
        ;; QUESTION SECTION:
        ;tg-net.				IN	A
        
        ;; ANSWER SECTION:
        tg-net.			600	IN	A	172.200.1.2
        tg-net.			600	IN	A	172.200.1.4
        tg-net.			600	IN	A	172.200.1.3
        
        ;; Query time: 0 msec
        ;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
        ;; WHEN: Wed Jan 17 09:38:57 UTC 2024
        ;; MSG SIZE  rcvd: 90
        ```
        
        - 요청을 tg-net에 보내면 응답하는 IP를 확인할 수 있다.
        - SERVER에서 DNS 서버의 주소를 확인할 수 있다.
        

## 컨테이너 proxy

### No config proxy

- Proxy 구성이 없으면 사용자의 요청은 직접 웹서버에 전달되어 서버 부담이 가중된다.
- 단일 웹서버 구성은 장애 발생 시 서비스 가용서에 치명적
- 다중 웹서버 구성으로 여러 사용자의 요청을 처리할 경우에도 요청한 부하를 적절히 분산시켜 주지 못하면 한 서버에 부하가 몰리는 Hotspot이 발생하는 등의 문제가 발생
- 최종 사용자 관점의 응답시간 만족을 얻기 힘들다.

### Proxy

- 요청자와 응답자 간의 중계 역할, 통신을 대리 수행하는 서버를 proxy server
- Proxy server의 위치에 따라 forward proxy, reverse proxy로 구분한다.
- 사용 사례
    - 다중 호스트(cluster)를 활용하여 트래픽 분산을 할 수 있는 Load Balancing 구현 - reverse
    - 클라이언트 요청 중 자주 사용되는 HTML, JS, CSS 등의 정적 파일을 caching 하여 서버 부하  및 네트워크 트래픽을 줄여 빠른 응답을 할 수 있는 캐시 서버 구현 - forward, reverse
    - 학교 및 사내망에서 보안을 위한 특정 사이트 접근 차단 구현 - forward
    - 무중단 배포를 통한 서비스 지속 - reverse
    - SSL 암호화 적용으로 보안 강화 - reverse
- Forward proxy
    - 클라이언트와 인터넷 사이에 있어서 client 정보가 서버에 노출 되지 않음
- reverse proxy
    - 클라이언트 요청을 서버 대신 받아서 전달
    - 클라이언트에게 서버 노출 안됨

### Nginx

- 기본 구성 값으로 web server를 실행. 동일 계열 점유율이 가장 높다.
- 추가 구성으로 “reverse proxy” 구현이 가능
- Kubernetes의 ingress controller로 “nginx ingress controller” 선택이 가능
- API 트래픽 처리를 고급 HTTP 처리 기능으로 사용 가능한 API Gateway 구성이 가능
- MSA 트래픽 처리를 위한 MicroGateway로 사용 가능
- 설정은 /etc/nginx 하위에 nginx.conf 변경을 통해 구성 가능
    
    ### Ngix reverse proxy
    
    - 클라이언트 요청이 80포트로 들어오면 준비해둔 애플리케이션의 서버의 주소로 각 서버로 트래픽을 분배
    - 기본 분배 방식(LoadBalancing)은 round-robin을 사용
    - 요청이 적은 서버로 분배하는 least_conn 방식
    - IP당 서버를 분배하는 ip_hash 등 여러가지 부하 분산 알고리즘을 사용 가능

### HAproxy

- 하드웨어 기반의 L4/L7 스위치를 대체하기 위한 오프소스하소프트웨어 솔루션
- TCP 및 HTTP 기반 애플리케이션을 위한 고가용성(Active-Passive), Load Balancing 및 프록시 기능을 제공하는 매우 빠르고 안정적인 무료 reverse proxy
- 주요 기능
    - SSL
    - Load Balancing
    - Active health check
    - KeepAlived (proxy 이중화)
        - Active으로 트래픽을 보내다가 문제가 발생하면 Passive를 Active로 변경하여 사용
- L4
    - IP를 이용한 트래픽 전달이 특징이다.
    - Haproxy L4 구성 시 IP와 Port를 기반으로 사용자 요청 트래픽을 전달하도록 구성
    - 요청에 대한 처리는 웹서버로 구성된 web1~3에 round-robin 방식으로 부하 분산 된다.
- L7
    - HTTP 기반의 URL를 이용한 트래픽 전달이 특징
    - 동일한 도메인의 하위에 존재하는 여러 웹 애플리케이션 서버를 사용 가능
    - [example.com/item](http://example.com/item) or [example.com/basket](http://example.com/basket)으로 연결된다.
    - 사용자의 요청과 설정에 따른 부하 분산

## Nginx를 활용한 컨테이너 proxy

### 실습 1

- nginx 설치
    
    ```bash
    sudo apt-get install nginx
    ```
    
- nginx 컨테이너 실행 - 실습용
    
    ```bash
    docker run -it -d -e SERVER_ROOT=5001 -p 5001:5001 -h alb-node01 -u root --name=alb-node01 dbgurum/nginxlb:1.0
    docker run -it -d -e SERVER_ROOT=5002 -p 5002:5002 -h alb-node02 -u root --name=alb-node02 dbgurum/nginxlb:1.0
    docker run -it -d -e SERVER_ROOT=5003 -p 5003:5003 -h alb-node03 -u root --name=alb-node03 dbgurum/nginxlb:1.0
    ```
    
    - 실습용 이미지가 docker 1.24 버전 이전에 만들어진 버전이어서 지금 버전에서 제대로 작동하지 않는 문제가 있다.
    - 솔루션
        
        ```bash
        sudo sed -i '/&GRUB_CMDLINE_LINUX/ s/"$/ systemd.unified_cgroup_hierarchy=0"/' /etc/default/grub
        sudo update-grub
        ```
        
        - Docker의 cgroup 버전을 1로 낮춘다.
        - /etc/default/grub에서 직접 systemd문구를 추가해도 된다.
- ngin.conf 새로작성
    
    ```bash
    events { worker_connections 1024; }
    http {
        #List of application servers
        upstream backend-alb {
            server 127.0.0.1:5001;
            server 127.0.0.1:5002;
            server 127.0.0.1:5003;
        }
        # Configuration for the server
        server {
            # Running port
            listen 80 default_server;
            # Proxying the connections
            location / {
                proxy_pass      http://backend-alb;
            }
        }
    }
    ```
    
    - nginx가 기본적으로 수행하는 웹 서버 기능에서 reverse proxy 기능을 하도록 수행
    - 이 상태에서 브라우저에 192.168.56.101 접속후 새로고침을 하면 연결되는 컨테이너가 계속 바뀌는 것을 확인 할 수 있다.

### 실습 2 - nginx container가 reverse proxy로 동작

- proxy-controller 생성
    
    ```bash
    docker run -d -p 8001:80 --name=proxy-container nginx:1.25.0-alpine
    ```
    
- nginx.conf 수정
    
    ```bash
    events { worker_connections 1024; }
    http {
        #List of application servers
        upstream backend-alb {
            server 192.168.56.101:5001;
            server 192.168.56.101:5002;
            server 192.168.56.101:5003;
        }
        # Configuration for the server
        server {
            # Running port
            listen 80 default_server;
            # Proxying the connections
            location / {
                proxy_pass      http://backend-alb;
            }
        }
    }
    ```
    
    - 컨테이너의 서버 주소를 호스트의 ip로 수정
- proxy-controller에 수정된 nginx.conf 복사
    
    ```bash
    docker cp nginx.conf proxy-controller:/etc/nginx/nginx.conf
    ```
    
- 호스트 nginx가 reverse proxy로 동작할 때와 동일한 기능을 보인다.

### 실습 3 - 가중치 추가

- nginx.conf 수정
    
    ```bash
    events { worker_connections 1024; }
    http {
        #List of application servers
        upstream backend-alb {
            server 192.168.56.101:5001 weight=60;
            server 192.168.56.101:5002 weight=20;
            server 192.168.56.101:5003 weight=20;
        }
        # Configuration for the server
        server {
            # Running port
            listen 80 default_server;
            # Proxying the connections
            location / {
                proxy_pass      http://backend-alb;
            }
        }
    }
    ```
    
    - server 가장 뒤에 가중치를 추가한다.
    - 가중치 비율만큼 트래픽을 보낸다.

## HAproxy를 활용한 컨테이너 proxy

### 실습 1 - 기본 mode http 방식

- 사용자 정의 네트워크 생성
    
    ```bash
    docker network create proxy-net
    ```
    
- 사용자 정의 네트워크를 사용하는 컨테이너 생성
    
    ```bash
    docker run -d --name=echo-web1 --net=proxy-net -h echo-web1 dbgurum/haproxy:echo
    docker run -d --name=echo-web2 --net=proxy-net -h echo-web2 dbgurum/haproxy:echo
    docker run -d --name=echo-web3 --net=proxy-net -h echo-web3 dbgurum/haproxy:echo
    ```
    
- haproxy.cfg 작성
    
    ```bash
    global
      stats socket /var/run/api.sock user haproxy group haproxy mode 660 level admin expose-fd listeners
      log stdout format raw local0 info
    
    defaults
      mode http
      timeout client 10s
      timeout connect 5s
      timeout server 10s
      timeout http-request 10s
      log global
    
    frontend stats
      bind *:8404
      stats enable
      stats uri /
      stats refresh 10s
    
    frontend myfrontend
      bind :80
      default_backend webservers
    
    backend webservers
      server s1 echo-web1:8080 check
      server s2 echo-web2:8080 check
      server s3 echo-web3:8080 check
    ```
    
    - default 하위의 mode에서 모드를 설정 가능
        - http : L7
        - tcp : L4
    - frontend stats
        - 자체적으로 트래픽 통계를 수집한다.
        - 기본으로 8404 포트를 사용
    - frontend myfrontend
        - 80번 포트를 사용
        - 80번 포트로 들어오는 트래픽을 webservers로 전달
    - backend webservers
        - backend로 사용할 webserver를 지정
- haproxy container 실행
    
    ```bash
    docker run -d --name=haproxy-container --net=proxy-net -p 80:80 -p 8404:8404 -v $(pwd):/usr/local/etc/haproxy:ro haproxytech/haproxy-alpine:2.5
    ```
    
    - webserver를 위한 80:80 포트 매핑과 트래픽 통계 수집을 위한 8404:8404 포트 매핑 설정
    - 통계 수집을 위한 볼륨 설정
- 브라우저에서 80번 포트로 접속한 후 새로고침을 반복하면 다른 컨테이너로 접속하는 것을 확인할 수 있다.

### 실습 2 - L7 URL 방식 - 1

- haproxy.cfg 수정
    
    ```bash
    global
      stats socket /var/run/api.sock user haproxy group haproxy mode 660 level admin expose-fd listeners
      log stdout format raw local0 info
    
    defaults
      mode http
      timeout client 10s
      timeout connect 5s
      timeout server 10s
      timeout http-request 10s
      log global
    
    frontend stats
      bind *:8404
      stats enable
      stats uri /
      stats refresh 10s
    
    frontend myfrontend
      bind :80
      default_backend webservers
    
      acl path-echo-web1 path_beg /echo-web1
      acl path-echo-web2 path_beg /echo-web2
      acl path-echo-web3 path_beg /echo-web3
    
      use_backend echo-web1_backend if path-echo-web1
      use_backend echo-web2_backend if path-echo-web2
      use_backend echo-web3_backend if path-echo-web3
    
    backend webservers
      balance roundrobin
      server s1 echo-web1:8080 check
      server s2 echo-web2:8080 check
      server s3 echo-web3:8080 check
    
    backend echo-web1_backend
      server s1 echo-web1:8080 check
    
    backend echo-web2_backend
      server s2 echo-web2:8080 check
    
    backend echo-web3_backend
      server s3 echo-web3:8080 check
    ```
    
    - frontend myfrontend
        - acl <frontend> path_bet /<server_URL>
            - 접근 제어 방식
            - 특정 URL로 접근하면 정해진 frontend로 트래픽 전달
        - use_backend <backend> if <frontend>
            - 특정 frontend로 트래픽이 전달되면 정해진 backend로 트래픽 전달
- 수정한 haproxy.cfg를 적용한 컨테이너 생성
- 기본 80번 포트로 접속하면 기존과 동일하게 다른 컨테이너 돌아가면서 접속하는 것을 확인할 수 있지만 맨뒤에 특정 url을 사용해서 접속하면 해당 url과 연결된 컨테이너만 연결된다.

### 실습 3 - L7 URL 방식 -2

- 하위 도메인에서 하나의 컨테이너가 아닌 여러 컨테이너를 사용한 로드밸런싱을 수행
- haproxy.cfg 수정
    
    ```bash
    global
      stats socket /var/run/api.sock user haproxy group haproxy mode 660 level admin expose-fd listeners
      log stdout format raw local0 info
    
    defaults
      mode http
      timeout client 10s
      timeout connect 5s
      timeout server 10s
      timeout http-request 10s
      log global
    
    frontend stats
      bind *:8404
      stats enable
      stats uri /
      stats refresh 10s
    
    frontend myfrontend
      bind :80
      default_backend webservers
    
      acl echo-web1-item path_beg /item
      acl echo-web2-item path_beg /item
      acl echo-web3-basket path_beg /basket
      acl echo-web4-basket path_beg /basket
    
      use_backend echo-web1_backend if echo-web1-item
      use_backend echo-web1_backend if echo-web2-item
      use_backend echo-web2_backend if echo-web3-basket
      use_backend echo-web2_backend if echo-web4-basket
    
    backend webservers
      balance roundrobin
      server s1 echo-web1-item:8080 check
      server s2 echo-web2-item:8080 check
      server s3 echo-web3-basket:8080 check
      server s4 echo-web3-basket:8080 check
    
    backend echo-web1_backend
      server s1 echo-web1-item:8080 check
      server s2 echo-web2-item:8080 check
    
    backend echo-web2_backend
      server s3 echo-web3-basket:8080 check
      server s4 echo-web4-basket:8080 check
    ```
    
    - frontend myfrontend
        - /item url로 들어오는 트래픽은 1, 2번 frontend로 전달
        - /basket url로 들어오는 트래픽은 3, 4번 frontend로 전달
        - 1, 2번 frontend로 들어오는 트래픽은 1번 backend로 전달
        - 3, 4번 frontend로 들어오는 트래픽은 2번 backend로 전달
    - backend echo-web1_backend, backend echo-web2_backend
        - backend 서버를 2개씩 그룹을 만든다.
        - 그룹 내부에서 로드밸런싱이 이루어진다.
- docker container 생성 - 서버
    
    ```bash
    docker run -d --name=echo-web1-item --net=proxy-net -h echo-web1-item dbgurum/haproxy:echo
    docker run -d --name=echo-web2-item --net=proxy-net -h echo-web2-item dbgurum/haproxy:echo
    docker run -d --name=echo-web3-basket --net=proxy-net -h echo-web3-basket dbgurum/haproxy:echo
    docker run -d --name=echo-web4-basket --net=proxy-net -h echo-web4-basket dbgurum/haproxy:echo
    ```
    
- 특정 url로 접속할 때 해당 그룹 안에서 로드밸런싱이 이루어지는 것을 확인할 수 있다.