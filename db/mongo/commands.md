# 자주 사용하는 명령어

MongoDB를 사용할 때 가장 빈번하게 사용하는 핵심 명령어들을 CRUD(생성, 조회, 수정, 삭제)와 관리 카테고리로 나누어 정리해 드립니다.

사용법:

```js
// db.컬렉션명.명령어()
db.users.find() // users 컬렉션에서 조회
db.orders.insertOne({...}) // orders 컬렉션에 삽입
```

## 1. 조회 (Read) - 가장 많이 사용

- find(query, projection): 조건에 맞는 모든 문서를 조회합니다.
- findOne(query): 조건에 맞는 문서 중 가장 먼저 발견된 단 하나만 반환합니다.
- countDocuments(query): 조건에 맞는 문서의 개수를 셉니다.
- distinct(field, query): 특정 필드의 중복을 제거한 값의 목록을 가져옵니다.

## 2. 생성 (Create)

- insertOne(document): 문서 1개를 삽입합니다.
- insertMany([docs]): 문서 여러 개를 배열 형태로 한 번에 삽입합니다.

## 3. 수정 (Update)

- updateOne(query, update): 조건에 맞는 첫 번째 문서만 수정합니다.
- updateMany(query, update): 조건에 맞는 모든 문서를 수정합니다.
- replaceOne(query, replacement): 조건에 맞는 문서를 새로운 문서로 아예 교체합니다.
- findOneAndUpdate(query, update): 수정 후, 수정 전(또는 후)의 문서를 결과값으로 반환합니다.

## 4. 삭제 (Delete)

- deleteOne(query): 조건에 맞는 문서 1개를 삭제합니다.
- deleteMany(query): 조건에 맞는 모든 문서를 삭제합니다.

## 5. 쿼리 보조 도구 (Cursor Methods)

find() 뒤에 붙여서 결과를 가공할 때 사용합니다.

- sort(spec): 결과 정렬 (1: 오름차순, -1: 내림차순)
- limit(n): 결과 개수 제한
- skip(n): 앞부분의 n개 문서를 건너뜀 (페이징 처리에 사용)

## 6. 관리 및 성능 확인

- aggregate([pipeline]): 복잡한 통계나 데이터 가공(Aggregation Framework)을 수행합니다.
- createIndex(keys): 성능 향상을 위한 인덱스를 생성합니다.
- explain(): 쿼리가 인덱스를 타는지, 실행 계획이 어떤지 분석할 때 사용합니다.
- getIndexes(): 현재 컬렉션에 생성된 인덱스 목록을 확인합니다.