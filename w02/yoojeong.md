## 데이터 모델과 질의 언어

### 학습할 내용

데이터 저장과 질의를 위한 다양한 범용 데이터 모델을 살펴본다. (관계형 모델, 문서 모델, 그리고 몇 가지 그래프 기반 데이터 모델)

다양한 질의 언어를 살펴보고 사용 사례를 비교한다.

 ### 관계형 모델과 문서 모델

- 오늘날 사용하는 대부분의 데이터 모델은 관계형 모델을 기반으로 하는 SQL이다.
- 데이터(SQL의 테이블)는 관계(relation)로 구성되고 각 관계는 순서 없는 tuple(SQL의 ROW)모음이다. 
- 사용 사례 : `트랜잭션 처리`(은행 거래, 항공 예약, 창고 재고 보관..), `일괄 처리`(고객 송장 작성, 급여 지불, 보고..)


#### NoSQL의 탄생

- Not Only SQL
- NoSQL 의 사용 이유
  - 대규모 데이터셋이나 매우 높은 쓰기 처리량 달성을 관계형 데이터베이스보다 쉽게 할 수 있는 확장성의 필요
  - 상용 데이터베이스 제품보다 무료 오픈소스 소프트웨어에 대한 선호도 확산
  - 관계형 모델에서 지원하지 않는 특수 질의 동작
  - 관계형 스키마의 제한에 대한 불만과 더욱 동적이고 표현력이 풍부한 데이터 모델에 대한 바람
- 애플리케이션의 다양한 요구사항 중에서는 RDB로 만족시킬 수 없는 요구사항들이 있고, 이러한 요구사항을 NoSQL이 충족시켜줄 수 있다.
- 현재는 NoSQL과 RDB가 같이 사용되고 있으며 이런 개념을 다중저장소 지속성(polyglot persistence)라 한다.

### 객체 관계형 불일치

- 데이터를 관계형 테이블에 저장하려면 어플리케이션 코드와 데이터베이스 모델 객체 사이에 거추장스러운 전환 계층이 필요하다
- 이런 모델 사이의 분리를 impedance mismatch 라고 부른다.
- 이러한 작업을 위해 객체 관계형 매핑(ORM) 프레임워크를 사용하기도 하지만 두 모델간의 차이를 완벽히 숨길 수 없다.

| 데이터 | RDB | JSON|
|:--:|:--:|:--:|
|![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/85367e16-39be-4faa-b1b7-58915458a91b)|![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/4b964e82-ad30-4b8e-b5fe-d95dc31095aa)|![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/50a92376-6155-4b5e-a0f5-99048b325808)|


- 프로필의 경우, 고유 식별자인 user_id로 식별하고, first_name, last_name 같은 경우는 사람마다 정확히 하나씩 있어 정형화가 가능하다.
- 하지만 사람마다 경력, 학력, 기간, 횟수 모두 다르기에 1:N 관계로 나타낼 수 있으며, 이 관계는 다양한 방법으로 나타낼 수 있다.
- 이력서 같은 데이터 구조는 모든 내용을 갖추고 있는 문서라서 JSON 표현에 매우 적합하다.
- JSON 표현은 다중 테이블 스키마보다 더 나은 지역성을 갖는다.
- 관계형 예제에서 프로필을 가져오려면 다중 질의 혹은 다중 조인을 수행해야 하지만 JSON 표현에서는 모든 관련 정보가 한 곳에 있어 질의 하나로 충분하다.

### 다대일과 다대다 관계

- id나 텍스트 문자열의 평문 저장 여부는 ‘중복’과 관련있다.
  -  id를 사용하는 경우 사람에게 의미있는 정보는 한 곳에만 저장하고 그것을 참조하는 모든 것은 id를 사용
  -  텍스트는 저장한다면 그것을 사용하는 모든 레코드에서 사람을 의미하는 정보를 중복해서 저장
- id 자체는 아무런 의미가 없기 떄문에 식별 정보를 변경해도 id는 변경할 필요가 없다
  - id 자체가 의미를 가진다면 변경 가능성이 있고, 만약 정보가 중복돼 있으면 모든 중복 항목을 변경해야 한다.
  - 쓰기 오버헤드와 불일치 위험이 있어 이런 중복을 제거하는 일이 데이터베이스의 정규화 이면에 놓인 핵심 개념이다.
- 중복된 데이터를 정규화하려면 다대일(many-to-one) 관계가 필요한데 문서 모델에는 다대일 관계가 적합하지 않다.
  - RDB 에서는 조인이 쉽기 때문에 id로 다른 테이블의 로우를 참조하는 방식
  - Document 에서는 1:N 트리 구조를 위해 조인이 필요하지 않지만 조인에 대한 지원이 약함
   


### 문서 데이터베이스는 역사를 반복하고 있나?

RDB 는 일상적으로 N:M 관계와 조인을 사용하지만
Document 는 데이터베이스에서 N:M 관계를 표현하는 데 어렵다.

이러한 논란은 가장 초기의 전산화 데이터베이스 시스템에도 있었다.


#### IBM의 계층 모델 (IMS)

- 계층모델은 문서 데이터베이스에서 사용하는 JSON 모델과 비슷
- JSON 구조와 마찬가지로 중첩된 레코드 트리로 표현
- Document 처럼 1:N 관계에서는 잘 동작하나 N:M 관계 표현은 어려웠고 조인은 지원하지 않았다

#### 네트워크 모델 (코다실 모델)

- 계층 모델의 트리구조에서 모든 레코드는 하나의 부모가 있지만 코다실 모델에서의 레코드는 다중 부모가 있을 수 있다. (다대일과 다대다 관계를 모델링 가능)
- 레코드에 접근하는 유일한 방법은 최상위 레코드에서부터 연속된 연결 경로(접근경로)를 따르는 방법
- 만약 레코드가 다중 부모를 가진다면 어플리케이션 코드는 모든 관계를 추적해야한다
- 원하는 데이터에 대한 경로가 없다면 접근 경로를 변경할 수 있지만 아주 많은 질의 코드를 살펴봐야 하고 새로운 접근경로를 다루기 위해 재작성해야 한다 (공수가 큼)

#### 관계형 모델

- 관계형 모델이 하는 일은 알려진 모든 데이터를 배치하는 것
- 관계는 튜플(로우)의 컬렉션이 전부로, 복잡한 중첩 구조나, 접근 경로도 없다.
- RDB 의 질의 최적화기(query optimizer)는 질의의 영역, 순서를 결정하고 사용할 색인을 자동으로 결정하며 새로운 색인을 선언하면 자동으로 적합한 색인을 사용
- 새로운 방식으로 데이터에 질의하고 싶은 경우 새로운 색인을 선언하기만 하면 질의는 자동으로 가장 적합한 색인을 사용
- 새로운 색인을 사용하기 위해 질의를 바꿀 필요가 없기 때문에 새로운 기능을 추가하는 작업이 훨씬 쉽다

#### 문서 데이터베이스와의 비교

- Document 는 별도의 테이블이 아닌 상위 레코드 내에 중첩 구조를 저장하는 측면에서 계층 모델과 유사
- 하지만 다대일과 다대다 관계를 표현할 때는 RDB 와 근본적으로 다르지 않게 고유한 식별자로 참조
  - RDB : **외래 키**
  - Document : **문서 참조**


 ### 관계형 데이터베이스와 오늘날의 문서 데이터베이스

#### 어떤 데이터 모델이 어플리케이션 코드를 더 간단하게 할까?

| 관계 | 추천 데이터모델 | 장점 |  단점 |
|:--:|:--:|:--|:--|
|1:N | Document | `스키마 유연성`, `지역성`에 기인한 `더 나은 성능`<br> 문서 형식의 데이터를 사용하는 어플리케이션의 경우 `데이터 구조의 유사성` | 중첩항목을 바로 참조하지못함<br> 조인 지원 미흡|
|N:M | RDB      |  데이터베이스 내 특화된 코드로 수행되는 조인 <br> `조인`, `다대일`, `다대다` 관계를 더 잘 지원 |스키마 경직성<br>상호 연결이 많은 데이터의 경우 그래프 모델이 최적 |

#### 문서 모델에서의 스키마 유연성

- 암묵적인 스키마가 있지만 데이터베이스는 이를 강요하지 않는다.
- 스키마가 없다는 뜻은 임의의 키와 값을 문서에 추가할 수 있고 읽을 때 클라이언트는 문서에 포함된 필드의 존재 여부를 보장하지 않는다는 의미이다.

| 접근방식 | 정의  | 예제 |
|:--:|:--|:--:|
| 쓰기 스키마 | 스키마는 명시적이고 데이터베이스는 쓰여진 모든 데이터가 스키마를 따르고 있음을 보장 |RDB|
| 읽기 스키마| 데이터 구조는 암묵적이고 데이터를 읽을 때만 해석됨| Document |


#### 질의를 위한 데이터 지역성(storage locality)

- 지역성의 이점은 한 번에 문서의 많은 부분을 필요로 하는 경우에만 해당
 - 대개 문서의 작은 부분에만 접근해도 전체 문서를 적재해야 하기에 큰 문서에서는 낭비일 수 있다.
 - 문서를 갱신할 때도 보통 전체 문서를 재작성함
 - 때문에 문서를 아주 작게 유지하면서 문서의 크기가 증가하는 쓰기를 피하라고 권장
- RDB 도 지역성 특성을 제공하기도 함
 - 구글의 스패너 DB:  부모 테이블 내에 테이블의 로우를 교차 배치되게끔 선언하는 스키마를 허용
 - 오라클 : 단위 클러스터에 두개 이상의 테이블을 함께 저장하는 다중 테이블 색인 클러스터 테이블 기능

#### 문서 데이터베이스와 관계형 데이터베이스의 통합


- RDB에 DocumentDB 통합
    - 대부분의 RDBMS (MySQL 빼고) : XML 지원
    - PostgresQL(>=9.3), MySQL(>=5.7), IBM DB2(>=10.5) : JSON 지원
- DocumentDB 에 RDB 통합
    - RethinkDB : 관계형 조인 지원
    - MongoDB driver : 자동으로 데이터베이스 참조 확인 (클라이언트 측 조인)
- 관계형 DB 와 문서 DB는 점점 상호보완 적으로 더 비슷해지는 중이다.


### 데이터를 위한 질의어

데이터를 질의 하는 방법에는 대표적으로 2가지가 있다.


[명령형 질의 ( IMS와 코다실 )]

```java
public List<Animal> getSharks() {
	animals.stream().filter(animal-> animal.equalFamily(SHARKS)).collect(toList());
}
```
- 순서에 따라 값이 달라지거나 문제가 생길수도 있기 때문에 개발자는 명령형 언어의 지시 순서에 신경을 써야 한다
- 병렬 처리도 까다로운 편

[선언형 질의 ( SQL )]

```SQL
SELECT * FROM animals WHERE family = 'Sharks';
```
- 방법이 아닌, 결과가 충족해야 하는 조건만 알면 됨
 - 데이터 변환(정렬(order by)
 - 그룹화(group by)
 - 집계(count, avg, sum, …)
- 어떤 색인이나 조인 함수를 지정할지나 순서에 대해서는 데이터베이스 시스템의 질의 최적화기가 한다
- 작업이 쉬우며 상세 구현은 숨겨져 있어서 질의를 변경하지 않고 DB 시스템 성능을 향상시킬수도 있다

### 맵리듀스 질의

- 맵리듀스는 많은 컴퓨터에서 대량의 데이터를 처리하기 위한 프로그래밍 모델
- 많은 문서를 대상으로 “Read-Only” 질의를 수행할 때 사용
- 맵 리듀스는 선언형 질의 언어도 왼전한 명령형 질의도 아닌 그 중간정도
- 몽고DB, 카우치DB 등 일부 NoSQL에서는 제한된 형태의 맵리듀스를 지원
- 여러 함수형 프로그래밍 언어에 있는 map과 reduce 함수를 기반


```SQL
# PostgreSQL
SELECT date_trunc('month', observation_timestamp) AS observation_month
			, sum(num_animals) AS total_animals
FROM observations
WHERE family = 'Shark'
GROUP BY observation_month;
```


```javascript
// mongoDB
db.collection.mapReduce({
    function map() {
        var year = this.timestamp.getFullYear();
        var month = this.timestamp.getMonth() + 1;
        emit(year + "-" + month, this.number);
    },
    function reduce(key, values) { reutrn Array.sum(values); },
    {
        query: { type: "A" },
        out: "monthlyCount"
    }
})
```

몽고DB의 map과 reduce 함수는 수행 할 떄 약간 제약사항이 있다.

- 두 함수는 순수 함수여야 한다.
	- 입력으로 전달된 데이터만 사용하고 추가적인 DB 질의를 수행 할 수 없게 하여 사이드이펙트가 없어야 한다.


- 맵리듀스 질의는 자바스크립트코드를 이용한 고급 질의가 가능하기에 훌륭하다
- 하지만 연계된 함수 두개를 작성하는게 때로 질의하나를 작성하는 것보다 어렵다는 문제가 있다
- 선언형 질의 언어는 질의 최적화기가 질의 성능을 높일 수 있다는 점도 고려되어, 몽고DB 2.2 에서는 집계 파이프라인(aggregation pipeline)이라 부르는 선언형 질의 언어 지원이 추가됨
	- 언어 표현 측면에서는 SQL의 부분집합과 유사하지만, JSON기반 구문을 사용

  
```SQL
db.observations.aggregate([
	{ $match: {family: "Sharks"}},
	{ $group: {
		_id: {
			year: { $year: "$observationTimestamp" },
			month: { $month: "$observationTimestamp" }
		},
		totalAnimals: { $sum: "$numAnimals" }
	}}
]);
```


### 그래프형 데이터 모델

| 관계 | 모델|
|:--|:--:|
|레코드 간 관계가 없거나 일대다(1:N) 관계 | 문서 모델|
|단순한 다대다 관계 | 관계형 모델|
|다대다(N:M)관계가 복잡해지는 경우| 그래프형 데이터 모델|

그래프는 두 유형의 객체로 이뤄진다. 

- 정점(vertex): 노드나 엔티티라고도 불린다. 
- 간선(edge): 관계나 호(arc)라고도 한다.

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/22428b13-1333-410d-aa5c-72a9ec795d16)


그래프는 다음과 같이 동종 데이터(ex: 사람, 웹 페이지, 통신서버등) 가 아닌 다른 유형의 정점과 간선을 연결함으로써 완전히 다른 유형의 객체를 일관성있게 저장할 수 있도록 해준다. 

이런 그래프에서 데이터를 구조화하고 질의하는 몇 가지 다른 방법이 있다. 

- 데이터 구조화 모델
	- 속성 그래프 모델
		- 네오포제이(Neo4j), 타이탄(Titan), 인피니티그래프(InfiniteGraph)
	- 트리플 저장소 모델
		- 데이토믹(Datomic), 알레그로그래프(Allegrograph)
- 그래프용 선언형 질의 언어
	- 사이퍼(Cyper)
	- 스파클(SPARQL)
	- 데이터로그(Datalog)
- 명령형 그래프 질의 언어
	- 그렘린(Gremlin)
- 그래프 처리 프레임워크
	- 프리글(Pregel)

   
#### 속성 그래프 모델
속성 그래프 모델의 정점과 간선은 다음과 같은 속성을 가진다. 

- Vertex
    - 고유한 식별자
    - 유출 간선 집합
    - 유입 간선 집합
    - 속성 컬랙션(Key-Value)
- Edge
    - 고유 식별자
    - 간선이 시작하는 정점(꼬리 정점)
    - 간선이 끝나는 정점(머리 정점)
    - 두 정점 간 관계 유형을 설명하는 레이블
    - 속성 컬랙션(Key-Value)
 
  
이러한 속성을 다음과 같이 두 개의 관계형 테이블로 구성해볼 수 있다. 
```SQL
CREATE TABLE vertices (
	vertex_id integer PRIMARY KEY,
	properties json
);

CREATE TABLE edges (
	edge_id integer PRIMARY KEY,
	tail_vertex integer REFERENCES verticees (vertex_id),
	head_vertex integer REFERENCES verticees (vertex_id),
	label text,
	properties json
)

CREATE INDEX edges_tails ON edges (tail_vertex);
CREATE INDEX edges_heads ON edges (head_vertex);
```

- 정점은 다른 정점과 간선으로 연결된다. 특정 유형과 관련 여부를 제한하는 스키마는 없다.
- 정점이 주어지면 정점의 유입과 유출 간선을 효율적으로 찾을 수 있고 그래프를 순회 할 수 있다.
즉 일련의 정점을 따라 앞뒤 방향으로 순회한다.
- 다른 유형의 관계에 서로 다른 레이블을 사용하면 단일 그래프에 다른 유형의 정보를 저장하면서도 데이터 모델을 깔끔하게 유지할 수 있다.


#### 사이퍼 질의 언어

속성 그래프를 위한 선언형 질의 언어인 사이퍼(Cyper)는 Neo4j 그래프 데이터베이스용으로 만들어졌다.  

- 데이터 모델 생성
  
```SQL
CREATE
	(NAmerica:Location {name: 'North America', type:'continent'}),
	(USA:Location {name:'United States', type:'country'}),
	(Idaho:Location {name:'Idaho', type:'state'}),
	(Lucy:Person {name: 'Lucy'}),
	(Idaho) -[:WITHIN]->(USA) -[:WITHIN]-> (NAmerica),
	(Lucy)  -[:BORN_IN]-> (IDaho)	
```

- 미국에서 유럽으로 이민 온 모든 사람들을 찾는 질의
```SQL
MATCH
	(person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (us:Location {name: 'United States'}),
	(person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (us:Location {name: 'Europe'})
RETURN person.name
```

- 질의 실행 방법
    1. 모든 사람을 조회로 시작 -> 사람들의 출생지와 거주지를 확인 -> 맞는 사람들만 반환
    2. 2개의 Location으로 시작 -> 미국과 유럽의 모든 위치 찾기를 진행 -> leaf 에 해당하는 정점 중 하나에 BORN_IN, LIVES_IN 유입 간선을 통해 발견된 사람들을 반환
 
#### SQL의 그래프 질의

반대로 그래프 데이터를 관계형 구조로 넣어서 SQL로 질의하는 것도 약간 어렵지만 가능하다. 

이는 미리 조인 수를 고정할 수 없기 때문인데, 미리 질의에 필요한 조인을 알 고 있는 관계형 데이터베이스와 달리 그래프 질의에서는 찾고자하는 정점을 찾기 전 가변적인 여러 간선을 순회 해야 하기 때문이다. 

사이퍼 질의에서 간선 순회는 `() -[:WITHIN*0..]→()` 문에서 발생하는데 이 가변 순회 경로에 대한 질의 개념을 SQL로 수행하기 위해서는 재귀 공통 테이블 식(recursive common table expression)(with recursive 문)을 사용해 표현할 수 있다. 

```SQL
WITH RECURSIVE

	in_usa(vertex_id) AS (
			SELECT vertex_id FROM vertices WHERE properties ->>'name' = 'United States'
		UNION
			SELECT edges.tail_vertex FROM edges
				JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
				WHERE edges.label = 'within'
	),

	in_europe(vertex_id) AS (
			SELECT vertex_id FROM vertices WHERE properties ->>'name' = 'Europe'
		UNION
			SELECT edges.tail_vertex FROM edges
				JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
				WHERE edges.label = 'within'
	),

	born_in_usa(vertex_id) AS (
			SELECT edges.tail_vertex FROM edges
				JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
				WHERE edges.label = 'born_in'
	),

	lives_in_europe(vertex_id) AS (
			SELECT edges.tail_vertex FROM edges
				JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
				WHERE edges.label = 'lives_in'
	)

SELECT vertices.properties->>'name'
FROM vertices
JOIN born_in_usa ON vertices.vertex_id = born_in_usa.vertex_id
JOIN lives_in_europe ON vertices.vertex_id = lives_in_europe.vertex_id;
```

 
#### 트리플 저장소와 스파클

- 속성 그래프 모델과 거의 동등
- 트리플 저장소에서는 정보를 주어(subject), 서술어(predicate), 목적어(object) 간단한 세 부분 구문 형식으로 저장
	- 주어 = 그래프의 정점, 목적어 = 문자열이나 숫자 같은 원시 데이터타입의 값 혹은 그래프의 다른 정점


#### 시맨틱 웹
