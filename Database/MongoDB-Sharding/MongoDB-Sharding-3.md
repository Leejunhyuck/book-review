샤딩
    각각의 데이터가 어떤 서버에 들어있는지를 저장하고 알려주는 역할이 필요하다.
    이처럼 샤딩을 하고 난 이후의 역할이 분배된 구성 요소들을 하나로 묶어 '샤드 클러스트'라고 한다.

    샤드 
        애플리케이션이 사용할 데이터를 나눠서 저장하고 있다.

    설정서버
        애플리케이션 데이터를 제외하고 샤드 클러스터가 작동하기 위해 필요한 모든 정보를 저장한다. 이런 정보를 메타데이터라고한다.

    라우터
        샤드와 설정 서버가 특정 정보를 저장하는 역할을 했다면 라우터는 직접적으로 드라이버의 명령을 받아서 어떤 작업을 수행할지 판단하고 결과물을 가져다 드라이버에 넘겨주는 역할을 한다.
    
    mongod 프로세스와 mongos 프로세스가 서로 통신 그 후 -> 샤드에 데이터 저장

샤드키
    샤드 클러스터에서 정보를 어떻게 나누어 저장하느지에 따라서 성능상의 문제가 발생할 수 있다. 예를들어 자주 사용하는 정보가 한쪽 샤드에 몰리는 상황을 생각해보자
    이런 경우, 대부분의 샤드는 쉬고 있는데도 서비스 속도가 저하되는 상황이 발생한다. 
    따라서 정보를 나누는 기준을 잘 설정해야한다. 기준이 되는 값이 샤드키이다.

프라이머리 샤드는   
    각각의 데이터베이스에 샤딩을 허용할수있다. 샤딩을 적용받은 데이터베이스는 프라이머리 샤드를 필수적으로 사져야 한다,
    데이터베이스 내에서 컬렉션을 저장하는 기본샤드다.


수직적 샤드
    1번샤드                                             2번샤드
    {A데이터베이스 컬렉션} {A데이터베이스 컬렉션}       {B데이터베이스 컬렉션} {B데이터베이스 컬렉션}

수평적 샤드
    1번 샤드                        2번 샤드            3번 샤드
    {가 컬렉션} {나 컬렉션의 1/3}   {나 컬렉션의 1/3}   {다 컬렉션의 1/3}
    나 컬렉션이 크기가 클 경우

청크
    샤드 사이에 분할된 정보를 효과적으로 관리하기 위해 '청크'를 사용한다. 샤드키값을 기준으로 한 도큐먼트의 묶음 이다 (각각의 청크는 샤드키를 기준으로 상한값과 하한값을 갖는다.
밸런서
    어떤 청크가 64mb를 넘어버리면, 둘로 쪼개지면서 크기가 반으로 줄어든다. 또한, 한쪽 샤드에 청크가 몰릴경우 샤드 클러스터의 밸런스가 샤드사이의 청크를 교환해준다
    
    sh.stopbalancer()
    도큐먼트가 대량으로 입력되거나 삭제되어서 밸런싱을 머추어야하는 상황
    샤드를 추가하거나 데이터를 복구할 때 쓰면 좋다.


범위 샤딩
샤드키 설정 원칙
    값의 종류가 다양해야한다 이유는 청크 분리 불가능 예시 성별,카테고리,대분류
    값이 골고루 분포해야한다       청크 분리 불가능,특정 샤드 작업 편중 예시 성씨,신장,체중
    값이 지속적으로 증가/감소되지 않아야 한다        특정 샤드 작업 편중 예시 신장,재고,수정날짜
해시 샤딩
    값의 종류가 다양해야 한다.
    값이 골고루 분포해야한다.
    값이 지속적으로 증가. 감소되지 않아야 한다.

샤딩 구역 설정
    샤드 클러스터에 범위 샤딩 혹인 해시샤딩을 구성하고 나서 '샤딩 구역' 을 설정할 수있다. 
    ex 샤드키 '1'           샤드키 '2'
       구역{KR}             구역{US}

    sh.addShardTag("샤드 이름","태그 이름")
    sh.addShardTagRange(
        "<데이터베이스>.<컬렉션>",
        {<샤드키>:<최솟값>},
        {<샤드키>:<최대값>},
        <태그 이름>
        )

    최솟값으 가지지 않으면 MinKey를 최댃값을 가지지 않으면 MaxKey로 범위를 설정하면 된다.

    샤딩구역이 필요한 경우
        - 샤드 성능이 다른 경우
        - 특정 정보를 일부 샤드에 나누어 저장해야 하는 경우
        - 지역별로 읽기 쓰기가 빈번한 경우

실습
    /db/data/config0-0
    /db/data/config0-1
    /db/data/config0-2
    설정 서버 구성

    mkdir -p /db/data/config0-0/rs0-0 /db/data/config0-0/rs0-1 /db/data/config0-0/rs0-2

    실행
        mongod --configsvr --replSet config0 --port 27018 --bind_ip localhost --dbpath /db/data/config0-0
    
    mongo --port 27018

    rs.initiate({
        _id:"config0",
        configsvr:ture,
        members: [
            {_id:0,host:"localhost:27018"},
            {_id:0,host:"localhost:27019"},
            {_id:0,host:"localhost:27020"}
        ]


    })

    샤드 구성

    /db/data/shard0-0
    /db/data/shard0-1
    /db/data/shard0-2
    설정 서버 구성

    mkdir -p /db/data/shard-0/rs0-0 /db/data/shard0-0/rs0-1 /db/data/shard0-0/rs0-2

    실행
        mongod --shardsvr --replSet shard0 --port 27021 --bind_ip localhost --dbpath /db/data/shard0-0
    
    mongo --port 27021

    rs.initiate({
        _id:"shard0",
        configsvr:ture,
        members: [
            {_id:0,host:"localhost:27021"},
            {_id:0,host:"localhost:27022"},
            {_id:0,host:"localhost:27023",arbiterOnly:true}
        ]


    })

    몽구스 설정하기 샤드클러스트를 이용하기 위해서 필요한 인스턴스이다.

    mongos --configdb config0/localhost:27018 --port 20000

    mongo --port 20000 

    sh.addShard("shard0/localhost:27021")

    user config
    db.shards.find() 결과를 확인 할 수 있다.

    mongo --port 20000
    show dbs
    car_accident

    use car_accident

    show collections

    sh.enableSharding("car_accident") -> 데이터베이스에 샤딩을 활성화

    범위 샤딩 구성하기
    db.by_month.createIndex({country:1})
    sh.shardCollection("car_accident.by_month",{
        country:1
    })

    헤시 샤딩 구성하기
    db.area.createIndex({country:"hashed"})
    sh.shardCollection("car_accident.area",{
        country:"hashed"
    })

    mongod 설정 파일 사용
    /etc/mongod.conf

    설정 사진들..

    샤드 클러스터 관리하기

    설정 서버 
    
    use config 
    show collections 

    여러 컬렉션들이 나옴 필요하면 찾아보기 

    use config
    db.status.find()
    //shards 조회

    sh.status() 
    //클러스터 상태 확인하기
    
사용자 인증
    security:
        authorization: enabled