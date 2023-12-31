# ch03. 저장소와 검색(part 1)

# 데이터베이스를 강력하게 만드는 데이터 구조

- 데이터베이스의 기본적인 오퍼레이션
    1. 데이터 저장
    2. 데이터 검색
- 데이터베이스의 저장과 검색 방법을 알아야 하는 이유?
    1. 직접 구현하기 보다는 사용 가능한 것 중 적합한 것을 쓰기 때문에 적합한 엔진을 선택하기 위해서
    2. 특정 작업 부하 유형에서 좋은 성능을 내게끔 저장소 엔진을 조정하기 위해서
- 데이터 베이스의 종류인 관계형 데이터베이스와 NoSQL이라 불리는 데이터 베이스 저장엔진에 대해서 설명, **로그 구조(log0structed) 계열** 저장소 엔진과 **B tree 같은 페이지 지향(page-oriented) 계열** 저장소 엔진을 검토

---

- 일반적으로 파일 추가 작업은 매우 효율적이기 때문에 데이터베이스 내부적으로 추가 전용(append-only) 데이터 파일인 **로그(log)**를 사용

  > 이 책에서 **로그**라는 단어를 어플리케이션 로그가 아닌 조금 더 일반적인 의미로 연속된 추가 전용 레코드로 사용한다.
>
- 특정 키의 값을 효율적으로 검색하기 위해서 색인 데이터 구조가 필요하다. 다양한 색인 구조에 대해 살펴보고 여러 색인 구조를 비교해본다.

  > 색인이란 어떤 메타데이터를 유지하여 이정표 역할을 통해 데이터의 위치를 빠르게 찾는다.
>
- 색인은 기본 데이터(primary data)에 파생된 추가적인 구조다. 많은 데이터베이스는 색인의 추가와 삭제를 허용한다. 데이터베이스 내용에는 영향은 없지만 검색 속도에 영향을 준다.
  데이터를 쓸 때마다 색인을 갱신해야 하기 때문에 색인이라는 추가적인 구조는 쓰기 작업에서 오버헤드가 발생한다.
  하지만 이것은 저장소 시스템에서 중요한 트레이드 오프. 색인을 잘 선택해 쓰기 작업의 필요 이상 부담을 주지 않고 애플리케이션에 가장 큰 이득을 안겨주어야 한다.

## 해시색인

- 키를 데이터 파일의 바이트 오프셋에 매핑해 인 메모리 해시 맵을 유지한다.
- 해시 맵을 모두 메모리에 유지하기 때문에 모든 키가 저장된다는 전제로 고성능 읽기와 쓰기를 보장
- 각 키의 값이 자주 갱신되는 상황에 매우 적합
- 결국 디스크 공간이 부족

  특정 크기의 세그먼트(segment)로 로그를 나누는 방식이 좋은 해결책이다. 특정 크기에 도달하면 세그먼트 파일을 닫고 새로운 세그먼트 파일로 로그를 나누는 방식이 좋은 해결책이다. 이럴 경우 중복된 키를 버리고 각 키의 최신 갱신 값만 유지하는 컴팩션을 수행할 수 있다.

- 고정된 세그먼트의 병합과 컴팩션은 백그라운드 쓰레드에서 수행할 수 있다.
- 실제 구현에서는 세부적으로 많은 사항을 고려해야 한다.
    1. 파일 형식: 바이트 단위의 문자열 길이를 부호화한 다음 원시 문자열을 부호화하는 바이너리 형식을 사용하는 편이 빠르고 간단하다.
    2. 레코드 삭제: 키와 관련된 값을 삭제하려면 특수한 삭제 레코드를 추가해야 한다.
    3. 고장(Crash) 복구: 각 세그먼트 해시 맵을 전부 읽으면서 각 키에 대한 최신 값을 복구하면 오래 걸리니 조금 더 빨리 로딩할 수 있게 스냅샷을 디스크에 저장해 복구 속도를 높인다.
    4. 부분적으로 레코드 쓰기: 체크섬을 통해 데이터베이스가 죽었을 때 손상된 부분을 탐지할 수 있다.
    5. 동시성 제어: 데이터 세그먼트는 쓰기 전용이거나 불변으로 다중 쓰레드에서 동시에 읽기를 할 수 있다.
- 추가 전용 로그가 여러 측면에서 좋은 설계인 이유
    1. 추가와 세그먼트 병합은 순차적인 쓰기 작업이기 때문에 무작위 쓰기보다 훨씬 빠르다.
    2. 세그먼트 파일이 추가 전용이나 불면이면 동시성과 고장 복구가 훨씬 간단하다.
    3. 오래된 세그먼트 병합은 시간이 지남에 따라 조각화 되는 데이터 파일 문제를 피할 수 있다.
- 해시 테이블 색인의 제한 사항
    1. 메모리에 저장해야 하므로 키가 너무 많으면 문제가 된다. 원칙적으로는 디스크에 해시 맵을 유지할 수 있지만 좋은 성능을 기대하기 어렵다. 무작위 접근 IO가 많고 디스크가 가득찼을 때 확장 비용이 비싸면 충돌 해소를 위해 성가신 로직이 필요하다.
    2. 해시 테이블은 범위 질의(range qeury)에 효율적이지 않다.

## SS테이블과 LSM 트리

- 키-밸류 쌍을 키로 정렬하면 **정렬된 문자열 테이블(Sorted String Table)** 또는 짧게 **SS테이블**
- 해시 테이블 대비 장점
    1. 세그먼트 병합 시 파일이 사용 가능한 메모리보다 크더라도 간단하고 효율적이다. 병합 정렬과 동일한 방식
    2. 파일의 특정 키를 찾기 위해 모든 키를 메모리에 유지할 필요가 없다. 정렬되어 있는 키 사이에 위치하고 있다는 것을 알고 있기 때문에 부분만 스캔하면 된다.
    3. 읽기 요청은 요청 범위 내에서 여러 키-쌍을 스캔해야 하기 때문에 해당 레코드들을 블록으로 그룹화하고 디스크에 쓰기 전에 압축 한다. 디스크와 IO 대역폭을 절약한다.

### SS테이블 생성과 유지

- 디스크 상 정렬된 구조를 유지하는 일은 가능하지만 메모리에서 하는 것이 훨씬 쉽다.
- 저장소 엔진 만드는 방법
    1. 쓰기가 들어오면 멤테이블(Memtable, 인메모리 균형 트리 ex. Balanced Tree) 데이터 구조에 추가한다.
    2. 멤테이블이 보통 수 메가바이트 정도의 임곗값보다 커지면 SS 테이블 파일로 디스크에 기록한다. SS테이블을 디스크에 기록하는 동안 쓰기는 새로운 멤테이블에 기록한다.
    3. 읽기 요청을 제공하려면 먼저 멤 테이블에서 키를 찾아야 한다. 그 다음 디스크 상의 가장 최신 세그먼트에서 찾는 다. 그 다음으로 두번 째 오래된 세그먼트 순서로 탐색한다.
    4. 가끔 세그먼트 파일을 합치고 덮어 쓰여지거나 삭제된 값을 버리는 병합과 컴팩션 과정을 수행한다. 이 과정은 백그라운드에서 수행된다.
- 아주 잘 동작하지만 데이터베이스 장애가 발생하면 디스크에 기록되지 않은 멤테이블에 있는 가장 최신 쓰기는 손실 된다. 이런 문제를 피하기 위해 즉시 쓰기를 추가할 수 있게 분리된 로그를 디스크 상에 유지해야 한다. 이 로그는 복원할 때만 필요하기 때문에 순서가 정렬되지 않아도 문제되지 않는다.

### SS테이블에서 LSM 트리 만들기

- 원래 이 색인 구조는 **로그 구조화 병합 트리(Log-Strcuted Merge-Tree or LSM 트리)**로 정렬된 파일 병합과 컴팩션 원리를 기반으로 하는 저장소 엔진을 LSM 저장소 엔진이라 부른다.
- 루씬(Lucene)은 엘리스틱서치나 솔라에서 사용하는 전문 검색 색인 엔진이다. 단어를 키로하고 해당 단어를 포함하는 문서ID 목록을 값으로하는 구조로 구현한다.

### 성능최적화

- LSM 트리에서는 데이터베이스에서 존재하지 않는 키를 찾는 경우 느릴 수 있다. 멤테이블 확인 후 가장 오래된 세그먼트까지 탐색해야 하기 때문이다. 이런 문제를 최적화하기 위해서 **블룸 필터(Bloom filter)**를 추가적으로 사용한다.
- SS테이블을 압축하고 병합하는 순서와 시기를 결정하는 다양한 전략이 있다.
    - 크기 계층(size-tiered)과 레벨 컴팩션(leveled compaction)이다. 크기 계층 컴팩션은 상대적으로 좀 더 새롭고 작은 SS테이블을 상대적으로 오래되고 큰 SS 테이블에 연이어 병합한다. 레벨 컴팩션은 키 범위를 더 작은 SS테이블로 나누고 오래된 데이터는 개별 ”레벨”로 이동하기 때문에 컴팩션을 점진적으로 진행해 디스크 공간을 덜 사용한다.

## B트리

B트리는 블록이나 페이지라는 고정 크기로 나누어서 한번에 하나의 페이지에 읽기 또는 쓰기를 한다. 각 페이지는 주소나 위치를 이용해 식별할 수 있다. 이를 이용하여 한 페이지에서 다른 페이지를 참조한다.

이러한 페이지 중 하나를 루프(root) 페이지로 지정되며 루프 페이지를 통해 개별 데이터의 정보가 있는 리프(leaf) 페이지에 접근하여 데이터를 찾는다. 이때 한 페이지에서 가지는 하위 페이지 수를 분기 계수(branching factor)라고 부른다. 새로운 데이터를 추가하게 되면 해당 키가 포함되는 페이지를 찾아 데이터를 추가한다. 데이터가 추가되어 분기 계수 이상이 될 경우 페이지를 둘로 나누고 상위 페이지가 새로운 키 범위를 알 수 있도록 갱신한다. 분기 계수가 500인 4kb 페이지의 4단계 트리는 256TB까지 저장할 수 있다.

### 신뢰할 수 있는 B 트리 만들기

B트리의 기본적인 쓰기 동작은 새로운 데이터를 디스크 상의 페이지에 덮어쓰는 것이다. 페이지를 덮어쓰는 과정에서 문제가 발생하면 데이터가 훼손되어 큰 문제가 될 수 있다. 이를 해결하기 위해서 쓰기 동작에 대한 재 실행을 위해 로그(Write-Ahaed Log, WAL)를 기록하고 이를 활용해 스스로 장애 복구를 한다.

한편으로 같은 페이지를 다시 쓰기 위해서 동시성 제어는 굉장히 중요한 부분이다. 이를 위해서  래치(latch)라는 잠금을 이용하여 데이터 구조를 보호한다.

### B트리 최적화

- 페이지 덮어쓰기와 고장 복구를 위한 WAL 대신 일부 데이터베이스에서는 쓰기 시 복사 방식(copy-on-write scheme)을 사용한다. 변경된 페이지는 다른 위치에 기록하고 상위 페이지의 새로운 버전을 만들어서 새로운 위치를 가르키게 한다. 이 접근 방식은 동시성 제어에도 유용하다.
- 페이지에서 키의 전체 값이 아닌 적절하게 범위를 식별할 수 있는 값만을 저장하여 더 많은 키를 페이지에 저장할 수 있도록 한다. 이럴 경우 분기 계수가 높아져 전체적인 트리의 높이가 낮아질 수 있다.
- 가능한 리프 페이지들이 연속되게 위치하도록 한다.

## **B트리와 LSM 트리 비교**

일반적으로 B트리는 읽기 성능이, LSM 트리는 쓰기 성능이 좋다고 한다. LSM 트리의 경우 조회 시에 컴팩션 단계에 있는 여러 데이터와 SS테이블을 읽어야 하기 때문이다. 하지만 실제로 벤치성능이 결정적이진 않고 실제 비즈니스 프로세스에 따라 달라질 수 있다.

### LSM 트리의 장점

B트리는 데이터를 저장하기 위해 쓰기 전 로그와 트리에 한번씩 최소 2번은 써야 한다. 만약 데이터가 조금 달라지거나 바뀌면 페이지를 전체 다시 써야하는 오버헤드가 있다. 이러한 동일한 데이터를 여러 번 쓰는 걸 쓰기 증폭이라고 한다.

LSM 트리는 이러한 쓰기 증폭이 상대적으로 적다. 컴팩션된 SS 트리에 쓰고 순차적으로 쓰기 때문에 압축률도 좋다. 따라서 같은 IO 대역폭 내에서 더 많은 데이터를 읽고 쓸 수가 있다.

### LSM 트리의 단점

LSM 트리는 컴팩션이라는 과정이 존재한다. 이 과정 중에 읽기와 쓰기 작업이 지연될 수 있다. 이럴 경우 쓰기 읽기 속도의 상위 백분위가 꽤 길어질 수 있다. 다른 문제로는 쓰기 속도가 너무 빠를 경우 컴팩션이 제대로 진행되지 않아 너무 많은 데이터를 해야한다.

또한 B트리는 색인의 리프에만 데이터가 존재하지만 LSM 트리는 여러 곳에 존재할 수 있다. 이럴 경우 강력한 트랜잭션 제어가 어려울 수 있다.
