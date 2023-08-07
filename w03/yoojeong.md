## 데이터 모델과 질의 언어

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
