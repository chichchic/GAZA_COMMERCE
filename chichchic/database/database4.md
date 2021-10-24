# B트리

정확한 일치, 범위 조건, 정렬 등 다양한 쿼리를 용이하게 처리할 수 있다.
키가 추가되거나 삭제되어도 밸런스를 유지할 수 있다.
mongodb에서 8kb를 할당할 수 있지만 평균 30byte 크기이다.
ordered index이다.

**ordered index**
dense Index files: 모든 search key값에 record가 존재하는 file로 바로 검색이 가능하다.
sparse index files: 일부의 index만을 구성해서 search key값을 만든다. 해당 index를 통해들어간 후 순차적으로 검색한다.(data가 너무 많을 경우 사용)
multi level index: outer index -> inner index -> real data 순으로 계층 구조를 가진다.

# 인덱스 사용시 주의사항
검색 속도는 증가하지만 수정, 삭제가 느려진다.
mongoDB에서는 collection당 최대 64개까지 index를 가질 수 있다.
일반적으로 2-3개 이상은 갖지 않는것이 좋다.
mongoDB에서 RDBMS index와 유사하게 작동한다.
인덱스 구축시 다른 read/write작업은 중단된다.(background option 사용시 동작할 수 있지만 효율이 떨어진다.)

## B+ Tree
root 부터 leaf까지 모두 같은 길이로 이루어딘다.
root(x), leaf(x) => 사이에 있는 internal note는 ceil(n/2)와 n 사이만큼의 자식을 가져야한다.
leaf node(실제 값을 가진 노드)는 ceil((n-1)/2)~n-1 값을 가져야한다.
root가 만약 leaf일 경우 0 ~ n-1개의 값을 가질 수 있다.
그렇지 않을 경우 최소 2개 이상의 children을 가져야한다.
ordered되어 있기에 leaf node에서 순차 검색이 가능하다.

### node structure
child의 수: n개, key의 개수: n-1개
tree의 height가 왠만해서는 3을 넘지 않는다.

### insert
들어가야할 위치를 search알고리즘을 통해서 들어가고 search-key값이 존재할 경우 record를 파일에 추가하고 없을 경우 분할 후 rebalance한다.
분할 작업은 절반으로 나누어 새로운 노드와 기존의 노드에 분배하게 된다.
(부모가 새로운 노드를 가르킬수 있도록 만드는데, 이때 부모가 가득찼을 경우 다시한번 분한일 일어난다.)

### deletion
search를 사용해 값을 제거한다.(leaf node에서)
이때, B+Tree의 조건을 맞추지 못할수 있다. 이를 위해 redistribute를 해야한다.
redistribute는 형제 노드와 결합하는 작업으로 하나의 single node로 만드는 작업이다.

# 샤딩
## 샤딩의 목적
- 데이터 분산 저장을 통해서 빅데이터를 관리하기 위함(시스템 성능 향상)
분산하지 않을 경우: 서비스의 성능 저하가 유발된다.(디스크를 사용하는 하드웨어 한계성, 물리적 한계. 샤딩하면 독립된 프로세스가 병렬로 처리함으로 뛰어넘을 수 있다.)
- 백업과 복구를 위해(데이터 유실 예방)
분산하지 않을 경우: 복구가 불가능하거나 엄청난 시간과 비용이 소모된다.

## 구조
client - mongos(라우터. 한대에서 이루어진듯한 효과를 제공. 여러개를 운영할 수 있으며 로드 벨런서를 두기도 한다.) - 샤드(각 db서버) - 레플리카(mongod)
config server(mongoDB의 meta data를 관리한다. 어떤 데이터가 어디에 어떻게 분산되어있는지)

## 특징
효율성 향상을 가장 큰 목적으로 한다.
(3대 이상의 서버로 성능 보장을 추천한다. 최소 2대 이상 필요)
기존 운영 방식보다 메모리를 20-30% 추가로 사용한다.

## config server
샤드 시스템에 대한 메타 데이터를 저장/관리하는 역할을 한다.
샤드와 별도의 서버에서 운용된다.
장애 발생에 대비해 최소 3대 이상을 추천한다.(1대여도 가능)
인덱스 검색이 빨라진다.
샤드에 비해 저사양으로 운용할 수 있다.

## mongos
하나 이상의 프로세스를 사용한다.
각 샤드 서버로 데이터를 분산해주는 프로세스(쓰고 읽는 작업)
개발자가 각 서버의 역할을 알 필요가 없다.
개발자가 샤딩 상태인지 리플리케이션 상태인지 알 필요가 없다.

## 샤딩 시스템 계층
중개자 계층(핵심): mongos
응용 계층으로부터 전달 된 질의를 분석하여 데이터 계층에 보내고 결과를 다시 응용계층으로 전달한다.

## shard key
여러개의 샤드로 분리될 기준 필드를 가르킨다.
partition과 load balancing에 기준이 된다.
데이터 저장과 성능에 절대적인 역향을 준다.
적절한 선택이 필요하다.(cardinality를 보고 데이터 분포가 넓으면(ex, 성별) low cardinality, 분포가 높으면(ex, 주민번호) high cardinality로 부른다.)
chunk migration의 횟수와 빈도를 결정한다.(빈번하게 발생시 chunk 크기를 키워야한다.)
*chunk란*
하나의 서버에 저장되는 데이터들을 여러개의 논리적 구조로 분할 저장하다 일정 데이터 양에 도달했을 때 2,3번째 서버로 분할 저장한다.

# Replication
고성능 DB의 핵심
Master/Slave 구조로 구성되어 있다.(수동으로 장애 조치를 취해야한다. 자동 불가)
고성능 미러링이 가능하다.
성능과 높은 가용성이라는 장점을 제공한다.

## 용도
데이터 일관성 유지
읽기 분산(성능 향상)
운영중 백업(예측 불가 상황 대비)
오프라인 일괄 작업 데이터 소스

## 동작 원리
쓰기는 Master만 가능하다.
쓰기가 실행되면 oplog에도 저장한다.(data 저장소와)
B+ 트리로 구성된 데이터 저장소는 쓰기를 수행한 결과만 저장한다.
oplog는 연산 수행과 관련된 명령 자체를 timestamp와 저장한다.
Slave는 주기적으로 Master에게 optime보다 큰 oplog를 요청한다.

## replica set
primary server: 첫 입력을 담당
secondary server: primary 제외 모두. primary에 문제가 생길 경우 바로 primary로 변환된다.

### 특징
primary는 secondary의 상태를 2초마다 체크한다.(heart beat)
secondary가 중단되어도 primary를 사용할 수 있다.(복구될 경우 밀린 data를 동기화 한다.)
primary가 멈출 경우 secondary를 primary화 한다.
primary의 heart beat는 복제 집합의 과반수 이상이어야 한다.(지키지 못할 경우 primary를 secondary화 한다. priority와 votes를 통해 primary를 결정)
외부에서는 primary를 알 수 없다.

### 한계
replica set의 최대 노드 개수는 12개이다.
투표 가능 노드 개수는 7개 이다.
통신지연만큼 시간차이가 발생한다.
Master Server가 죽었을 때 write가 실행되면 데이터가 유실된다.(저널링을 통해 예방하지만 group commit 주기안에서 data가 날아갈 수 있다.)

# Map Reduce
대용량의 데이터를 안전하고 빠르게 처리하기 위한 방법
한대 이상의 하드웨어를 활요하는 분산 프로그래밍 모델
대용량 파일에 대한 로그 분석, 색인 구축, 검색에 탁월한 능력을 가지고 있다.
분산하여 연산 후 합치는 기술
맵과 reduce 단계로 구분된다.
맵 단계는 입출력으로 key-value 형태를 가진다.(분산에 필수)
데이터를 섞어 병합후 리듀스를 통해 최종 결과를 제공한다.
사용자가 임의로 코딩이 가능하다.
분할된 조각이 작을수록 더 좋은 효과를 제공한다.
너무 작으면 분할 단계에서 overhead가 발생한다.
RDBMS의 보완체로서 사용된다.(보다 유연하고 수정 없이 여러번 읽는 작업에 유리)
일괄처리  방식으로 dataset 분석에 적합하다.

## 장점
대용량 처리 단순화
특정 모델, 스키마에 의존적이지 않다.
저장 구조의 독립성을 가진다.
높은 확장성을 보인다.

## 단점
불편한 스키마 질의문
DBMS보다 낮은 성능
개발 환경이 불편한다.
단순 데이터 처리에만 최적화되어있다.

## Hadoop의 HDFS
- client: 송수신 요청
- name node: HDFS 전체 제어. 메타 데이터 저장
- data node: 데이터 저장 역할
