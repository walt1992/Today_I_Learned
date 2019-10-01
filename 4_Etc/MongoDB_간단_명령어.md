# MongoDB 간단 사용 명령어 정리 

> (window기준)으로 정리되어 있다.

먼저 MongoDB를 실행하기 위해서는 기본적으로 `C:\` 아래에 `data/db`의 디렉토리가 생성되어야 한다.

이제 몽고디비가 설치된 디렉토리로 이동해 다음 경로로 이동한다 .

    C:\'program files'\mongodb\server\bin

이곳에서 다음 명령어를 입력해 **몽고디비 서버 실행**

      mongod

**프롬프트 사용을 위해 새로 CMD를 열어 다음을 입력**

       mongo

* ADMIN 계정 생성하기

        use admin
        db.createUser( { user : 'username', pwd :'password', roles['root']})

* ADMIN 계정으로 로그인하기

        use admin -u 'username' -p 'password'

* 데이터 베이스 생성하기

        use 'DBname'

* DB목록 확인하기
  
        show dbs

* 현재 사용중인 DB 확인하기

        db

* 컬렉션 생성하기

        db.createCollection('collectionname')

* 컬렉션 목록 확인하기 

        show collections

* 다큐먼트 생성하기

        db.[collectionname].save( {..... key : value.....})


### 데이터 조회하기 


* 컬렉션 내 모든 다큐먼트 조회하기

        db.[collectionname].find({})

* 컬렉션 내 모든 다큐먼트 조회하기 (특정 필드값만)

        db.[collectionname].find({}, { [fieldName] : [1(true) | 0(false)], ...})

* 컬렉션 특정 조건부 다큐먼트 조회하기

        db.[collectionname].find({ [fieldName] : '[fieldValue]'}, )

    fieldValue앞에 다음 키워드를 넣을 수 있음

    *    `$gt` 초과
    *    `$gte` 이상
    *    `$lt` 미만
    *    `$lte` 이하
    *    `$ne` 같지않음
    *    `$lte` 이하
    *    `or` 또는
    *    `in` 배열 중 하나
    
* 검색결과 정렬하기

        db.[collectionname].find({ [fieldName] : '[fieldValue]'}, ).sort( [fieldname] : [ 0 | -1])
    

* 조회할 다큐먼트 개수


        db.[collectionname].find({ [fieldName] : '[fieldValue]'}, ).limit( num).skip(num2)

    num2개의 개수를 건너뛴 이후로 num개 만큼 조회하기

### 데이터 수정하기


    db.[collectionname].update({ [fieldName] : '[fieldValue]'}, { $set : { [changeFieldName] : '[chagneValue]'}})


`$set`키워드를 사용하면 일부 필드만 변경하는 업데이트를 할 수 있음. 만약 **$set을 생략할 경우 _id를 제외한 전체 다큐먼트가 수정됨.**

### 데이터 삭제하기

    db.[collectionname].remove({ [fieldname] : '[fieldValue]'})