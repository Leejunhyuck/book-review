1. 스키마가 자주 바뀌는 환경
ex) 
1.애자일 개발 방법론이나 나선형 모델의 시제품을 만들 때와 같이 최종적인 개발 명세가 확정되지 않는 상황
2.로깅을 하기 위한 용도로 사용할 수도 있다. 다양한 형식의 로그를 기록할 때에도 매우 유용하다.
2. 분산 컴퓨팅 환경
샤딩과 복제를 DBMS 수준에서 지원해서 여러 관련 기능들을 보다 효과적으로 적용할 수 있다.
-복제는 '고가용성' 환경이 필요할때 고려해볼만하다. (항상 사용할 수 있는 상태)
-샤딩은 저장용량이 늘어나면서 읽기,쓰기 속도가 필요한 만큼 나오지 않을 때 적용할 수 있다.

* 리눅스에서 mongodb설치시 권한 설정 (sudo chmod -R go+w /data/db


도큐먼트는 BSON 구조는 값으로 매우 다양한 데이터 타입을 가진다.

db.dropDatabase() 
db.collection.drop()
db.collection.renameCollection(바꿀이름)

Capped 컬렉션
정해진 크기를 초과하게 되면, 자동으로 가장 오래된 데이터를 삭제하게된다
db.createCollection(<컬렉션이름>,{capped:true, size:<제한할 크기>)
capped 컬렉션은 로그 데이터나, 일정 시간 동안만 보관하는 통계 데이터를 보관하고 싶을 때 유용하게 사용 할 수 있다.

뷰
집계 파이프라인 문법 사용


WriteConcern
mongodb는 기본적으로 메모리메 미리 저장한 후, 천천히장기 저장장치로 데이터를 옮긴다. 따라서 데이터베이스에 갑저가 문제(정전 등)이 생기면 메모리에만 담겨 있어
데이터가 손실 된다.


db.user.insertMany({다큐먼트1},{다큐먼트2} ....)
여러 다큐먼트를 한번에 넣기

 

find 
    - 쿼리 
        db.containerBox.find({name:"가위"})
    - 점연산자 
        - {name:{firstName:"Karoid",lastName:"Jeong"}}
          db.A.find({"name.firstName"::Karoid"})
        - {numbers: [101,32,21,11]}
          db.A.find({"numbers.0":52)
    - 프로젝션
        - 어떤 도큐먼트를 불러올지를 결정하는 파라미터
        db.containerBox.find(null,{name:true})
        db.containerBox.find(null,{name:1})

        null => {name:'가위'}  조건 찾기
    - cursor 커서 
        모든 정보를 직접 반환하지 않고 커서를 반환한다.
        toArray()를 메소드를 사용하면 커서로부터 정보를 전부 가져올 수 있다.
            - 만약 find문의 모든 값이 다 필요하지 않다면 toArray 메소드는 비효율적이다.
            - 또, 필요한 도큐먼트의 총 크기가 매우 크다면 toArray 메소드를 사용 할 시 메모리 용량을 초과 할수도있다.

수정
    replaceOne (대체)
    기존의 값 유지 되지않고, 교체되어 버린다.
    - db.user.replaceOne({username:"karoid"},{
        username:"Karpoid",
        status:"Sleep",
        points:100,
        password:2222
    },(upsert:true))

        - upsert 
            참/거짓 값을 설정할 수 있다, 참값으로 설정된 경우 쿼리로 찾은 도큐먼트가 없다면, 찾은 내용으로 도큐먼트를 생성해서 수정하게 된다. 
    
    updateOne, updateMany (수정)
        -db.containerBox.updateMany({name:"bear"},{$set:{name:"teddy bear",category:"toy"}})
            ($unset 도큐먼트상의 field 필드를 제거한다.)

수정 배열 연산자
    db.character.insertMany([
        {name:'x',inventory:['pen','cloth','pen']},
        {name:'y',inventory:['book','cloth'],position:{x:1,y:5}},
        {name:'z',inventory:['wood','pen'],position:{x:0,y:9}}
    ])

    db.character.updateMany({},{
        $set:{"inventory.$[penElem]":"pencil"},
        {arrayFilters:[{penElem:'pen'}]}
    })

    $set:{"inventory.$[]":"pencil"} -> 배열에 pen 요소가 두개 있어도 모두 pencil로 바뀌게 된다.

삭제는 수정과 매우 유사하다

트랜잭션 명령어
    session = db.getMongo()startSesstion()
    session().startTransaction({readConcern :{level:"snapshot"},
    writeConcern:{w:"majority"}});
    
    <원하는 작업을 수행>
    session.commitTransaction();
    session.endSession();    
    <문제가 생길 경우 중간에 작업을 취소>
    session.abortTransaction();


$text 연산자
    MongoDB의 $text 연산자는 해당 컬렉션의 텍스트 인덱스 안에서만 작동하기 때문에, 문자열 인덱스를 생성해야 한다.
    
    db.stores.insertMany([
        {_id:1, name :"java Hut",description:"Coffee and cakes"},
        {_id:2, name :"asdf",description:"Coffee and cakes"},
        {_id:3, name :"xcxc",description:"Coffee and cakes"},
        {_id:4, name :"hhh",description:"Coffee and cakes"},
        {_id:5, name :"jjj",description:"Coffee and cakes"},
        {_id:6, name :"jkkt",description:"Coffee and cakes"},
        {_id:7, name :"ccc",description:"Coffee and cakes"},
    ])

    db.stores.createIndex({name:"text",description:"text"}) - 문자열 인덱스를 생성
    db.stores.find({$text:{$search:"bake coffee cake"}})

    - 대소문자를 구분하지 않고 검색을 한다.

    db.stores.find({$text:{$search:"\"coffee shop\""}})
    여러 단어를 띄어쓰기와 함께 사용하게 되면 해당 단어들 중 일부만 포함된 도큐먼트라도 불러오게 된다.

    db.stores.find({$text :{$search:"shopped"}})
    shop이라는 단어의 진행형인 shopping이 포함된 도큐먼트를 불러온다

배열 연산자
    db.inventory.find({tags:{$elemMatch:{$gt:10,%lt:5}}})
    하나라도 두 조건을 동시에 만족하는 값을 가져옴

    db.inventory.find({tags:{$all:["red","blank"]}})
    순서와 상관없이 모든 요소를 가지는 값을 가져옴

    db.inventory.find({"tags":{size:3},....})
    배열의 해당 크기와 같은 도큐먼트를 찾는다

프로젝션 연산자
    프로젝션에서는 점연산자가 첫번째 배열을 가르키지 않는다.
    
    $slice등을 사용
        db.inventory.find({},tags:{$slice:1}})
        tags 필드의 첫 번째 요소만 출력 된다.

        마지막 요소로부터 n개를 출력하고 싶다면 연산자의 값으로 -n을 설정하면 된다.
        {$slice:[1,2]}

    elemMatch
        배열 속에서 특정 조건을 만족하는 요소만 노출

    $연산자 
        tags 필드의 값 중 "red"를 갖는 도큐먼트를 찾는경우
        db.inventory.find({tags:"red"},{"tags.$":true})



집계 명령어
    집계 방법론과 효율성
         도큐먼트를 집계하는 방법은 크게 세 가지가 있다.
         1. 데이터베이스의 모든 정보를 불러와 애플리케이션 단게에서 집계하는 방법
         2. MongoDB의 맵-리듀스 기능을 이용하는 방법
         3. MongoDB의 집계 파이프라인 기능을 이용하는 방법 

         집계 파이프라인은 MongoDB 내부에서 작동, 다른 방법들은 정보교환을 위해 메모리가 필요하게 된다.
         집계 파이프라인 방식으로 원하는 결과를 얻지 못할 수도 있다.
         그럴 경우, 맵-리듀스 방식을 사용하면 된다.

        - 맵-리듀스의 작동 방식 1
        
        Rating
        {_id:1,rating:1,user_id:2}        -->       {1:2}           -->     {_id:1,value:2}
        {_id:1,rating:1,user_id:2}        map       {2:3}           -->     {_id:2,value:3}
                                                    {3:[4,1]}       -->     {_id:3,value:2}
                                                    {4:[5,8]}       -->     {_id:4,value:2}
        
        db.rating.mapReduce(mapper, reducer, {out:{inline:1}})

        - 맵-리듀스의 작동 방식 2
        
        Rating                                                     reduce           중간 결과물
        {_id:1,rating:1,user_id:2}        -->       {1:2}           -->       ----- {_id:1,value:2}
        {_id:1,rating:1,user_id:2}        map       {2:3}           -->       |     {_id:2,value:3}
                                                                              | 
                                          -->       {3:[4,1]}       -->       V     {_id:3,value:2}
                                                    {4:[5,8]}       -->     --->    {_id:4,value:2}  최종결과물
                                                                           reduce 

        - 파이프라인 방식
        $project (어떤 필드를 만들고 어떤 필드를 숨길지 설정)
            db.rating.aggregate([
                {
                    $project:{_id:0,rating:1,hello:"new field"}
                }
            ])
            {"rating":1,"hello":"new field"}    

            db.rating.aggregate([
                {
                    $project:{_id:0,multiply:{
                        $multiply:[$_id","user_id"]
                    }}
                }
            ])
            {"multiply":2} 

        $group(그룹화이다.)
            db.rating.aggregate([
                {
                    $group:{_id:"$rating",count:{$sum:1}}
                }
            ])

        $match(도큐먼트를 필터링해서 반환한다. find문과 비슷한 역할이다.)
            db.rating.aggregate([
                {
                    $match:{rating:{$gte:4}}
                },
                {
                    $group:{_id:"$rating",user_ids:{$push:"$user_id"}}
                }
            ])

            {"_id" :5 ,"user_ids:[11,12,10,9]}

            평점이 4 이상인 사용자의 id들을 배열의 형태로 정리

             db.rating.aggregate([
                {
                    $match:{rating:{$gte:4}}
                },
                {
                    $group:{_id:"$rating",user_ids:{$push:"$user_id"}}
                },
                {
                $unwind:"$user_ids"
                }
            ])

            {"_id" :5 ,"user_ids:9}
            {"_id" :5 ,"user_ids:10}
            {"_id" :5 ,"user_ids:11}

        $out(도큐먼트를 저장)
            
             db.rating.aggregate([
                {
                    $match:{rating:{$gte:4}}
                },
                {
                    $group:{_id:"$rating",user_ids:{$push:"$user_id"}}
                },
                {
                $unwind:"$user_ids"
                },
                {
                $out:"user_ids_by_rating"    
                }
            ])

        $limit(입력 도큐먼트 중 지정된 숫자만큼만 출력)
        $skip(입력 도큐먼트 중 지정된 숫자만큼 건너뛰고 출력)
        $sort(도큐먼스틑 정렬 1이면 오름차순,-1이면 내림차순)
        
뷰 생성하고 삭제하기
    db.createView(<뷰 이름>,<출처 컬렉션>,<파이프라인 스테이지 배열>,)
    프로젝트 연산자 전부 사용하지 못하고, 사용할 수있는 명령어가 제한적이다.

데이터 모델링과 인덱싱
    - 레퍼런스 방식
        정보의 양이 늘어날수록 크기가 작은 도큐먼트의 개수가 늘어나게 된다.
        도큐먼트의 크기는 작고, 정보를 추가하려면 도큐먼트를 추가한다.
        연결된 정보를 모두 불러오는데 더 오래 걸린다.
        연결된 정보를 일부만 불러올 때 더 빠르다.

    - 임베디드 방식
        도큐먼트의 크기가 무한히 커질 수도 있다.
        도큐먼트의 크기는 크고, 정보를 추가하려면 도큐먼트를 수정한다.
        임베디드 되어 있는 정보를 더 빨리 불러 올 수 있다.
        도큐먼트 전체를 읽어야 해서 느리다.
        장점 
            기본적인 도큐먼트 생성 및 삭제 작업은 하나의 도큐먼트에 대해 원자성이 지켜진다. (하지만 최근 mongodb가 트랜잭션을 지원하면서 크게 장점이 되지 않는다. 다수의 도큐먼트도 원자성을 보존할 수 있기 때문)
            거의 대부분의 글을 노출시켜야 한다면 레퍼런스 방식이 좋은 성능
            하나의 서버에서는 읽는 다면 속도차이가 별로 나지 않지만 여러대의 서버로 샤딩 되어 있다면 속도 차이가 날 수 밖에 없다.


    결론                주로 읽기 작업 위주                         주로 쓰기 작업 위주
    도큐먼트가 크다     임베디드 된 정보를 대부분 불러올때 유리         불리하다.
    도큐먼트가 작다     임베디드 된 정보를 자주 불러와야 하면 유리      조금 불리하다.


다중키 인덱스

    배열 값에 대한 검색 상황을 대비에 '배열에 대한 인덱스'를 지원
텍스트 인덱스

    $text 연산자를 사용하기 위해서 생성 되어야 하는 인덱스가 바로 텍스트 인덱스, 영어의 경우 단어의 원형으로 인덱스를 생성 한국어는 형태소별 지원이 되지 않음
해시 인덱스
    핵시인덱스를 포함한 복합인덱스 생성불가, 배열을 값으로 가지는 필드에 설정 불가
    주로 해시샤딩으로 사용하는 것이 일반적

인덱스 명령어
    단일키 인덱스
        db.movie.createIndex(
        {평점:1 },)
    복학키 인덱스
        db.movie.createIndex(
            {평점:1,점수:-1},
        )
    다중키 인덱스 (임베디드 도큐먼트)
        db.movie.createIndex(
            {"리뷰.제목":1},
        )
    텍스트 인덱스가
        db.movie.createIndex(
            {제목:"text"},
        )
    해시 인덱스
        db.movie.createIndex(
            {배급시 :"hashed"},
        )

    {name:"배급사 해시 인덱스"} -> 인덱스 이름 설정가능

    TTL 인덱스
        일정 시간이 지나면 자동으로 도큐먼트를 삭제하도록 만들어줌
    
    고유 인덱스 (같은 값이 저장되는 것을 방지할 수있게 도와준다.)
        {unique: true}

    인덱스 조회 및 삭제 명령
    db.collection.getIndexes() 컬렉션의 모든 인덱스를 보여준다
    db.collection.dropIndex(인덱스 설정 혹은 이름) 인덱스 삭제
    db.collection.dropIndexs() 모든 인덱스 삭제


    find.explain() 어떤 과정을 거쳐서 쿼리가 진행되었는지 확인 가능하다.
    만약 인덱스가 사용되었다면 IXSCAN 스테이지가 포홤된 쿼리 계획이 나타나야 한다.

    (*파이프라인에서는 db.rating.explain().aggregate)

    모든 쿼리를 explain으로 확인 할수 없으니 프로파일러를 통하여 확인가능
    db.setProfilingLevel(2)
    db.setProfilingLevel(1,50) 50ms 이상인 작업은 모두 기록하게 된다.

    현재 실행되고 있는 잠금필드
    db.currentOp()
    