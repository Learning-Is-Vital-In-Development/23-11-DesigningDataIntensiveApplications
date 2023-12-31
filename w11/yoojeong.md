## 트랜잭션

- 트랜잭션의 개념
  - 몇개의 읽기와 쓰기를 하나의 논리적 단위로 묶는 방법
  - 전체가 성공(commit) 하거나 실패(abort, 롤백)
- 트랜잭션의 사용 이유
  - 신뢰성(내결함성)을 지닌 시스템 구현하기 위한 다양한 문제들을 단순화하는 메커니즘으로 채택
  - 오류 부분적인 실패를 걱정할 필요가 없다.
  - 동시성 문제의 해결할 수 있다.
  - 어떤 경우는 트랜잭션을 완화하거나 사용않는 것이 안전/성능 상의 이점이 있다
- 논의해봐야 할 문제
  - 동시성 문제
  - race condition
  - 격리 수준의 구현 : read commited, snapshot isolation, serialization


### 애매모호한 트랜잭션의 개념

- 전통적인 RDB : "값진 데이터"가 있는 "중대한 애플리케이션"에 필수적인 요구사항이다.
- 새로 등장한 NoSQL : 분산환경에서 높은 성능과 고가용성을 유지하려면 트랜잭션을 포기해야한다. 트랜잭션은 위 두 관점보다 훨씬 약한 보장이다.

#### ACID의 의미

> 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)

1. 원자성(Atomicity)

여러 쓰기 작업이 하나의 원자적인 트랜잭션으로 묶여 있는데 결함 때문에 커밋(commit)될 수 없다면 abort되고 데이터베이스는 이 트랜잭션에서 지금까지 실행한 쓰기를 무시하거나 취소해야 한다.

2. 일관성(Consistency)

일관성이란 단어는 여러 의미로 쓰인다. 

- 5장의 복제 일관성(replica consistency)과 비동기식으로 복제 시스템의 최종적 일관성(eventual consistency) 문제
- 일관성 해싱은 어떤 시스템들에서 재균형화를 위해 사용되는 파티셔닝 방법
- CAP 정리에서 일관성이란 단어는 선형성(linearizability)을 의미
- **ACID의 일관성은 불변식이 유효한 데이터베이스에서 시작하고 트랙잭션에서 실행된 모든 쓰기가 유효성을 보존한다면 불변식이 항상 만족해야 한다는 의미**


일관성은 실제로는 데이터베이스 속성이 아닌 애플리케이션의 속성이다.<br>
일관성의 아이디어는 애플리케이션의 불변식에 의존하며 일관성을 유지하도록 트랜잭션을 올바르게 정의하는 것도 애플리케이션의 책임이다.

3. 격리성(Isolation)

> race condition
![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/9a24a9d1-e26b-4751-b355-854915c34bcb)


격리성은 트랜잭션은 다른 트랜잭션을 방해 할 수 없는 것을 말한다. 즉, 여러 트랜잭션이 동시에 실행되었더라도 그 결과가 순차적으로 실행되엇을 때와 결과가 동일해야함을 말한다

직렬성 격리(serializable isolation)는 성능 손해를 동반하므로 현실에서는 거의 사용되지 않는다.<br>
오라클의 경우 직렬성이라는 격리 수준은 있지만 실제로는 직렬성보다 보장이 약한 스냅숏 격리를 구현한 것이다.

4. 지속성(Durability)

지속성은 트랜잭션이 성공적으로 커밋됐다면 하드웨어 결함이 발생하거나 데이터베이스가 죽더라도 트랜잭션에서 기록한 모든 데이터는 손실되지 않는다는 보장이다.<br>
지속성을 보장한다는 의미는 commit 전 다른 노드의 복제완료까지를 의미한다.

- 단일 노드 데이터베이스에서 지속성은 일반적으로 데이터가 하드디스크나 SSD 같은 비휘발성 자장소에 기록됐다는 뜻이다.
- 복제 기능이 있는 데이터베이스에서 지속성은 데이터가 성공적으로 다른 노드 몇 개에 복사됐다는 것을 의미한다.

#### 단일 객체 연산과 다중 객체 연산

1. 격리성 위반

격리성은 사용자2가 삽입된 이메일과 갱신된 개수를 모두 보거나 모두 보지 못하게 하고 일관성이 깨진 중간 지점은 없게 해준다

> dirty read
![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/8016fcd2-ae69-489b-a7f7-53660db6e813)


2. 원자성의 필요성

어느 시점에서 오류가 발생하면 우편함의 내용과 읽지 않은 메시지 개수가 동기화되지 않을 수 있다
원자적 트랜젝션은 개수 갱신이 실패하면 트랜젝션이 어보트되고 삽입된 이메일은 롤백된다

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/829ee886-e57d-456c-a5f5-b0007dbed143)


### 완화된 격리 수준

동시성 문제(경쟁조건)는 트랜잭션이 다른 트랙잭션에서 동시에 변경한 데이터를 읽거나 두 트랜잭션이 동시에 같은 데이터를 변경하려고 할 때만 나타난다.
떄문에 테스트로 발견,재현 및 추론이 어렵고, 데이터베이스는 오랫동안 트랜잭션 격리를 제공함으로써 애플리케이션 개발자들에게 동시성 문제를 감추려고 했다.

직렬성 격리는 여러 트랜잭션이 직렬적으로 실행되어 동일한 결과를 얻음을 보장하지만, 현실에서는 성능 비용이 있고 많은 데이터베이스들은 어떤 동시성 이슈로부터는 보호해주지만 모든 이슈로부터 보호해주지는 않는, 완화된 격리 수준을 사용한다.

#### 커밋 후 읽기

- 더티읽기(dirty reads)
  - 다른 트랜잭션에서 커밋되지 않은 데이터를 볼 수 있으면 이를 더티 읽기라고 부른다.
  - 문제점
    - 부분적으로 갱신된 상태를 보는 것은 사용자에게 혼란을 주며 다른 트랜잭션들이 잘못된 결정을 하는 원인이 될 수도 있다.
    - 트랜잭션이 abort되면 그때까지 쓴 내용은 모두 rollback 돼야 한다.
- 더티쓰기(dirty writes)
  - 먼저 쓴 트랜잭션이 commit/abort 될 때까지 두번째 쓰기를 지연시키는 방법을 사용한다.
  - 트랜잭션이 다중객체를 갱신하면 나쁜결과를 유발할 수 있는데, 커밋 후 읽기는 이러한 사고를 막아준다.
  - 하지만, 커밋 후 읽기는 카운터 증가 사이의 경쟁조건은 막지못한다.


커밋 후 읽기는 오라클 11g, 포스트그레스큐엘, SQL 서버 2012, 멤SQL(MemSQL) 등 많은 데이터베이스에서 기본 설정이다.

- DB에서 읽을 때 커밋된 데이터만 보게 된다 (no dirty read)
- DB에 쓸 때 커밋된 데이터만 덮어쓰게 된다 (no dirty write)

  
가장 흔한 방법으로 데이터베이스는 row lock 을 사용해 더티 쓰기를 방지한다. 트랜잭션에서 특정 객체(로우나 문서)를 변경하고 싶다면 잠금을 획득해야 하고 트랜잭션이 커밋되거나 어보트될 때까지 잠금을 보유하고 있어야 한다. 오직 한 트랜잭션만 어떤 주어진 객체에 대한 잠금을 보유할 수 있다.

쓰여진 모든 객체에 대해 데이터베이스는 과거에 커밋된 값과 현재 쓰기 잠금을 갖고 있는 트랜잭션에서 쓴 새로운 값을 기억하여 더티 읽기를 방지한다. 해당 트랜잭션이 실행 중인 동안 그 객체를 읽는 다른 트랜잭션들은 과거의 값을, 새 값이 커밋된 후에는 새 값을 읽을 수 있게 된다. 읽기 잠금은 읽기만 실행하는 여러 트랜잭션들이 오랫동안 실행되는 쓰기 트랜잭션 하나가 완료될 떄까지 기다려야 하기 떄문에 운용성이 나빠 현실에서는 잘 동작하지 않는다.


#### 스냅숏 격리와 반복 읽기

> 비반복읽기(nonrepeatable read) or 읽기 스큐(read skew)
![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/b714a10d-2e7c-48ad-acff-ee360dcb2260)

100달러를 계좌1에서 계좌2로 이체하는 트랜잭션이 처리되고 있는 순간에 앨리스가 계좌잔고를 보게되면 일시적으로 100달러가 사라진 것 처럼 보인다.

이런 일시적인 비일관성은 nonrepeatable read(비반복읽기) 혹은 read skew(읽기 스큐)라고 하며, 감내할 수 있는 정도의 문제이다.<br>
그런데 이러한 일시적인 관성을 감내하기 어려운 경우가 있다.(data freeze가 필요)

- 백업
- 분석 질의와 무결성 확인(데이터 무결성 검증 또는 오염 모니터링)


이를 위해 `스냅숏 격리`를 사용한다. 각 트랜잭션은 일관된 스냅숏으로 부터 읽는다.

> 구현
![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/9ca2f92c-84d6-445c-ad11-42925018524a)


- Multi-version concurrency control(MVCC): 데이터베이스가 객체의 여러 버전을 유지한다.
- 스냅숏 격리를 지원하는 저장소 엔진은 보통 커밋 후 읽기 격리를 위해서도 MVCC를 사용한다.
- 전형적인 방법은 커밋 후 읽기는 질의마다 독립된 스냅숏을 사용하고 스냅숏 격리는 전체 트랜잭션에 대해 동일한 스냅숏을 사용한다.

반복 읽기는 오라클에서는 직렬성(serializable), PostgreSQL과 MySQL에서는 반복읽기(repeatable read)라고 한다. <br>
이렇게 이름이 혼란스러운 이유는 SQL 표준에 스냅숏 격리의 개념이 없기 때문이다. SQL 표준은 1975년의 시스템 R의 격리 수준 정의 기반으로 하고 그 당시에는 스냅숏 격리가 발명되지 않았다. 대신 비슷해보이는 반복 읽기는 SQL 표준에 정의돼 있다.


#### 갱신 손실 방지

read-modify-write 주기에서 발생한다. 만약 두 트랜잭션이 이 작업을 동시에 하면 두번째 쓰기 작업이 첫번째 변경을 포함하지 않으므로 변경 중 하나는 손실될 수 있다.

갱신 손실을 방지하는 법은

- `원자적 쓰기`: 보통 객체를 읽을 때 그 객체에 독점적인(exclusive) 잠금을 획득해서 구현한다. 그래서 갱신이 적용될 때까지 다른 트랜잭션에서 그 객체를 읽지 못하게 한다(cursor stability). 다른 선택지는 그냥 모든 원자적 연산을 단일 스레드에서 실행되도록 강제하는 것이다. 원자적 연산이 사용될 수 있는 상황에서는 보통 최선의 선택이다.
- `명시적 잠금`: 애플리케이션에서 갱신할 객체를 명시적으로 잠근다. 갱쟁조건을 유발하지 않도록 신중하게 개발되어야 한다.
- `갱신 손실 자동 감지`: read-modify-write 주기를 순차적으로 허용하는게 아니라 병렬 실행을 허용하고 트랜잭션 관리자가 갱신 손실을 발견하면 트랜잭션을 어보트시키고 read-modify-write 주기를 재시도하도록 강제하는 방법이다. 자동으로 갱신 손실이 감지되어 오류가 덜 발생하게 해준다.
- `Compare-and-set`: 값을 마지막으로 읽은 후로 변경되지 않았을 때만 갱신을 허용함으로써 갱신 손실을 회피한다. 데이터베이스가 오래된 스냅숏으로 부터 읽는 것을 허용한다면 갱신 손실을 막지 못할 수 있다. 따라서 데이터베이스의 compare-and-set 연산에 의존하기 전에 먼저 안전한지 확인해야 한다.

복제가 적용된 데이터베이스에서 갱신 손실을 막는 것은 차원이 다른 문제이다.

- 데이터의 최신 복사본이 하나만 있다고 가정되지 않는 상황에서는 잠금과 compare-and-set 연산을 적용할 수 없다.
- 복제가 적용된 데이터베이스에서 흔히 쓰는 방법은 쓰기가 동시에 실행될 때 한 값에 대해 여러 개의 충돌된 버전(sibling)을 생성하는 것을 허용하고 사후에 애플리케이션 코드나 특별한 데이터 구조를 사용해 충돌을 해소하고 이 버전을 변합하는 것이다.
- 원자적 연산은 복제 상황에서도 잘 동작한다. 특히 교환 법칙(즉 다른 복제본에 다른 순서로 연산을 적용해도 같은 결과나오는 경우)이 성립하는 연산이라면 그렇다.
- Last write wins(LWW) 충돌해소 방법은 갱신손실 문제가 발생하기 쉽지만 많은 복제 데이터베이스의 기본설정이다.

#### 쓰기 스큐와 팬텀

쓰기 스큐는 두 트랜잭션이 같은 객체들을 읽어서 그 중 일부를 갱신할 때 나타날 수 있다(다른 트랜잭션은 다른 객체를 갱신한다).<br>
다른 트랜잭션이 하나의 동일한 객체를 갱신하는 특별한 경우에 (타이밍에 따라) 더티 쓰기나 갱실 손실 이상 현상을 겪게 된다.

> 회의실 예약 시스템, 사용자명 획득, 이중 사용(dobule-spending) 방지...
![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/5246c467-3c1c-4ea1-9885-c7362a909f27)

쓰기 스큐 패턴

1. SELECT 질의가 어떤 검색 조건에 부합하는 로우를 검색함으로써 어떤 요구사항을 만족하는지 확인한다.
2. 첫번째 질의의 결과에 따라 애플리케이션 코드는 어떻게 진행할지 결정한다.
3. 애플리케이션이 계속 처리하기로 결정했다면 데이터베이스에 쓰고(INSERT, UPDATE, DELETE) 트랜잭션을 커밋한다.

1단계의 로우를 잠금으로써(SELECT FOR UPDATE) 트랜잭션을 안전하게 만들고 쓰기 스큐를 회피할 수 있다. 하지만 1단계의 질의가 아무 로우도 반환하지 않으면 `SELECT FOR UPDATE`는 아무 것도 잠글 수 없다.

어떤 트랙잭션에서 실행한 쓰기가 다른 트랜잭션의 검색 질의 결과를 바꾸는 효과를 팬텀(phantom)이라고 한다.

해결법

- 제약 조건을 설정할 수 있다(e.g. 유일성, 외래키, 특정값 제약조건 등). 트리거나 구체화 뷰(material view)를 사용해 구현할 수 있다.
- 직렬성 격리 수준을 사용할 수 없다면 트랜잭션이 의존하는 로우를 명시적으로 잠그는 것이 차선책이다(SELECT FOR UPDATE). FOR UPDATE는 데이터베이스에게 이 질의가 반환하는 모든 로우를 잠그라고 지시한다.
- 충돌 구체화(materializing conflict): 팬텀을 데이터베이스에 존재하는 구체적인 로우 직합에 대한 잠금 충돌로 반환한다(e.g. 잠글 대상을 만들기 위해 회의실과 시간 범위에 해당하는 로우를 미리 만듦). 충돌 구체화하는 방법을 알아내기 어렵고 오류가 발생하기 쉽다. 또한 동시성 제어 메커니즘이 애플리케이션 데이터모델로 새어 나오는 것도 좋지 않다.
- 직렬성(serializability)
