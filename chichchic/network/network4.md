## NetworkLayer
data plane, control plane, forwarding router 등을 살펴본다.

transport layer에서 온 segment를 목적지로 보내는것이 역할이다.
sender는 segment를 capsulation, receiver는 datagram에서 segment를 추출한다.
모든 호스트 + 라우터에 Network Layer는 필요하다.
레이어간의 독립성을 보장하기에 transport layer에서는 network layer의 경로를 알 수 없다.

**대표 기능**
- forwarding: 각 라우터에서 어떤 output port로 내보낼지 결정
- routing: send-receiver의 경로. (forwarding table을 만든다.)
원하는 경로로 가기위한 output port 결정이 결국 forwarding이다.

**data plane 영역**
어떤 output port로 내보내야 할지 결정(forwarding function)
**control plane 영역**
출발지에서 목적지까지의 전체경로를 결정한다.

### per-router control plane
traditional routing approch
라우팅 알고리즘이 각각 라우터에 존재한다.
라우터간 상호작용한 결과로 table을 생성한다.
단점: 각각 라우터에 알고리즘이 존재해서 update가 어렵다.

### logically centralized control plane
RC(Remote controller)와 CA(Control Agent)를 만들어
알고리즘 update시 RC가 CA로 전달하고, CA는 table을 변경한다.

## Network Service Model
datagram을 위해 network layer가 제공하는 서비스란 무엇인가.
-> 반듯이 전달, 순서보장, 40ms안에 전달, 1m bps 보장 ...

**delay jitter**
네트워크 상황으로 인해 voice data 사이에 공백이 생기는 문제
internet은 보장하지 않는다.
ATM (cbr, vbr, abr, ubr 등이 있다. 별로 중요하지 않은 내용)

## Router architecture
4가지 컴포넌트
1. router input port, 실존 포트. dataplane(hard ware)
2. router output port, 실존 포트. dataplane(hard ware)
3. processor (software)
4. high-seed switching fabric, dataplane(hard ware)

### input port
- physical layer(line termination)
bit stream을 packet으로 만든 후 link-layer로 올려준다.
- link layer
packet error check
- look up, forwarding, queuing
packet이 가는곳이 어디인지
output port는 어디인지
packet 도착속도가 빠를때는 queuing

**forwarding 기법 2가지**
1. destination 기반. 목적지 IP 기반
destination address range가 적혀있다.
logest prefix match(가장 길게 match되는것을 선택)
don't care을 저장할 수 있는 content addressable memory(content를 통해 주소 반환)이다.
one clock cycle(매우 빠르게)에 address memory를 돌려준다.
특별한 hardware가 필요하다.(비싸다)
2. generalized forwarding. header field 값을 보고 결정

**cast 종류**
1. uni cast. source와 destination 1:1 매칭
2. multi cast. source와 destitnation 1:m 매칭. 중복 발생 최소화 기법이 필요하다.(영상 스트리밍) 네트워크 자원 절약이 가능하다.

### switching fabrics
memory, bus, crossbar(interconnection) 형태로 구성되어있다.
1) memory
   packet이 들어오면 system bus를 통해 memory에 저장한 후 다시 읽어 output port로 보낸다.
   input oprt와 output port가 memory를 공유한다.
   switching speed: input port -> output port 까지 얼마나 빠르게 forwarding하는가
   메모리의 속도에 따라 결정된다. input과 output은 동시에 writing/read할수 없기에 속도가 1/2가 된다.(단일 메모리 단점)
2) bus
   input port와 output port 각각에 memory가 존재한다.
   bus가 각각을 연결한다.
   output port label이 존재, input port에서 data를 쓸 때 label도 같이 작성한다.
   모든 output port 는 input port를 보고 label이 맞을 경우 buffer에 추가한다.
   -> 굉장히 빠르지만 bus가 한번에 한 packet만 사용 가능하다.(단점)
3) interconnection
   crossbar형태로 구성되어있다. 교차점마다 switch가 존재한다.
   전송시에만 on, 나머지는 off하는 방식으로 여러 input/output이 동작할수 있다.

### input port queuing
빠져나가는 속도 < 들어오는 속도 일때 block이 발생한다.
메모리에는 한계가 존재하기에 넘치면 packet 유실이 발생한다.
쌓이면 스케줄링에 따라서 순서를 결정한다.

block이 발생했을 때 다음 packet은 block이 되지 않을 수 있다.
input port queue의 head에서 동일한 output port로 나가려고 할 때 fabrics가 먼저 처리한 packet을 제외하고 block될수 있다.
이를 HOL(head of the line) blocking이라고 한다.
이 해결책으로 virtual queue를 제작하고 input port 에서 나온 packet을 저장하고 head에서 블락 될 경우 다음 packet을 순서대로 사용할 수 있도록 하는 방식이다.

### output port queuing
output port queuing -> link layer protocol -> bit stream으로 나간다.
switching을 통해 들어온 packet이 처리속도(link capacity)보다 많을 경우 congestion이 발생한다.(packet loss도 발생할 수 있다.)
*우선순위 설정 가능*
network schedueling은 주로 output port queuing
priority scheduling이 가장 좋은 성능을 보여준다.

### buffer의 memory 크기는 얼마나될까.
마냥 크다고 좋은것은 아니다.
buffer가 커지면 기다려야하는 시간이 올라간다.(buffer bloat)
경험적 법칙에 따르면 RTT(250ms) * link capacity(10Gbps) / N(flow size)^0.5 이다.

### scheduling mechanism
1) FiFo
   buffer가 가득찰 경우, tail drop(마지막 버림), priority drop(우선순위가 낮은 packet 버림), random drop(임의로 버림)
2) Priority
   우선순위를 결정해서 먼저 서비스(IP주소, port번호, header정보 사용)
   ex) high priority queue와 low priority queue를 분리한다.
3) Round Robin
   multi queue를 번갈아가면서 처리하는 방식
   * work - conserving(일 보존) 순서를 지키다 처리할게 없으면 쉬지 않고 다른 처리작업 실행
4) Weighted Fair Queuing(WFQ)
   generalized Round Robin
   소량의 트랙이 대량의 트래픽에 의해 손해를 보지 않도록 packet flow별로 다른 queue를 두어 트래픽을 조절(공정성)
   특정 기준에 따라 가중치를 정하여 같은 양의 트래픽을 가진 packet flow에서도 차별을 두는 방식(가중치)

## internet protocl
routing protocol -> fowarding table을 관리
IP protocol -> 주소 체계 설정, data gram format, packet처리 체계 설정
ICMP protocol -> Error reporting, router signaling(주변 라우터 상태를 공유할 때 사용)

**IP datagram Format**
1. IP version number
2. header length
3. data type
4. time to live
5. upper layer(UDP/TCP)
6. length
7. 16bit identifier/flags/fragment offset(datagram을 쪼개서 사용할때 이용)
8. option -> time stamp
9. data(payload, 전송 data. 이게 payload인지는 관점에 따라 다르다.)

## IP fragmentation, ressembly
MTU(max-transfer-size. 최대 크기는 protocol마다 다르다.)를 위해
fragmentation(분열)과 ressembly(재조립)을 제공한다.
transport layer에서 큰 segment가 내려올 경우 network layer에서 잘라서 전송한다. (받는측에서 재조립)
-> 이때 fragment offset에 (이전 data + 지금 보내는 data) / 8만큼을 표기해주어야한다.

## IPV4 주소체계
- 32bit
- interface(ram카드)에 할당한다. host가 아님
- host router와 physycal link사이에 connection 제공
- nic(network interface card)와 같은 interface가 존재한다.
- interface는 여러개 있어야한다.(각각 할당 받는게 가능)

### subnet
관리자가 네트워크 성능을 향상시키기 위해 자원을 효율적으로 분배하는 것.
네트워크 영역과 호스트 영역을 분할한다.
공정된 부분: subnet part -> 라우터 개입x 물리적으로 data 주고받는것이 가능
변경된 부분: host part

host/router의 interface들을 이용해서 고립된 network 형성
subnetmask: 필요한 네트워크 주소만 호스트 IP로 할당할 수 있게 만들어 네트워크 낭비를 방지한다.
ex) 192.168.0.3, 192.168.0.4 에서 192.168.0은 네트워크 영역, 3과 4는 호스트IP. 0과 255호스트 IP는 예약되어있어 사용 불가.

### class inter domain routing(CIDR)
subnet 크기 결정 x(임의)
class => A/8,B/16,C/24,D/32로 과거에 사용. 이러한 크기별로 규격화 하지 않는것이 CIDR이다.
IP주소 뒤에 앞에서부터 범주화된 크기를 적어서 보다 유연하게 IP주소를 여러 네트워크 영역으로 나누게된다.
ex) 192.168.0.0/16

### DHCP(dynamic host configuration protocol) 동작 방법
DHCP란 호스트의 IP주소와 각종 TCP/IP 프로토콜의 기본 설정을 클라이언트에게 자동으로 제공해주는 프로토콜이다.
DHCP는 네트워크에 사용되는 IP주소를 DHCP서버가 중압 집중식으로 관리하는 클라이언트/서버 모델을 사용한다.
DHCP지원 클라이언트는 네트워크 부팅과정에서 DHCP서버에 IP주소를 요청하고 이를 얻을수 있습니다.
장점: PC의 수가 많거나 PC자체 변동사항이 많은 경우 IP설정이 자동으로 되기 때문에 효율적으로 사용 가능하고, IP를 자동으로 할당해주기 때문에 IP 충돌을 막을 수 있다.
단점: DHCP 서버에 의존하기에 서버가 다운되면 IP할당이 이루어지지 못한다.

host가 network join -> "DHCP discover" msg 전송
DHCP server가 "DHCP offer" msg 전송(자신의 주소와 사용 가능 주소를 포함)
host가 "DHCP request"를 통해 주소를 사용할것이라 전송
sever가 ACK를 보내줌(사용하게되는 주소를 포함한 네트워크 정보와 기간을 알려줌)

### Network Address Translation(NAT)
공유기를 통해서 여러 Host를 사용할 수 있음
한정된 주소를 더 많이 사용할 수 있도록 해줌
local에서만 사용하는 IP: 보안성이 우수해진다.

source IP Address, port#를 NAT IP address, new port#로 변경
변경후 기억은 NAT router가 한다.

**논란사항**
IP주소와 port 번호를 봐야한다. -> router 철학에 어긋난다.
+ IPv6로 충분하다.
+ 단대 단 통신에 위배된다.

**IPv6**
IPv4의 고갈로 인해 만든 프로토콜. header의 크기가 40으로 고정되고 간단해진다.
priority, flow label, next hader, source address, destination address
(checksum이 없고, option이 header 밖으로 나왔다.)
router들을 바꿔야 하는 문제로 아직 사용은 불가지만 많이 변경되고 있다.
IPv4 payload에 IPv6 datagram encapsulation을 해주어 혼동을 방지할 수 있다.(IPv6끼리는 그냥 전송)

### Generalized forwarding & SDN
단순 forwarding에서 확장해 여러 port로 보낼 수 있고, packet drop으로 잘못된 packet을 검출할 수 있다.
flow table 사용(forwarding table 대체)
centralized node(server/controller)가 계산하고 분배한 값(table)을 각 router들이 가지고 있는다.
header(dest address field)/counter(특정 flow drop packet개수 세기, byte수)/action(단순 forwarding/multicast forwarding/drop/header변경 결정)
으로 이루어진다.

**open flow**
Rule: matching에 사용할 필드
Action: forwarding encapsulation등 동작 정의
stats: packet/byte counter
성능 향상을 위해 layer간 간섭이 발생하게된다.

**match와 action을 활용하는 법**
1. router
match: longest destination(IP prefix)
action: forward out a link
2. switch
match: destination MAC address
action: foward or flood

등 다양한 기능을 사용(디바이스 흉내)
계산은 결국 control plane(중앙 SDN)에서 수행

### control plane(sofware)의 2가지 접근 방식
per router control(traditional)
logically centralized control(software defined networking)

### routing protocol
좋은 경로 선택을 위한것 -> 비용, 시간, congestion 등(다익스트라, DP등 사용)

**표기 법**
그래프로 표현. C(w, z)

### routing algorithm classification
global: network topology를 모든 node가 알고 있다.(link state 알고리즘)
decentralized: 이웃의 정보만 알고 있음. (distance vector 알고리즘)

static: 경로가 빠르게 변함
dynamic: 경로가 빠르게 변함

link stat => 다익스트라
주변 node들간 cost들을 소통하면 global의 특징을 가질 수 있음
O(n^2), O(n log n)까지 가능
oscillations possible 가능성이 있다.(양방향 cost가 다르므로 방향을 계속 변경하게 되는 현상)

distance vector => bellman-ford equation 사용
d_x(Y) = min(C(X, Y), d_v(Y)) 현재까지의 최소 Cost(예상치)
distance vector = D_x(집합) = [D_x(Y): Y는 N에 원소]
이웃들하고만 cost를 공유한다.
*특징*
비 동기적 반복, 분산형태로 구현 가능
각 노드가, wait(이웃의 변화) - 자신의 상태 계산 - 변화가 생기면 이웃에 알림 - wait을 반복하는 구조
*문제*
link cost가 변할 경우 cost가 증가했을 때 최소값 갱신이 바로 불가능함. out date data로 인한 혼동
*해결책*
poisoned reverse.(가고자 하는 경로의 값을 일부러 무한대 값으로 전달해서 빠른 수정)

message complexity
LS: O(NE)
DV: exchange neighbor

speed covergence
LS: O(n^2) or O(NE) 하지만 oscillation이 발생할 수 있다.
DV: 상화별로 다르다. routing loop -> count to infinity문제가 유발된다.

router 오작동
LS: router의 문제가 다른곳으로 퍼지지 않는다.
DV: router의 문제가 전파되는 문제가 발생한다.(error propagate)

linkstate => OSDF
distance vector=> BGP

**OSPF**
network를 flat한 상태로 보지 않고 계층적(subnet 단위)으로 본다.
administrative autonomy(내부적으로 알아서 하도록 내버려 둔다.)
autonomous system(AS): domain, ISP(다른곳과 상관없이 자신만의 routing 정책이 가능하다.)

AS안에서 routing: intraAS. 같은 routing protocol을 사용해야한다. gateway router(AS의 edge router)가 AS 사이에 routing
AS 사이에 routing: interAS. 다른 routing protocol을 사용할 수도 있다.
목적지가 intraAS안에 있으면 intra routing protocol만 사용하면 되지만, interAS에 있을 경우 둘 다 사용해야한다.

*interAS routing의 역할*
gateway: 현재 사용자가 위치한 네트워크에서 다른 네트워크로 가기 위해 반드시 거쳐야 하는 거점을 의미.
밖으로 나가는 datagram을 받을 경우 gateway로 forwarding후 밖으로 보낸다.
reachable information(prefix 및 그 길이로 표현된 정보)을 주변에 전파해야한다.

*interAS protocol*
Interior gateway protocol이라고도 부른다. IGRP, OSPF, RIP등으로 불린다.

*특징*
OSPF는 누구나 사용가능하고 link state alogorithm(다익스트라)를 사용한다.
모든 node가 network topology를 알고있어야한다.
AS간 topology 공유는 쉽지만 외부 AS끼리는 정책적(보안)인 문제가 생길 수 있다. (따라서 BGP를 사용)
주변에 알리는걸 flooding이라고 한다.
OSPF msg를 통해서 보내진다. ISIS routing protocol도 OSPF와 거의 유사하다.

*고급 기술(선택)*
security: 인증을 서로 한 후 정보 교환
한개의 source에서 동일한 destination까지 동일 cost 경로 제작 가능. load balancing에 좋다.
TOS(type of service) 서로다른 servicec에 다라 cost를 다르게 할수 있다.
uni-cost, multi-cost 지원
계층적으로 OSPF 구성이 가능하다.(large domain에서)
=> area border router를 묶어주는 backbone
=> backbone을 묶어주는 boundary router가 존재한다.
link-state advertisement는 area안에서만 가능하다.
자신의 area에서만 topology를 가질 수 있다.
다른 area로 전송하고싶을 경우에는 자신의 topology를 "요약"해서 보낸다.
backbone router는 backbone 전용 OSPF를 사용한다.
boundary router는 다른 AS로 연결되는 router이다.

**ISP: BGP(border gateway protocol)**
domain 사이의 protocol. 전체 네트워크에서 사용 가능하다.
eBGP: 인접한 AS로부터 reachability 정보를 수집한다.(inter)
iBGP: AS 안에서 reachability 정보를 전파한다.(intra)
상대와 traffic 전송 협정을 하는 policy가 중요하다.

**session**
거의 영구적인 TCP 연결을 통해 두 router가 BGP msg를 주고받을 수 있게 하는것
path vertor protocol: subnet이 내부에 존재한다는것을 다른 AS에게 알려주기 위한것
ex) (AS3, X)를 eBGP를 통해 리스트 형태로 전파
경로 정보 2가지 속성
AS-PATH: 어떤 AS를 통해야하는가.
NEXT-HOP: 다음 AS로 가기위한 Internal router
policy-based routing: 협정에 따라서 경로를 결정하는 정책

BGP Message
open -> TCP connect 열기
update -> 새로운 경로 홍보
keep alive -> update 없이도 connection을 지속
notification -> error 알림 or connection 종료 알림

BGP router selection
1. policy를 통해서 결정
2. AS-PATH중 가장 짧은것으로 결정
3. 가장 가까운 NEXT-HOP router를 통해서 결정(hot poatato routing. 다른 router를 생각하지 않으며 전체 경로 또한 생각하지 않는다.)

achieving policy via advertisement
provider-가입(연결)-customer, network가 있다고 가정
TODO: 내용 검색해도 나오지 않음

**intraAS와 interAS를 구분하는 이유**
1. inter는 서로 다른 IP끼리 소통하고 있으므로
2. intra는 제일 좋은 경로만 고민하면 되므로
3. SCALE 측면에서는 계층적 routing이 가능해져서(table size, update비용 감소. traffic 감소)
4. performance 측면. interAS는 성능<정책 우선순위일 수 있어 성능이 최우선이 아닐수 있다.