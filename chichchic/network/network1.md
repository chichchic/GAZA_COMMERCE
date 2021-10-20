## protocol
- 통신할 때 무조건 사용되어야 함.
- 서버와 클라이언트간의 사전확인을 위해 사용.
- 규칙들이 정의되어 있다.

**TCP**
- 순서와 송신확인을 보장해주는 신뢰성 있는 통신
- flow control이 가능하다(보내는 파일을 받는사람의 속도에 맞춰준다.)
- congestion control이 가능하다.(네트워크 상황에 맞춰서 속도를 조절해준다.)

**UDP**
- 아무것도 보장해주지 않는다.
- 하지만, 속도가 매우 빠르다.

## 구성 요소
**edge**
client: 원할 때 사용한다.
server: 항시 대기하고 있는다.

**link**
유선, 무선 모두 가능하다.

**router**
데이터를 전송해주는 core 부분을 뜻한다.
router와 router의 연결(빠르고 대역폭이 큰 선을 사용한다.), router와 edge의 연결이 존재한다.

## Router의 전달방식
### packet-switching. 작은 블록의 패킷으로 데이터를 전송하며 데이터를 전송하는 동안만 네트워크 자원을 사용하는 방식

**관련 용어**
- processing delay: 잘 전달중인지 확인 중 걸리는 시간.
- transmission delay: 전송에 걸리는 시간(bandwidth와 반비례).
계산식: length of bits(L) / transmission rate(link bandwidth, R).
End2End delay => transmission delay * hops(사이에 있는 네트워크 장비 개수)
- propagation delay: 마지막 패킷이 다음 라우터로 가는 시간. 길이, 빛의 속력에 의해 결정되므로 우리가 개선할 수 있는것이 없다.
- buffer(queue): packet을 저장해두는 router 공간
- queuing delay: packet buffer에 머물러 있는 시간

transmission delay, processing delay는 개선이 가능하지만, queuing delay는 현실적으로 개선이 불가능하다.
packet 유실은 대부분 queue가 꽉 찼을때 발생한다. TCP에서는 이를 알 수 있다.
이때 packet을 다시 보내는건 router가 할수 없으므로 edge가 해야한다.(모든 작업은 edge가 한다)

### circuit-swtchig
FDM - 주파수 단위로 나눠서 통신하는 방식
TDM - 시간 단위로 나눠서 통신하는 방식

circuit-switching은 시간별로 효율이 차이나며, 많은 사람이 사용하기 어렵다.
반면, packet-switching은 언제든 누구나 사용이 가능하다.
효율성 측면에서 보다 좋아 packet-switching을 대부분 사용하고있다.
하지만, 몰려서 사용할 때 문제가 발생한다.

## 네트워크의 2가지 기능
Fowarding: 목적지를 가기위한 경로(주변 router 중 어디로 가야하는지)
Routing: 출발지에서 목적지까지의 경로를 결정하는 것. 전체 경로를 봐야한다.(미리 계산한 후 Fowarding)

circuit-switching은 미리 사용할 경로를 예약하기에 속도가 보장된다.
packet-switching은 속도가 보장이 안된다.(queuing delay 또는 packet loss시 더 느리다)

## 4가지의 delay
1. transmission(전송 지연)
- 패킷의 모든 비트들을 link로 전송하는 시간(L/R)
2. nodal processing(노드처리 지연 - 별로 지연되지 않는다)
- 라우터에서의 처리 지연
- 패킷 헤더를 조사하고 어느 출력 링크로 보낼 지 결정하는 시간
- 패킷의 비트 수준 오류를 조사하는데 필요한 시간
- 고속 라우터에서 처리 지연은 수 밀리초이다.
3. queuing
- queue에서 대기하는 시간
R: link bandwidth
L: Packet Length
a: average packet arrival rate

La/R ~ 0 => delay small
La/R ~ 1 => delay long
La/R > 1 => almost Infinity
4. propagation
- link에서 다음 router까지의 시간

### Thoughput(처리율)
instantaneous: 순간 처리율
average: 평균 처리율
속도가 다른 pipe가 2개 있다면, 작은것에 맞춰서 보내게 된다. => bottle neck link

## OSI/ISO 7layer
기능을 명확하게 하며, 업데이트가 용이할수 있도록 나눈다.
하지만, 기능 중복이 발생하며, 다른 레이어의 기능을 못쓰는 단점이 있다.
1. physical: bit를 전달한다.
2. data link: Frame을 전달한다. 노드와 노드사이의 데이터를 전달한다. 신뢰성을 위해 흐름제어 및 오류제어 기능을 제공한다.
3. network: packet을 전달한다. 송신측에서 최종 목적지까지 패킷을 전달한다.
4. transport: segment를 전달한다. TCP/UDP에 해당하는 layer. 연결/흐름 제어를 담당한다.
5. session: Message를 전달한다. 통신하는 프로세스 사이의 대화제어 및 동기화를 담당한다.
6. presentation: Message를 전달한다. 데이터의 변환 압축, 암호화를 담당한다.
7. application: Message를 전달한다. IMAP, SMTP, HTTP등 사용자가 원하는 최종 목표에 해당한다.

application layer는 운영체제 위에서 돌아가는 process 중 1개이며, 다른 컴퓨터와 통신한다는 점만 다른 process와 다르다.
IP(어디 host)와 Port(host안의 process주소)를 사용하여 주소를 설정한다.(보통은 DNS를 사용한다.)

**transport layer에서 app이 필요로 한것**
data 무결성, 처리율, 보안, 걸리는 시간

## HTTP
request와 response로 구성되어있다.
TCP를 사용해야만 한다. => 이에따른 비용이 필요하다.
stateless(요청에 응답만 보내고 기억하지 않는다.)하므로 빠르고 많은것을 처리할 수 있다.

non-presistent HTTP: 매번 TCP연결을 하는 방식
presistent HTTP: TCP 연결을 유지하는 방식. 매 object마다 1RTT(round trip time)만큼 이득을 본다.

stateless의 단점을 극복하기 위해서 cookie를 사용한다.
이를 통해 사용자를 식별하거나 정보수집 및 사용자 추적이 가능하다.

## Proxy
request를 proxy에 요청한다.
proxy는 origin server로 요청한다. 동일한 request의 경우 proxy에서 바로 응답한다.
**장점**
서버: load balancing
클라이언트: Qkfms threh
proxy: traffic 감소 -> 비용 감소
**단점**
cache와 서버 사이에 동기화 문제 발생
보안상의 Issue가 존재.
TODO: 어떤 보안상의 Issue가 존재하지? 오히려 안전한거 아닌가.

## cache
사용하려는 Data를 미리 가져다 두는 일
bandwith를 확장하는것은 너무 비싸며, proxy보다 싸게 효율을 높일 수 있다.
**효율 계산**
hit rate를 높이는게 가장 중요하다.
(1 - hit rate) * accesslink / bandwidth

**conditional get**
If-Modified-Since을 request header에 포함해서 요청할 경우.
지정된 날짜 이후 데이터가 수정된 경우 200과 함께 리소스를 돌려준다.
수정되지 않았을 경우 304 응답만을 보낸다. 이를 통해 동기화 문제를 해결한다.

## DNS
TABLE로 매칭시켜 주소를 관리해주는 서비스
하지만, 너무 많아서 관리가 어렵다. 따라서 계층화하여 사용한다.
ex) root -> (com, org, edu) ...
single point of failure(DNS가 고장날 경우 먹통이 되는 문제)를 막기 위해 root DNS server가 동일한 사본으로 13개 존재한다.

**local DNS name server**
이전에 DNS server를 통해 받아온 IP 주소를 local에서 기억하는 것.

DNS는 UDP를 사용한다. 동기화 문제를 해결하기 위해 유효기간을 설정한다.

**구성요소**
name, value, type, ttl(time to leave, 유효기간)

## email 구성
- user agent: 메일 확인용 브라우저
- mail server: 네이버, 구글 등 서비스를 제공해주는 곳
- simple mail transfer protocl(SMTP, push 프로토콜이다)

mail은 HTTP 적용이 어렵다.
email의 경우 받는 사람이 항상 online이 아니기때문에 mail server가 대신 받아줘야한다.
예시)
1. A가 A_server로 SMTP 프로토콜로 보낸다.
2. A_server가 B_server로 SMTP 프로토콜로 보낸다.
3. B가 B_server에 mail access protocol(POP, IMAP 등)을 사용해서 메일을 읽어온다.