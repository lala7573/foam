# 2. 핵심 컨셉

## 2.1 Models, Schemas, Connections, Documents, Queries

- Model는 MongodB에 저장되어있는 오브젝트를 표현하는 클래스.
- Schema 모델 설정을 표현하는 클래스
- Connection는 한 개 이상의 Mongodb 서버 프로세스와 연결되는 소켓의 집합을 표현하는 클래스
- Document 는 모델의 인스턴스. save() 쓸 수 있음
- Query 는 Mongoose가 MongoDB로 보내는 연산을 표현

## 2.2 Documents: Casting, Validation, and Change Tracking

- documents 는 casting, validating, 스키마에 정의되어있는 paths의 tracking changes을 책임 짐

- 캐스팅 Casting

  ```
  const schema = Schema({ name: String, age: Number });
  const Person = mongoose.model('Person', schema);
  const doc = new Person({});
  doc.age = 'not a number'; // CastError
  doc.age = undefined; // Succeed as null
  ```

  Mongodb는 undefined를 허용하지만 nodejs mongodb driver는 허용하지 않음

- 유효성 검사 Validation

  - required 옵션
  - Strings: enum, match, minlength, maxlength, lowercase
  - Numbers: min, max
  - Dates: min, max

  ```
  const schema = new mongoose.Schema({ status: {
    type: String,
    enum: ['Started', 'Running', 'Finished']
  } });

  const Status = mongoose.model('Status', schema);
  const doc = new Status({ status: 'not exists' });
  doc.save().catch(error => {console.log(error)}) // ValidationError
  ```

- 변경 탐지 Change Tracking
  ```
  const doc = await Person.findOne();
  doc.name; // "연주"
  doc.name = '뽀짝';
  doc.modifiedPaths(); // ['name']
  await doc.save();
  // save시에 변경된 path들만 mongodb로 보냄. age에 대한 것은 보내지 않음
  ```

## 2.3 스키마와 스키마 타입 Schemas and Schema Type

- schema 는 모델을 위한 설정 object 이다. 스키마의 가장 큰 책임은, 모델이 cast, validate, track changes 할 paths를 정의하는 것이다.
  많은 몽구스 개발자들이 schema와 model 용어를 교환가능한 것처럼 사용하지만 둘은 다르다. OOP 용어로 말하면, model은 schema를 갖고있고(has a), model은 document이다(is a).
- SchemaType 의 기능: casting, validation, getter and setters
- Mongodb 3.4에서 Decimal128 타입 (decimal floating points) 도입됨. mongodb decimal은 네이티브 자바스크립트 덧셈/뺄셈을 연산을 쓸 수 없으므로, getter/setter를 이용해야함

```
const accountSchema = Schema({
  balance: {
    type: mongoose.Decimal128,
    get: v => parseFloat(v.toString()),
    set: v => mongoose.Types.Decimal128.fromString(v.toFixed(2))
  }
});
const Account = mongoose.model('Account', accountSchema);

const account = new Account({ balance: 0.1 });
account.balance += 0.2;
account.balance; // 0.3
```

## 2.5 virtuals

getter/setter로도 computed property를 다룰 수 있지만, virtual를 사용하는 것이 더 나은 선택

- 아래와 같이 설정하면 json 으로 변환할 때에 virtual도 포함시킬 수 있다.

```
const opts = { toJSON: { virtuals: true } };
const userSchema = Schema({ email: String }, opts);
...
doc.toJSON() or JSON.stringify(doc)
```

    - JSON.stringify는 toJSON() 의 결과를 사용함
    ```
    const myObject = { toJSON: () => 42};
    JSON.stringify({ prop: myObject }); // {"prop": 42}
    ```

## 2.6 Queries

Mongoose는 명시적으로 쿼리를 실행시켜주어야 한다. `exec()`또는 `then()` 을 통해서 실행할 수 있다. 쿼리를 `await`하면 JavaScript runtime이 then을 내부적으로 호출한다.

## 2.7 Mongoose Global and Multiple Connection

대부분의 앱에서는 require('mongoose')로 Mongoose global 혹은 singleton으로 사용하게 될 것이다. 하지만 multiple connection으로 구성하고 싶다면 아래와 같이 하면 됨

```
const mongoose = require('mongoose');
const conn1 = mongoose.createConnection('mongodb://localhost:27017/db1' { useNewUrlParser: true });
const conn2 = mongoose.createConnection('mongodb://localhost:27017/db2' { useNewUrlParser: true });
const Model1 = conn1.model('Test', mongoose.Schema({ name: String }));
const Model2 = conn2.model('Test', mongoose.Schema({ name: String }));
```
