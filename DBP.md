

## Transaction

트랜젝션은 ACID 를 만족해야함

- Atomicity : 모든 작업이 반영되거나 모두 롤백되는 특성입니다
- Consistency : 데이터는 미리 정의된 규칙에서만 수정이 가능한 특성을 의미합니다. 숫자컬럼에 문자열값을 저장이 안되도록 보장해줍니다.
- Isolation : A와 B 두개의 트랜젝션이 실행되고 있을 때, A의 작업들이 B에게 보여지는 정도를 의미합니다.

- durability(영구성) : 한번 반영(커밋)된 트랜젝션의 내용은 영원히 적용되는 특성을 의미합니다.

#### state

- active : Initial 상태
- Failed : 실패 직후 상태
- Partially commited : 메인메모리에 read/write가 실행된 상태
- commited: disk에 메인메모리의 변경상황이 적용됨
- Aborted : 메인메모리의 변경된 데이터를 롤백
- terminated



### compensating transaction

commit이후에는 transaction rollback이 불가하니 commit된 변경을 돌리는 트랜젝션



# LOCK

### 1. Lock의 길이

1.1.  lock이 길어진다 

자원 선점 시간이 길어진다 -> 병렬처리 성능 저하

prevent inconsistency 

1.2. lock이 짧아진다

병렬처리 성능 상승  

inconsistency 확률 증가



Conflict equivalent: S를 연산을 변경해서 S'으로 만들수 있다면 Conflict Equivalent

Conflict Serializability : sequential한 스케줄과 conflict equivalent한것이 conflict serializablity. 



Lock-based protocol:

- inconsistent #observation1 -> short term일때 발생

- deadlock #observation2

- starvation  #observation3



### Lock-based protocol

2PL을 사용하면 conflict serializability 보장.

deadlock, cacading rollback은 계속 발생

cacading rollback은 strict 2PL을 통해 막을수있음. 

#### Strict 2PL

Lock-X에 대해서는 Commit 직전에 unlock을 배치함으로써 cacading rollback을 막을 수있다.

대신 Concurrency 성능이 하락한다.

#### rigourous 2PL

Lock-X뿐만아니라 Lock-S도 commit전까지 풀지않는다

#### 2PL with lock conversion

2PL의 concurrency 성능이 떨어지는걸방지하기위해

read전에는 lock-s만 걸고

write전에 lock-s(A)가 걸려있다면 upgrade(A), 아무락도 걸려있지않다면 lock-x(A)를 건다.



### Lock Manager

트랜젝션으로 부터 lock요청을 받으면 어떻게 처리할지를 정해준다.

#### lock 

만약 락을 걸수있다면 (lock list가 비어있거나, 현재 부여된lock과 요청이 온 lock이 compatible하다면) permit.

현재 실행중인 lock과 incompatible하다면 awating.

#### unlock

1. acknowledge발생.
2. 다음 transaction에게 자원에대해 permit.



## Graph-based protocol

conflict serializability 보장

high-level concurrency

deadlock free

- commit dependency 사용



## Time-Based protocol

TS(Ti) < TS(Tj)라고할때 (Tj가 더 늦게 시작한 트랜젝션),

READ(X)

- TS(Ti) < TS-WRITE(X)   -> abort 

  나보다 늦게시작한 트렌젝션이 쓴건 안읽는다.

WRITE(X)

- TS(Ti) < TS-READ(X) -> abort

  나보다 늦게시작한 트렌젝션이 읽은거에는 안씀

- TS(Ti) < TS-WRITE(X) -> abort

  나보다 늦게시작한 트렌젝션이 쓴거에는 안덮어씀\

Lock방식이 아니라 deadlock이 없음

starvation도 없음
