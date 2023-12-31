# 08. 분산 시스템의 골칫거리

최대한 비관적으로 시스템을 바라보자. 분산 시스템을 다루는 것은 한 컴퓨터에서 실행되는 소프트웨어를 작성하는 것과 근본적으로 다룬다.

# 결함과 부분 장애

- 우리는 컴퓨터에 내부 결함이 발생하면 잘못된 결과가 반환하는 것이 아니라 완전한 동작을 하지 않기를 바란다.
- 네트워크로 연결된 여러 컴퓨터는 근본적으로 다르다. 분산 시스템의 어떤 부분은 잘 동작하지만 다른 부분은 예측할 수 없는 방식으로 고장난다. 이를 부분 장애(partial failure)라고 한다.
- 부분 장애는 **비결정적**이어서 어렵다.
    - 뭐가 성공했는지 알 수 없고 메시지가 네트워크를 거쳐 전송되는 시간도 비결정적이다.

비결정성과 부분 장애 가능성이 분산 시스템을 다루기 어렵게 한다.

## 클라우드 컴퓨팅과 슈퍼컴퓨팅

대규모 컴퓨팅 시스템 구축 방법에 관한 철학

- 대규모 컴퓨팅의 한쪽 끝에는 고성능 컴퓨팅(high-performance computing, HPC) 분야가 있다. 수천 개의 CPU를 가진 슈퍼컴퓨터는 일기예보 등의 계산 비용이 높은 과학 계산 작업 등에 쓰인다.
- 다른 극단에 클라우드 컴퓨팅이 있다. 멀티 테넌트 데이터센터, IP 네트워크 컴퓨터, 신축적(elastic)/주문식(on-demand) 자원 할당, 계량 결제(metered billing)와 흔히 관련된다.
- 전통적인 기업형 데이터센터는 이 두 극단의 중간 지점에 있다.

슈퍼 컴퓨팅

- 장애 대응
    - 계산 상태를 지속성 있는 저장소에 체크포인트로 저장
    - 노드에 장애가 발생하면 전체 클러스터 작업부하를 중단하는 해결책을 주로 사용
    - 장애가 발생한 노드가 복구된 후 마지막 체크포인트부터 계산을 재시작
- 분산 시스템 보다는 **단일 노드 컴퓨터**에 가까움
    - **부분장애 → 전체 장애로 확대하는 방법으로 주로 처리**
- 전형적으로 특화된 하드웨어를 사용해 구축
    - 노드가 신뢰성이 높고 공유 메모리와 원격 직접 메모리 접근을 사용해 통신
- 통신 패턴이 정해진 HPC 작업부하에서 높은 성능을 보여주는 다차원 메시(mesh)나 토러스(torus) 같은 특화된 네트워크 토폴로지를 자주 사용한다.

인터넷 서비스

- 대부분의 애플리케이션은 사용자에게 지연 시간이 낮은 서비스를 제공해야하기 때문에 온라인이다.
- 클러스터를 중단시켜 서비스를 이용할 수 없게 만드는 방법은 허용하지 않는다.
- 상용 장비를 사용해 구축하고 더 낮은 비용으로 동일한 성능을 제공한다. 따라서 실패율도 높다.
- IP와 이더넷을 기반으로 높은 양단 대역폭(bisection bandwidth)을 제공하기 위해 클로스 토폴로지(clos topology)로 연결

시스템이 장애가 난 노드를 감내할 수 있고 전체적으로 계속 동작할 수 있다면 이는 운영과 유지보수에 유용한 특성이 된다.

- 순회식 업그레이드
    - 서비스를 계속 제공하면서 한 번에 노드 하나씩 재시작

**분산 시스템이 동작하게 만드려면 부분 장애 가능성을 받아들이고 소프트웨어에 내결함성 매커니즘을 넣어야 한다.**

→ 신뢰성 없는 구성 요소를 사용해 신뢰성 있는 시스템을 구축해야한다.

작은 시스템이라도 부분 장애를 고려해야한다. 결함 처리는 소프트웨어 설계의 일부여야 하며 결함이 발생하면 소프트웨어가 어떻게 동작할지 알아야 한다.

결함이 드물 것이라 가정하고 최선의 상황을 바라기만 하는 것은 현명하지 못하다. 테스트 환경에서 인위적으로 이런 상황을 만들어서 어떤 일이 생기는지 중요하다. 분산시스템에서는 의심, 비관주의, 편집증은 값어치를 한다.

> **신뢰성 없는 구성 요소를 사용해 신뢰성 있는 시스템 구축하기**
> 예시)
> 
> ****- 오류 수정 코드(error-correcting code): 무선 네트워크에서 발생하는 전파 장애 등의 이유로 가끔 일부 비트가 잘못되는 통신 채널을 통해 디지털 데이터를 정확히 전송하게 해준다.
> 
> - IP(Internet Protocol)는 신뢰성이 없다. IP를 사용할 때 패킷은 누락 또는 지연되거나 중복될 수 있다. TCP(Transmission Control Protocol)는 IP 위에서 더욱 신뢰성이 높은 전송 계층을 제공한다. TCP는 손실된 패킷을 재전송하고 중복된 것은 제거하여 패킷을 보낸 순서에 맞춰 재조립되도록 보장해준다.
> 
> 시스템은 기반이 되는 부분보다 높은 신뢰성을 갖출 수 있지만 신뢰성을 높이는 데에는 제한이 있다. 
> 
> - 오류수정코드는 소량의 단일 비트 오류는 해결할 수 있지만 전파 방해에 영향을 심하게 받으면 통신 채널을 통해 전송할 수 있는 데이터의 양에 근본적인 제한이 생긴다.
> - TCP는 패킷 손실 및 중복, 순서가 섞이는 문제를 감추지만 네트워크 지연을 해결할 수 없다.
> 
> 좀 더 신뢰성 있는 상위 시스템은 완벽하지는 않아도 유용하다. 저수준 결함 중 일부분을 처리해주므로 남은 결함은 상황에 따라 따져보고 처리하면 쉽기 때문이다.

# 신뢰성 없는 네트워크

이 책에서 다루는 분산 시스템은 비공유 시스템, 네트워크로 연결된 다수의 장비이다.

- 네트워크는 장비들이 통신하는 유일한 수단
- 각 장비는 자신만의 메모리와 디스크를 가지고 있으며 독립적으로 격리되어있다.
- 인터넷과 데이터센터 내부 네트워크의 대부분은 **비동기 패킷 네트워크(asynchronous packet network)**이다.

- 네트워크는 메시지가 언제 도착할지 혹은 메시지가 도착하기는 할 것인지 보장하지 않는다.
    1. 요청이 손실됐을 수 있다.
       -  누군가 네트워크 케이블을 뽑았을 수도 있다.
    2. 요청이 큐에서 대기하다가 나중에 전송될 수 있다.
       -  네트워크나 수신자에 과부하가 걸렸을 수도 있다.
    3. 원격 노드에 장애가 생겼을 수 있다.
       -  죽었거나 전원이 나갔을 수 있다.
    4. 원격 노드가 일시적으로 응답하기를 멈췄지만 나중에 다시 응답하기 시작할 수 있다.
       -  가비지 컬렉션 휴지가 길어졌을 수 있다.
    5. 원격 노드가 요청을 처리했지만 응답이 네트워크에서 손실됐을 수 있다.
       -  네트워크 스위치의 설정이 잘못됐을 수 있다.
    6. 원격 노드가 요청을 처리했지만 응답이 지연되다가 나중에 전송될 수 있다.
       -  네트워크나 요청을 보낸 장비에 과부하가 걸렸을 수 있다.
    ![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/ae730028-6917-47f1-ab4c-c95ee62b3c8e)
    요청을 보낸 후 응답을 받지 못했다면 요청이 손실됐는지, 원격 노드가 다운됐는지, 응답이 손실됐는지 구별할 수 없다.

- 전송 측은 패킷이 전송됐는지 아닌지 구별할 수 없다. 유일한 선택지는 수신 측에서 응답 메시지를 보내는 것이지만 응답 메시지도 손실되거나 지연될 수 있다.
- 다른 노드로 요청을 보내서 응답을 받지 못했다면 **그 이유를 아는 것은 불가능**하다.
    - 이런 문제를 다루는 흔항 방법이 **타임아웃**이다.

## 현실의 네트워크 결함

- 네트워크 장비를 중복 추가하는 것은 기대만큼 결함을 줄여주지 않는다.

> **네트워크 분단**
> 네트워크 결함 때문에 네트워크 일부가 다른 쪽과 차단되는 것을 네트워크 분단(network partition)이나 네트워크 분리(netsplit)라고 부른다. 이 책에서는 더 일반적인 용어인 네트워크 결함(network fault)를 사용한다.

- 네트워크 결함이 드물더라도 결함이 일어날 수 있다. 따라서 소프트웨어가 이를 처리해야한다. 네트워크 상으로 통신할 때마다 실패할 가능성이 있고 이를 피할 방법은 없다.
- 네트워크 결함의 오류 처리가 정의되고 테스트하지 않으면 예측 못한 문제가 발생할 수 있다.
- 네트워크 결함을 견뎌내도록(tolerating) 처리할 필요가 없다. 하지만 소프트웨어가 네트워크 문제에 어떻게 반응하는지 알고 시스템이 그로부터 복구할 수 있도록 보장해야 한다.
    - 고의로 네트워크 문제를 유발하고 시스템의 반응을 테스트하는 것도 좋은 방법이다(카오스 몽키(Chaos Monkey)의 기반이되는 생각)

## 결함 감지

시스템은 결함이 있는 노드를 감지할 수 있어야 한다.

- 로드 밸런서는 죽은 노드로 요청을 그만 보내야 한다.
    - 죽은 노드는 순번에 빠진 것으로 간주해야 하낟.
- 단일 리더 복제를 사용하는 분산 데이터베이스에서 리더에 장애가 나면 팔로워 중 하나가 리더로 승격해야 한다.

특정한 환경에서는 노드가 동작하지 않는다고 명시적인 피드백을 받을 수 있다.

## 타임아웃과 기약 없는 지연

타임아웃만이 결함을 감지하는 확실한 수단이라면 타임아웃은 얼마나 길어야 할까? → 답은 없다.

- 타임아웃이 길면 노드가 죽었다고 선언될 때까지 시간이 길어진다.
- 타임아웃이 짧으면 결함을 빨리 발견하지만 노드가 일시적으로 느려졌을 뿐인데도 죽었다고 잘못 선언할 위험이 높아진다.
    - 노드가 실제로는 살아있고 동작이 실행중일 때 실패했다고 판단하고 다른 노드가 역할을 넘겨 받으면 그 동작을 두 번 실행하게 될지도 모른다.
        - ex) 이메일을 2번 발송할 수도 있다.
    - 노드의 책무를 다른 노드로 전달해야 해서 네트워크와 다른 노드에 추가적인 부하를 준다.
        - 실제로는 죽지 않고 과부하 때문에 응답이 느릴 수 있다. 그러면 연쇄 장애를 유발하고 모든 노드가 서로를 죽었다고 판단할수도 있다.

**패킷의 최대 지연 시간이 보장된 네트워크를 사용하는 상상 속 시스템**

1. 모든 패킷은 어떤 시간 `d` 내에 전송되거나 손실되지만 전송 시간이 결코 `d`보다 걸리지 않는다.
2. 장애가 나지 않은 노드는 항상 요청을 `r` 시간 내에 처리한다고 보장할 수 있다고 가정한다.
3. 이 경우 성공한 요청은 모두 `2d + r` 시간 내에 응답을 받는다고 보장할 수 있다.
    1. 오고가는 시간(`2d`) + 처리시간(`r`)

→ 우리가 사용하는 시스템은 어떤것도 보장하지 못한다.

비동기 네트워크는 **기약 없는 지연(unbounded delay)**이 있고 서버 구현은 대부분 어떤 최대 시간 내에 요청을 처리한다고 보장할 수 없다. 시스템이 대부분의 시간에 빠르다는 것은 장애 감지에 충분치 않다.

### 네트워크 혼잡과 큐 대기

컴퓨터 네트워크에서 패킷 지연의 변동성은 큐 대기 때문인 경우가 많다.

- 여러 노드가 동시에 같은 목적지로 패킷을 보내려고 하면 네트워크 스위치는 패킷을 큐에 넣고 한 번에 하나씩 목적지 네트워크 링크로 넘겨야 한다. 네트워크 링크가 붐비면 패킷은 슬롯을 얻을 수 있을 때까지 잠시 기다려야 할 수 있다(네트워크 혼잡(network congestion). 네트워크가 정상적이라도 들어오는 데이터가 많아 스위치 큐를 꽉 채우게 되면 패킷이 유실되어 재 전송해야한다.
  ![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/120ee2c6-bc3c-4b88-994a-d5fa41f665f3)
  같은 목적지로 네트워크 트래픽을 보내면 스위치 큐가 가득찰 수 있다.

- 패킷이 목적지 장비에 도착했을 때 CPU 코어가 바쁘면 처리할 준비가 될때까지 OS 큐에 집어넣는다. 장비의 부하에 따라 큐 대기 시간은 제각각이다.
- 가상 환경에서 실행되는 OS는 다른 가상 장비가 CPU 코어를 사용하는 동안 멈출 때가 흔하다. 이 시간동안 가상 장비는 네트워크에서 어떤 데이터도 받아들 일 수 없다. 가상 장비 모니터가 들어오는 데이터를 큐에 넣어서 네트워크 지연의 변동성을 더욱 증가시킨다.
- TCP는 흐름 제어(flow control)를 수행한다. 혼잡 회피(congestion avoidance)나 배압(backpressure)이라는 흐름제어는 노드가 네트워크 링크나 수신 노드에 과부하를 가하지 않도록 자신의 송신율을 제한하는 것이다. 데이터가 네트워크로 들어가기 전에도 부가적인 큐 대기를 할 수 있다.
- TCP는 어떤 타임아웃 안에 확인 응답을 받지 않으면 패킷이 손실됐다고 간주하고 손실된 패킷은 자동으로 재전송한다. 애플리케이션에게는 패킷 손실이나 재전송이 보이지 않지만 그 결과로 생기는 지연은 보인다.

> **TCP 대 UDP**
> 
> 화상 회의나 인터넷 전화처럼 지연 시간에 민감한 애플리케이션은 TCP 대신 UDP를 사용한다.
> 
> 신뢰성과 지연 변동성 사이에 트레이드오프 관계가 있다.
> 
> UDP: 흐름 제어를 하지 않고 손실된 패킷을 재전송하지 않으므로 네트워크 지연이 변하게 하는 원인 중 일부를 제거한다. 
> 
> 전화 통화에서 스피커로 데이터가 재생되기 전에 손실된 패킷을 재전송하기에는 시간이 충분치 않다. 이 경우에 패킷을 재전송하는 게 의미가 없다. 대신 애플리케이션은 잃어버린 패킷에 해당하는 시간 슬롯을 침묵으로 채우고 스트림에서 계속 이동해야 한다. 재시도는 사람 계층에서 대신 실행된다.(다시 좀 말해줄래? 잠시 끊김;;)

큐 대기 지연은 시스템이 최대 용량에 가까울 때 광범위하게 발생한다. 예비 용량이 풍부한 시스템은 쉽게 큐를 비울 수 있지만 사용률이 높은 시스템은 긴 큐가 매우 빨리 만들어진다.

- 클라우드 환경에서는 자원을 공유하기 때문에 포화되기 쉽다.
- 이런 환경에서 실험적으로 타임아웃을 선택하는 것이 좋다.
- 더 좋은 방법은 시스템이 지속적으로 응답 시간과 변동성(지터(jitter))을 측정하고 관찰된 응답 시간 분포에 따라 타임아웃을 자동으로 조절하게 하면 좋다.
    - 파이 증가 장애 감지기를 사용하면 되며 예시로 아카(akka)와 카산드라가 있다.
    - TCP 재전송 타임아웃도 비슷하게 동작한다

## 동기 네트워크 대 비동기 네트워크

패킷 전송 지연 시간의 최대치가 고정돼 있고 패킷을 유실하지 않는 네트워크에 기댈 수 있다면 분산 시스템은 단순해진다.

**동기식 네트워크**

- 전화 통화에서는 회선(circuit)이 만들어져 두 통화에 대해 보장된 양의 대역폭이 할당된다. 회선은 통화가 끝날 때까지 유지된다.
- 데이터가 여러 라우터를 거치더라도 큐 대기 문제를 겪지 않는다. 큐 대기가 없으므로 네트워크 종단 지연 시간의 최대치가 고정되어있다.
- 제한 있는 지연(bounded delay)

### 그냥 네트워크 지연을 예측 가능하게 만들 수는 없을까?

회선은 만들어져있는 동안 다른 누구도 사용할 수 없는 고정된 양의 예약된 대역폭이다. TCP 연결의 패킷은 가용한 네트워크 대역폭을 기회주의적으로 사용한다. TCP에 가변 크기의 데이터 블록을 보내면 가능하면 짧은 시간 안에 전송하려고 한다. TCP 연결이 유휴 상태에 있는 동안 어떤 대역폭도 사용하지 않는다.

패킷 교환(packet-swtich) 프로토콜

- IP와 이더넷이 사용
- 큐 대기 영향으로 네트워크에 기약 없는 지연이 있음

데이터센터 네트워크와 인터넷이 패킷 교환을 사용하는 이유

- 순간적으로 몰리는 트래픽(bursty traffic)에 최적화
- 웹 페이지 요청, 이메일 전송, 파일 전송은 특별한 대역폭 요구사항이 없고 빨리 완료되기를 바랄 뿐이다.
- 가용한 네트워크 용량에 맞춰 데이터 전송률을 동적으로 조절한다.

회선을 통한 파일 전송

- 추정치가 너무 낮으면 네트워크 용량을 쓰지 않고 남겨 둔 상태로 전송이 불필요하게 느려짐
- 추정치가 너무 높으면 회선은 구성될 수 없다.
- 순간적으로 몰리는 데이터 전송에 회선을 쓰면 네트워크 용량을 낭비하고 전송이 불필요하게 느려진다.

회선 교환과 패킷 교환을 모두 지원하는 하이브리드 네트워크를 만드려는 시도도 있다. 서비스 품질(quality of service, QoS, 패킷에 우선순위를 매기고 스케줄링)과 진입 제어(admission control, 전송 측에서 전송률을 제한)를 잘 쓰면 패킷 네트워크에서 회선 교환을 흉내 내거나 통계적으로 제한 있는 지연을 제공하는 것도 가능하다.

하지만 공개 클라우드에서 사용할 수 없고 인터넷 통신도 안된다. 따라서 네트워크 혼잡, 큐 대기, 기역 없는 지연이 발생하는 것을 가정해야 하고 타임아웃에 ‘올바른’ 값은 없으며 실험을 통해 결정해야 한다.

> **지연 시간과 자원 사용률**

# 신뢰성 없는 시계

애플리케이션은 여러가지 상황에서 시계에 의존한다.

**지속시간(시점과 시점 사이의 구간)**

1. 이 요청이 타임아웃됐나?
2. 이 서비스의 99분위 응답 시간은 어떻게 되나?
3. 이 서비스는 지난 5분 동안 평균 초당 몇 개의 질의를 처리했나?
4. 사용자가 우리 사이트에서 시간을 얼마나 보냈나?

**시점(특정 날짜의 특정 시간에 발생한 이벤트)**

1. 이 기사가 언제 게시됐나?
2. 며칠 몇 시에 미리 알림 이메일을 보내야 하나?
3. 이 캐시 항목은 언제 만료되나?
4. 로그 파일에 남은 이 오류 메시지의 타임스탬프는 무엇인가?

분산 시스템에서 통신은 즉각적이지 않으므로 시간을 다루기 까다롭다. 네트워크에 있는 개별 장비는 자신의 시계를 가지고 있어 개념이 상이할 수 있다. 이를 동기화하기 위한 매커니즘으로는 네트워크 시간 프로토콜(Network Time Protocol, NTP)를 사용할 수 있다. 이 서버들은 다시 GPS 수신자 같은 더욱 정확한 시간 출처로부터 시간을 얻는다.

## 단조 시계 대 일 기준 시계

현대 컴퓨터는 일 기준 시계(time-of-day clock)과 단조 시계(monotonic clock) 최소 두 종류의 시계를 갖고 있다. 둘 다 시간을 측정하지만 다른 목적으로 사용한다.

### 일 기준 시계

직관적으로 시계에 기대하는 일을 한다.

- 달력에 따라 현재 날짜와 시간을 반환(벽시계 시간(wall-clock)이라고도 한다.)
    - 리눅스의 clock_gettime
    - 자바의 System.currentTimeMillis()는 에포크(epoch) 이래로 흐른 초 수를 ㅂ반환한다.
    - 윤초는 세지 않으며 에포크는 그레고리력에 따르면 UTC(협정세계시) 1970년 1월 1일 자정을 가리킨다.
- 일 기준 시계는 보통 NTP로 동기화된다.
    - 그러나 종종 시간이 안맞아 측정하는데 적합하지 않는 경우가 있다.
- 일 기준 시계는 역사적으로 매우 거친(coarse-grained) 해상도를 가진다.
    - 최근에는 큰 문제가 되지 않는다.

### 단조 시계

- 타임아웃이나 서비스 응답 시간 같은 지속 시간을 재는데 적합하다.
    - 리눅스의 clock_gettime
    - 자바의 System.nanoTime()
- 시간이 항상 앞으로 흐른다. 일 기준 시계는 역으로 흐를수도 있다.
- 하지만 절대적인 값은 의미가 없다.
- CPU 소켓이 있는 서버는 CPU마다 독립된 타이머가 있을 수 있다. 이 타이머는 다른 CPU와 반드시 동기화되지는 않다.
    - 운영체제는 차이를 보정해서 애플리케이션 스레드가 여러 CPU에 걸쳐 스케줄링 되더라도 단조적으로 보이게 하려고 한다. 하지만 곧이 곧대로 받아들이지 않는게 현명하다.
- NTP는 컴퓨터의 로컬 시계가 NTP보다 빠르거나 느리면 진도수를 조정할 수 있다.
    - 시계를 돌린다(slewing)고 한다.

## 시계 동기화와 정확도

단조 시계는 동기화가 필요 없지만 일 기준 시계는 NTP 서버나 다른 외부 시간 출처에 맞춰 설정돼야 유용하다.

하드웨어 시계와 NTP는 여러 문제가 발생할 수 있다.

- 컴퓨터의 수정 시계가 정확하지 않은 드리프트(drift) 현상이 생긴다.
- 컴퓨터 시계과 NTP 서버와 많은 차이가 나면 동기화가 거부되거나 로컬 시계가 강제로 리셋될 수 있다.

시계 정확도를 높이기 위한 방법으로 해결하는 방법이 있다. GPS 수신기, 정밀 시간 프로토콜 배포 및 모니터링을 할 수 있다. 하지만 쉬운 방법이 아니다.

## 동기화된 시계에 의존하기

동기화된 시계가 필요한 소프트웨어를 사용한다면 필수적으로 모든 장비 사이의 시계 차이를 조심스럽게 모니터링 해야한다. 다른 노드와 시계가 너무 차이나는 노드는 죽은 것으로 선언하고 클러스터에서 제거해야한다.

### 이벤트 순서화용 타임스탬프

여러 노드에 걸친 이벤트들의 순서를 정하는 문제가 있다. 다중리더복제에서 쓰기 연산이 일어났을 때

![image](https://github.com/eastperson/23-11-DesigningDataIntensiveApplications/assets/66561524/3ae7a251-cfca-4f15-a46f-e3c9878c1cf8)
노드간의 시계가 다르면 클라이언트 A보다 인과성 측면에서 나중에 쓰지만 B가 쓸 대 사용하는 타임스탬프가 더 이르다.

쓰기가 다른 노드로 복제될 때 Node1에서 쓰기가 너 빠른 시점인데도 불구하고 x=2를 사용할 때 타임스탬프가 더 이르기 때문에 순서가 섞인다.

- 이 전략은 최종 쓰기 승리(last write wins, LWW)라고 불리느며 다중 리더 복제와 카산드라와 리악같은 리더 없는 데이터베이스에서 사용된다.
    - 이를 해결하기 위해 서버가 아니라 클라이언트에서 생성하는 구현도 있지만 근본적인 LWW 해결방법은 아니다.
    1. 시계가 뒤쳐지는 노드는 시계가 빠른 노드가 먼저 쓴 내용을 그들 사이에 차이나는 시간이 흐를 때까지 덮어쓸 수 있다.
    2. 빠른 시간 내에 연속으로 실행되는 것과 진짜 동시에 쓰기가 실행되는 것을 구분할 수 없다. 인과성 위반을 막기 위해서는 버전 백터 같은 인과성 추적 메커니즘이 필요하다.
    3. 두 노드가 독립적으로 동일한 타임스탬프를 가진 쓰기 작업을 만들 수 있다. 이런 충돌을 해소하려면 같은 값을 다르게 만들어줄 부가적인 값이 필요하지만 이 방법도 인과성 위반으로 이어질 수 있다.

‘최근’ 값을 유지하고 싶더라도 ‘최근’의 정의를 정확히하며 그 시계는 틀릴 수도 있다는 것을 알아야 한다. NTP 동기화를 정확하게하는 방법은 불가능에 가깝다. 올바른 순서화를 위해서는 시계 출처가 측정하려고 하는 대상보다 훨씬 더 정확해야한다.

논리적 시계(logical clock): 진동하는 수정 대신 증가하는 카운터를 기반으로 하며 이벤트 순서화의 안전한 대안

- 일 기준 시간이나 경과한 초 수를 측정하지 않고 이벤트의 상대적인 순서만 측정한다.

물리적 시계(physical clock): 일 기준 시계와 단조 시계는 실제 경과 시간을 측정

### 시계 읽기는 신뢰 구간이 있다

시계읽기는 어떤 시점으로 생각하는 것은 타당하지 않다. 언떤 신뢰 구간에 속하는 시간의 범위로 읽는 게 낫다.

불확실성 경계는 시간 출처를 기반으로 계산할 수 있다.

- GPS 수신기나 컴퓨터에 부착된 원자 시계가 있으면 제조사가 예상한 오차범위가 있다.
- 서버로부터 얻는다면 서버와 마지막으로 동기화한 시간 이후로 예상되는 시계 드리프트에 NTP 서버의 불확실성을 더하고 그 서버와 통신할 때 걸리는 네트워크 왕복 시간을 더한 값을 기반으로 한다.

시계는 불확실성 계산을 기반으로 실제 현재 시간이 그 구간 안의 어딘가에 있다는 것을 안다.

### 전역 스냅숏용 동기화된 시계

데이터베이스가 여러 데이터센터에 있는 여러 장비에 분산되어 있을 때는 코디네이션이 필요하다. 스냅숏 격리를 구현하기 위해 트랜잭션ID를 생성해야하는데 전욕 단조 증가 트랜잭션 ID를 생성하기 어렵다. 트랜잭션은 ID는 인과성을 반영해야 한다.

트랜잭션 타임스탬프가 인과성을 반영하는 것을 보장하기 위해서는 스패너는 읽기 쓰기 트랜잭션을 커밋하기 전에 의도적으로 신뢰 구간의 길이만큼 기다린다. 트랜잭션은 충분히 나중에 실행되는 게 보장되므로 신뢰 구간이 겹치지 않는다.

분산 트랜잭션 시멘틱용으로 시계 동기화를 스는 것은 활발히 연구되는 분야다.이 아이디어는 흥미롭지만 구글 외에는 아직 주류 데이터베이스에서 구현한 사례가 없다.

## 프로세스 중단

단일 장비에서 다중 스레드 코드를 작성할 때 안전하게 만드는 방법

- 뮤텍스
- 세마포어
- 원자적 카운터
- 잠금 없는 자료구조
- 블로킹 큐

→ 이것들을 분산시스템용으로 번영하기가 쉽지 않다. 분산시스템은 공유 메모리가 없고 신뢰성없는 네트워크를 통해 메시지를 보내기 때문이다.

### 응답 시간 보장

프로그래밍 언어와 운영체제에서 스레드와 프로세스는 기약 없는 시간동안 중단될 수 있다. 충분히 열심히 노력하면 중단의 원인을 제거할 수 있다.

- 데드라인(deadline)
- 엄격한 실시간 시스템(hard real-time)

### 가비지 컬렉션의 영향을 제한하기

GC 중단을 노드가 잠시동안 계획적으로 중단되는 것으로 간주하고 노드가 가비지 컬렉션을 하는 동안 클라이언트로부터의 요청 다른 노드들이 처리하게 하는 것이다.

