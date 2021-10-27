## Transport Layer
**용어 정리**
Application Layer: Message
Transport Layer: segment
Network Layer: Packet
Link Layer: Frame

### Multiplexing / Demultiplexing
Transport Layer에서 기본으로 제공하는 기능이다.
Multiplexing: 여러 process가 보낸 정보를 하나의 통로로 보내기 위해 합치는 과정
Demultiplexing: 하나의 통로에서 온 다양한 정보를 여러 process로 나눠주는 과정.

TCP와 UDP 모두 제공하지만 방식이 다르다.

1. UDP의 경우

connection less 방식
1:1 mapping(하나의 process는 정해진 process로만 전해지는 것)이 없다.
따라서 다른 process로 전송이 가능하다. 다순히 segment header의 port번호로만 destination을 확인한다.

총 4개의 헤더필드로 구성되어 있다.
source port(보내는 측) # dest port(받는 측) # length(전체 길이) # checksum

*checksum*
16bit 단위 검사(2개의 16bit)
둘을 더한 후 overflow 된 값은 제일 낮은 비트에 더하고 이후 2의 보수를 취한다.
2. TCP의 경우

connection oriented 방식
1:1로 mapping되어 있다.
TCP socket에는 고유 ID 4가지 정보가 있는데
source IP, source Port, destination IP, destination Port 로 각각을 구분 가능하다.

신뢰성 있는 데이터 전송(RDT, reliable data transfer) * TCP 설명 아님
신뢰성 있는 데이터 전송과 수신 시이에 unreliable channel이 끼어있다.
unreliable channel에서 bit error, packet 유실등이 발생할 수 있는데 이를 극복할 경우 reliable 하다 할 수 있다.

**rdt 1.0**
신뢰성 있다고 가정하고 진행
sender: packet을 만들고 udt_send(pack)을 보낸다.
receiver: deliver_data(data)를 받고 extract(packet, data)를 받는다.

**rdt 2.0**
신뢰성이 없다고 가정하고 진행.
1. cecksum을 통해 에러를 확인한다.
2. 에러가 없을 경우 ACKs를 보내고 에러가 있으면 NAKs를 보낸다(피드백)
   1. NAKs를 받을 경우 다시 data를 보낸다.
   2. 이때 receiver에서 data가 중복되었는지 확인이 가능해야한다. 따라서 sequence number를 함께 보낸다.
   (중복일 경우 msg를 버리지만 ACKs는 보내야한다.)

**rdt 2.2**
ACK에 sequence number를 붙여 마지막으로 받은 data를 구분한다. (NAK가 필요없어짐)

**rdt 3.0**
packet loss가 발생할 경우
일정 시간동안 답이 오지 않을 경우 재전송한다.
짧을경우 중복이 많아지고, 길면 재전송이 오래걸린다.

상황가정
1. packet loss
timeout 이후 data 재전송
2. ACKs loss
timeout 이후 data 재전송
3. delay ACK / premaure timeout
timeout 이후 data 재전송
뒤늦게 들어온 ACK에 반응해 data 전송(중복 발생)
하지만 정상적으로 작동은 할 수 있다.

### stop and wait operation
t = 0 ~ t = L/R -> firstPacket ~ last packet (transmission delay)
이후 ACK를 받을때까지(RTT)

U_sender = (L/R) / (L/R) + RTT => (링크 사용시간 / 패킷을 한번 보내기 위한 시간)

이 방식은 효율이 좋지 않다. 이를 극복하기 위해 pipelining으로 해결한다.

### pipelined protocols
**![Screen Shot 2021-10-19 at 10.16.49 PM](https://raw.githubusercontent.com/chichchic/GAZA_COMMERCE/main/chichchic/images/GBN_SR.png)

**Go-Back-N**
N개의 packet을 연속으로 보낸다.
receiver는 cumulative ack만 보낸다.

header에 k-bits Seq를 붙인다.
size가 N개인 window를 사용한다.
window의 시작점(send_based, 이미 ack를 받은 data 이후에 보낼 data)에서만 timer를 측정하고 만료될 경우 이후 데이터 전부를 재전송한다.
순서에 맞지 않을 경우 무시하고 순차적으로 들어온 seq#중 제일 높은것만 ACK를 보낸다.


**Selective Repeat**
N개 packet 연속은 동일하다.
individual ack를 사용한다.
ack를 받지 못한 packet만 재 전송한다.

unAcked pkt들마다 timer를 측정한다.
*sender*
window 내부에 보낼 수 있는 packet을 보낸다.
timeout일 경우 resend
send_based의 ACK를 받으면 window +1
*receiver*
window 안에 있는 pkt를 받으면 buffer에 넣는다.(순서를 맞출수 있도록)
in-order일 경우 receive_base +1
window 밖에 있을 경우 무시한다.

**Selective Repeat의 딜레마**
모든 ACK가 유실될 경우 재전송되는 pkt과 receiver window의 범위가 포함될 수 있어 잘못된 data를 받을 수 있다.
따라서 window size를 줄이거나 seq#를 늘려야한다.(window size의 2배 이상으로 seq#를 설정해야 함)

## TCP:overview
P2P: one sender 2 one receiver 매칭 (1:1)
reliable, pipelined(병렬적인 수행), full duplex data(양쪽 모두 server, receiver가 된다.)
connection-oriented(connection이 맺어져야 시작), flow controlled(네트워크.receiver의 상황에 맞춰 속도 결정)

**header 정리**
port number: 출발지, 도착지를 결정한다.
sequence number: 받은 정보 or 재전송 정보를 확인한다. 시퀀스의 first byte number로 표현한다.
ACK number: NCK 대신 사용한다. 순차가 보장된채로 받은 가장 마지막 number로 표현한다.

### TCP, RTT, timeout
segment를 보낼 때 timer가 시작된다.
RTT + a 만큼을 timer 시간으로 설정한다.
RTT는 매 시간마다 측정한다. 이때, 재전송한 Seg의 RTT는 제외하여 계산한다.
항상 측정된 값에만 의존할 경우 변화 폭이 너무 크기에 Estimated RTT를 사용한다.
*ERTT = (1 - a) * ERTT + a * Sample RTT, a = 0.125(공학적 결정으로 변경 가능하다)*
*Timout Interval = ERTT + 4 * DevRtt, DevRtt = Safety margin*
실제 데이터 전송시 Timeout Interval보다 큰 값들이 많아 safety margin을 준 Timeout Interval을 사용한다.
### reliable data transfer
send buff(재전송을 위해)와 recv buff(순서가 맞지 않게 도착한 경우를 위해)를 사용한다.
TCP는 window 크기만큼만 data를 보낼 수 있다.
ACK number가 3번 중복된 값이 올 경우(같은 값이 4번 올 때) expire되기 전에도 재전송한다.(TCP fast retrasmit)
=> 순서가 맞지 않게 도착했을 경우 이전의 ACK number가 중복해서 도착하게 된다.

