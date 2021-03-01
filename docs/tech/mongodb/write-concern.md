# WriteConcern

[https://docs.mongodb.com/manual/reference/write-concern/](https://docs.mongodb.com/manual/reference/write-concern/)

write concern는 mongodb에 요청된 쓰기 동작의 ack의 레벨에 대해 알려준다. sharded cluster인 경우, mongos 인스턴스는 샤드에게 write concern을 넘긴다.
여러 도큐먼트에 대한 트랜젝션 시, 각 operation 레벨의 설정이 아닌 트랜잭션 단위에서 write concern이 설정되어야 한다. 트랙잭션에서 각 write operation마다 write concern을 명시적으로 설정하지 말 것!
mongodb 4.4 부터는 전역 디폴트 write concern을 설정할 수 있다. 명시적으로 설정하지 않은 작업은 전역 설정을 따라감 (setDefaultRWConcern)
```
{ w: <value>, j: <boolean>, wtimeout: <number> }
```

- standealone, replica set에서의 동작이 다르다. 당연히 자세한 건 문서를 참조하는 게 좋겠다.
    - standalone
      j를 false를 주면 in-memory인 경우, true를 주면 on-disk journal 이다.
    - replica set
      w가 majority인가, 특정 숫자인가에 따라 동작이 달라지게 된다. 마찬가지로 j가 true인 경우 on-disk journal, false면 in-memory 이다.

### 옵션 설명
- w
    - <number>
        - 1 (default) request ack from standalone or primary
        2인 경우 standalone or primary + one of the secondaries.
        3인 경우 standalone or primary + both secondaries.
        - 0: no ack 요청. socket exception, network error등을 확인할 수 있다.
            - j : true 와 함께 주면 j: true 가 우선순위를 가진다. request ack from standalone or primary
        - Hidden, delayed, priority 0 member는 이 옵션을 통해 찾을 수 있다.
        - Delayed secondaries는 설정된 slaveDelay보다 빠르게 ack를 반환 할 수 없다
    - "majority"
        - calculated majority의 갯수 만큼 ack 요청
        [https://docs.mongodb.com/manual/reference/write-concern/#calculating-majority-for-write-concern](https://docs.mongodb.com/manual/reference/write-concern/#calculating-majority-for-write-concern)
            - 모든 voting member (arbiter 포함)의 majority
            - 모든 data-bearing voting member의 majority
        - members[n].votes = 1 or 0. replica set election
            - arbitor (elections for primary)는 무조건 1
            [https://docs.mongodb.com/manual/core/replica-set-members/#arbiter](https://docs.mongodb.com/manual/core/replica-set-members/#arbiter)
    - <custom write concern name>
        - tagged member에 대한 write concern을 작성할 수 있다
        예를 들어, Custom Multi-Datacenter Write Concern
- j
    - on-disk journal에 써졌는지 ack를 확인
    - w: <value>에 써져있는 갯수 만큼 ack를 확인한다.
    - 이것 자체만으로 replica set primary failover로 인해 쓰기가 롤백되지 않는다는 보장을 하지 않음 ?
    - journaling이 켜져있는 경우 w: "majority" 는 j: true를 암시한다.
- wtimeout
    - millisecond 단위, w ≥ 1 에서만 적용됨
    - 0이면, without wtimeout과 같음
    - 정의하지 않으면, write concern이 unreachable일 경우 무한히 block 됨
    - 결국 성공하더라도 정해진 limit이 지나면 에러를 리턴함
    - wtimeout을 초과하기 전에 이미 성공한 데이터 변경에 대한 undo를 수행하지 않음