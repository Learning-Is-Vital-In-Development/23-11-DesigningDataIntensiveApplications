## 파티셔닝

- 거대한 데이터셋을 서브셋으로 분리하여 관리하는 방법(aka 샤딩)
- 데이터 파티셔닝의 주된 이유는 **확장성**
  - 대용량 데이터셋을 여러 디스크에 분산시켜 질의 부하를 여러 프로세스로 분산시킬 수 있다
  - 각 노드에서 쿼리를 처리할 수 있으므로 질의 처리량을 늘릴 수 있다

<details>
<summary>파티셔닝 & 샤딩</summary>

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/699bebd3-a6f5-4b99-8467-334fd1bbf6b6)

- Partitioning    
  - 테이블에 데이터가 많아지면 인덱스(B-Tree)또한 커지게 되고 테이블에 읽기/쓰기가 있을 때 마다 인덱스에서 처리되는 시간도 늘어난다.
  - 이를 방지하기 위해 테이블을 적당한 크기로 쪼개는 작업을 수행한다.

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/1363dd86-bbb4-4af7-9a4f-d484c0c330a7)

- Sharding
  - 네트워크 요청에 대한 부하를 각각의 서버로 분산시키기 위해 사용된다.

</details>


### 파티셔닝과 복제

![image](https://github.com/rachel5004/23-11-DesigningDataIntensiveApplications/assets/75432228/c2b68d30-ffcb-4c5d-9745-5b318ac5145f)

파티셔닝과 복제를 동시에 적용할 수 있다.

이 경우에는 복제의 모든 특성이 적용된다. 즉, 복제와 파티셔닝은 서로 연관이 없다.


### 키-값 데이터 파티셔닝

파티셔닝이 고르게 이뤄지지 않아 다른 파티션보다 데이터가 많거나 질의를 많이 받는 파티션이 있다면 쏠렸다(skewed)고 말하며 불균형하게 부하가 높은 파티션을 핫스팟(hot spot)이라고 한다.

#### 키 범위 기준 파티셔닝

백과사전처럼 각 파티션에 연속된 범위의 키를 할당하는 것


