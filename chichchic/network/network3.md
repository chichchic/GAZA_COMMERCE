## Flow Control
받을 수 있는 양보다 많이 보내지 않기 위해서 사용(congestion contorl에 비교할게 적다.)

application process에서 데이터를 받고자 할 때 read() syscall을 호출해 TCP socket receiver buffer에서 데어터를 받아간다.
sender는 receiver buffer의 상황에 맞게 data를 보내야한다.
이를 위해 "receive window"를 헤더를 통해서 보내준다.
따라서, **sender가 보낼 수 있는 양은 receiver에 의존적이다**

*seg를 보내는 경우*
1. App->write()->send
2. App->read()->ACK
deadlock이 발생할 수 있다.
이를 해결하기 위해 rwnd=0을 받을 경우 주기적으로 probe packet(1byte data)를 전송한다.

### Segment 크기 정하기

**Nagle's algorithm(sender의 입장)**
1. first는 1byte만 보낸다.
2. App에서 오는 data를 축적, seg가 가득찰 경우 send, ACK가 돌아올 경우 send (2 반복)
3. network 상황이 좋을 경우 seg를 감소, App이 빠를경우 seg증가

**clark's solution**
1. seg MAX 만큼 rwnd가 공간이 없을 경우 rwnd=0으로 보낸다. (최대치만 받도록 유도)
2. 500ms 뒤에 ACK를 보내고, 만양 기다리는 동안 다른 data가 도착할 경우 ACK보내지 않아 횟수를 감소시킬수 있다.(Delayed ACK)

## Connection(hand shake)
**3-way handshake**

UAPRSF 중 A(ACK), S(Sync), F(FIN)헤더 사용
1. 시작시 S bit=1, Seq = X를 receiver에 전송
2. receiver는 S bit=1, A bit=1, seq = Y, AckNum = X+1로 전송한다.
3. A bit=1, ACKnum = Y+1을 전송한다.

**closing**
1. F bit = 1, seq = x (client)
2. A bit = 1, ACKnum = x+1 (server)
3. F bit = 1, Seq = Y (server)
4. A bit = 1, ACKnum = Y+1 (client)

4번 값이 유실될 수 있으므로 Server측 3번이 timeout되는 시간 * 2만큼 Client는 연결을 끊지 않고 대기해야한다.

## Congestion control
network가 감당할 수 있는 만큼만 보내는 방식
다수의 사람들이 사용하기에 복잡한 문제이다.
=> 혼잡도를 파악하고 보내는 양을 조절하는 방법이 핵심

**보내는 양과 받는 양의 격차가 생기는 경우**
1. application layer에 비해 tranport layer의 데이터가 많아지는 경우(재전송이 발생했을 때)
2. upstream router(가장 가까운 라우터)를 각각 전부 차지했을 경우 packet이 drop되어버린다.
모두 최대한으로 사용할 경우 아무도 보내지 못하는 모순이 발생

### TCP Congestion Control(AIMD, additive increase multiplicative decrease)
receiver window, cogestion window 두가지의 size가 sender의 window size에 영향을 준다. 둘중 작은 값을 사용한다.
전송이 잘 이루어질 경우 점진적으 증가시킨다.
전송에 문제가 생길 경우 절반으로 감소시킨다.

**sender limit transmission**
Last byte sent - Last byte acted <= CWND

initially CWND = 1MSS(maximum segment size * TCP sending rate, rate 는 CWND / RTT 과 비슷(byte/sec))
이후 SSthresh hold를 만날때까지 2배씩 증가

*timeout이 3duplicate보다 나쁜 상황이다*
timeout은 1MSS로 설정, ssthresh값은 현재의 1/2로 설정한다.
3duplicate 되지 않도록 thresh hold 값을 넘은 후에는 점진적으로 증가한다.(congestion이 일어나지 않을것이라 가정)

### TCP throughput
네트워크에 의해 결정되는 요소, 고정값이 아니다.
avg TCP throughput = 3/4 * w/RTT (bytes/sec)

### TCP fairness
작업이 반복되면 fairness가 보장된다.
- ECN, IP Header에 담아 Network Congestion 정보를 Router가 인지할수 있도록 한다.
- RED(Random Early Drop), 큐의 평균 크기를 감시하면서 통계적 확률에 의거해 패킷을 탈락시킨다.