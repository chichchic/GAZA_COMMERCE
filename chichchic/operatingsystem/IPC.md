# IPC(Inter Process Communication)
프로세스간 통신을 위해 커널에서 제공하는 기능이다.
IPC 통신에서 프로세스 간 데이터를 동기화하고 보호하기 위해 세마포어와 뮤텍스를 사용한다.(공유된 자원에 하나의 프로세스만 접근할 수 있도록)

## 종류

### 공유 메모리
데이터 자체를 공유하도록 만드는 방식
프로세스 사이에 buffer공간을 만들고 data를 in, out하도록한다.
이러한 경우 n:m의 상태에서는 누구에게 보내고 누구로부터 받을지 명확하게 알수없는 문제가 발생한다.

### Message Passing
create, send, receive, delete 기능만 호출하면 OS가 알아서 동작한다.
direct 방식
보내는 쪽과 받는쪽이 정확하게 명시적으로 Link된다.
Producer와 Consumer 1:m 매칭 형식이다.
indirect 방식
보내는 쪽에 대한 명시만 존재한다. 받는쪽에서는 보낸곳의 포트를 통해서 data recive syscall을 한다.
Producer와 Consumer n:m 매칭이 가능하다.

blocking i/o(synchronous)
buffer size가 보내고자 하는 data보다 작을 경우 나눠서 보내게 되는데, 이때 recieve를 통해서 buffer가 비워질때까지 Consumer가 waiting하고 있는 방식
non-blocking i/o(asynchronous)
buffer size가 보내고자 하는 data보다 작을 경우 나눠서 보내게 되는데, 이때 recieve를 통해서 buffer가 비워질때까지 Consumer가 자신의 일을 하고있는 상태

### Signals
특정 이벤트를 프로세스에게 알리는 IPC의 메커니즘.
이는 동기적으로(ex. 잘못된 메모리 접근), 비동기적으로(ex. 중단) 발생할수 있다.
software interrupt로 생각할 수 있다. Signal은 특정 이벤트에 의해 생성되어 프로세스에 전달된다.
이때 signal handler가 signal을 처리하게된다.
모든 signal은 신호 처리 시 커널모드가 실행되는 기본 핸들러가 존재한다. 사용자 정의 signal handler로 재정의 할 수 도 있다.
프로세스는 signal을 다른 프로세스에게 보낼 수 있다.
네트워크 상의 원격 프로세스에 접근하여 프로시저 똔느 함수를 호출하여 사용할 수 있는 방법이다.

### PIPE
파이프는 두 프로세스가 통신할 수 있는 통로이다.
초기 UNIX 시스템에서 최소의 IPC 메커니즘 중 하나이다.

**Ordinary Pipe vs Named Pipe**
Ordinary Pipe
전형적인 Producer-Consumer 컨셉의 파이프 방식.
Producer는 파이프의 write-end에서 데이터를 쓸 수 있고, Consumer는 read-end에서 데이터를 읽을 수 있다.(단방향 통신, 두 프로세스 사이에 부모-자식 관계여야한다.)
양쪽으로 송수신을 원할경우 Pipe를 2개 만들어야한다.
매우 간단하게 사용이 가능하지만 이중 통신할 경우 구현이 복잡해진다.

Named Pipe(익명 파이프)
통신할 프로세스를 명확하게 알 수 없을 때 사용하는 방식
통신을 위해 이름있는 파일을 사용한다.
양방향 통신이 가능하다.
읽기/쓰기를 동시에 하는것은 불가능하다. 따라서 전이중 통신을 위해서는 Ordinary Pipe처럼 2개를 만들어야한다.

### Memory Map
공유 메모리처럼 메모리를 공유하는 방식이지만, 파일을 메모리에 맵핑시켜 공유하는 방식이다.(공유 매개체가 파일 + 메모리이다.)
주로 파일같은 대용량 데이터를 공유해야할 때 사용한다.