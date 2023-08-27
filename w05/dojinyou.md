# ch03. 데이터 저장과 검색(part 2)

## 기타 색인 구조

대표적인 키-밸류 색인에는 데이터베이스의 Primary Key가 있다. 보조 색인(Second Index)를 사용하는 것도 일반적인 방법이다. 보조 색인은 PK와 다르게 키가 유일하지 않을 수 있다. 이것을 해결하기 위해서는 식별자의 해당하는 많은 로우 식별자 목록을 만들거나 로우 식별자를 추가해서 고유하기 만들 수 있다. 이 방법 모두 B트리와 로그 구조화 색인 모두 사용할 수 있다.

### 색인 안에 값 저장하기

색인의 값은 실제 값이거나 실제 값을 가르키는 참조이다. 이때 참조를 가지고 있을 경우 실제 값이 저장된 곳을 힙파일(heap file)이라고 한다. 이러한 힙파일 접근은 데이터의 중복을 피할 수 있어 일반적으로 사용된다.

값이 변경되지 않을 경우 키를 변경이 유연하지만 값을 새로 쓸 경우 오버헤드가 크다. 이를 해결하기 위해서 색인된 로우를 바로 저장하는 클러스터드 색인(Clustered Index)를 사용한다.

이 뿐만 아니라 비클러스터 인덱스라도 커버링 인덱스(Covering Index)를 통해 특정 열에 대한 응답을 색인 내에서 해결할 수 있다.

모든 종류의 데이터 복제와 마찬가지로 읽기 성능은 높일 수 있지만 추가적인 저장소와 쓰기 과정이 필요하고 복제에 대한 불일치 파악을 어플리케이션에서 할 수 없어 트랜잭션 보장을 강화하기 위한 별도의 노력이 필요하다.

### 다중 컬럼 색인

여러 가지 컬럼의 조건을 동시에 줄 경우 B트리나 LSM트리는 효율적으로 대응하기 어렵다. 이런 경우에는 R 트리와 같은 공간 전문 색인과 같이 다차원 색인에 적합한 형태를 사용하는 것이다. PostgreSQL은 일반화된 검색 트리(Generalized Search Index)를 사용했다.

### 전문 검색과 퍼지 색인

이전에 나온 색인들은 모두 같이 일치하거나 범위를 탐색할 때 효율적으로 동작했다. 하지만 이러한 형태는 애매모호한(fuzzy) 질의에 대해서는 제대로 대응하기 어렵다.

이를 대응하기 위해서는 전문 검색엔진과 같은 것을 사용할 수 있따. 루씬은 문서나 질의에 대해 특정 편집 거리 내 단어를 검색할 수 있다.

### 모든 것을 메모리에 보관

디스크에 대한 해결책으로 다양한 방법들을 이야기했지만 램 가격이 점점 낮아져 현재는 데이터를 모두 메모리에 보관할 수도 있다. 이런 이유로 인 메모리 데이터베이스가 개발되었다.

멤캐시와 같은 휘발성 캐시도 있지만 로그나 스냅샷 같은 기법이나 특수한 하드웨어를 이용하여 지속성을 가지는 것을 목표로 하기도 한다.

인메모리 데이터베이스의 성능적 장점은 디스크에서 읽지 않는 것이 아니라 다양한 자료구조와 쉬운 구현이다. 또한 최근 연구에 따르면 가용 메모리보다 더 큰 데이터 셋을 제공하기 위해 안티 캐싱(anti caching)접근 방식을 통해 사용하지 않은 데이터를 디스크로 내보내고 나중에 다시 접근할 때 메모리에 올려서 사용할 수 있다.

나아가 비휘발성 메모리(non-valatile memory) 기술이 앞으로 주목될 수 있다.

# 트랜잭션 처리나 분석?

온라인 상의 데이터 처리는 OLTP(OnLine Transaction Process)와 OLAP(OnLine Analytic Process)로 나누어 진다. 전통적인 OLTP는 마켓에서 시작된 트랜잭션을 의미하며 최근에는 짧은 지연의 읽기 쓰기를 의미할 수도 있다. OLAP는 데이터의 집계를 통한 분석을 위한 용도로 사용되며 OLTP와 다르게 많은 데이터를 집계하며 현재의 상태가 아닌 히스토리를 기반으로 접근한다는 것이 다르다. 따라서 이러한 용도에 맞는 데이터 저장소를 별도로 구축 하기도 하는데 이것을 “데이터 웨어하우스”라고 한다.

## 데이터 웨어하우징

OTLP의 환경에서 분석용 질의를 할 경우 비용이 비싸서 OLTP 성능에 영향을 줄 수 있다. 따라서 별도의 웨어하우스에서 영향을 주지 않고 질의하는 것이 좋다. 이를 위해서는 대게 개별적으로 운영되는 다양한 OLTP 데이터 저장소에서 추출(Extract)하고 적절한 스키마로 변환(transform)하여 데이터 웨어하우스에 적재(load)한다.(ETL과정)

### OTLP 데이터베이스와 데이터 웨어하우스의 차이점

SQL의 기반의 데이터 웨어하우스를 구축할 수도 있지만 실제로 이러한 저장소는 전혀 다른 질의에 최적화 되어 있다. 하나의 데이터베이스에서도 둘 다 지원하는 경우도 있지만 점차 분리하는 추세이다.

## 분석용 스키마: 별 모양 스키마와 눈꽃송이 모양 스키마

데이터 분석용 데이터 형태는 크게 다르지 않고 별 모양 스키마(Start Scheme)를 사용한다. 별 모양 스키마는 중앙에 사실 테이블이라는 이벤트를 저장하고 있고 주변의 차원 테이블들은 이러한 이벤트에 대한 각종 메타 정보(누가, 언제, 어디서, 무엇을, 어떻게, 왜 등)를 가지고 있다. 이것을 조금 더 정규화하고 차원 테이블들을 더욱 세분화하여 눈꽃송이 모양 스키마(Snowflake Scheme)도 많이 사용된다.

# 칼럼 지향 저장소

데이터 분석 관점에서는 100개 이상의 칼럼 중 3,4개 정도의 컬럼에 접근하길 원한다. 하지만 Row기반의 DBMS들은 해당 칼럼 접근을 하기 위해 100개 이상의 칼럼들을 모두 가져오는 행위를 해야 한다. Row기반의 DBMS는 연관성이 높은 같은 ROW의 데이터들을 함께 배치하고 있기 때문이다. 이러한 불필요한 오버헤드를 제거하기 위해서 컬럼기반으로 저장하고 관리하는 컬럼 지향 저장소가 적절하다. 필요한 컬럼만 읽어서 가져온다.

## 칼럼 압축

칼럼 값의 자세히 살펴보면 굉장히 많은 값이 반복적으로 나타난다. 실제로 Row의 수보다 해당 칼럼의 유니크한 값의 수가 압도적으로 적다. 따라서 이를 비트맵 부호화로 압축해서 표현할 수 있다. 유니크한 값에 해당하는 지 여부를 값의 수 만큼 길이를 가진 비트의 0(없음)과 1(있음)로 나타낼 수 있다.

만약 유니크한 값이 꽤 많다면 대부분 0으로 채워지는 현상이 발생한다. 이를 희소(sparse)하다고 한다. 이럴 경우 런 렝스 부호화(run-length encoding)을 통해서 압축을 할 수 있다. 가령 예를 들어 9,1 인 경우 0이 9개, 1이 1개, 나머지가 모두 0임을 의미한다. 이러듯 연속된 0을 압축적으로 표현한다.

### 메모리 대역폭과 벡터화 처리

수백만 로우를 스캔해야하는 데이터 웨어하우스 질의 특성상 디스크에서 메모리로 데이터를 가져오는 대역폭이 병목이 된다. 이 뿐만 아니라 CPU의 분기 예측 실패(branch misprediction)와 버블(bubble)을 피하며 CPU가 단일 명령 다중 데이터 명령을 사용하게끔 신경 써야 한다.

칼럼 저장소 배치는 CPU 주기를 효율적으로 사용하기 위해서 L1캐시의 크기에 딱 맞는 덩어리로 가져오고 이를 타이트 루프(tight loop)에서 반복한다. 칼럼 압축을 통해 더 많은 데이터를 L1에 맞춰 저장할 수 있고 비트 연산을 통해 비트 덩어리를 바로 연산할 수 있게 한다. 이런 기법을 벡터화 처리(vectorized processing)라고 한다.

## 칼럼 저장소와 순서 정렬

칼럼 저장소의 경우 row의 순서가 중요하지 않다. 꼭 삽입된 순서로 칼럼들을 정렬하고 있을 필요가 없기 때문이다. 따라서 적절한 다른 값을 통해 정렬할 수 있다.이를 이용하면 유사한 데이터가 인접하게 붙어 있게 되고 압축률이 더욱 좋아진다.

### 다양한 순서 정렬

다양한 질의는 서로 다른 순서의 도움을 받기 때문에 같은 데이터라도 다양한 방식으로 정렬해두고 질의에 맞는 적절한 데이터를 사용할 수 있다.

## 칼럼 지향 저장소에 쓰기

데이터 웨어하우스에서 대용량 데이터에 대한 읽기 전용 질의의 최적화는 합리적이다. 하지만 쓰기가 매우 어려워진다. 이 부분에 대해서 앞선 LSM 트리라는 좋은 해결책을 설명했다. 모든 쓰기는 먼저 인 메모리 저장소로 이동해 정렬된 구조에 추가하고 디스크에 쓸 준비한다. 충분히 모이면 디스크 칼럼 파일에 병합하고 대량으로 새로운 파일에 기록한다.

## 집계: 데이터 큐브와 구체화 뷰

모든 데이터 웨어하우스에서 칼럼 기반 저장소가 필수는 아니다. 전통적인 로우 기반 저장소도 사용 가능하다.

데이터 웨어하우스의 다른 측면으로는 구체화 집계(materialized aggregate)가 있다. 동일한 집계를 여러 곳에서 이용한다면 매번 계산하는 것은 낭비다. 이러한 캐시를 만들어야한다면 구체화 뷰(materialized view)가 좋은 방법이다.

관계형 데이터베이스 구체화 뷰는 비정규화된 복사본이기 때문에 원본 데이터가 변경될 경우 쓰기 비용이 비싸다.

장점은 집계 함수의 계산이 되어 있어서 별도로 계산하지 않아도 빠르게 동작된다는 것이지만 단점은 이 데이터가 유연하게 동작하지 않는다.