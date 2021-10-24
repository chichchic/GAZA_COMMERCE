# query processing
기본 진행과정
1. query parsing -> relation algebra로 변환
2. optimization -> 무엇을 먼저 실행해야 효과적인지 확인(통계 정보를 활용)
3. evaluation(평가)

## measure of query cost
시간에 영향을 주는 요소
- disk access(seek + rotation latency + transfer(block read, block write(read back 때문에 write비용이 더 크다.)))
- cpu
- 네트워크 요소

T_t: time to transfer one block
T_s: time for one seek
b(transfer cost) * T_t + s(seek cost) * T_s => access time cost
항상 최악의 경우로 생각해야한다.

# transation
## ACID
1. atomicity(원자성)
더 이상 부분 tn행될 수 없는 단위. All or Nothing.
2. Durability(지속성)
한번 완료된 transaction의 결과는 permanent(영구히)하게 변경된다.
3. consistency(일관성)
A<->B의 데이터는 일관되게 유지되어야 한다.
4. isolation(고립성, 독립성)
transaction 진행중에 다른 transaction 실행이 불가하다.(다른 transaction의 존재를 몰라야한다.)

## state
Active: 진행중
partially committed: 마지막 instruction 완료 상태
fail: 실패
aborted: 재실행, 종료 가능
committed: 성공

## concurrent execution
동시성 제어 전략: machanisms to achieve isolation
processor와 disk utilization 향상
평균 response time을 감소시킨다.

## schedules
transaction의 실행 순서를 고려한다.
빠른 속도와 동시에 일관성이 보장되어야만 한다.

## serializability(직렬성)
concurrent schedule이 serial schedule과 같다면 serializability하다 할수 있다.

- 데이터 일관성을 위해서는 read/write만 확인하면 된다.

## conflicting serializability
충돌이 발생하는 두 명령이가 순차적으로 실행되면 결국 결과가 동일하게 나온다.(실질적 충돌이 없는것으로 생각 가능)

## testing for serializability
T_i와 T_j가 있을 때 T_i가 우선 write or read 해야할 경우 T_i -> T_j로 표기한다.
만약 순환이 생길경우 serializability가 보장되지 않음을 알 수 있다.

## recoverable schedules
T_i -> T_j이면 commit 순서는 T_i이후 T_j여야 한다.(fail이 발생할 경우를 대비하기 위해서)
T_i가 먼저 commit되는 transaction의 경우만 recoverable하다.

## cascading rollbacks
T_i -> T_j -> T_k 일때 T_i에서 abort가 생기면 전부 roll back 해야한다.(시스템에서 성능상 좋지 못하다.)
따라서 cascadeless schedules를 통해 성능을 향상시켜주는것이 좋다.
**cascadeless schedules**
T_i가 write하고 T_j가 read한다고 할 때
T_j가 read전에 T_i가 먼저 commit할 경우 cascade가 필요 없어진다.

## concurrency control
- 동시 실행과 순차 실행의 결과가 같으면 직렬성이 있는 것이다.
- recoverable / cascadeless를 보장하기 위해 노력해야한다.(궁극적인 목표)
- serializablity를 보장하는 concurrency control protocols를 제작함

**lock-based protocols**
lock을 통해 data 접근을 막아 동시성을 제어한다.
2개의 lock(Transaction 단위로 권한을 가진다.)
- Exclusive(x) Mode
배타적 Lock
혼자서만 X r/w 가능
r보다는 w를 위해 사용(따라서 write lock이라고도 부른다. lock-x)

- Shared(s) Mode
write 불가
읽기만 가능하게 바꾼다.
모두 읽기가 가능하다.(다른 transaction도)
read lock이라고도 부른다.
lock-s

* lock-s는 다른 transaction이 lock을 걸어도 또 다른 transaction이 lock-s가 가능하다.
* lock-x는 lock이 걸려있으면 다른 transaction은 불가능하다.
* lock-s와 lock-x는 동시에 lock할수 없다.
* lock이 걸려있어 접근이 불가능 할 경우 wait 상태가 된다.
(deadlock이 걸릴수 있음을 유의해야한다. starvation또한 일어날수 있다. 하나의 victim(rollback cost가 적은것)을 정해서 해결한다.)

**two-phase locking protocol**
2단계로 나누어 lock을 건다.
concurrency manager가 phase를 관리한다.
1) growing phase
lock을 거는것은 가능하지만 lock을 푸는것은 불가능하다.
2) shirinkin phase
lock을 거는것은 불가능하지만 lock을 푸는것은 가능하다.

final lock을 얻은 순서(lock point 순으로 실행 가능)
여전히 deadlock에서 자유롭지는 않다.

**strict two-phase locking protocol**
lock-x transaction이 commit/abort 하기 전까지 unlock을 수행하지 않는다.(cascading rollback을 예방한다.)

**rigorous two-phase locking protocol**
모든 lock에 대해 commit/abort 전까지 unlock하지 않는다.(deadlock이 예방된다.)

## lock conversions
two phase locking을 사용하되
lock-s -> lock-x, lock-x -> lock-s 와같은 증감 단계를 추가한다.(동시성이 조금 향상된다.)

**dead lock(handling)**
OS에서 해결하는 방식과 유사하다.
Dead Lock의 조건은 4가지 이다.(모두 만족해야만 deadlock)
상호배제: 모든것을 공유하지 않는 배타적 자원을 사용한다.
점유 및 대기: 이미 점유중이면서 요청을 한다.
비선점: 자원 강탈이 허용되지 않는다.
환영대기: circular wait 허용

## timeout scheme
요청 후 일정시간 이상 lock 획득을 실패할 경우 rollback을 시킨다.

## deadlock detection
wait for graph를 사용한다.
V->transaction
E->lock요청
이때 cycle이 발생하면 deadlock이 발생한다.(victim을 결정해서 해결한다.)
* total rollback이 발생할 수 있다.(연쇄적 roll back을 항상 고려해야한다.)
* starvation이 일어날 수 있다.(cost에 victim 이력을 기록한다.)

## transaction failure
logical error
system error
system crash(갑자기 전원이 꺼질 경우)

## recovery algorithm
1. log 확인
2. 복구 후 실행될 조치들(redo or undo)

## data access
physical block -> hard disk
buffer block -> main memory
input main memory <- physical block
output main memory -> physical block
read work area <- main memory
write work area -> main memory

write후 바로 output할 필요는 없다(비효율적)

## recovery and Atomicity
원자성 보장을 위해서는 data 수정 전에 변경사항을 저장해 두어야한다.(log)
log는 stable storage에 저장해야한다.(memory x)
buffer에 먼저 기록한 후 disk로 flush한다.
<T_i, X, V_1(old), V_2(new)> 의 형태로 로그에 기록한다.

transaction이 끝난 후 <T_i commit>기록 이후 commit한다.
1. deferred database modification -> commit 이후 data base 수정
2. immediately database modification -> commit 되기 전에 DB값을 수정한다.

## concurrency control and recovery
여러개의 transaction이 수행될 수 있게 보장해야한다.
-> lock을 통해서 보장이 가능하다.
isolation이 보장되어야한다.
(혼자만 data를 쓰듯이 transaction끼리는 서로를 신경쓰지 않아야한다.)

## redo and undo operation
log에 commit or abort가 없을 경우 undo, 있을 경우 redo
undo: V_1 to X -> 실행순서: log 뒤에서부터
redo: V_2 to X -> 실행 순서: log 앞에서부터
undo 이후 <T_i, X, V>기록 후 <T_i abort>를 기록한다.
redo 이후에는 기록하지 않는다.

## check point
log 정보가 너무 쌓이면 할일이 너무 많아진다.
따라서 주기적으로 <checkpoint L>을 생성한다.
check point를 저장할 때는 모든 updating을 미룬다.
현재 수행중인 transaction을 기록한다.
이후 걸려있는 transaction의 start한 시점부터 log를 이용해 redo/undo를 실시한다.