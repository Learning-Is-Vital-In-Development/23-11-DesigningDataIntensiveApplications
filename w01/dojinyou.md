# 신뢰할 수 있고 확장 가능하며 유지보수하기 쉬운 애플리케이션

현대 애플리케이션은 CPU와 같은 계산의 성능보다 다뤄야하는 데이터의 크기, 형태, 속도에서 문제를 겪게 된다. 이를 데이터 중심 애플리케이션이라고 한다.

이러한 데이터 중심 애플리케이션을 만들기 위해서는 표준 구성 요소 블럭들이 있다.

- 데이터베이스
- 캐시
- 검색 색인
- 스트림 처리(비동기 처리 요청을 위한)
- 배치 처리(주기적 대용량 데이터 처리)

# 데이터 시스템에 대한 생각

데이터 시스템은 여러 가지 도구들을 아우른다.  현대에는 다양한 이유로 이들을 전통적으로 구분하는 것은 무의미해졌다.

먼저 데이터 저장과 처리를 위한 다양한 도구들이 최근에 만들어졌다. 한편으로 하나의 도구로 온전히 요구사항을 만족하기가 어려워졌다. 따라서 도구를 효율적으로 쓸 수 있도록 태스크를 구성하고 이를 적절히 코드 레벨에서 추상화하여 연결한다.

애플리케이션의 관심사는 크게 3가지로 신뢰성, 확장성, 유지보수성이다. 신뢰성은 간단하게 다양한 오류 상황(하드웨어, 소프트웨어, 인력 등)에서도 정상적인 동작에 대한 기대이다. 확장성은 다양한 요청의 변화에 대해서 유연한 대응에 대한 기대이다. 유지보수성은 시간이 지남에 따라 다양한 사람이 시스템을 작업하기 때문에 모든 이들이 생산성을 가지고 작업할 수 있다는 기대를 의미한다.

# 신뢰성(reliability)

신뢰성이란 소프트웨어는 다음과 같은 기대를 의미한다.

- 사용자가 기대한 기능을 수행한다.
- 사용자가 범한 실수나 예상치 못한 사용법을 허용 한다.
- 예상된 부하와 데이터  양에서 충분한 필수적인 기능을 제공한다.
- 시스템에 허가되지 않은 접근과 오남용을 방지한다.

결함은 잘못되는 것을 의미하고 결함을 예측하고 대처할 수 있는 것을 내결함성이라고 한다. 장애와 결함을 다르다. 결함은 일부 구성요소가 시스템의 기대를 벗어나는 것을 의미하지만 장애는 사용자에게 필요한 서비스를 제공하지 못하고 시스템 전체가 멈춘 경우다.

## 하드웨어 결함

하드웨어의 장애는 대용량 데이터센터에서는 굉장히 빈번하게 발생하고 약 10000개의 하드 디스크가 있다면 일반적으로 하루에 1대가 결함이 발생하는 것이 기대된다.

이를 해결하기 위해서는 일반적으로 하드웨어 구성요소를 중복으로 가져가 핫스왑이 되도록 하여 내결함성을 높일 수 있습니다.

## 소프트웨어 오류

하드웨어의 결함을 독립적인 낮은 상관 관계를 갖지만 시스템 내 체계적 오류는 높은 상관 관계를 가지고 다수의 시스템 오류를 발생시킬 수 있다.

이와 같은 예로는 다음의 경우들이 있다.

- 잘못된 특정 입력이 있을 때 모든 애플리케이션 서버 인스턴스가 죽는 소프트웨어 버그
- CPU 시간, 메모리, 디스크 공간, 네트워크 대역폭처럼 공유 자원을 과도하게 사용하는 일부 프로세스
- 시스템의 속도가 느려져서 반응이 없거나 잘못된 응답을 하는 서비스
- 한 구성 요소의 작은 결함이 다른 구성 요소의 결함으로 이어져 더 많은 결함을 발생 시키는 연쇄 장애(cascading failure)

소프트웨어 오류에 대한 신속한 해결책은 없다. 여러 가지 다양한 장치로 지속적으로 확인하고 경고를 발생 시켜야 한다.

## 인적 오류

신뢰할 수 없는 인간을 제어하기 위한 방법들

- 오류의 가능성 자체가 적은 쪽으로 시스템 설계하기
- 사람이 실수할 수 있는 부분 분리하기.  Sandbox 제공
- 모든 수준에서 철저한 자동 테스트하기
- 장애 발생 시 쉽고 빠르게 롤백하고 배포는 천천히 롤아웃하자
- 성능 지표와 오류 등을 정확하게 모니터링 하자.
- 조작 교육과 실습을 실행하자.

# 확장성(scalability)

확장성은 증가한 부하에 대처하는 시스템 능력을 설명하는 데 사용하는 용어지만 일차원적인 용어는 아니다.

## 부하 기술하기

부하를 기술하기 위해서는 매개변수를 잘 정의해야 한다. 적절한 매개변수의 선택은 시스템 설계에 따라 달라질 수 있다.

## 성능 기술하기

성능을 테스트하기 위해서는 기존 자원에서 매개변수를 높이면서 변화를 추적하거나 특정 매개변수에 맞는 자원을 찾는 방식으로 테스트할 수 있다.

이때 지표로는 처리량 또는 응답시간이 사용될 수 있다. 응답시간은 지연시간과 다르게 클라이언트가 느끼는 요청에 대한 응답시간이다.

이러한 지표들은 매 요청마다 같지 않고 다양한 값을 가지므로 분포를 통해 접근 해야 한다. 이때 꼬리 지연시간(tail latency)라고 불리는 상위 응답 시간은 사용자 경험에 직접적인 영향을 주기 때문에 중요하다.

또한 서버는 병렬로 소수의 작업만 처리가 가능하기 때문에 소수의 느린 요청 처리에 의해 후속 요청이 지연된다. 이러한 현상을 선수 차단(head-of-line blocking)이라고 한다. 다른 여러 가지 요청의 응답이 준비되더라도 하나의 늦은 응답이 있을 경우 전체 응답 시간에 영향을 주어 후속 응답이 늦어지는 현상을 발생 시킨다.

## 부하 대응 접근 방식

확장성을 위해 일반적으로 장비의 성능을 올리는 스케일 업이나 다수의 비슷한 사양의 장비를 추가하는 스케일 아웃을 고려한다. 다수의 장비에 부하를 분산하는 아키텍처를 비공유(shared-nothing) 아키텍처라고 한다. 일부 시스템은 부하의 변경에 따라 자원이 탄력적으로 운영된다.

대게 대규모로 동작하는 시스템의 아키텍처는 어플리케이션에 특화되어 있다. 아키텍처를 결정하는 다양한 요소들이 동일하지 않기 때문이다. 특정 애플리케이션은 주요 동작이 무엇인지 잘 하지 않는 동작인지 무엇인지 등 가정을 기반으로 설계된다. 이러한 가정은 곧 부하 매개변수가 된다. 하지만 이런 애플리케이션 특화 아키텍처도 익숙한 패턴으로 나열된 범용적인 구성 요소로 구축한다.

# 유지보수성(maintainability)

유지보수를 하는 걸 사람들은 좋아하지 않는다. 그래서 유지보수하기 쉽도록 설계 해야 한다.  이렇게 설계하는 원칙 3가지는 다음과 같다.

1. 운용성(operability) : 원활하게 운영하기 쉬워야 한다.
2. 단순성(simplicity) : 시스템에서 복잡도를 최대한 제거하여 단순하게 만들어서 이해가 쉬워야 한다.
3. 발전성(evolability): 엔지니어가 이후에 손쉽게 변경할 수 있어야 한다. 이 속성은 유연성(extensibility), 수정 가능성(modifiability), 적응성(plasticity)으로 알려져있다.

## 운용성: 운영의 편리함 만들기

좋은 운영팀은 동일하게 반복되는 태스크를 자동화 하고 고부가가치에 집중할 수 있도록 돕는다.

## 단순성: 복잡도 관리하기

복잡도를 낮게 만들고 단순화 시켜야 하는데 이것은 우발적 복잡도(accidental complexity)를 낮춘다는 뜻일 수 있다. 우발적 복잡도를 낮추기 위해서는 좋은 추상화를 통해 깔끔하고 직관적은 인터페이스를 제공하고 구현을 숨겨야 한다.

## 발전성: 변화를 쉽게 만들기

요구사항이 변경되지 않을 가능성은 매우매우 적다. 애자일 기법은 아주 작은 단위에 대해서 빠르고 민첩하게 요구사항에 대응하고 변화하는 것을 이야기한다. 하지만 대규모 시스템에서의 유연한 대응과 변화를 위해 민첩성이 필요한데 이 민첩성을 발전성이라고 하겠다.

# 정리

애플리케이션이 유용하기 위해서는 다양한 요구사항을 충족해야 한다. 기능적 요구사항과 비 기능적 요구사항이 있고 비 기능적 요구사항인 신뢰성, 확장성, 유지보수성에 대해서 이야기했다.

신뢰성은 결함이 발생해도 올바르게 동작한다는 의미이다. 다른 말로 내결함성이라고 할 수 있다. 결함은 하드웨어, 소프트웨어, 사람에게서 발생할 수 있다.

확장성은 부하가 증가해도 좋은 성능을 유지하기 위한 전략이다. 매개변수를 기반으로 다양한 전략을 수립할 수 있다.

유지보수성은 좋은 추상화를 통해 복잡도를 낮추고 반복되는 일을 자동화하여 쉽게 편리하게 운용하며 고부가가치에 집중할 수 있도록 하는 것이다.
