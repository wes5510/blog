# Altas cloud의 trigger 테스트

## 배경

웹 어플리케이션은 "DB 값 <-> 가공 <-> 유효성 검사 <-> 브라우저 출력" 흐름을 갖는다.  

백엔드에서 개발자가 구현하는 것은 "가공"과 "유효성 검사"이다. 이 두 단계에서 휴면 에러가 가장 많이 발생하기 때문에 두 단계를 제거하고 싶었다.   

"유효성 검사"는 DB 스키마와 에러 처리를 잘하면 대부분 해결할 수 있을 것이라고 생각했다.  
"가공"은 여전히 개발자 몫이였고 이걸 해결하기 위해서 생각한게 "가공을 DB에서 해준다면?" -> "DB에 트리거를 연결해서 해보자!"였다.  

마침 [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)가 free tire가 있었고 trigger라는 기능도 있어서 사용해봤다.  

## 본론
### 1. 테스트용 어플리케이션 구현
단지 comment를 cli로 입력받아서 mongodb의 collection에 값을 넣어준다.

```javascript
#!/usr/bin/env node

const { program } = require("commander");

program
  .option("-u, --db-url <url>", "mongodb url")
  .option("-d, --db-name <name>", "mongodb db name")
  .option("-c, --comment <comment>", "inserted comment in db")
  .parse(process.argv);

const URL = program.dbUrl;
const DB_NAME = program.dbName;
const COMMENT = program.comment;

const ConnectDB = require('./connectDb');
const service = require('./service');

const main = async () => {
  const db = await ConnectDB(URL, DB_NAME);

  const ret = await service.insertComment(db, COMMENT);
  console.log({ ret });
};

main()
  .then(() => {
    console.log("Complete");
    process.exit(0);
  })
  .catch((err) => {
    console.error(err);
    process.exit(1);
  });
```
모든 코드는 [wes5510/insert-comments](https://github.com/wes5510/insert-comments) 을 참고하길바란다.

### 2. MongoDB Atlas 설정
1. mongodb organization 생성
2. 프로젝트 생성
3. 클러스터와 DB, Collection 생성
4. Trigger 생성
    1. 프로젝트에 들어가서 좌측 사이드 메뉴의 [Triggers]를 클릭한 후 [Add Trigger]를 클릭한다.
    2. [Link Cluster]를 comment가 삽입되는 클러스터를 선택한다.
    3. [Cluster Name], [Database Name], [Collection Name]도 역시 comment가 삽입되는 곳으로 설정한다.
    4. [Operation Type]은 어플리케이션이 삽입뿐이 안하므로 [Insert]만 선택한다.
    5. Function 코드를 입력한다. 나는 아래와 같이 입력했다.
        ```javascript
        exports = function(changeEvent) {
          const docId = changeEvent.documentKey._id;
  
          const mongodb = context.services.get("Cluster0"); // commentsLog가 있는 클러스터 이름이다. 잘못 넣으면 오류가 남.로그를 봐도 알기 어려움.
          const logCollection = mongodb.db("test").collection("commentsLog");
          logCollection
            .insertOne({ commentId: docId, event: 'create' })
           .then(result => console.log(`Successfully inserted item with _id: ${result.insertedId}`))
            .catch(err => console.error(`Failed to insert item: ${err}`));
        }
       ```
     6. [Save]를 클릭하여 생성한다.

### 3. 실행

comments-insert 어플리케이션을 아래같이 실행하면
```sh
./index.js -u "<url>" -d "test" -c "hello"
```
commentsLog collection에도 `{ commentId: OOOOO, event: 'create' }`가 입력된 걸 확인할 수 있다.

## 결론
- 디버그 힘듬
- 목표했던 "가공" 단계의 제거가 아닌 "가공" 단계를 DB 트리거 기능으로 이동뿐이 안됨
