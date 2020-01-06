# Transaction

## Transaction 4가지 특성

### Atomicity

- 트랜잭션의 작업이 부분적으로 실행되거나 중단되지 않는 것을 보장한다.
- All or Noting의 개념으로서 작업 단위를 일부분만 실행하지 않는다.

### Consistency

- 트랜잭션이 성공적으로 완료되면 일관적인 DB상태(데이터의 타입, 값 등)를 유지한다.

### Isolation

- 트랜잭션 수행시 다른 트랜잭션의 작업이 끼어들지 못하도록 보장한다.
- 즉, 트랜잭션끼리는 서로를 간섭할 수 없다.

### Durability

- 성공적으로 수행된 트랜잭션은 영원히 반영이 된다.
- commit을 하면 현재 상태는 영원히 보장된다.

## Trasaction Isolation Level 4단계

> 낮은 transaction isolation level로 생길 수 있는 문제점
> #### Dirty Read
> - 커밋되지 않은 수정 중인 데이터를 다른 트랜잭션에서 읽을 수 있도록 허용할 때 발생
> #### Non-Repeatable Read (=Inconsistent Analysis)
> - 한 트랜잭션 내에서 같은 쿼리를 두 번 수행할 때 그 사이에 다른 트랜잭션이 값을 수정 또는 삭제함으로써 두 쿼리의 결과가 상이하게 나타나는 비 일관성 발생
> ### Phantom Read
> - 한 트랜잭션 안에서 일정 범위의 레코드를 두 번 이상 읽을 때, 첫 번째 쿼리에서 없던 레코드가 두 번째 쿼리에서 나타나는 현상.
> - 트랜잭션 도중 새로운 레코드가 삽입되는 것을 허용하기 때문에 나타남.
> - 이를 방지하기 위해서는 쓰기 잠금을 걸어야 한다.

### READ UNCOMMITTED

#### 특징
- 각 트랜잭션에서의 변경 내용이 COMMIT이나 ROLLBACK 여부에 상관 없이 다른 트랜잭션에서 값을 읽을 수 있다.
- Consistency에 문제가 많은 격리 수준이기 때문에 사용하지 않는 것을 권장한다.
- Read Committed와는 다른 값이 읽힐 수 있다.
- 일반적으로 그냥 최신 업데이트 값을 읽는다.
- 상당히 위험하다!

#### 문제점
- DIRTY READ

### READ COMMITTED

#### 특징
- Dirty Read와 같은 현상은 발생하지 않는다.
- 실제 테이블 값을 가져오는 것이 아니라 Undo 영역에 백업된 레코드에서 값을 가져온다.
- 같은 트랜잭션에서는 최근의 스냅샷을 읽는다.

#### 문제점
- Non-Repeatable Read
- 이러한 문제는 주로 입금, 출금 처리가 진행되는 금전적인 처리에서 주로 발생한다.
- 데이터의 consistency는 깨지고, 버그는 찾기 어려워 진다.

### REPEATABLE READ

#### 특징

##### MySQL
- MySQL의 기본 동작 모드
- 첫 번째 읽기에 스냅샷을 생성함
- 이후 동일 트랜잭션에서는 스냅샷에서부터 값을 읽음
	- Phantom read가 발생하지 않는다.
- 잠금의 대상은 unique index, secondary index의 유무에 따라 달라짐

##### General
- 트랜잭션이 완료될 때까지 다른 사용자는 그 영역에 해당되는 데이터에 대한 수정이 불가능합니다. 가령, Select col1 from A where {condition}을 수행하였고 이 조건에 해당하는 데이터가 2건이 있는 경우(col1=1과 5) 다른 사용자가 col1이 1이나 5인 Row에 대한 UPDATE이 불가능합니다. 하지만, 이 조건에 해당하는 Row를 INSERT하는 것은 가능합니다. 따라서 동일 트랜잭션에서 Read되는 데이터 수가 달라질 수 있습니다.

#### 문제점
- PHANTOM READ

### SERIALIZABLE

#### 특징
- 가장 단순한 격리 수준이지만 가장 엄격한 격리 수준
- 완벽한 읽기 일관성 모드를 제공
- MySQL에서는 모든 SELECT문에 shared lock(Read lock)이 걸린다.
- MySQL에서는 

#### 문제점

- 성능 측면에서는 동시 처리성능이 가장 낮다.
	- 데이터베이스에서 거의 사용되지 않는다.


## Transaction 관리 팁

- autocommit을 끄자 (특히 JDBC 등에서 주의)
- 긴 트랜잭션은 데드락의 원인
- 배치 작업 중간에 커밋을 하자
- 아무것도 하지 않는 트랜잭션 / 커넥션의 주의!
- 트랜잭션 중간에 사용자 입력이 존재하면 안됨!
- 서버 모니터링은 주기적으로

[ref1](https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation)
[ref2](https://victorydntmd.tistory.com/129)
[ref3](http://egloos.zum.com/ljlave/v/1530887)
[ref4](https://effectivesquid.tistory.com/entry/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-Isolation-Level)