# 일괄처리

- 서비스(온라인 시스템): 요청하나가 오면 가능한 빨리 요청을 처리 해 응답을 되돌려 보내려 한다. 때로는 가용성이 매우 중요하여 클라이언트가 서비스에 접근 못하면 오류 메세지를 받을 수 있다.
- 일괄 처리 시스템(오프라인 시스템): 매우 큰 입력을 받아 작업을 수행하고 결과 데이터를 생산한다. 매우 오래 걸리는 작업으로 입력된 데이터 중 특정 크기를 처리할 때 걸리는 시간을 통해 처리량을 나타난다.
- 스트림 처리 시스템(준 실시간 시스템): 입력 데이터를 소비하고 출력 데이터를 생산한다. 일괄 처리 작업은 정해진 크기의 입력에 대해서 작동하지만 스트림 처리는 이벤트 발생한 직후 바로 작동한다.

이번 장에서는 맵 리듀스를 알아보고 다른 일괄 처리 알고리즘과 프레임워크도 살펴본다. 그리고 현대 데이터 시스템에서는 이것을 어떻게 사용하는지 알아본다.

# 유닉스 도구로 일괄 처리하기

웹 서버에 요청이 올 때마다 로그 파일에 한 줄씩 추가한다고 가정해보자. 크게 포맷과 실제 데이터가 있을 것이다.

## 단순 로그 분석

인기 있는 웹사이트 5개를 뽑는다면 아래와 같이 처리할 수 있다.

```bash
cat /var/log/nginx/access.log \
  awk '{print $7}' \
  sort \
  uniq -c \
  sort -r -n \
  head -n 5
```

1. 로그를 읽는다.
2. 줄마다 공백으로 분리해서 요청 URL의 7번째에 해당하는 필드만 출력한다.
3. 요청 URL을 정렬한다.
4. 인접한 두 줄이 같은 지 확인해서 중복을 제거한다. `-c` 옵션은 중복 횟수를 함께 출력하는 옵션이다.
5. `-n` 옵션을 사용해 매 줄 첫번째로 나타나는 숫자(중복 횟수)로 재 정렬한다. `-r` 은 내림차순을 의미
6. 마지막으로 `head` 명령어를 통해서 앞에서 5줄만`(-n 5)` 출력한다.

유닉스 도구에 익숙하지 않다면 앞에서 사용한 명령줄을 이해하기 쉽지 않다. 하지만 수 기가 바이트의 로그 파일을 수초 내로 처리할 수 있고 필요에 따라 분석 방법도 수정할 수 있기 때문에 강력한 방식이다.

### 연쇄 명령 대 맞춤형 프로그램

유닉스 인쇄 대신 같은 작업을 하는 간단한 프로그램을 작성할 수도 있다. 유닉스 연쇄 파이프라인보다 간결하지는 않지만 더 읽기 쉽다. 그러나 표면적인 문제는 문법적인 차이를 빼고도 실행 흐름이 크게 다르다.

### 정렬 대 인 메모리 집계

루비 스크립트는 URL 해시 테이블을 메모리에 유지하며 URL 수를 매핑한다. 하지만 유닉스 파이프라인에서는 이런 해시 테이블이 아닌 정렬된 목록에서 같은 URL이 반복해서 나타난다.

작업 세트가 충분히 작다면 인메모리 해시 테이블도 잘 작동한다. 반면 허용 메모리보다 작업 세트가 크다면 정렬 접근법을 사용하는 것이 좋다. 먼저 데이터 청크를 메모리에서 정렬하고 청크를 세그먼트 파일로 디스크에 저장한다. 그 다음에 각각 정렬된 세그먼트 파일을 여러 개의 한 개의 큰 정렬 파일로 병합한다. 병합 정렬을 순차적 접근 패턴을 따르므로 디스크에서 좋은 성능을 낼 수 있다.

GNU Coreutils(리눅스)에 포함된 sort 유틸리티는 메모리보다 큰 데이터셋을 자동으로 디스크로 보내고 자동으로 여러 CPU 코어에서 병렬로 정렬한다.

## 유닉스 철학

[Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy)

유닉스 쉘을 사용하면 작은 프로그램들을 가지고 놀랄만큼 강력한 데이터 처리 작업을 쉽게 **구성**할 수 있다. 이런 프로그램 중 다수를 다른 그룹에 속하는 사람들이 만들었지만 유연한 방식으로 함께 조합할 수 있다.

### 동일 인터페이스

특정 프로그램이 어떤 프로그램과도 연결 가능하려면 프로그램 모두가 동일한 입출력 인터페이스를 사용해야 한다는 의미다. 유닉스에서 인터페이스는 파일이다.

### 로직과 연결의 분리

유닉스의 도구의 다른 특징으로 표준 입력(stdin)과 표준 출력(stdout)을 사용한다는 점을 들 수 있다. 프로그램은 입력이 어디서 들어오고 출력이 어디로 나가는 지 신경 쓰거나 알 필요가 없다.

그러나 stdin, stdout을 사용할 때는 몇 가지 제약사항이 있다. 여러 개의 입력과 여러 개의 출력이 필요할 때는 불가능하지 않지만 까다롭다. 출력을 네트워크와 연결하지 못한다.

### 투명성과 실험

유닉스가 성공적인 이유 중 하나는 진행사항을 파악하기 쉽기 때문이다.

- 유닉스 명령에 들어가는 입력 파일은 불변으로 처리된다.
- 어느 시점이든 파이프라인을 중단하고 출력을 파이프를 통해 less로 보내 원하는 형태가 나오는지 확인 할 수 있다.
- 특정 파이프라인 단계의 출력을 파일에 쓰고 그 파일을 다음 단계 입력으로 사용할 수 있다. 파이프 라인을 다시 시작하지 않고 다음 단계부터 시작할 수 있다.

유닉스 도구를 사용하는데 가장 큰 제약은 단일 장비에서만 동작한다는 점이다. 이점은 하둡같은 도구가 필요한 이유이다.

# 맵리듀스와 분산 파일 시스템

맵리듀스는 유닉스 도구와 비슷하지만 수천대 장비로 분산해 실행이 가능하다는 점에서 차이가 있다. 단일 맵리듀스 작업은 하나 이상의 입력을 받아서 하나 이상의 출력을 만들어 낸다. 입력을 수정하지 않기 때문에 출력 외에 부수적 효과는 없다. 출력 파일은 순차적으로 쓰여지고 한번 쓰여지면 고쳐지지 않는다. 입 출력으로 파일 시스템 상의 파일을 사용한다. 하둡 맵리듀스가 사용하는 파일 시스템을 HDFS(Hadoop Distributed File System)이다.

HDFS는 비공유 원칙을 기반으로 한다. HDFS은 각 장비에서 실행되는 데몬 프로세스로 구성된다. 데몬 프로세스는 외부에서 파일 시스템에 접근할 수 있는 네트워크를 제공하고 **네임노드(NameNode)**라고 불리는 중앙 서버는 특정 파일 블록이 어떤 장비에 저장되어 있는 지 추적한다.

장비가 죽거나 디스크 실패하는 경우를 대비해 여러 장비에 복제된다.

> 삭제 코딩: 저장 공간의 효율성을 높이려고 설계된 복제 방식. 이레이저 코드를 사용해 데이터를 인코딩하고, 데이터가 손실되면 디코딩 과정을 거쳐 원본 데이터를 복구함.
이때 많이 사용되는 알고리즘 중 하나가 [Reed-Solomon Code](https://www.slideshare.net/RamSinghYadav2/implementation-of-reed-solomon-codes-basics)다.
>

## 맵리듀스 작업 실행하기

맵리듀스의 작업 하나는 4단계로 수행한다.

1. 파일을 나누어 레코드를 만드는데 입력 형식 파서 사용
2. 맵(구현): 매퍼는 모든 입력 레코드마다 한번 호출되며 키와 값을 출력하는 작업
3. 정렬 단계로 맵리듀스에 내재되어 있음.
4. 리듀스(구현): 키-값 쌍을 받아서 같은 키를 가진 레코드를 모아서 값의 집합에 반복적인 리듀스를 호출한다.

### 맵리듀스의 분산실행

유닉스와 맵 리듀스의 가장 큰 차이는 병렬로 수행하는 코드를 여러 장비에서 동시에 처리 가능하다는 점이다. 맵리듀스의 병렬실행은 파티셔닝을 기반으로 한다. 입력값으로는 보통 HDFS상의 디렉토리를 사용하는 것이 일반적이고 디렉토리 내 파일 또는 파일 블록을 독립된 맵 태스크에서 처리할 파티션으로 간주한다.

데이터 가까이에서 연산하기 위해서 가능하다면 입력 파일이 있는 장비에서 수행하려고 한다.

키-값 쌍이 반드시 정렬돼야 데이터셋이 매우 크기 때문에 단계를 나누어 정렬한다. 각 맵 태스크는 기 해시값을 기반으로 출력을 리듀서로 파티셔닝한다. 그리고 각 파티션을 매퍼의 로컬 디스크에 정렬된 파일로 기록한다.

매퍼가 입력 파일을 정렬된 출력 파일 기록하기를 완료하면 스케쥴러는 그때 출력 파일을 가져올 수 있다고 리듀서에게 알려준다. 정렬 한 뒤에 매퍼로부터 데이터 파티션을 복사하는 과정을 셔플(suffle)이라 한다.

### 맵리듀스 워크플로

맵리듀스 작업 하나로는 해결할 수 있는 문제가 적기 때문에 이를 맵리듀스 워크플로(workflow)로 연결하여 복잡한 문제를 해결하는 것이 일반적이다. 하둡 맵리듀스 프레임워크는 워크플로를 명시적으로 지원하지 않기 때문에 암묵적으로 이전 출력 디렉토리를 다음 입력 디렉토리로 사용하여 연결한다.

그렇기 때문에 이전 작업이 완전히 끝나야지 다음 작업을 수행할 수 있다. 이러한 의존성 관리를 위해 다양한 스케쥴러가 개발되었다. 예로 우지, 아즈카반, 루이지, 에어플로, 핀볼 등이 있다.

이런 스케쥴러의 많은 일괄 처리 작업을 유지 보수할 때 유용한 관리 기능이 있는데 이를 위한 도구도 많다. 예로 피그, 하이브, 캐스캐이딩, 크런치, 플룸자바 같은 다양한 하둡용 고수준 도구들이 있다.

## 리듀스 사이드 조인과 그룹화

여러 데이터 셋에서 한 레코드가 다른 레코드와 연관이 있는 것은 일반적이다. 이를 표현하기 위해서 관계형 데이터 모델의 외래키, 문서형 데이터 모델의 문서 참조, 그래프 모델에서의 간선 등의 개념을 사용한다. 데이터베이스에서 적은 수의 레코드에 관련된 질의라면 색인(Index)를 사용해 더 빠르게 검색한다. 맵리듀스에는 일반적으로 이야기하는 색인의 개념은 없다.

적은 수의 레코드를 읽을 때는 전체 테이블 스캔(full table scan)을 하게 되면 탐색 비용이 너무 많이 든다. 하지만 일반적으로 분석 질의는 대량의 레코드를 대상으로 집계 연산을 한다.

### 사용자 활동 이벤트 분석 예제

사용자 활동 이벤트에는 단지 사용자 식별자(ID)만 포함하고 있다. 사용자 프로필의 나이나 생일을 이용하기 위해선 데이터베이스의 프로필과 조인해야한다.

가장 손쉬운 조인 방법은 식별자마다 DB에 질의하는 것이다. 안될 건 없지만 나쁜 성능으로 고생할 것이다. 일괄 처리에서 처리량을 높이기 위해서는 한 장비 내에서 연산을 수행해야한다. 따라서 더 좋은 방법은 사용자 데이터 베이스의 사본을 가져와 사용자 활동 이벤트 로그가 저장된 분산 파일 시스템에 넣는 방법이다.

### 정렬 병합 조인

매퍼는 입력 레코드로부터 키와 밸류를 추출하는 것이 목적이라는 것을 잊지 말자. 맵리듀스 프레임워크에서 키로 매퍼의 출력을 파티셔닝해 키-값 쌍으로 저열ㄹ한다면 같은 사용자 활동 이벤트와 사용자 레코드는 리듀서의 입력으로 인접해서 들어간다.

리듀서가 항상 사용자 데이터베이스를 먼저 보고 활동 이벤트를 시간 순으로 보게 하는 식으로 맵리듀스에서 작업 레코드를 재배열하기도 한다. 이 기술은 보조 정렬(secondary sort)이라 한다.

리듀서 함수는 모든 사용자 식별자당 1번씩만 호출되고 보조 정렬로 생년 월일가 첫번째 값임을 예상할 수 있다. 이후 같은 사용자 ID의 이벤트를 순회해 URL과 연령 쌍을 출력한다.

리듀서는 특정 ID에 대해서 1번만 호출되기 때문에 1명의 사용자 프로필을 메모리에 유지하면 되고 네트워크로 아무 요청도 보낼 필요가 없다. 이 알고리즘을 정렬 병합 조인(sort-merge join)이라고 한다.

### 같은 곳으로 연관된 데이터 가져오기

병합 정렬 조인 중 매퍼와 정렬 프로세스는 특정 사용자 ID로 조인 연산을 할 때 필요한 데이터를 한 곳으로 모은다. 그래서 사용자 ID별로 리듀서를 한번만 호출한다.

매퍼가 키-값쌍을 보낼 때 키는 값을 보낼 목적지의 주소 역할을 한다. 같은 키라면 같은 목적지로 배달된다.

### 그룹화

조인 외에도 “같은 곳으로 관련 데이터를 모으는” 일반적인 사용 유형은 SQL의 GROUP BY 절과 같이 특정 키로 레코드를 그룹화 하는 것이다.

맵리듀스로 그룹화 연산을 구현하는 가장 간단한 방법은 매퍼가 키-값 쌍을 생성할 때 그룹화할 대상을 키로 하는 것이다. 그러면 파티션 및 정렬 프로세스가 같은 키를 가진 모든 레코드를 같은 리듀서로 모은다.

특정 사용자가 취한 일련의 활동을 찾기 위해 사용자 세션별로 활동 이벤트를 수집 분석할 때도 일반적으로 그룹화를 사용한다. 이 과정을 세션화(sessionization)라고 한다.

사용자 요청을 받은 웬 서버가 여러 개라면 특정 사용자의 활동 이벤트는 여러 서버로 분산돼 각각 다른 로그 파일에 저장된다. 이때 적절한 식별자를 그룹화 키로 사용해 특정 사용자의 활동 이벤트를 한 군데로 모으면 세션화를 구현할 수 있다.

### 쏠림 다루기

키 하나가 너무 많은 데이터가 연관된다면 “같은 키를 가지는 모든 레코드를 같은 장소로 모으는” 패턴은 제대로 작동하지 않는다. 이렇게 불균형한 활성 데이터베이스 레코드를 **린치핀 객체(linchpin object)** 또는 **핫 키(hot key)**라 한다.

이러한 린치핀 객체의 데이터를 한 리듀서에 모은다면 쏠림 현상이 생기고 이를 **핫스팟**이라 한다. 모든 매퍼와 리듀서가 완전히 끝나야지 맵리듀스 작업이 끝나기 때문에 가장 느린 리듀서가 끝나길 기다려야 한다.

핫 키가 존재하는 경우 이를 완화할 몇가지 알고리즘이 있다. 한가지는 피그(pig)에 있는 쏠림 조인(skewed join) 메서드는 어떤 키가 핫 키인지 결정하기 위해 샘플링 작업을 수행한다. 실제 조인을 수행할 때는 매퍼는 핫키를 가진 레코드는 여러 리듀서 중 임의의로 선택한 하나로 보낸다. 핫키로 조인할 다른 입력은 핫 키가 전송된 모든 리듀서에 복제한다. 이 기법은 핫 키를 여러 리듀서에 퍼뜨려서 처리하게 하는 방법이다. 여러 리듀서로 복제하는 비용이 들지만 벙렬화 효과가 훨씬 크다.

크런치(Crunch)에서 제공하는 공유 조인(shared join) 메서드가 이 기법과 비슷하지만 샘플이 대신 핫키를 명시적으로 지정해야 한다.

하이브(Hive)는 쏠린 조인을 최적화할 때 핫 키는 테이블의 메타  데이터에 명시적으로 지정하고 핫 키와 관련된 레코드를 나머지 키와는 별도의 파일에 저장한다. 해당 테이블에서 조인할 때 핫 키를 가지는 레코드느느 맵 사이드 조인(map-side join)을 사용해 처리한다.

핫 키의 레코드를 그룹화하고 집계하는 과정은 2개의 단계로 구성된다. 첫번째는 임의의 리듀서로 보내고 일부를 그룹화하고 키별로 집계해 간소화한 값을 출력한다. 두번째는 첫번째 과정을 통해 나온 결과를 모두 결합해 하나의 값으로 만든다.

## 맵 사이드 조인

지난 절에서 설명한 여러 조인 알고리즘은 실제 조인 로직을 리듀서에서 수행하기 때문에 리듀스 사이드 조인(reduce-side join)이라 한다. 리듀스 사이드 조인의 장점은 입력 데이터에 대한 특정 가정이 필요 없다는 점이다. 그러나 정렬 후 리듀서로 복사한 뒤 리듀서 입력을 병합하는 모든 과정에서 드는 비용이 상당하다는 단점이 있다.

반면 입력 데이터에 대해 특정 가정이 가능하다면 맵사이드 조인(map-side join)으로 불리는 기법으로 더 빠르게 수행할 수 있다. 리듀서는 물론 정렬 작업도 없다, 대신 각 매퍼가 할 작업은 분산 파일 시스템에서 단순히 입력 파일 블럭 하나를 읽어 해당 분산 파일 시스템에 출력하는 것이 전부다.

### 브로드 캐스트 해시 조인

맵사이드 조인은 작은 데이터셋과 매우 큰 데이터셋을 조인하는 경우 간단히 적용해볼 수 있다. 이때 작은 데이터셋은 메모리에 모두 올릴 수 있을 정도로 충분히 작아야 한다. 우선 매퍼가 시작할 때 작은 데이터셋을 해시 테이블에 넣는다. 이후 상요자 활동 이벤트를 모두 스캔하고 각 이벤트의 사용자 프로필을 해시 테이블에서 간단히 조회한다.

이 간단하고 효율적인 알고리즘을 브로드캐스트 해시 조인(broadcast hash join)이라 한다. 브로드 캐스트라는 단어는 큰 입력의 파티션 하나를 담당하는 각 매퍼는 작은 입력 전체를 읽는다는 것을 의미한다.

작은 조인 입력을 인메모리 해시 테이블로 적재하는 대신 로컬 디스크에 읽기 전용 색인으로 작은 조인 입력을 저장하기도 한다. 이 중 자주 사용되는 부분은 운영체제의 페이지 캐시에 남아 전체 데이터를 메모리에 올리지 못해도 인메모리처럼 빠르게 임의 접근 조회가 가능하다.

### 파티션 해시 조인

같은 방식으로 해시할 데이터를 파티셔닝을 통해 처리할 수도 있다. 정상적으로 동작할 경우 조인할 레코드가 모두 같은 파티션에 위치한다. 그래서 각 매퍼는 각 입력 데이터셋 중 파티션 한개만 읽어도 충분하다. 이를 통해 해시 테이블에 적재해야 할 데이터양을 줄일 수 있다. 하이브(Hive)에서는 버킷 맵 조인(bucketed map join)이라 한다.

### 맵 사이드 병합 조인

파티셔닝 뿐만 아니라 정렬까지 되어 있다면 변형된 맵 사이드 조인을 적용할 수 있다. 이때는 입력 크기가 메모리에 적재 가능한지 고려할 필요가 없다. 매퍼는 리듀서에서 일반적으로 수행하는 것과 동일한 병합 연산을 수행할 수 있기 때문이다. 수행 과정을 보면 오름차순으로 양쪽 입력 모두 점진적으로 읽어 키가 동일한 레코드를 맞춘다.

맵 사이드 병합 조인(map-side merge join)이 가능하다는 건 선행 맵 리듀스 작업이 이미 입력 데이터셋을 파티셔닝하고 정렬해 놓았다는 뜻이다.

### 맵 사이드 조인을 사용하는 맵 리듀스 워크 플로

맵 사이드 조인과 리듀서 사이드 조인을 결정하는 것은 이전 리듀서의 출력에 의존한다. 리듀스 사이드 조인은 조인 키로 파티셔닝하고 정렬해서 출력한다. 맵 사이드 조인은 큰 입력과 동일한 방법으로 파티셔닝하고 정렬한다.

앞서 살펴본 것처럼 맵 사이드 조인을 하기 위해서는 제약조건이 따른다.  조인 전략을 최적화할 때는 분산 파일 시스템 내 저장된 데이터셋의 물리적 레이아웃 파악이 중요하다. 파티션이 몇개인지, 데이터가 어떤 키를 기준으로 파티셔닝되고 정렬됐는지도 꼭 알아야 한다.

## 일괄 처리 워크플로의 출력

이러한 일괄 처리는 어디에 적합할까? 트랜잭션 처리도, 분석도 아니다. 일괄 처리는 입력 데이터셋을 대부분 스캔하는 것이 일반적이라 분석에 더 가깝다. 그러나 맵 리듀스 작업의 워크플로는 분석 목적으로 사용하는 SQL 질의와는 다르다.

### 검색 색인 구축

맵리듀스는 구글에서 검색 엔진에 사용할 색인을 구축하기 위해 사용되었다. 정해진 문서 집합을 대상으로 전문 검색이 필요하다면 일괄 처리가 색인을 구축하는데 매우 효율적이다. 문서 기준으로 파티셔닝해 색인을 구축하는 과정은 병렬화가 매우 잘된다. 키워드로 색인에 질의하는 연산은 읽기 전용이라 한번 생성하면 불변이다.

색인된 문서 집합이 변한다면 다시 전체 색인 워크플로를 재수행하는 것도 하나의 방법이다. 문서 중 일부만 바뀐다면 연산량이 너무 비싸지만 색인 과정을 문서가 들어오면 색인이 나오는 식으로 쉽게 추론할 수 있다.다른 방법으로 증분 색인도 구축하는 것이 가능하다.

### 일괄 처리의 출력으로 키-값을 저장

배치 프로세스의 출력을 웹 애플리케이션의 질의하는 데이터베이스로 보내는 방법이 있을까? 가장 확실한 방법은 직접 매퍼와 리듀서 내에서 선호하는 데이터베이스 클라이언트 라이브러리를 사용해 일괄 처리 작업이 한번에 레코드 하나씩 데이터베이스 서버로 직접 요청을 보내는 것이다. 물론 좋은 아이디어는 아니다. 상당히 느리고 일괄 처리가 기대하는 속도로 데이터베이스에 쓰기 요청을 보내면 과부하에 걸릴 수 있다. 맵리듀스는 작업이 성공한 경우에만 출력을 생성하는 것을 보장한다. 어떤 태스크가 실패하더라도 재시도 한다. 그러나 작업 내부에서 외부 시스템에 기록한다면 부수효과가 만들어진다.

더 좋은 방법은 일괄 처리 작업 내부에 완전히 새로운데이터베이스를 구축해 분산 파일 시스템의 작업 출력 디렉터리에 저장하는 방법이다. 이 데이터 파일은 기록되고 나면 불변이고 bulk로 적재해 읽기 전용 질의를 처리할 수 있다.

### 일괄 처리 출력에 대한 철학

맵리듀스 작업도 유닉스 철학과 마찬가지로 출력을 취급한다. 입력을 불변으로 처리하고 부수효과를 피하기 때문에 좋은 성능을 내면서 유지보수가 간단하다.

버그가 있는 코드를 배포해 데이터베이스에 잘못된 데이터가 저장되면(부수효과) 버그 있는 코드를 수정한다고해서 데이터베이스의 데이터가 수정되진 않는다. 쉽게 되돌릴 수 있는 실수라면 되돌리기 어려운 환경보다 더 빠르게 기능개발을 할 수 있다.(비가역성 최소화, minimizing irreversibility)

유닉스와 하둡은 몇가지 차이도 있다. 유닉스는 텍스트 파일을 가정하고 처리하기 때문에 입력 파싱에 부담이 있지만 하둡은 특별한 스키마를 사용하기 때문에 저수준 구문 변환 작업 중 일부를 하지 않아도 된다.

## 하둡과 분산 데이터베이스의 비교

### 저장소의 다양성

데이터 베이스는 특정 모델을 따라 데이터를 구조화 해야 한다. 반면 분산 파일 시스템은 파일은 어떤 데이터 모델과 인코딩을 사용해도 기록할 수 있다. 커다란 조직의 다양한 데이터를 한 곳에 모으는 것만으로도 큰 가치가 있다. 원시 데이털르 수집하고 스키마 설계는 나중에 고민하면 데이터 수집 속도가 올라간다.

제약없는 데이터 덤핑(data dumping)은 데이터를 해석하는 부담을 소비자에게 이전시킨다. 데이터를 여러 목적에 적합한 다양한 관점으로 변환이 가능하다.

따라서 하둡은 ETL 프로세스 구현하는데 종종 사용되기도 한다.

### 처리모델의 다양성

HDFS와 맵리듀스가 있으면 그 위에 SQL 질의 실행엔진을 구축할 수 있는데 HIVE 프로젝트가 바로 그런 역할을 한다.

맵리듀스가 어떤 형태의 처리에서는 성능이 나쁘다는 점을 깨달았다. 그래서 하둡 위에서 다른 다양한 처리 모델이 개발됐다. 임의 접근이 가능한 Hbase나 MPP 스타일의 분석 데이터베이스를 포함한다.

### 빈번하게 발생하는 결함을 줄이는 설계

맵리듀스와 MPP 데이터베이스는 크게 결함을 다루는 방식과 디스크 사용 방식에서 큰 차이를 보인다. MPP 데이터베이스는 질의 실행 중에 한 장비만 죽어도 전체 질의를 중단한다. 그러면 사용자는 질의를 다시 제출하거나 자동으로 재실행해야 한다. 또한 디스크에서 데이터 읽는 비용을 피하기 위해 조인 같은 방식을 사용해 가능하면 메모리에 많은 데이터를 유지한다.

맵리듀스는 맵 또는 리듀스 태스크의 실패를 견딜 수 있다. 또한 데이터를 되도록 디스크에 기록하려 한다. 내결함성 확보 및 메모리에 올리기에 데이터가 너무 크기 때문이다.

맵 리듀스 방식은 대용량 처리에 적합하다. 많은 데이터를 처리하고 오랜 시간 수행하는 작업이라면 그 사이에 최소한 태스크 하나는 실패할 가능성이 높다. 이 경우 작업 하나의 실패로 전체 태스크를 다시 실행하는 것은 큰 손해다.

맵리듀스를 설계한 구글의 환경은 온라인 프러덕션 서비스와 오프라인 일괄 처리 작업이 같은 장비에서 실행된다. 각 태스크는 컨테이너를 사용하여 자원을 제어한다. 모든 태스크는 우선순위가 있어 저순위의 태스크 자원을 고순위 태스크가 선점할 수 있다.

이러한 환경에서 일괄 처리는 상대적으로 우선순위가 낮다. 실제로 구글에서 어떤 맵리듀스 태스크가 한시간 정도 수행된다면 태스크가 다른 우선순위 높은 태스크에 의해 종료될 확률이 5%이다. 10분간 수행되는 100개의 태스크를 가지고 있다면 작업 완료되기 전에 적어도 하나 이상의 태스크가 종료될 확률은 50%를 넘어간다.

이런 이유로 맵리듀스는 태스크 종료가 예상치 못하게 자주 발생하더라도 견딜 수 있도록 설계되어 있다. 만약 태스크가 자주 종료되지 않는 호나경이라면 맵리듀스를 이런식으로 설계한 점은 선뜻 이해하기 어렵다.