# InsertMany
mongodb 3.6 기준 

[https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/)

- ordered default = true
- ordered false 인 경우
    - document가 unordered로 insert 되고, 퍼포먼스를 올리기 위해 mongod에 의해 reorder된다
    - 최대 사이즈는 maxWriteBatchSize이며 default는 10만개이다. [ isMaster.maxWriteBatchSize ]
        - error message oversize를 막기 위함이다. limit을 넘으면 client driver가 10만개 chunk로 보냄
        - high-level API 사용시에만 나눠주고,  db.runCommand() 는 나눠서 보내지 않음
    - single batch의 error report가 너무 커지면 Mongodb는 모든 남은 error message들을 empty string으로 치환시킨다. 현재는 최소 2개의 메세지이고 총 메세지 사이즈가 1MB를 넘을 경우에 위 경우가 발생한다
        - size와 grouping 매커니즘은 내부 성능 상세이고, 이후 버전에서는 바뀔 수 있다
    - 샤드 collection 인 경우, ordered list를 실행하면 각 operation은 이전 operation이 완료될 때 까지 기다린 후에 실행이 되므로, unordered보다 일반적으로 느리다.
- ErrorHandling
    - 기본적으로 BulkWriteError를 던진다
    - WriteConcern Error를 제외하고, ordered operation은 에러가 발생하면 동작을 멈춘다. unordered는 큐에 있는 모든 wriet operation을 계속 한다
- multi document 트랜잭션에서 insertMany를 쓸 수 있다.
    - 트랜잭션 내부에서는 새 collection의 생성은 허용되지 않는다.
    - transaction 작성 시 write concern을 각 operation level로 명시적으로 설정하지 않는다. transaction level 에서 write concern을 준다.
    [https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)
