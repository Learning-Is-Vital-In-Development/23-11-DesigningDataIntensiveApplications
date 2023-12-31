`5장의 핵심 내용`

- 노드 간 변경을 복제하기 위한 세 알고리즘 : **단일 리더**, **다중 리더**, **리더 없는** 복제
- 복제 시 고려해야 할 트레이드오프 : 동기식 vs 비동기식, 잘못된 복제본 처리 방법.
- 복제 지연 문제 : 쓰기 후 읽기 일관성 & 단조 읽기 보장 & 일관된 순서로 읽기 보장

---

## 들어가기에 앞서

1부에서는 DB가 1개 일때만 생각했는데, 2부에서는 DB가 여러개인 케이스들을 볼것이다.

이번 장에서는 데이터 셋이 아주 작아 각 장비에 전체 데이터셋의 복사본을 보유할 수 있다고 가정하여 **복제본**을 만들었을 때의 **다양한 종류의 장애**와 **대처 방법**에 대해 다룰 것이다. 
(데이터의 용량, 컴퓨터의 용량 한계는 고려하지 않고 장비 1대에 모든 데이터를 저장할 수 있다는 가정이다.) 

# 0. 복제

**복제란?** 네트워크로 연결된 여러 장비(노드)에 동일한 데이터의 복사본을 유지한다는 의미. 

**복제의 필요성**

- 지연 시간 감소 : 사용자와 가까운 지리적 위치
- 고가용성 : 시스템 일부 장애 발생해도 지속적 동작 가능
- 읽기 처리량 증가 : 읽기 질의 제공 장비 수 확장

**복제 알고리즘**

- 단일 리더(single-leader)
- 다중 리더(multi-leader)
- 리더가 없음(leaderless)

# 1. 단일 **리더 기반 복제(leader-based replication)**

### 1-0. 리더와 팔로워

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85903675-8147-479c-a855-6c17bf86398f/Untitled.png)

**`복제 서버(replica)`**

- 데이터 베이스의 복사본 저장하는 각 노드
- 이때 모든 복제 서버에 모든 데이터가 있다는 사실을 보장하는 방법은?
    - **통칭** : 
    리더 기반 복제(leader-based replication) = 능동(active)
    수동(passive) = 마스터 슬레이브(master slave)
    - `리더`(leader), 마스터/프라이머리(primary) : 복제 서버 중 하나
        - 예) 클라이언트가 데이터 베이스에 쓰기 요청 → 요청은 리더에게 감 
        → 리더는 로컬 저장소에 새 데이터 기록
    - `팔로워`(follwer),읽기 복제 서버(read replica), 슬레이브 2차(secondary), 핫대기(hot standby) 
    :다른 복제 서버
        - 예) 리더가 로컬 장소에 새로운 데이터 기록할 때마다 
        → 복제로그(replicationlog)/변경스트림(change stream)의 일부로 팔로워에게 전송
        → 각 팔로워가 리더로부터 로그 받으면 리더가 처리한 것과 동일한 순서로 모든 쓰기 적용
        → 이에 맞게 데이터베이스 로컬 복사본 갱신 처리

## 1-1.동기식 / 비동기식 / 반동기식 복제

| 동기식 | 비동기식 | 반동기식 (semi-synchronous) |
| --- | --- | --- |
| - 팔로워 변경 완료까지 대기
(장점) 팔로워-리더 간 데이터 일관성. 리더 장애 시 팔로워 데이터 신뢰로 사용 가능.
(단점) 팔로워가 응답이 없으면 쓰기가 처리되지 않는다 | - 팔로워 응답 대기 X
(장점) 팔로워 -리더 간 장애 격리
(단점) 리더 장애 시 데이터 유실 가능성 존재 
내구성 약하나, 많은 팔로워 또는 지리직 분산 시 비동기식 복제 사용. | - 팔로워1 동기식/팔로워2 비동기식
- 적어도 두 노드(리더 & 동기식 팔로워)에 데이터 최신 복사본 보장 |

`동기식 vs 비동기식`

- **동기식 복제는 한 노드의 장애가 전체 시스템을 멈추게 하므로 비현실적이다.**
- 현실적으로 **팔로워 하나만 동기식**으로, 나머지는 비동기식으로 복제하는 방식을 사용할 수 있다.
    - 이 방식이 **`반동기식(semi-synchronouse)`**이라고 한다.
- **보통 리더 기반 복제는 팔로워를 지리적으로 분산하여 단점을 보완하고 완전히 비동기식으로 구성된다.**

## 1-2.새로운 팔로워 설정

새로운 팔로워가 리더의 데이터 복제본을 정확히 갖고 있는지 어떻게 보장하는가?

- (리더의 데이터베이스는 지속적으로 추가/변경/삭제 등이 일어나고 있음) 
DB의 쓰기를 잠그고 복제한다면, 일관성은 획득하나, 고가용성은 희생하게 된다.
    
    > 고가용성(high availability,HA)
    시스템이 장애가 나는 상황에서도 사용 가능하도록 유지하는 능력을 의미
    > 
    > 
    > ![위 가용성이 굉장히 높은 경우(>= 99.9999..) 또는 이에 근접하다면 시스템이 고가용성을 지녔다고 표현한다.](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/100829f5-54a9-4321-b69d-bd03de3374f5/Untitled.png)
    > 
    > 위 가용성이 굉장히 높은 경우(>= 99.9999..) 또는 이에 근접하다면 시스템이 고가용성을 지녔다고 표현한다.
    > 

따라서 가용성을 위해 중단 없이 팔로워를 설정하려면 다음과 같이 한다.

1. 리더의 데이터베이스 일정 시점 리더  DB 스냅샷 저장.
2. 스냅샷을 새로운 팔로워 노드에 복사
3. 팔로워는 리더에 연결해 스냅샷 이후 변경분 요청. 
(로그의 정확한 위치 필요. eg. MySQL의 binlog coordinate, PostgresQL의 log sequence number)
4. 팔로워가 스냅샷 이후 데이터 변경의 미처리분(backlog)를 
모두 처리했을 때 "따라잡았다(caught up)"고 보고, 
리더에 발생하는 데이터 변화 처리 가능.

## 1-3. 노드 중단 처리 ***(Handling Node Outages)***

**목표**: 개별 노드의 장애에도 전체 시스템이 동작하게끔 유지하고 노드 중단의 영향을 최소화하는 것

- **장점**: 중단 시간 없이 개별 노드를 재부팅할 수 있다.(운영과 유지보수에 장점)

리더 기반 복제에서 고가용성은 어떻게 달성하는가?

- `팔로워 장애`(따라잡기 복구), `리더 장애`(장애 복구)
    - `팔로워 장애`
    (장애 복구 과정)
        - 팔로워 장애 시 마지막으로 처리한 트랜잭션 조회. 이 후 변경분에 대해 팔로워는 리더에 요청.
    - `리더 장애` → 자동 장애 복구 기능이 있더라도, 수동으로 장애 복구를 하는 쪽을 선호하는 경우도 많다.
    (장애 복구 과정 - **수동**)
        - 팔로워 중 하나를 새로운 리더로 삼아야 한다.
        - 클라이언트는 새로운 리더로 쓰기를 전송해야 하므로 재설정이 필요.
        - 다른 팔로워는 새로운 리더를 바라봐야 한다.
        
        (장애 복구 과정 - **자동**)
        
        - 리더가 장애인지 판단한다.
            - 판단할 수 있는 확실한 방법은 없음 → 보통 타임아웃을 사용
            - 노드 간 메시지를 주고 받고 일정시간 응답하지 않는 노드는 죽은 것으로 간주
                - 예외도 존재, 리더가 계획된 유지 보수를 위해 의도적으로 중단되는 경우
        - 새로운 리더를 선택한다.
            - 복제 노드들이 새로운 리더를 선출
            - 또는 제어 노드(controller node)가 새로운 리더를 임명
            - 최신 데이터 변경사항을 가진 복제 서버가 새로운 리더의 가장 적합한 후보로 지목된다.
        - 새로운 리더 사용을 위해 시스템을 재설정한다.
            - 클라이언트의 쓰기 요청, 팔로워의 데이터 변경 로그 재설정
            - 이전 리더가 복구되는 경우 이전 리더가 새로운 리더를 인식하고 자신은 팔로워가 된다.
        
        (문제점)
        
        - 스플릿 브레인(Split Brain)
        두 노드가 모두 자신이 리더라고 믿는 문제
        - 내구성을 보장하지 X 
        비동기식 복제 사용 시 새로운 리더는 이전 리더가 실패하기 전에 이전 리더의 쓰기를 일부 수진 못할 수 있음.
        - 죽었다고 판단하기에 적절한 타임아웃 값을 정하기가 어렵다.
            - 타임아웃이 길면 → 복구에 너무 오랜 시간이 소요
            - 타임아웃이 짧으면 → 불필요한 장애복구 발생

## 1-4. 복제 로그 구현

**리더 기반 복제의 다양한 복제 방법**

| 복제 방법 특징 | 복제 방법 | 정의 | 설명 |
| --- | --- | --- | --- |
| 애플리케이션의 관여 없이 DB 시스템에 의해 구현 | 구문(Statement) 기반 복제(statement-based replication) | 요청받은 구문을 기록하고 쓰기를 실행한 다음 구문을 팔로워에게 전송 | - 서버 간의 비결정적 함수 결과값 차이 때문에 복제가 깨질 수 있다. (NOW(), RAND() 등은 복제 서버마다 다른 값을 생성할 가능성이 존재)

- 부수 효과 존재 구문(트리거, 스토어드 프로시저, 사용자 정의 함수) : 다른 결과 발생 가능성 존재

- 자동증가 칼럼 또는 DB의 데이터에 의존(update .. where …) 시 정확히 같은 순서 실행 보장되어야 함. |
| 애플리케이션의 관여 없이 DB 시스템에 의해 구현 | 쓰기 전 로그
(WAL, write-ahead log) 배송 | 로그 구조화 저장소 엔진의 경우 로그 자체가 저장소의 주요 부분. 로그 세그먼트는 작게 유지되고 백그라운드로 가비지 컬렉션. | - 개별 디스크 블록에 덮어쓰는 B 트리의 경우 모든 변경은 쓰기 전 로그(Write-ahead log, WAL)에 쓰기 때문에 고장 이후 일관성 있는 상태로 색인 복원 가능.
MySQL Inno DB 엔진에서의 WAL은 Redo Log

- 리더는 디스크에 로그 기록 + 팔로워에게 네트워크로 로그 전송. 팔로워는 해당 로그 처리 통해 리더와 동일한 복제본 생성.

(단점) 
로그가 제일 저수준의 데이터를 기술. 저장소 엔진에 의존적. 저장소 형식 변경 시 문제 발생. |
| 애플리케이션의 관여 없이 DB 시스템에 의해 구현 | 논리적(로우 기반) 로그 복제(row-based replication) | 로그를 저장소 엔진과 분리하기 위한 대안으로 복제와 저장소 엔진에 각기 다른 로그 형식을 사용한다. |  |
| 복제 방식에 유연성이 요구되며 애플리케이션이 관여 | 트리거 기반 복제 | - 사용자 정의 애플리케이션 코드를 등록할 수 있다.
- 데이터 변경 시(쓰기 트랜잭션) 자동으로 실행된다.
- 트리거를 통해 데이터 변경을 분리된 테이블에 로깅한다.
- 이 테이블에 기록된 데이터 변경을 외부 프로세스가 읽고 처리한다.
- 필요한 애플리케이션 로직 적용 후 다른 시스템에 데이터 변경을 복제한다.
ex. Oracle의 Databus, PostgresQL의 Bucardo |  |

## 1-5. **복제 지연 문제**

**읽기 확장(read-scaling) 아키텍처 → 하나의 리더와 여러 팔로워로 구성**

단일 노드에 **쓰기(리더)** & 복제 노드에서 **읽기(팔로워)** : 읽기 확장 아키텍처. 읽기 처리량을 증가시키는 방법. 
사실상 **`비동기 팔로워`**에서만 동작 가능한 방법. 수많은 팔로워 노드 복제를 기다릴 수 없기 때문.

비동기 복제 방식은 팔로워가 뒤쳐지면 오래된 정보를 볼 가능성이 존재한다. 즉, 리더에서 팔로워 데이터 반영까지 지연이 있을 수 있음(**복제 지연**). 
하지만 이런 불일치는 일시적인 상태이고, 팔로워가 결국 따라잡아 리더와 일치하게 된다. => `최종적 일관성`.

> **최종적 일관성 (Eventual Consistency)**
분산 컴퓨팅 환경에서 사용되는 일관성 모델 중 하나. 일시적으로는 데이터의 일관성이 깨지는 것을 허용한다.
그러나 최종적으로는 (데이터의 변경사항이 없다면) 데이터 대한 모든 접근들에 대해 마지막으로 갱신된 값을 반환하는 것을 보장한다.
> 

**복제 지현 발생 사례 세 가지**

1. 자신이 쓴 내용 읽기 (read-after-write consistency(read-your-write))

> 
> 
> - 자신이 쓴 내용을 바로 다시 읽기 했을 때 복제 서버엔 쓰기가 반영되지 않아 쓰기 전 데이터를 볼 가능성이 존재한다. → 최종적으로 반영됨(eventual consistency)
> → 데이터 변경 직후에는 사용자가 보기엔 데이터가 유실된 것처럼 보이기 때문에 문제가 있다.
> - 따라서 이러한 문제 방지를 위해 **`쓰기 후 읽기 일관성(자신의 쓰기 읽기 일관성)`은 자신이 갱신한 내용에 대해서는 일관성 보장이 필요. 단, 다른 사용자에 대해서는 일관성을 보장하지 않는다.** 
> → 이러한 보장을 통해 자신의 입력이 올바르게 저장됐음을 보장할 수 있게 된다.
> - 구현방법
>     - 사용자가 수정한 내용은 리더로만 읽게 하며, 다른 사용자 프로필은 팔로워에서 읽도록 하는 방법
>         - 실제로 무엇이 수정되었는지 파악할 방법이 필요한데 소셜 네트워크에서 사용자 프로필 정보 같은 경우 자신만 수정이 가능하므로 자신의 프로필은 무조건 리더에서 읽게하는 규칙
>         - 여러 사용자가 편집 가능한 경우라면 마지막 갱신 시각을 찾아서 1분 내에 있는 갱신은 리더에서 읽고 복제 지연이 1분이 넘는 경우 팔로워에 질의를 금지하도록 하는 방법
>     - 클라이언트는 최근 쓰기 타임스탬프를 알 수 있으므로 클라이언트의 타임스탬프를 통해 복제 서버가 타임스탬프까지 따라잡을 때 까지 질의를 대기
1. 단조(****Monotonic)**** 읽기

> **사용자가 시간이 꺼꾸로 흐르는 현상을 목격할 수 있다.**
> 
> - 복제 서버 A, B 중 A서버에만 동기화가 되어있는 시점에 사용자가 처음엔 A 서버( 빠른 팔로워 DB)를 통해 데이터를 읽을땐 데이터가 반환되지만 그 다음에 B 서버(비교적 느린 팔로워 DB)를 통해 데이터를 읽으면 데이터가 반환되지 않을 것이다.
> - 자신이 쓴 내용 읽기 문제와의 차이점은 '저장 내역을 한 번이라도 본 적이 있다'는 점이다. 분명히 내가 쓴 댓글을 잘 확인했다. 그런데 새로고침을 하니 내 댓글이 사라지고 마치 시간이 거꾸로 간 것처럼 느껴지는 것이다.
> - **`단조 읽기(monotonic read)`**는 이런 종류의 이상 현상이 발생하지 않음을 보장한다.
>     - **각 사용자는 항상 동일한 복제 서버에서 수행되게끔 하면 이를 해결할 수 있다.**
>     → 예) 각 사용자별로 무조건 하나의 팔로워 DB만 접근하도록 구현하는 방법
1. 일관된 순서 읽기

> 
> 
> - 쿼리를 전송한 순서가 꼬일 수 있다.  데이터가 서로 다른 파티션에 저장되고 파티션 리더가 다를 때 실제 시간상으론 query A -> query B 순으로 입력되었지만 
> 리더와 팔로워 간 지연이 일어나면서 실제로 데이터를 보는 사람들인 팔로워들은 query B -> query A 순으로 복제가 될 가능성이 있다.
> - **`일관된 순서로 읽기(Consistent Prefix Read)`**는 이런 종류의 이상 현상이 발생하지 않음을 보장한다.
>     - 쓰기가 특정 순서로 발생하면 이 쓰기를 읽는 모든 사용자는 같은 순서로 쓰여진 내용을 보게 됨을 보장한다. (샤딩과도 연관되어 있음)

## 1-6. **복제 지연을 위한 해결책**

- 최종적 일관성으로 인한 복제 지연이 애플리케이션에 얼마나 영향을 끼치는지 파악할 필요가 있다.
    - 지연에 대한 영향이 크다면 강력한 일관성을 보장할 수 있도록 시스템을 설계해야 한다.
- 앞에서 설명된 방식으로 애플리케이션에서 강력한 보장을 제공할 수도 있지만 애플리케이션에서 다루기엔 복잡하다.
- **트랜잭션**은 이러한 문제에 대한 해결을 데이터베이스 단에서 보장해주지만 **분산 데이터베이스로 전환되면서 많은 시스템이 트랜잭션을 포기했다.**
    - 트랜잭션이 성능과 가용성 측면에서 너무 비싸고, 확장 가능한 시스템에서는 어쩔 수 없이 최종적 일관성을 사용해야한다는 주장이 존재.

# 2. **다중 리더 복제**

단일 리더 방식은 리더가 단일 장애점이 되기 때문에, 가용성을 위해 노드를 하나 이상 두는 것이 합리적이다. 

쓰기 처리를 하는 각 노드는 데이터 변경을 모든 노드에 전달하는데 이를 `다중 리더`설정 (마스터 마스터, 액티브/액티브 복제라고도 함)이라고 한다.

- 여기서 각 리더는 동시에 다른 리더의 팔로워 역할도함

## 2-1. 다중 데이터 센터 운영

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/894a0a8f-101c-4b85-98e9-c81ab05c0d6a/Untitled.png)

- 모든 쓰기를 해당 리더를 거쳐야 하고, 리더 연결이 불가능한 경우 쓰기 불가능한 단점 보완
- 각 데이터센터마다 리더 존재
- 단점
    - 동일 데이터를 다른 두 개의 데이터센터에서 동일 변경 가능 -> 쓰기 충돌 반드시 해소해야하는 문제
    - 자동 증가 키, 트리거, 무결성 제약 등 문제 소지 가능성 부분 존재
- 다중 데이터센터에서의 단일 리더 설정과 다중 리더 설정
    
    
    |  | 단일리더 | 다중리더 |
    | --- | --- | --- |
    | 성능 | 쓰기 지연 ⬆ | 지연 ⬇ (성능 Good) |
    | 데이터센터 중단내성 | 다른 데이터센터 팔로워의 리더 승진 | 데이터센터 리더 간 독립성으로 상호 영향 X |
    | 네트워크 | 데이터센터 내 연결에 민감(동기식 사용) | 네트워크 민감성 떨어짐(비동기 사용) |

## 2-2. 오프라인 작업을 하는 클라이언트

- 오프라인으로도 애플리케이션이 동작될 수 있는 경우에도 다중 리더 방식이 사용될 수 있다.
    - 캘린더같은 앱은 인터넷이 연결되어 있지 않아도 언제든지 캘린더 정보를 볼 수 있고 저장할 수 있다.
- **모든 디바이스는 리더처럼 동작하는 로컬 데이터베이스로 볼 수 있다.**
- 디바이스의 인터넷이 연결되면 로컬에 변경된 데이터가 복제 서버로 동기화 된다.
- **아키텍처 관점에서 보면 이 설정은 근본적으로 데이터 센터 간 다중 리더 복제와 동일하다.**

### **쓰기 충돌 다루기**

- **다중 리더 복제의 가장 큰 문제는 쓰기 충돌이다. 이를 위해 충돌 해소가 필요하다.**
    - 각 사용자의 변경이 로컬 리더에 정상 적용되고 동기화될 때 충돌이 발생할 수 있다.
1. 충돌 회피 
    - **충돌을 처리하는 가장 단순한 방법은 충돌을 피하는 것이다.** 충돌 해소는 어렵기 때문에 회피가 가장 간단한 전략이고 권장된다.
    - 특정 레코드의 모든 쓰기는 동일한 리더를 거치도록 처리
2. 일관 상태 수렴
    - 모든 복제 서버가 동일해야 함이 원칙
    - 수렴(convergent) : 모든 변경이 복제돼 모든 복제 서버에 동일한 최종 값이 전달되게 해야 함
3. 충돌 해소
    - 각 쓰기에 고유 ID를 두고 가장 높은 ID를 가진 쓰기를 고르고 나머지는 버리는 방식을 사용할 수 있다. **ID가 타임스탬프라면 최종 쓰기 승리(Last Write Wins)라고 부른다.**
    - 어떻게든 값을 병합하도록 하여 충돌을 해결할 수도 있다.(텍스트를 사전 순으로 정렬하여 병합한다.)
    - 충돌을 모두 기록해 나중에 사용자를 통해 충돌을 해결하도록 한다
4. 사용자 정의 충돌 해소 로직
    - 대부분 다중 리더 복제도구는 애플리케이션 코드를 사용해 충돌 해소 로직 작성 ( 애플리케이션에 따라 적합한 충돌 해소 방법이 다름)
    - 쓰기 수행 중 :
        - 복제된 변경사항 로그에서 데이터베이스 시스템 충돌 감지되면 충돌 핸들러 호출 (백그라운드에서 실행)
    - 읽기 수행 중 :
        - 충돌 감지 시 모든 충돌 쓰기 저장
        - 다음 번 읽기 시 여러 데이터 반환. 애플리케이션은 사용자에게 충돌 내용 보여주거나 자동으로 충돌 해소해 결과를 데이터베이스에 기록 → 카우치DB가 이렇게 동작
5. 자동 충돌 해소
    - 충돌 없는 복제 데이터 타입
        - Set, Map, 정렬 목록, 카운터 등을 위한 데이터 구조의 집합
    - 병합 가능한 영속 데이터 구조
    - Git 처럼 명시적으로 히스토리 추적하고 삼중 병합 함수를 사용한다

### **다중 리더 복제 토폴로지**

- **`복제 토폴로지`**는 쓰기를 한 노드에서 다른 노드로 전달하는 통신 경로를 말한다.
- 리더가 두개라면 다중 데이터센터 운영 구조처럼만 사용할 수 있지만 **리더가 둘 이상이라면 다양한 토폴로지가 가능하다.**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/dc23d780-df5a-49b0-b69b-7f9eb14087f2/Untitled.png)

- **`원형 토폴로지`**는 각 노드가 하나의 노드로부터 쓰기를 받고, 이 쓰기를 다른 한 노드에게만 전달한다.
    - MySQL에서 기본적으로 제공
- **`별 모양 토폴로지`**는 지정된 루트 노드 하나가 다른 모든 노드에 쓰기를 전달한다. 트리로 일반화된다.
    - 노드 장애 시 노드 간 복제 메시지 흐름에 방해를 준다.
- **`전체 연결 토폴로지`**는 모든 리더가 각자의 쓰기를 다른 모든 리더에게 전송한다.
    - 가장 일반적인 토폴로지이다.

**장단점**

- **원형**과 **별 모양 토폴로지**는 하나의 노드에 장애가 다른 노드 간 복제 흐름의 방해를 주기 때문에 **내결함성이 좋지 않다.**
- 전체 연결 토폴로지는 단일 장애점을 피할 수 있어 내결함성이 훨씬 더 좋다.
    - 다만 리더간의 네트워크 연결 속도가 다르다면 **전체 연결 토폴로지는 일부 복제 데이터가 다른 데이터를 추월하여 일관된 순서로 데이터가 복제되지 않을 수 있다.**
- 올바른 이벤트 정렬을 위한 버전 벡터 기법으로 해결 가능

→ 따라서 다중 리더 복제 시스템을 사용하려면 이런 문제를 인지하고 문서를 주의깊게 읽은 다음 데이터 베이스를 철저하게 테스트해봐야 함
