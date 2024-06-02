# EC2 배치 그룹
EC2 배치 그룹은 워크로드에 따라 상호 의존적인 EC2 인스턴스를 그룹으로 묶어 인프라에 배치하는 것을 의미합니다. 세 가지 배치 전략이 있으며 워크로드의 요구사항에 적절한 전략을 선택하여 배치합니다.

## 배치 전략

| 전략 | 설명 |
| --- | --- |
| Cluster - 클러스터 | 단일 가용 영역 내에서 지연 시간이 짧은 하드웨어 설정으로 인스턴스를 그룹화. 높은 성능을 제공하지만 높은 위험성도 가짐. |
| Spread - 분산 배치 그룹 | 서로 다른 하드웨어에 인스턴스가 분산 배치. 가용 영역별 배치 그룹은 7개의 인스턴스만 가질 수 있다는 제한 사항이 존재. 크됨리티컬 애플리케이션이 있는 경우 사용. |
| Partition - 분할 배치 그룹 | 분산 배치 그룹과 비슷하게 인스턴스를 분산시키는 전략. 인스턴스가 여러 파티션에 거쳐 배치되며 파티션은 가용 영역 내의 서로 다른 하드웨어 렉 세트에 의존. 인스턴스 자체가 다른 실패로부터 격리되지는 않지만 파티션만큼은 오류가 발생한 파티션과 격리. 파티션 하나당 수백개의 인스턴스로 확장될 수 있고 Hadoop, Cassandra, Kafka와 같은 애플리케이션을 실행 가능. |

### Cluster

<img src="/images/AWS_placement_group_1.png" width="50%" height="50%" title="aws placement group 1" alt="aws placement group 1">    

- 클러스터는 모든 인스턴스가 같은 가용영역의 같은 렉에 존재한다.
- 10GB 속도 정도의 매우 짧은 지연 시간을 가진 네트워크를 원할 때 사용한다.
- 렉에서 실패나 오류가 발생하면 모든 EC2 인스턴스가 다운되며 전체 스택에 걸쳐 실패가 전파될 위험이 높다.
- Use Cases:
    - 빨리 완료되어야 하는 빅데이터 분석
    - 높은 네트워크 처리량을 요구하는 애플리케이션 실행

### Spread

<img src="/images/AWS_placement_group_2.png" width="50%" height="50%" title="aws placement group 2" alt="aws placement group 2">    

- 분산 배치 그룹은 실패의 위험성을 최소화하는데 집중한다.
- 모든 EC2 인스턴스가 서로 다른 하드웨어에 위치한다.
- 장점
    - 여러 가용영역에 걸쳐 있어 동시 실패의 위험을 줄일 수 있다.
    - 인스턴스가 다른 인스턴스와 물리적으로 분리되어 있다.
- 단점
    - 배치 그룹당 7개의 인스턴스로 제한된다.
- Use Cases
    - 가용성을 극대화하고 위험성을 줄여야하는 애플리케이션
    - 인스턴스 오류를 서로 격리해야하는 크리티컬 애플리케이션

### Partition

<img src="/images/AWS_placement_group_3.png" width="50%" height="50%" title="aws placement group 3" alt="aws placement group 3">    

- 분할 배치 그룹은 여러 가용 영역의 파티션에 인스턴스를 분산할 수 있으며 가용 영역 당 최대 7개의 파티션을 가질 수 있다.
- 파티션에는 수백개의 EC2 인스턴스가 있을 수 있다.
- 파티션은 다른 파티션과 물리적으로 분리되어 다른 파티션의 실패나 오류에서 격리된다.
- EC2 인스턴스가 위치한 파티션을 알기 위해 메타데이터 서비스를 이용하여 액세스하는 옵션이 있다.
- Use Cases
    - 파티션들 전반에 걸쳐 데이터와 서버를 퍼뜨려 두도록 파티션 인식 가능한 애플리케이션
    - HDFS Hbase, Cassandra, Apache Kafka

### Create Placement Group

<img src="/images/AWS_placement_group_4.png" width="50%" height="50%" title="aws placement group 4" alt="aws placement group 4">    

1. 배치 그룹 이름과 전략을 선택 후 생성할 수 있다.
2. 인스턴스를 생성시 고급 세부 정보에서 배치 그룹을 선택 후 생성해야 배치 전략에 맞는 인스턴스 생성을 할 수 있다.