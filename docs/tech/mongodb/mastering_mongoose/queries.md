# 3. 쿼리

query 혹은 aggregate를 이용헤서 Mongodb로부터 데이터를 가져올 수 있다. chainable API를 지원함

```
const query = Model.find({ age: { $lte: 30 } });
query instanceof mongoose.Query; // true
```

## 3.1 쿼리 실행하는 법

- query에 await 하기
- Query#then()
- Query#exec()

## 3.2 쿼리 operation
실제 Mongoose가 실행하는 operation은 Query#op 속성에 string으로 저장되어있다.

- Read
  - count(deprecated), countDocument, find, findOne, dictinct, estimatedDocumentCount(#doc in collection)
- Update
  - findOneAndReplace(=replaceOne() + 기존 doc 반환), findOneAndUpdate(=updateOne() + 기존 doc 반환), replaceOne, update, updateMany, updateOne
- Delete
  - deleteOne, deleteMany, findOneAndDelete(=deleteOne() + 결과 반환), findOneAndRemove(=remove() + deleted document반환, deprecated), remove (모든 doc 삭제, deprecated)

```
await Model.
find({ name: 'Will Riker' }). updateOne({}, { rank: 'Commander' });

// Equivalent, without using chaining syntax
await Model.updateOne({ name: 'Will Riker' }, { rank: 'Commander' });
```

- `.find().updateOne()` != `.findOneAndUpdate()`
- `.findOneAndUpdate()`는 mongoDB server에서 구분되는 atomic operation 임
- Chaining은 updateOne으로 끝남
- updateOne은 다른 필드는 그대로 둔 채로 변경하려는 필드만 변경하지만 replaceOne은 다른 필드들을 모두 삭제함

## 3.3 쿼리 opeator
- $eq, $gt, $gte, $in, $lt, $lte, $ne, $nin, $regex
- $exists, $type
  - $type: double, string, object, array, binData, objectId, bool, null, regex, int, timestamp, long, decimal, number(alias double, int, long and decimal)
- Geospatial: $geoIntersects, $geoWithin, $near, $nearSphere
- Array: $all, $size, $elemMatch

## 3.4 Updated operators
- $set, $unset, $setOnInsert $min, $max
- Numeric Update Operators: $inc, $mul
- Array Update Operators: $push, $addToSet, $pull, $pullAll, $pop

## 3.5 정렬
- string은 ascii 코드에 기반해 비교함. '' < 'A' < 'Z' < 'a' < 'z'
- 여러 타입이 있는 경우 순서. (낮은 것에서 높은 것)
  - Null
  - Numbers
  - Strings
  - Objects
  - Arrays
  - binData
  - objectId
  - Boolean
  - Date
  - Timestamp
  - Regular Expression
```
let doc = await Test.findOneAndUpdate({}, { $min: { value: 'a' } }); 
doc.value; // 42. 'a' > 42

doc = await Test.findOneAndUpdate({}, { $min: { value: null } }); 
doc.value; // null. null < 42
```

## 3.6 Limit, Skip, Project
```
SearchResult.find()
  .sort({ order: 1 })
  .skip(page * perPage)
  .limit(perPage)
  .select({ ... });
```

## 3.7 query validator
Mongoose는 기본적으로 query를 validate하지 않는다. 필요하다면 query에 runValidator 옵션을 enable 시켜주어야한다.

- custom validator
```
const schema = Schema({
  name: String,
  age: Number,
  rank: { type: String, validate: ageRankValidator }
});

function ageRankValidator(v) {
  const message = 'Captains must be at least 30';
  assert(v !== 'Captain' || this.age >= 30, message); 
}

const update = { age: 29, rank: 'Captain' };
const opts = { runValidators: true, context: 'query' };
await Character.findOneAndUpdate({}, update, opts);
// Throws 'Validation failed: rank: Captains must be at least 30'
```
