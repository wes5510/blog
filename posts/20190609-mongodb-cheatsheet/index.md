# Mongodb Cheatsheet

### 소개

- 문서 지향 데이터베이스
- 유연한 모델인 '문서'를 사용
- 고정된 스키마가 없고 필요할 때 필드를 추가, 제거하여 개발 생산성을 향상 시킴
- 분산 확장을 염두에 두고 설계됨
    - 복제 셋, 샤딩 등을 이용하여 쉽게 분산처리할 수 있음.

### 문서

- 키는 문자열(UTF-8), 중복될 수 없음, 대소문자 구별
- 키/값 쌍으로 정렬되어 있음({ "a": 1, "b": 1 }, { "b": 1, "a": 1 }은 다름)

### 콜렉션

- 문서의 모음
- 동적 스키마를 가짐(콜렉션 내 문서들은 모두 다른 구조를 가질 수 있음)
- 인덱스를 사용하기 위해 특정 스키마를 갖는 걸 권장함

### 데이터베이스

- 콜렉션 모음
- 파일로 디스크에 저장됨

### 데이터형

- null, boolean, 숫자, 문자열, 날짜, 정규표현식, 배열, 내장문서, ObjectId, 이진 데이터, 코드가 있다.
- ObjectId
총 12바이트이다. 아래와 같이 구성되어 있음

    ```
    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
    |   타임스탬프   |   장비    |  PID  |    증가랑    |
    ```

### 삽입

- 삽입: `db.collection.insert({ 'a': 1 })`
- 요소 추가
    - `db.collection.update({ 'a': 1 }, { '$push': { 'posts': 'abc' } })`
    - 지정된 키가 존재하면 추가, 아니면 새로운 배열 생성 후 추가
    - `$each` 사용하면 여러개 값 한 번에 추가 가능. `db.collection.update({ 'a': 1 }, { '$push': { 'posts': { '$each': ['a', 'b', 'c'] } } })`
    - `$slice`를 결합하여 요소 개수 제한. `db.collection.update({ 'a': 1 }, { '$push': { 'top5': { '$each': [ 'a', 'b' ], '$slice': -5 } } })`
- 배열에 중복값 없이 요소 추가
    - `$ne` 사용: `db.collection.update({ 'posts': { '$ne': '1' } }, { '$push': { 'posts': '1' }' })`. `$addToSet` 사용. `db.collection.update({}, { '$addToSet': { 'posts': 1 } })`
    - `$addToSet`사용: `db.collection.update({}, { '$addToSet': { 'posts': { '$each': ['a', 'b', 'c'] } } })`

### 삭제

- 삭제: `db.collection.remove({ 'a': 1 })`
- 콜렉션 전체 삭제: `db.collection.drop()`
- 요소 제거
    - 배열의 처음 제거: `db.collection.update({ 'a': 1 }, { '$pop': { 'posts': -1 } })` .
    - 배열의 끝 제거: `db.collection.update({ 'a': 1 }, { '$pop': { 'posts': 1 } })`.
    - 조건에 일치하는 모든 요소 제거: `db.collection.update({ 'a': 1 }, { '$pull': { 'posts': 'a' } })`.

### 갱신

- 문서 갱신은 원자적
- 필드 숫자 증가, 감소: `db.collection.update({ 'a': 1 }, { '$inc': { 'size': 1 } })`
- 배열 인덱스를 이용한 수정: `db.collection.update({ 'auth': 'lee' }, { '$inc': { 'comments.1.votes': 1 } })`
- 첫 번째로 일치하는 요소 수정: `db.collection.update({ 'auth': 'lee' }, { '$set': { 'comments.$.votes': 1 } })`
- 문서의 크기가 변경되면 수정이 느리다. 문서가 커지면 공간이 맞지않아 콜렉션의 다른 공간으로 이동하기 때문
- 문서의 크기를 변경시키지 않는 수정은 효율적이다. 예를 들어 `$inc`.
- 이동이 잦아 큰 빈 공간이 생길 때, 로그 `info DFM::findAll(): extent a:23C13 was empty, skipping ahead`
- 잦은 이동, 삽입, 삭제를 하는 콜렉션 설정: `db.runCommand({ 'collMod': collection, 'usePowerOf2Sizes': ture })`. 콜렉션의 모든 후속 할당을 2^n크기의 블록에 있게 설정
- 원자적 갱신 입력
    - 해당 조건에 만족하는 문서가 없을 경우 입력 후 갱신한 문서를 만든다: `db.collection.update({ 'auth': 'lee' }, { '$inc': { 'viewCount': 1 } }, true)`. `{ 'auth': 'lee', 'viewCount': 1 }` 반환
    - 해당 조건에 만족하는 문서가 없을 경우 입력, 만족하는 문서가 있을 경우 입력 안함: `db.collection.update({ 'auth': 'lee' }, { '$setOnInsert': { 'createAt': new Date() } })`
- 다중 문서 갱신: 4번째 인자를 true로 설정. `db.collection.update({ 'auth': 'lee' }, { '$inc': { 'viewCount': 1 } }, false, true)`
- 갱신 전 문서 반환 후 갱신: `db.collection.findAndModify({ 'auth': 'lee' }, { '$inc': { 'viewCount': 1 } })`. 이전 `viewCount : 24`면 24를 반환하고 실제로 25로 갱신.
- `findAndModify`인자: `query` 문서를 찾는 조건, `sort` 결과 정렬, `update` 찾은 문서 갱신, `remove` 찾은 문서 삭제 여부 boolean값, `new` 반환 문서가 갱신하기 전 문서인지 갱신 후 문서인지 설정하는 boolean값, `fields` 반환 문서 필드 설정. `upsert` 갱신 입력 여부 boolean값.

### 조회

- or 쿼리: 하나 키에 대한 or 쿼리는 `$in` 사용. 여러 키에 대한 or 쿼리는 `$or`사용. `$nin`은 `$in`과 반대 배열 내 조건과 일치하지 않는 문서 반환.
- `$or`보다는 `$in`를 사용하라. `$or`은 설정된 값의 결과를 병합하는 작업(중복 제거도 포함)이 있기 때문에 `$in`를 사용하는 게 효율적이다.
- `$ne`는 비효율적이다. `$ne`로 설정된 것 이외의 모든 인덱스를 살펴봐야하기 때문이다.
- `$not`은 대부분 테이블 스캔을 수행한다.
- `$nin`은 항상 테이블 스캔을 수행한다.
- `null`은 자신과 일치하는 것뿐만 아니라 존재하지 않는것도 반환한다.
- 여러 개 요소와 일치하는 문서: `db.collection.find({ 'tags': { $all: [ 'devops', 'network' ] } })` 배열에 순서에 상관없이. 'devops', 'network'가 포함된 문서를 반환한다.
- 배열을 포함하는 문서에 범위 쿼리를 수행할 때 `min()`와 `max()`함수를 사용하는 것이 좋음: `db.collection.find({ 'a': { '$gt': 1, '$lt': 10 } }).min({ 'a': 1 }).max({ 'a': 10 })`
- 내장 문서를 찾을 때는 점 표기법이 좋다. `db.collection.find({ 'name.first': 'kh', 'name.last': 'lee' })`. 내장 문서의 키 순서가 바뀌면 `db.collection.find({ 'name': { 'first': 'kh', 'last': 'lee' } })` 명령어는 아무것도 반환하지 않는다.
- `$where` 쿼리는 보안상, 인덱스 사용 불가하기 때문에 사용하지 말아야한다.
- `aggregate` 사용
    - `$match`를 가장 앞쪽에 배치하면 불필요한 문서를 먼저 걸러낼 수 있으며, `$project`이나 `$group`를 사용하기 전에 쿼리가 인덱스를 사용할 수 있다.
    - 필드명을 수정하면 인덱스를 사용할 수 없다.
- 하나의 `aggregate`가 메모리 20% 이상을 사용하면 오류가 발생한다.

### 인덱스

- 인덱스 생성: `db.collection.ensureIndex({ 'id': 1 })`
- 인덱스를 생성하면 모든 읽기, 갱신, 삭제가 더 오래 걸린다. 문서 자체뿐만 아니라 몽고디비가 모든 인덱스를 갱신해야 되기 때문이다. 콜렉션에 두세 개 이상의 인덱스를 갖지 않는 게 좋다.
- 인덱스는 정렬 앞부분에 놓일 경우에만 정렬에 도움이 된다.
- `db.collection.find().sort({ 'id': 1, 'auth': 1 })`는 `db.collection.ensureIndex({ 'id': 1, 'auth': 1 })`로 하면 `find()`만으로 해결 가능하다.
- 인덱스가 되지 않은 `sort()`를 사용 시, 몽고디비는 결과를 반환하기 전 메모리에서 정렬하므로 인덱스를 최대한 활용하는 게 좋다. `explain`에서 `scanAndOrder: true`가 된다.
- `find()`를 사용할 때, `limit()`를 사용하는 것은 효율적이다.
- `db.collection.find({ 'indexing key': 1, query... })` 은 처음 index된 키로 범위를 좁히므로 효율적이다.
- 만약 index가 `{ 'id': 1 }`로 설정되었을 경우, `{ 'id': -1 }`로 정렬한다면 몽고 디비가 최적화를 해주기 때문에 `{ 'id': -1 }`를 index로 설정할 필요가 없다.
- 인덱스가 상용자에 의해 요구되는 `select`를 모두 포함하고 있다면 퀴리가 커버링된다고 한다. 커버드 쿼리는 `explain`에서 `indexOnly: true`이다.
- 내장 문서에 대한 인덱스 생성: `db.collection.ensureIndex({ 'user.name': 1 })`
- 배열을 인덱싱할 수 있지만 단일 값 인덱스 보다 휠씬 비싸고 비효율적이기 때문에 권장하지 않는다.
- 인덱스 키가 유니크할 수록 인덱스에 도움이 된다.
- 큰 콜렉션, 큰 문서, 선택적 쿼리에서 인덱스는 적합하다.
- 작은 콜렌션, 작은 문서, 비선택적 쿼리에서 테이블 스캔이 적합하다.
- 제한 콜렉션; 생성될 때, 크기와 갯수가 고정되어 생성된다. 빈 공간이 없다면 가장 오래된 문서가 지워지고 새로운 문서를 넣는다.
- TTL 인덱스는 각 문서에 대해 유효 시간을 설정할 수 있다. 유효시간이 지나면 그 문서는 삭제된다. `db.collection.ensureIndex({ 'expireTime': 1 }, { 'expreAfterSeconds': 60*60*24 })`는 24시간 후에 삭제된다.
- 전문 인덱스
    - 문장을 빠르게 검색하고 다국어 형태소 분석, 정지단어를 위한 내장된 기능을 제공한다.
    - 항상 오프라인으로 생성하거나 성능이 문제되지 않을 때 생성해야한다.
    - 전문 인덱스가 생성된 콜렉션은 쓰기 성능이 떨어질 수 도 있다.
    - 전문 인덱스 생성: `db.collection.ensureIndex({ 'desc': 'text'})`
    - 전문 인덱스는 오직 하나만 생성할 수 있다.
- 서비스 중 인덱스를 구축할 때는 `background` 옵션을 사용해라. 기본적으로 인덱스를 구축할 때 몽고디비는 데이터베이스의 모든 읽기와 쓰기를 중단한다.

### 스키마

- 내장 방식(중복 데이터 전체를 가지고 있는) 스키마는 작은 하위문서, 정기적으로 변하지 않는 데이터, 결과적인 일관성이 허용될 때, 증가량이 적은문서, 조회하려는 두 번째 쿼리를 수행하기 위해 자주 필요한 데이터, 빠른 읽기 특징을 가지고 있을 때 좋다.
- 참조 방식(유니크한 키 데이터만 가지고 있는) 스키마는 큰 하위문서, 자주 변하는 데이터, 즉각적인 일관성이 필요할 때, 증가량이 많은 문서, 결과에서 자주 제외되는 데이터, 빠른 쓰기 특징을 가지고 있을 때 좋다.

### [쿼리 분석](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)
