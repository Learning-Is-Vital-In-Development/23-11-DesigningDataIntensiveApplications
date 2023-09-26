### 리더 없는 복제

> 리더의 개념을 버리고 모든 복제 서버가 쓰기 작업을 할 수 있게 허용하는 방식

- DynamoDB 에서 다시 리더 없는 복제 개념이 유행되어 리악, 카산드라, 볼드모트 등이 존재한다. (다이나모 스타일)
- 코디네이터 노드(주키퍼와 같은)가 클라이언트를 대신에 클라이언트의 쓰기 요청을 전송해준다.
  - 리더 데이터베이스와 달리 코디네이터 노드는 특정 순서로 쓰기를 수행하지는 않는다.
 

#### 노드가 다운됐을 때 데이터베이스에 쓰기

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/39f600f3-e80b-49ac-8a11-530e96335514)


- 세 개의 노드를 가진 데이터베이스가 있고, 그 중 하나가 다운된 경우
  - 클라이언트는 모든 노드 서버에 쓰기요청을 하기 때문에 데이터는 유실되지 않는다.
  - 다만 다운된 노드가 다시 온라인이 되었을 때 이 서버는 오래된 데이터를 가지고 있어서 읽기요청 시에 오래된 데이터가 올 수 있다.
  - 읽기 요청을 병렬로 여러 노드에 전송하고 버전을 확인하여 최신 데이터를 사용한다.


#### 읽기 복구와 안티 엔트로피

모든 복제 서버가 동일한 데이터를 가지는 일관성을 보장할 수 있어야 한다. <br>
장애가 발생한 노드가 장애 복구 후 일관성을 회복해야 하는데 다이나모 스타일 데이터스토어는 두 가지 메커니즘을 주로 사용한다. 

- 읽기 복구(Read repair)
  - 오래된 값이라는 사실을 알면 해당 복제 서버에 새로운 값을 다시 기록한다.
  - 이 접근 방식은 값을 자주 읽는 상황에 적합하다.
- 안티 엔트로피 처리 (Anti-entropy process)
  - 백그라운드 프로세스를 두고 복제 서버 간 데이터 차이를 지속적으로 찾아 누락된 데이터를 하나의 복제 서버에서 다른 서버로 복사한다.
 

#### 읽기와 쓰기를 위한 정족수

```
n: 총 복제 서버의 개수
w: 쓰기를 성공한 복제 서버의 개수
r: 읽기를 수행하기 위해 질의해야 하는 복제 서버의 개수
```

위의 예제에서 적어도 두개의 복제서버를 읽으면 두개 중 적어도 하나는 최신 값인지 확인 할 수 있듯이, **w + r > n 이면, 읽을 때 최신 값을 얻을 것으로 기대한다.**
<br> 이런 r과 w를 따르는 읽기와 쓰기를 **정족수 읽기와 쓰기**라고 부른다.


정족수 조건이 w+r > n 이면 아래와 같이 사용 불가능한 노드를 용인한다.

- w < n 이면 노드 하나를 사용할 수 없어도 여전히 쓰기를 처리할 수 있다.
- r < n 이면 노드 하나를 사용할 수 없어도 여전히 읽기를 처리할 수 있다.
- n = 3, w = 2, r = 2 이면 사용 불가능한 노드 하나를 용인한다.
- n = 5, w = 3, r = 3 이면 사용 불가능한 노드 둘을 용인한다.


![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/be2828c7-de58-450e-806f-938ed894ab9e)

필요한 w나 r개 노드보다 사용 가능한 노드가 적다면 쓰기나 읽기는 에러를 반환한다.<br>
하지만 노드는 다양한 이유로 사용이 불가능해질 수 있기 때문에 다양한 종류의 오류를 구별할 필요는 없다.


#### 정족수 일관성의 한계

읽기, 쓰기 정족수가 w + r > n 을 만족하는 경우에도 오래된 값을 반환하는 에지 케이스가 있다.

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/eb975292-5839-4231-8708-457bb7a633de)

1. 느슨한 정족수(↓)를 사용할 경우 쓰기가 완료된 노드와 읽기 대상 노드가 겹치지 않을 수 있다.
  - 즉, r개의 노드와 w개의 노드가 겹치지 않는 것을 보장하지 않는다.
2. 동시에 두 개의 쓰기가 발생할 경우 순서를 알 수 없기에 동시 쓰기를 합쳐야 하는데, 만약 타임스탬프를 기준으로 순서를 정하면(LWW) 시계 스큐(clock skew)로 쓰기가 유실될 수 있다. \
3. 읽기와 쓰기가 동시에 발생하면, 쓰기는 일부 복제 서버에만 반영될 수 있다.
  - 이 경우 읽기가 최신 값을 반환하는지 여부가 분명하지 않다.
 
![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/8a77929d-c6fe-4fea-8d3d-5f177ba54127)

4. 쓰기가 일부 성공 일부 실패로 설정한 w 수보다 성공 서버가 낮을 경우
  - 성공한 복제 서버에서는 롤백하지 않는다.
  - 쓰기가 실패한 것으로 보고될 경우 이어지는 읽기에 해당 쓰기 값이 반환될 수도 아닐 수도 있다.
 
5. 새 값을 전달하는 노드가 고장날 경우
  - 예전 값을 가진 노드에서 데이터가 복원되고
  - 새로운 값을 저장한 노드 수가 W보다 낮아져 정족수 조건이 깨질 수 있다.
6. 모든 과정이 정상 동작해도 시점의 문제로 에지 케이스가 있을 수 있다. (선형성과 정족수)


> 결국 정족수는 설정해 최근의 쓴 값을 반환하게끔 설정할 수는 있지만 절대적으로 보장할 수는 없다. 견고한 보장을 위해서 일반적으로 트랜잭션이나 합의가 필요하다.

#### 최신성 모니터링

운영 관점에서 DB가 최신 결과를 반환하는지 모니터링하는 것은 매우 중요하다. 

- 리더 기반 복제
  - 복제 지연에 대한 지표를 노출하며 모니터링 시스템에 제공된다.
  - 모든 쓰기가 리더를 통하기에 가능한 구조이다.
  - 리더의 현재 위치 - 팔로워의 현재 위치 = 복제 지연량
- 리더 없는 복제
  - 쓰기가 적용된 순서를 고정할 수 없기 때문에 모니터링이 더 어렵다.
  - 읽기 복구만 사용(안티 엔트로피X)하는 경우 읽기가 드문 값은 얼마나 오래된 값인지 제한이 없어 아주 오래된 값일 수 있다. 
 
#### 느슨한 정족수와 암시된 핸드오프

- 느슨한 정족수
  - 일단 쓰기를 받고 값이 보통 저장되는 n개 노드에 속하진 않지만 연결할 수 있는 노드에 기록하는 방법
  - w, r의 성공 응답이 필요하지만 값을 위해 지정한 n개의 홈 노드에 없는 노드가 포함될 수 있음을 허용한다.
- 암시된 핸드오프
  - 네트워크 장애 상황이 해제되면 한 노드가 다른 노드를 위해 일시적으로 수용한 모든 쓰기를 해당 홈 노드로 전송한다.


**느슨한 정족수의 장점**

- 쓰기 가용성을 높일 수 있다.
  - 모든 w개의 노드를 사용할 수 있는 동안 뿐만 아니라 일시적으로 n 이외의 일부 노드에 기록될 수 있다.
  - 단, 이 경우 w + r > n 이라도 키의 최신 값을 읽는다고 보장할 수는 없게 된다.
- 느슨한 정족수는 지속성에 대한 보장으로 데이터가 w 노드 이외에 저장될 수도 있다는 뜻이다.
- 암시된 핸드오프가 완료될 때까지는 r 노드의 읽기가 저장된 데이터를 본다는 보장은 없다.


### 동시 쓰기 감지

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/217ae1a9-1024-4d4c-8bad-22c243493885)

다이나모 스타일 데이터베이스는 동시에 같은 키에 쓰는 것을 허용하기에 엄격한 정족수를 사용해도 충돌이 발생할 수 있다. 


#### 최종 쓰기 승리(동시 쓰기 버리기, LWW)

각 복제본이 가진 “예전” 값을 버리고 가장 “최신” 값으로 덮어쓰는 방식이다.<br>
어떤 쓰기가 “최신”인지를 결정할 수 있는 한 모든 쓰기는 최종적으로 모든 복제 서버에 복사되어 동일한 값으로 수렴된다

- 쓰기는 자연적인 순서가 없지만 임의로 순서를 정할 수 있다.
- 쓰기에 타임스탬프를 붙여서 “최신”값을 선택하는 방법은 **LWW** 라고 한다.
- 최종적 수렴 달성이 목표지만 지속성은 희생된다.
    - 동일한 키에 여러번의 동시쓰기가 있다면 1개만 남고 나머지는 무시된다.
- 데이터 손실을 허용하지 않는다면 LWW는 적합하지 않다.


#### “이전 발생” 관계와 동시성

- 이전 발생
  - 작업 B가 작업 A에 대해 알거나 A에 의존적이거나, 어떤 방식으로든 A를 기반으로 한다면
  - 작업 A는 작업 B의 **이전 발생**이라 한다.

- 동시 작업
  - 작업이 다른 작업보다 먼저 발생하지 않으면 (어느 작업도 다른 작업에 대해 알지 못하면)
  - **동시 작업** 이라고 한다.


#### 이전 발생관계 파악하기

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/99bbe398-b2d2-4974-9737-27480d676c72)

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/49a832c6-7406-4b84-8407-cb75024f671f)


1. client1 - 우유 추가, version1 = [(우유)]
2. client2 - 달걀 추가, version2 = [(우유) , (달걀)]
3. client1 - 밀가루 추가, version3 = [(우유, 밀가루), (달걀)]
    - 1번 응답인 우유에 밀가루 추가, version2 값 달걀
4. client2 - 햄 추가, version4 = [(달걀, 우유, 햄), (우유, 밀가루)]
    - 2번 응답인 우유, 달걀에 햄 추가, version3의 우유, 밀가루
5. client1 - 베이컨 추가, version5 = [(우유, 밀가루, 달걀, 베이컨), (달걀, 우유, 햄)]
    - 3번 응답인 우유, 밀가루, 달걀에 베이컨 추가, version4의 달걀, 우유, 햄


동시, 이전 발생 결정 알고리즘

- 서버가 모든 키에 대한 버전 번호 유지하고 키를 기록 할 때마다 버전 번호를 증가시킨다. 기록한 값은 새로운 버전 번호를 가지고 저장된다.
- 클라이언트가 키를 읽을 때는 최신 버전과 덮어쓰지 않은 모든 값을 반환한다.
- 클라이언트가 키를 기록할 때는 이전 읽기의 버전 번호를 포함해야 하고 이전 읽기에서 받은 모든 값을 함께 합쳐야 한다.
- 서버가 특정 번호를 가진 쓰기를 받을 때는  해당 버전 이하 모든 값을 덮어쓸 수 있지만 높은 버전 번호의 값은 유지해야 한다.


#### 동시에 쓴 값 병합

여러 작업이 동시에 발생하면 클라이언트는 동시에 쓴 값을 합쳐 정리해야 한다. (리약은 이런 동시 값을 **형제(sibling) 값**이라 부른다.)

동시에 쓴 데이터를 병합하는 방법은 다음과 같다.

- LWW를 통해 병합
    - 타임스탬프를 통해 하나의 값을 선택한다.
    - 데이터 손실이 발생한다.
- Sibling 끼리 합집합을 취하는 것
    - 장바구니 예제에서 최종 두 개의 형제는 [우유, 밀가루, 달걀, 베이컨]과 [달걀, 우유, 햄]이다.
    - 이들의 합집합은 [우유, 밀가루, 달걀, 베이컨, 햄]이고 중복은 없다.
     
> 시스템은 형제(동시값)를 병합할때 상품을 제거했음을 나타내기 위해 해당 버전 번호에 표시를 남겨둬야하는데 이를 `툼스톤`이라고 한다.

#### 버전 벡터

다중 복제본의 동시쓰기를 받아들이기 위해서는 키당 버전 번호뿐만 아니라 복제본당 버전 번호도 사용해야 한다.<br>
각 복제본은 쓰기를 처리할 때 자체 버전번호를 증가시키고 각기 다른 복제본의 버전 번호도 계속 추적해야 한다. <br>
모든 복제본의 버전 번호 모음을 **버전 벡터** 라고 한다.