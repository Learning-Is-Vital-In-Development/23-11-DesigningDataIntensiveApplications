
![image](https://github.com/Learning-Is-Vital-In-Development/23-11-DesigningDataIntensiveApplications/assets/60343930/a83a055c-8f5e-40f3-a9f4-e287b348209c)

## 선형성
데이터 복사본이 하나만 있고 데이터 대상으로 수행하는 모든 연산은 원자적인 것처럼 보이게 만드는 것.
### 선형성 시스템
-> 클라이언트가 쓰기를 성공적으로 완료하자마자 
   그 데이터베이스를 읽는 모든 클라이언트는 방금 쓰여진 값 확인 가능
-> 최근 갱신 된 값, 캐시나 복제본 X 
-> 최신성 보장(recency guarantee)

### 비선형 시스탬
-> 스포츠 웹사이트 예시 
    - 한명은 최신 한명은 db복제본 전달돼서 아직 경기 진행중
    - 오랜된 결과를 반환한 건 선형선 위반

###그렇다면 선형성 부여는 어떻게?
-> 시스템에 데이터 복사본이 하나뿐인 것처럼 보이게 만들기
1) 2) 3) 4)

### 선형성/직렬성
- 직렬성 : 모든 트랜잭션이 여러 객체(로우,문서,레코드)를 읽고 쓸 수 있는 상황에서 트랜잭션의 격리 속성
- 선형성 : 레지스터에 실행되는 읽기와 쓰기에 대한 최신성 보장
-> DB는 모두 제공 가능 (엄격한 직렬성 strict serializability, 강한 단일 복사본 직렬성 strong one-copy serializability,)

![image](https://github.com/Learning-Is-Vital-In-Development/23-11-DesigningDataIntensiveApplications/assets/60343930/876987a6-d5a1-4943-bd8e-4b9784e38057)
![image](https://github.com/Learning-Is-Vital-In-Development/23-11-DesigningDataIntensiveApplications/assets/60343930/4e44222a-e2fe-473f-950c-aac550d3c6c4)

### 잠금과 리더 선출 
-> 단일 리더 복제의 리더 split brain X -> 진짜로 1개만
    -> 리더 선출 방법 : 잠금 
        - 모든 노드 시작시 잠금 획득 시도 -> 성공한 노드가 리더
        - 잠금을 어떻게 구현하든지 선형적이어야함.
        - 모든 노드는 어느 노드가 잠금을 소유하는지 동의해야됨.

### 분산 잠금과 리더 선출 구현 => 코디네이션 서비스 [아파치 주키퍼(apache zookeeper) / etcd ].
-> 코디네이션 서비스는 합의 알고리즘을 사용해 선형성 연산을 내결함성이 있는 방식으로 구현 


### 제약조건과 유일성 보장
-> 이메일/사용자명 : 유일한 식별조건 , 제약 조건 강제는 선형성으로 이루어짐
-> 사용자명이 "잠금"을 획득 하는 것, 이는 원자적 compare-and-set과 매우 비슷
-> 모든 노드가 동의하는 하나의 최신값이 있기를 요구 
    ex) 은행 계좌 잔고(음수x), 창고에 있는 재고보다 더 팔리지 않게, 동시에 같은 좌석 예약x
    - 실제 애플리케이션은 때때로 이런 제약 조건을 느슨하게 다뤄도 됨(항공편 오버부킹 시 다른 항공편 옮기고 보상)
-> 그러나 관계형 디비에서 전형적으로 볼 수 있는 엄격한 유일성 제약 조건은 선형성이 필요!
    - 외래키, 속성 제약조건 같은 다른 종류의 제약 조건은 선형성 요구x 구현 가능

###채널간 타이밍 의존성
-> 파일저장 서비스가 선형적이면 이 시스템은 잘 동작함 (비선형일시 경쟁 조건 위험 발생)
-> 웹서버와 크기 변경 모듈 사이에 두가지 다른 통신 채널 (경쟁 조건 비교) 
    - 파일 저장소 / 메세지 큐 , 데이터베이스 복제 / 앨리스 입과 밥의 귀(현실 오디오 채널)
-> 선형성이 경쟁조건을 회피하는 유일한 방법은 아니지만 이해하기에 가장 단순함.
-> (그림) 파일 저장 서비스가 선형적이면 이 시스템을 잘 동작한다. 선형적이지 않으면 경쟁 조건의 위험이 있다.

![image](https://github.com/Learning-Is-Vital-In-Development/23-11-DesigningDataIntensiveApplications/assets/60343930/2dd2357e-15d5-42fb-8915-7cc657e82907)

### 선형성 시스템 구현하기
선형성 시맨틱 제공하는 시스템은 어떻게 구현?
-> 데이터 복사본 하나만 사용하기 : 결함을 이겨낼 수 없음, 하나의 복사본을 저장한 노드에 장애가 나면 
                            데이터가 손실되거나 적어도 노드가 살아날 때까지 접근할 수 없기 때문에
-> 내결함성을 지니도록 만드는 가장 흔한 방법
단일 리더 복제(선형적일 수 있음)
합의 알고리즘(선형적)
다중 리더 복제(비선형적)
리더 없는 복제(아마도 비선형적)

###선형성과 정족수
다이나모 스타일 모델에서 엄격한 정족수를 사용한 읽기 쓰기는 선형적인 것 처럼 보인다.

### 선형성 시스템 구현하기
선형성 시맨틱 제공하는 시스템은 어떻게 구현?
-> 데이터 복사본 하나만 사용하기 : 결함을 이겨낼 수 없음, 하나의 복사본을 저장한 노드에 장애가 나면 
                            데이터가 손실되거나 적어도 노드가 살아날 때까지 접근할 수 없기 때문에
-> 내결함성을 지니도록 만드는 가장 흔한 방법
단일 리더 복제(선형적일 수 있음)
합의 알고리즘(선형적)
다중 리더 복제(비선형적)
리더 없는 복제(아마도 비선형적)

###선형성과 정족수
다이나모 스타일 모델에서 엄격한 정족수를 사용한 읽기 쓰기는 선형적인 것 처럼 보인다.
n(복제 서버) = 3, w(쓰기 노드) = 3, r(읽기 노드) = 2 → 정족수 조건 만족 (w + r > n) 
이지만 선형적이지 않음 성능상 불이익이나 선형성을 만족시킬 수 없기 떄문에 
다이나모 스타일의 복제를 하는 리더 없는 시스템은 비선형적임
![image](https://github.com/Learning-Is-Vital-In-Development/23-11-DesigningDataIntensiveApplications/assets/60343930/430fb1c7-a22c-4b3e-baca-b40383ff1ce9)


### 선형성의 비용
네트워크가 끊겼을 때 
-> 다중 리더 데이터 베이스 : 각 데이터센터는 정상 동작, 한 DC에 쓰인 내용이 비동기로 다른 DC에 복제되므로
                          쓰기는 큐에 쌓였다가 네트워크 연결이 복구되면 전달된다.
-> 단일 리더 설정 데이터 베이스 : 네트워크가 끊기면 팔로워 데이터센터로 접속한 클라이언트 들은 리더로 연결할 수 없으므로
                              아무것도 쓸 수 없고 선형성 읽기도 못함.
### CAP 정리
-> Consistency(일관성), Availability(가용성), Partition tolerance(분단 내성)
선형성 데이터베이스라면 이러한 선형성과 가용성의 트레이드오프 문제가 있음
오직 하나의 일관성 모델(=선형성) 과 한 종류의 결함(네트워크 분단 or 연결이 끊긴 살아있는 노드) 만 고려함
다른 부분인 네트워크 지연, 트레이드 오프 등에 대해 고려하지 않으므로 시스템 설계 시 고려할 실용적인 가치가 없음

-> 선형성을 제거하는 이유는 내결함성이 아니라 성능이다. (내결함성과 성능은 트레이드오프)
-> 최신 다중코어 CPU의 RAM조차 선형적이지 않을 정도로 선형적인 시스템은 드물다.
    - 모든 CPU코어가 저마다 메모리 캐시와 저장 버퍼를 갖기 때문이다. 메모리 접근은 기본적으로 캐시로 가고 변경은 메모리에 비동기 기록
    - 캐시에서 데이터 접근하는게 메인 메모리보다 훨씬 빠르다. 그러나 이렇게 캐시에 데이터복사본이 다양하게 생기면, 
     이러한 복사본은 비동기 갱신이므로 선형성이 손실된다.
------
## 순서화 보장
선형성 레지스터는 데이터 복사본이 하나만 있는 것처럼 동작하고 모든 연산이 어느 시점에 원자적으로 효과가 나타나는 것처럼 보인다고 했다.
-> 이 정의는 연산들이 어떤 잘 정의된 순서대로 실행됨을 암시함. 
-> 순서화, 선형성, 합의 사이에 깊은 연결 관계가 있다. 

### 순서화와 인과성
순서화가 인과성을 보존하는데 도움을 준다. 
-> 일관된 순서로 읽기 : 질문이 답변되면 분명히 그 질문이 먼저 있어야하게 때문에, 질문과 답변 사이에 인과적 의존성이 있다함
                    : 로우가 갱신되기 전 생성되는 것
-> 동시 쓰기 감지 : A.B가 있으면 각각 먼저 실행될수도 동시(서로의존성x)에 될수도 있다. 바꿔말하면 둘다 서로에대해 알지못한다고 확신할 수 있다.
-> 트랜잭션용 스냅숏 격리 : 트랜잭션은 일관된 스냅숏으로부터읽는다. 여기서 일관적이란 인과성에 일관적(consistent with causalilty)
                        : 읽기 스큐는 인과성을 위반하는 상태에 있는 데이터를 읽는 것을 의미한다.

인과성은 이벤트에 순서를 부과한다. 순서를 시키면 그 시스템은 인과적으로 일관적(causally consistent)라고 한다. 
-> 예) 스냅숏 격리는 인과적 일관성을 제공한다. 데이터 베이스에서 읽어서 데이터의 어떤 조각을 봤다면 그보다 인과적으로 먼저 발생한
데이터도 볼 수 있어야한다. (도중에 삭제되지 않은 이상)

### 인과적 순서가 전체 순서는 아니다. 
total order는 자연수는 명확하지만 수학적 집합은 명확하지 못하다. -> 비교불가(incomparable)하기 때문에 부분적으로 순서가 정해진다.(partially ordered)
-> 전체 순서와 부분 순서의 차이점은 다른 데이터베이스 일관성 모델에 반영된다. 
선형성과 인과성 정의에 따르면 선형성 데이터 스토어에는 동시적 연산이 없다. 
-> 데이터 스토어는 동시성 없이 하나의 타임라인을 따라 단일 데이터 복사본에 연산을 실행해 모든 요청이 한 시점에 원자적으로 처리되도록 보장함
-> 동시성은 타임라인이 갈라졌다 다시 합쳐지는 것을 의미함.
-> 예) 깃 같은 분산 버전 관리 시스템 : 깃 버전 히스토리는 인과적 의존성 그래프와 매우 유사. 종종하나의 커밋은 다른 것보다 일직선 상에서 나중에 실행되지만 
                                    때때로 브랜치를 만들고 이렇게 동시에 만들어진 커밋을 머지할 수 도 있기 때문.

### 선형성은 인과적 일관성보다 강하다
인과적 순서와 선형성 사이에는 어떤 관계가 있을까?
-> 선형성은 인과성을 내포한다. 
선형성은 인과성을 보존하는 유일한 방법이 아니다.
(시스템을 선형적으로 만들었는데 네트워크 지연이 크면(지리적분산) 성능과 가용성에 해가 됨. 
때문에 어떤 분산 데이터 시스템들은 선형성을 포기해서 더 좋은 성능을 달성하지만 사용하기는 어렵다.)
-> 절충 가능 : 선형성이 필요한 시스템에 진짜로 필요한 것은 인과적 일관성이며 이는 더 효율적으로 구현가능, 최종적 일관성 시스템과 성능 및 가용성 특성이
비슷하면서 인과성을 보존하는 새로운 종류의 데이터베이스를 연구하고 있다. (프로덕션 시스템에 반영x)

### 인과적 의존성 담기
### 비인과적 일련번호 생성기

### 램포트 타임 스탬프
인과성에 일관적인 일련번호를 생성하는 간단한 방법 
레슬리 램포트가 1978년 논문에서 제안한 , 램포트 타임스탬프(Lamport timestamp) : 분산시스템 분야에서 가장 많이 인용된 논문 중 하나
-> 각 노드는 고유 식별자를 갖고 각 노드는 처리한 연산 개수를 카운터로 유지, 램포트 타임스탬프는 그냥 (카운터, 노드ID)의 쌍이다. 
두 노드는 떄떄로 카운터 값이 같을수 있지만 타임스탬프에 노드ID를 포함시켜 각 타임스탬프는 유니크하게 됨.
-> 버전 벡터보다 램포트 타임스탬프가 좋은 점은 크기가 작다는 것.

### 타임스탬프 순서화로는 충분하지 않다.
램포트 타임스탬프가 인과성에 일관적인 연산의 전체 순서를 정희하지만, 분산 시스템의 여러 공통문제 해결에는 불충분하다.
-> 예) 사용자명으로 계정이 유일하게 식별할 수 있도록 보장하는 시스템: 두사용자가 동일한 사용자명 생성시 둘중한명만 성공해야함.(리더와 잠금 301pg)
    ->언뜻 보기에는 연산의 전체 순서화가 램포트 타임스탬프와 같은 걸 사용하면 문제를 해결할 수 있을거라 본다. 하지만 당장 요청이 성공/실패해야하는지
      결정해야할 떄는 안된다. 
    ->노드는 다른 노드가 동일한 사용자명으로 계정 생성 처리르 하고 있는지와 다른 노드가 그 연산에 어떤 타임스템프를 배정할지 모르기 떄문
    -> 다른 노드중 하나에 장애가 생기거나 네트워크 문제가 생기면 시스템이 멈춘다. 이것은 우리가 필요한 내결함성 시스템 유형이 아님
    -> 이 모든 문제는 연산이 종료 된 후에 드러나게 된다. 
언제 전체 순서가 확장되는지 알아야한다는 아이디어를 전체 순서 브로드캐스트의 주제로 다룬다.



### 1. 전체 순서 브로드캐스트
1)단일 CPU코어에서만 실행된다면 연산의 전체 순서를 정하기 쉽다. -> cpu에서 실행된 순서가 바로 전체 순서
2)분산시스템에서는 모든 노드에서 연산의 전체 순서가 동일하도록 합의하기 까다로움.
-> 타임스탬프/일련번호를 사용한 순서화를 이전에 설명했지만 단일 리더 복제만큼 강력하지는 않다. 
### 2. 단일 리더 복제
    -> 단일 리더 복제는 한 노드를 리더로 설정하고 리더의 단일 cpu코어에서 모든 연산을 차례대로 배열하므로 
       연산 전체 순서를 정한다.
       이때 문제는 처리량이 단일 리더가 처리할 수 있는 수준을 넘어설 때 시스템을 어떻게 확장할 것인가 이다.
### 4. 분산시스템 분야에서는 이러한 문제를 아래와 같이 칭한다.
   전체순서브로드캐스트(total order broadcast) / 원자적 브로드캐스트(atomic broadcast)

### 5. 전체순서브로드캐스트(total order broadcast)
= 보통 노드 사이에 메시지를 교환하는 프로토콜로 기술된다. 아래 두가지 속성을 만족해야함.
1) 신뢰성 있는 전달(메세지 손실x,메세지가 한 노드에 전달되면 모든 노드에 전달)
2) 전체 순서가 정해진 전달 ( 메세지는 모든 노드에 같은 순서로 전달 됨)
-> 전체 순서 브로드 캐스트는 네트워크 결함이 있더라도 신뢰성과 순서화 속성이 항상 만족되도록 보장해야함.
 예) 네트워크가 복구되면 이후에 올바른 순서로 전달

### 6. 전체 순서 브로드 캐스트 사용하기
전체 순서 브로드 캐스트를 구현 한 서비스 : 주키퍼 & etcd 
-> 데이터베이스 복제에 전체 순서브로드캐스트는 필요하다!
    1) 상태 기계 복제(state machine replication)
    - 모든 메세지가 데이터베이스에 쓰기를 나타내고, 모든 복제서버가 같은 쓰기 연산을 같은 순서로 처리하면
    복제 서버들은 서로 일관성 있는 상태를 유지한다. (일시적인 복제지연 제외)
    2) 직렬성 트랜잭션 구현 
    - 모든 메세지가 스토어드 프로시저로 실행되는 결정적 트랜잭션을 나타낸다면, 모든 노드가 그 메세지들을 같은 순서로 처리한다면
    데이터베이스의 파티션과 복제본은 서로 일관적인 상태를 유지한다.
-> 메세지가 전달되는 시점에 순서가 고정됨. (후속 메시지가 전달 됐다면 노드는 그 순서의 앞에 메세지를 소급적으로 끼워넣는게 허용X
-> LOG를 만드는 방법 중 하나. 메세지 전달은 로그에 추가하는 것과 비슷하다. 모든 노드가 같은 메시지를 같은 순서로 전달해야하므로
     모든 노드는 로그를 읽어서 순서가 동일한 메세지를 볼 수 있다.
-> 잠금 서비스 구현에도 유용, 모든 메시지들은 로그에 나타난 순서대로 
   일련번호(주키퍼에서는 zxid)가 단조증가하면서 붙어 펜싱토큰역할 수행

### 7. 전체 순서 브로드 캐스트를 사용해 선형성 저장소 구현하기
선형성 시스템에는 연산의 전체 순서가 있지만, 이게 선형성이 전체 순서 브로드 캐스트와 같다는 의미는 아니다. 
- 전체 순서 브로드캐스트는 비동기식이다. 메세지는 고정된 순서로 신뢰성 있게 전달되도록 보장되지만 언제 메세지가 전달될지는 보장x
- 선형성은 최신성 보장이다. 읽기가 최근에 쓰여진 값 보는게 보장된다.

전체 순서 브로드캐스트 구현이 있다면 이를 기반으로 선형성 저장소를 만들 수 있다.
예) 사용자명으로 사용자 게쩡을 유일하게 식별하도록 보장
-> 사용 가능한 모든 사용자마다 원자적(compare-and-set) 연산이 구현된 선형성 저장소를 가질 수 있다고 상상해보자. 
   모든 레지스터는 초기에 Null 값을 가진다. (사용자명이 점유되지 x) 
   여러 사용자가 동시에 같은 사용자명을 하려한다면 compare-and-set연산 중 하나만 성공. 다른 compare-and-set은 널이 아닌 값을 보기 때문

