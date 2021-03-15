# MongoDB CRUD Concepts
https://docs.mongodb.com/manual/core/crud/

mongodb 문서에는 mongodb CRUD 컨셉이라는 페이지가 따로 존재한다. mongodb crud operations 의 각 기능들을 설명하고 mongodb에서 일어나는 CRUD 작업의 추가적인 컨셉에 대한 정보를 포함하고 있다.
일단 목차를 살펴보자.

## 원자성, 일관성과 분산 작업들
일반적으로 데이터베이스는 ACID (원자성, 일관성, 고립성, 지속성) 속성 중 최대 세가지를 갖는다고 한다. mongodb는 뭘 가지고 있을까?

-  원자성과 트랜잭선
   -  원자성
      -  쓰기 작업은 단일 도큐먼트 레벨의 원자성을 가짐
   -  다중 도큐먼트 트랜잭션
      -  단일 쓰기 작업이 여러 도큐먼트를 변경하는 경우(e.g. updateMany), 각 도큐먼트의 변경은 atomic하다. 그러나 전체 작업은 atomic하지 않음
      -  다중 도큐먼트 쓰기 작업을 수행할 때, 단일 쓰기 작업이나 다중 쓰기 작업 등 다른 작업이 interleave될 수 있음
      -  다중 도큐먼트에 대한 reads & writes 의 원자성이 필요한 경우, 다중 도큐먼트 트랜젝션을 쓸 수 있음
         -  4.0, replica set에서 다중 도큐먼트 트랜젝션을 사용 가능
         -  4.2 부터 샤드 클러스터에 다중 도큐먼트 트랜잭션을 지원하는 분산 트랜잭션을 도입함. 이는 기존 replica set에서의 다중 도큐먼트 트랜잭션을 포함함. 
     - 트랜잭션을 지원하긴 하지만, 단일 도큐먼트 쓰기 작업에 비해 매우 큰 성능이 필요하고, 다중 트랜잭션이 가능한 것이 효율적인 스키마 디자인의 대체제가 될 수 없음. 데이터와 use case를 위한 많은 시나리오에서, 비정규화한 데이터 모델은 계속 최적일 것이다. 다시 말해, 많은 시나리오에서 데이터 모델링을 적절히 하는 것이 다중 도큐먼트 트랜잭션에 대한 필요를 줄일 것. 
     - 그래도 꼭 트랜잭션을 쓰고 싶다면 [production considerations](https://docs.mongodb.com/manual/core/transactions-production-consideration/) 문서를 참조하자.
   - 동시성 제어
     - 동시성 제어를 통해 데이터 불일치 또는 충돌 없이 여러 앱을 동시에 실행 가능
     - 방법1) unique index를 만들면, 삽입, 업데이트로 인해 중복 데이터가 생성되지 않음. 
     - 방법2) 쓰기 작업에 대해 쿼리 예측에서 필드의 예측된 현재값을 명시해주는 것 <- 무슨 말인지 잘 모르겠다.
-  읽기 고립, 일관성, 최신성(recency)
   -  고립 보장 (Isolation guarantees)
   -  
      -  Read Uncommited
      -  Read Uncommitted and single document atomicity
      -  Read Uncommitted and multiple document writes
      -  cursor snapshot
   -  단일 쓰기 (Monotonic Write)
      -  Mongodb는 standalone과 replica set에 대하여 기본적으로 monotonic 쓰기 보장을 제공
      -  샤드 클러스터에서 monotonic 쓰기를 위해서는 casual consistency를 참조
   - 실시간 정렬
   - 인과적 일관성 (causal consistency)
-  분산 쿼리
-  findAndModify를 통한 선형 읽기(Linearizable Reads via findAndModify)
## 쿼리 플랜, 성능, 분석
- 쿼리 플랜
- 쿼리 최적화
- 쿼리 성능 분석
- 쓰기 작업 성능
## 그 위의 것들..
- tailable cursors
