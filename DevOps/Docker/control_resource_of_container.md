# 컨테이너 자원 제어
docker를 사용해서 컨테이너가 소모하는 시스템 자원을 제한할 수 있다. 컨테이너를 하나만 실행할 때는 자원 제한을 하지 않아도 큰 문제가 없지만 다중 컨테이너 실행 환경에서는 컨테이너들이 필요한 자원을 충분히 할당받지 못하는 성능 문제가 발생한다. 그러므로 컨테이너에 적절한 자원 할당을 통해 서비스가 일관된 성능을 낼 수 있도록 해야한다.


## CPU 자원 소비 제어

### 컨테이너 CPU 리소스에 대한 런타임 제한

- CPU 자원을 어떤 프로세스에 얼마나 할당하는지를 정책으로 만드는 것을 CPU 스케줄링이라고 한다.
- CPU 사용량 제한을 위해 CFS(Completely Fair Scheduler) 스케줄러를 사용한다.
    - 모든 프로세스가 공평하게 CPU 사용 시간을 제공 받도록 하는 OS 알고리즘
- 옵션
    - —cpu-shares
        - 컨테이너가 사용할 수 있는 CPU 사용 시간에 대한 가중치를 설정
        - 기본값 1024, 2048 설정 시 다른 컨테이너에 비해 2배의 사용 시간 할당
    - —cpuset-cpus
        - 보유한 CPU core 번호를 지정하여 컨테이너가 해당 core만 사용하도록 설정
        - 0 / 1-2 / 0, 2, 3 으로 지정 가능
    - —cpus
        - 컨테이너가 사용할 수 있는 CPU 사용 비율 지정
        - —cpus=0.25 설정 시 지정 된 core 수의 25% 사용 가능

### CPU time scheduling, —cpu-shares

- 가중치 적용은 많은 프로세스가 동시에 CPU를 사용하게 될 때 자동으로 적용되면 그렇지 않은 경우에는 적용되지 않는다.
- CPU 사용 시간 가중치가 1024, 512인 컨테이너를 하나씩 생성
    
    ```bash
    docker run -d --name=cpu_1024 --cpu-shares 1024 leecloudo/stress:1.0 stress --cpu 4
    docker run -d --name=cpu_512 --cpu-shares 512 leecloudo/stress:1.0 stress --cpu 4
    ```
    
- ps로 CPU 사용량 확인
    
    ```bash
    ps -auxf | grep stress
    root        6202  0.0  0.0   7488  1536 ?        Ss   18:35   0:00  \_ stress --cpu 4
    root        6232 71.9  0.0   7488   128 ?        R    18:35   0:57      \_ stress --cpu 4
    root        6233 70.8  0.0   7488   128 ?        R    18:35   0:56      \_ stress --cpu 4
    root        6234 70.4  0.0   7488   128 ?        R    18:35   0:56      \_ stress --cpu 4
    root        6235 69.1  0.0   7488   128 ?        R    18:35   0:55      \_ stress --cpu 4
    root        6332  0.0  0.0   7488  1536 ?        Ss   18:36   0:00  \_ stress --cpu 4
    root        6362 32.1  0.0   7488   128 ?        R    18:36   0:19      \_ stress --cpu 4
    root        6363 31.3  0.0   7488   128 ?        R    18:36   0:19      \_ stress --cpu 4
    root        6364 31.6  0.0   7488   128 ?        R    18:36   0:19      \_ stress --cpu 4
    root        6365 32.3  0.0   7488   128 ?        R    18:36   0:19      \_ stress --cpu 4
    ```
    
    - CPU 사용 시간을 확인하면 가중치가 1024인 컨테이너가 512인 컨테이너에 비해 더 많은 사용시간을 가져가는 것을 확인할 수 있다.
- Cadvisor 모니터링 확인        
    <img src="/images/control_resource_of_container_1.png" width="50%" height="50%" title="control resources 1" alt="control resources 1">      
    

### CPU 지정, —cpuset-cpus

- 사용할 CPU 번호를 지정하여 컨테이너 실행
    
    ```bash
     docker run -d --name cpuset_1 --cpuset-cpus=2 leecloudo/stress:1.0 stress --cpu 1
    ```
    
- htop 확인     
    <img src="/images/control_resource_of_container_2.png" width="50%" height="50%" title="control resources 2" alt="control resources 2">      
    
    - 지정한 2번 core의 사용량이 100%인 것을 확인
- cadvisor 확인     
    <img src="/images/control_resource_of_container_3.png" width="50%" height="50%" title="control resources 3" alt="control resources 3">      
    
    - htop에서 확인한 것과 동일하게 2번 core의 사용량만 증가한 것을 확인

### CPU 사용 비율 지정, —cpus

- stress 컨테이너 실행 후 docker update를 사용하여 cpu 사용 비율을 조정
    
    ```bash
    
    docker run -d --name=cpuset_1 --cpuset-cpus=2 leecloudo/stress:1.0 stress --cpu 1
    docker update --cpus=0.2 cpuset_1
    ```
    
- 사용 비율 조정 전     
    <img src="/images/control_resource_of_container_4.png" width="50%" height="50%" title="control resources 4" alt="control resources 4">      
    
    - 2번 core 사용량이 100%까지 올라가는 것을 확인
- 사용 비율 조정 전     
    <img src="/images/control_resource_of_container_5.png" width="50%" height="50%" title="control resources 5" alt="control resources 5">      
    
    - 20% 내외로 사용량이 조정이 된 것을 확인
- 두 개 이상 코어를 사용하는 stress 컨테이너 생성 후 사용 비율 조정
    
    ```bash
    
    docker run -d --name=cpuset_2 --cpuset-cpus=0,3 leecloudo/stress:1.0 stress --cpu 2
    docker update --cpus=0.2 cpuset_1
    ```
    
- 2개 이상 코어를 사용하는 컨테이너의 사용 비율 조정은 core 합산 사용량을 지정하는 것       
    <img src="/images/control_resource_of_container_6.png" width="50%" height="50%" title="control resources 6" alt="control resources 6">      
    
    - 0, 3 core의 사용 비율이 20% 이내 인것을 확인할 수 있다.

## Memory 자원 소비 제어

### 컨테이너 Memory 리소스에 대한 런타임 제한

- Docker hostos의 총 메모리 양과 작업에 사용될 예상 메모리 크기를 사전에 파악하여 메모리 최적화를 유지해야 한다.
- 특정 컨테이너가 과도한 메모리 사용 시 메모리 부족(OOM, out of memory)으로 인해 프로세스, 컨테이너의 예기치 못한 강제 종료가 발생할 수 있다.
    - OOM Killer가 프로세스를 Kill하지 못하도록 보호 : —oom-kill-disable
- 컨테이너들의 과도한 메모리 사용으로 인해 docker daemon이 커널에 의해 강제 종료되면 전체 컨테이너 서비스들에 영향을 주게 된다.
- Docker hostos의 메모리 사용량 확인과 컨테이너의 메모리 사용 제한이 중요
    - —memory, -m
        - (hard limit) 컨테이너가 사용하는 최대 메모리 사용량 제한
        - 설정 값을 초과해서 사용하면 OOM 발생, 최소 6MB
    - —memory-reservation
        - (soft limit) Docker가 contention을 감지하거나 hostos의 메모리 가용율이 현저히 떨어지는 경우 활성화되어 최소한의 보장 값으로 사용
        - (soft limit) -m=1g —memory-reservation=500m (최대 1g 사용이 가능하고, 적어도 500m 사용 보장)
    - —memory-swap
        - 컨테이너가 사용할 수 있는 swap memory 사용량 제한 (-1은 무제한)
        - -m=300m 설정 시 자동으로 —memory-swap=600m 설정됨.
        - 전체 600m에서 -m 값을 뺀 나머지만큼 swap이 사용된다는 의미

### memory hard limit, —memory, -m

- 메모리 크기를 제한한 컨테이너를 실행
    
    ```bash
    docker run -d --memory=1g --name=nginx_mem_1g nginx
    ```
    
- docker inspect로 할당된 메모리 크기 확인
    
    ```bash
    docker inspect nginx_mem_1g | grep -i memory
                "Memory": 1073741824,
                "MemoryReservation": 0,
                "MemorySwap": 2147483648,
                "MemorySwappiness": null,
    ```
    
    - 1g가 할당된 것을 확인할 수 있다.
    - swap을 확인하면 할당된 1g와 스왑 영역 1g가 합쳐진 용량이 표시된다.
- cadvisor      
    <img src="/images/control_resource_of_container_7.png" width="50%" height="50%" title="control resources 7" alt="control resources 7">      
    
    - cadvisor에서 해당 컨테이너의 메모리가 1.00GB로 제한된 것을 확인할 수 있다.

### swap memory limit, —memory-swap

- 컨테이너의 스왑 영역 크기를 제한한 컨테이너 생성
    
    ```bash
    docker run -m=200m --memory-swap=300m -itd --name=mem-test ubuntu:22.04
    ```
    
    - 메모리 할당이 200MB, swap 영역이 100MB인 컨테이너 생성
- docker inspect로 확인
    
    ```bash
    docker inspect mem-test | grep -i memory
    						"Memory": 209715200,
                "MemoryReservation": 0,
                "MemorySwap": 314572800,
                "MemorySwappiness": null,
    ```
    
    - MemorySwap이 300MB인 것을 확인 가능

### docker update

- docker 컨테이너의 리소스는 docker update를 통해서 바로 적용이 가능한 부분이다.
- 컨테이너 동작에 부족한 메모리를 설정한 컨테이너 실행 및 확인
    
    ```bash
    docker run -itd --memory=6m --name=mydb -e MYSQL_ROOT_PASSWORD=pass123# mysql:5.7-debian
    
    docker ps -a
    CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS                      PORTS                                       NAMES
    cfe6da27ce1e   mysql:5.7-debian                  "docker-entrypoint.s…"   12 seconds ago   Exited (1) 11 seconds ago                                               mydb
    
    docker logs mydb
    2024-01-18 12:59:15+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.42-1debian10 started.
    2024-01-18 12:59:15+00:00 [ERROR] [Entrypoint]: mysqld failed while attempting to check config
    	command was: mysqld --verbose --help --log-bin-index=/tmp/tmp.Yp41CbZ9Xt
    ```
    
    - 메모리가 6MB인 MySQL 컨테이너가 실행되지 못하고 곧바로 exit된 것을 확인할 수 있다.
    - log에는 mysqld가 실행되지 못했다는 것만 나와있지만 설정된 메모리 크기로 추측하였을 때 메모리 부족으로 실행 종료가 된것을 알 수 있다.
        - mysql은 최소 200MB 정도의 메모리를 필요로 한다.
- 리소스 업데이트
    
    ```bash
    docker update --memory=300m --memory-swap=600m mydb
    ```
    
    - 메모리 할당을 300MB로 재설정
- 리소스 업데이트 후 컨테이너를 시작하면 정상 동작하는 것을 확인할 수 있다.

### memory 과부하 테스트

- stress를 사용해서 메모리에 과부하 테스트 실행
    
    ```bash
    docker run -it --rm --memory=200m --memory-swap=200m leecloudo/stress:1.0 stress --vm 1 --vm-bytes 250m -t 10s
    stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
    stress: FAIL: [1] (415) <-- worker 8 got signal 9
    stress: WARN: [1] (417) now reaping child worker processes
    stress: FAIL: [1] (421) kill error: No such process
    stress: FAIL: [1] (451) failed run completed in 0s
    ```
    
    - 요청 메모리의 크기가 할당된 메모리보다 크고 swap 영역이 없어 stress 프로세스 동작이 실패
    
    ```bash
    docker run -it --rm --memory=200m --memory-swap=200m leecloudo/stress:1.0 stress --vm 1 --vm-bytes 150m -t 10s
    stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
    stress: info: [1] successful run completed in 10s
    ```
    
    - stress 테스트로 설정한 메모리 할당량보다 적은 메모리를 요청했을 때
    - 문제 없이 통과하는 것을 확인할 수 있다.
    
    ```bash
    docker run -it --rm --memory=200m leecloudo/stress:1.0 stress --vm 1 --vm-bytes 350m -t 10s
    stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
    stress: info: [1] successful run completed in 10s
    ```
    
    - memory swap을 설정하지 않아 기본적으로 200m의 영역이 추가로 존재해 할당된 메모리보다 많은 메모리량을 stress 테스트해도 정상 동작 했다.
    
    ```bash
    docker run -it --rm --memory=200m leecloudo/stress:1.0 stress --vm 1 --vm-bytes 400m -t 10s
    stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
    stress: FAIL: [1] (415) <-- worker 7 got signal 9
    stress: WARN: [1] (417) now reaping child worker processes
    stress: FAIL: [1] (421) kill error: No such process
    stress: FAIL: [1] (451) failed run completed in 0s
    ```
    
    - 할당된 메모리와 swap 영역의 메모리 양을 합친 것 이상으로 메모리 테스트 시 실패한다.

## Disk 자원 소비 제어

### 컨테이너 Disk 리소스에 대한 런타임 제한

- Docker image는 기본적으로 dockr host의 공간을 사용하므로 지속적인 사용량 관찰이 요구된다.
- 컨테이너 I/O 제한을 설정하지 않으면 컨테이너 내부 I/O bandwidth 제한이 설정되지 않기 때문에 옵션을 통해 Block I/O 제한이 필요하다.
- Direct I/O의 경우에만 Block I/O가 제한되며 Bufferd I/O는 해당되지 않는다.
- 디스크 Block I/O 작업에 대한 제한은 디스크 성능 지표인 IOPS와 MBPS에 따른다.
    - —blkio-weight, —blkio-weight-device
        - Block I/O의 할당량(Quota)을 10~1000으로 설정
        - 기본값 500
    - —devic-read-bps, —device-write-bps
        - 특정 device에 MBPS를 제한한다.
        - 초당 Block throughput을 의미, b, kb, mb, gb 단위로 제한
    - —device-read-iops, -device-write-iops
        - 특정 device에 IOPS를 제한
        - 초당 Block I/O 횟수를 의미, 0 이상의 정수로 표기
- Disk I/O 측정 도구, iostat
    - sysstat 패키지 설치 필요
        
        ```bash
        sudo apt-get install sysstat
        ```
        

### MBPS(Mege Byte per Second)

- ubuntu 컨테이너에서 진행
    - dd 명령어 실행
        
        ```bash
        dd if=/dev/zero of=blkmb.out bs=1M count=10 oflag=direct
        10+0 records in
        10+0 records out
        10485760 bytes (10 MB, 10 MiB) copied, 0.0118019 s, 888 MB/s
        ```
        
        - /dev/zero 디렉토리에 block i/o를 실행
        - block size 1m, 10개
        - direct i/o 사용
        - 제한 없이 사용했을 때 초당 888 MB/s의 성능이 나왔다.
- 쓰기 성능이 제한된 ubuntu 컨테이너 실행
    
    ```bash
    docker run -it --rm --device-write-bps /dev/sdb:1mb ubuntu:22.04 bash
    ```
    
    - dd 명령어 실행
        
        ```bash
        dd if=/dev/zero of=blkmb.out bs=1M count=10 oflag=direct
        10+0 records in
        10+0 records out
        10485760 bytes (10 MB, 10 MiB) copied, 10.0053 s, 1.0 MB/s
        ```
        
        - 쓰기 성능이 1.0MB/s로 제한되어 제한이 없을 때에 비해 시간이 오래 걸렸다.

### IOPS

- ubuntu 컨테이너에서 진행
    - dd 명령어 실행
        
        ```bash
        dd if=/dev/zero of=blkmb.out bs=1M count=10 oflag=direct
        10+0 records in
        10+0 records out
        10485760 bytes (10 MB, 10 MiB) copied, 0.0135619 s, 773 MB/s
        ```
        
        - 제한 없이 사용했을 때 초당 773 MB/s의 성능이 나왔다.
- IOPS가 제한된 ubuntu 컨테이너 실행
    
    ```bash
    docker run -it --rm --device-write-iops /dev/sdb:10 ubuntu:22.04 bash
    ```
    
    - dd 명령어 실행
        
        ```bash
        dd if=/dev/zero of=blkmb.out bs=1M count=10 oflag=direct
        10+0 records in
        10+0 records out
        10485760 bytes (10 MB, 10 MiB) copied, 1.90569 s, 5.5 MB/s
        ```
        
        - 초당 I/O 성능을 10으로 제한했을 때 초당 5.5MB/s의 성능으로 제한된다.