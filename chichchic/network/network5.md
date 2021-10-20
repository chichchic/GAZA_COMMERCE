## SDN control plane
router controlplane을 내부에 가지고 있으면 update가 어렵다.
이를 해결하고자 per-router plane이 제시되고있다.

locally centralized control 방식(remote control)으로 개선
-> 중앙 통합 관리(SDN의 핵심 Approach이다.)

사용 이유
1. 네트워크 관리가 용이하다.
2. table-based forwarding에 programming이 가능하다(open flow)
3. control plane이 모두에게 open되어있다. (특별한 interface가 없다. open interface)

Q: source와 destination에 따라 중간 경로를 다르게 할 수 있을까?
A: SDN에서는 가능하다. traffic engineering이라고 부른다.
Q: Destination과 source가 같은데도 경로를 다르게 할 수 있을까?
A: SDN에서는 가능하다. SDN을 통해 경로의 선택폭이 늘어난다.

### SDN
핵심: Generalized forwarding
단순히 destination만 보지 않고 다양한 요소를 확인한다.
이를 위해 control plane과 data plane을 분리한다.
data plane의 switch 입장에서 control plane은 완벽히 외부이다.
programable control update로 다루어진다.

SDN data plane(switch, hard ware)
- fast, simple, generalized forwarding이 hardware에 구현되어있어야한다.
- table-based switch control의 API가 제공되어야한다.(open flow)
- centeralized control과 통신이 가능해야한다.(통신 과정을 위한 API도 요구된다.)


### SDN controller(OS같은 입장)
network state 정보가 필요하다.
분산 시스템에 구현(distribute system) -> 각기 다른 도메인에 구현
서버 입장에서는 single point of failure를 방지할 수 있다.
논리적으로는 centeralized 되어있다.(router 입장)

### SDN network-control plane(brain에 해당)
프로그램 설치하듯 사용 가능
써드파티(제3자)에 의한 서비스가 등장할 수 있음

### open flow
controller와 switch 사이에 control정보나 switch 상태 정보를 controller로 보낼 때 사용(interface 역할) (TCP, 암호화 제공)
메세지 정보를 1) controller 2 switch, 2) switch 2 controller, 3) symmetric(그외) 로 구분한다.
open flow는 프로토콜 중 1개에 불과하다.(정답은 없다.)

**switch와 controller 통신 예제**
1. s1에서 s2로 link failure 발생: port 정보를 알아낸다.
2. open flow를 통해 failure가 발생한 port를 알려준다.
3. SDN controller가 그 상태 정보를 받는다.
4. link sate정보를 update한다.
5. routing정보를 update(필요시) - algorithm을 통해 계산
6. flow table component와 interation해 flow table을 만든다.
7. open flow를 통해 update 정보를 전달한다.

### SDN의 도전 과제
오류에 얼마나 잘 대처하는가(안정성 check, error propagation check)
security 문제가 더욱 예민해짐(propagation이 가능해짐)
특정 요구사항이 가능한가.(realtime, ultra-reliable, ultra-sequre 둥)

## ICMP(internet control message protocol)
host와 router 사이에 error message를 report 하기 위해서 사용
(Ping같은 echo reply), TTL 등을 사용

### Trance route on ICMP
바로 앞의 router에게 3번의 probes를 전송해 응답이 돌아온 시간을 평균내서 round trip time을 계산한다.
각 router마다 측정. destination host에 도달 or "port unreachable" msg(오류)가 올 때 까지
UDP seg로 구성되어있다.
TTL사용:
몇 hop을 살아가는지 확인. TTL에 홉 크기를 넣어서 보낸다. router는 datagram을 버리고 type2에 해당하는 ICMP를 넣어 source로 전송한다.
도달 시간까지가 round trip time이다(3번한 후 평균)
홉이 증가해도 시간이 감소할 수는 있으나 일반적으로는 증가한다.

## SNMP(network management)
autonomous system(AS)는 수많은 interaction
SNMP란 직접 network를 관리하는 관리자의 protocol이다.
network management는 합리적 비용으로 요구사항을 맞추려하고 이를 위해 정보를 수집한다.
managed device들이 존재한다.(agent 설치)
data들은 manage entity(server)로 전송한다.
이를 기반으로 각각 manage device(object라고 부름)를 관리한다.(MIB라고 부른다)

### MIB 정보 전달 2가지 방식
1) managing entity - request > agent data. agent data - response > managing entity
2) agent data - trap msg > managing  entity (오류 발생시 request없이 msg를 보낸다.)