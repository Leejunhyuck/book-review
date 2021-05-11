복제
    가장 큰 목적 : 서비스 중인 mongodb 인스턴스에 문제가 생겼을 때, 정보가 복제된 다른 인스턴스를 이용해서 문제가 생긴 인스턴스를 대체하는 것이다.
    
    하지만, 복제된 인스턴스는 변경된 정보를 동기화하는데 시간이 걸린다.

    복제를 수행 하기 위해서는 여러 mongod 인스턴스가 모인 "묶음"이 필요하다. 이 묶음에 속하게 되면 서로의 정보를 동기화한다. 이런 묶음을 복제세트라고하고 이에는 3가지로 구성된다.
    1. 클라이언트와 읽기 및 쓰기 작업을 하는 '프라이머리 구성원' 
    2. 프라이머리 구성원의 정보를 동기화하는 '세컨더리 구성원'
    3. 정보를 저장하지는 않고 복제 세트의 복구를 돕는 '아비터 구성원' 이렇게 세 가지 종류의 구성원이 복제 세트를 구성한다.

    서로 살아 있는지 10초마다 핑을 보내는 작업을 수행한다. 이를 하트비트라고한다.

    복제 세트 구성원의 기능

                프라이머리                  세컨더리                아비터

    주요역할    클라이언트와 정보교환       프라이머리 정보 복제        투표진행
    정보저장            O                           O                   X
    선거에서역할    문제가 발생              피선거권,선거권 보유      선거권 보유


    새컨더리가 프라이머리가 되고 프라미어리보다 최신의 정보는 지운다. 그래서 롤백이 되는데 소실되어서는 안되는 민감한 정보를 저장해야 하는 경우에는 WriteConcern을 보수적으로 설정해야한다.

    mongod
        --replSet <복제 세트 이름>
        --port <포트번호>
        --bind_ip <연결할아이피>
        --dbpath <정보를 저장할 경로>
        --oplogSize <오피로그 크기>
    
    mkdir -p /db/data/rs-0 /db/data/rs-1 /db/data/rs-2
    
    포트 27018, 27019, 27020

    mongo --port 27018 실행 
    
    //rs()이라는 이름의 복제 세트로 묶는다
    rs.initiate({
        _id:"rs0",
        members:[
            {_id:0,host:"localhost:27018"},
            {_id:0,host:"localhost:27019"},
            {_id:0,host:"localhost:27020"},
        ]

    })

    rs.status() // 상세 정보
    프라이머리에서 데이터 넣고, 세컨더리에서 조회
    rs.slaveOk() //세컨더리는 기본값으로 셸을 사용하지 못하게 되어있다. 사용가능하게 하기

    
    배포 환경에 복제 세트 구성
        mongod.conf 파일을 수정해서 인스턴스의 기본 값이 외부와 연결이 가능하도록 설정해야 한다.
        /etc/mongod.conf 결로에 설정파일 존재

        net:
            bindIp: 127.0.0.1
            port: 27017

    복제 세트 세부 설정
        rs.conf()
        rs.reconfig(<복제옵션>) // 세컨더리의 우선순위 변경 가능

    새로운 구성원을 복제 세트에 추가하려고 할때 add 명령어를 이용하면 추가 할 수 있다.
    rs.add({
    {_id:,
    host:,
    설정값들..
    }
    })
    
    rs.remove("128.20.1.33") //복제 세트에서 제외 하고자 할 떄에는 해당 구성원의 인스턴스를 종류하고, remove 명령어를 주어서 구성원을 제외하면 된다.


    읽기 선호 설정이 가능하다 
        기본은 프라이머리에서 읽어오지만, 읽기 작업을 분산시킬수 있다. ( 드라이버에서 관리, 식제 개발할 떄에는 드라이버의 설정에 맞추어 개발해야 한다.)
    
    1.프라이머리 구성원에서 읽는 상태
    2.프라이머리 구성원에서 우선적으로 읽는 상태(읽기 작업이 밀리게 되면 세컨더리로부터 정보를 불러오기 때문에 변경사항 반영이 좀 늦을수 있다.)
    3.세컨더리 구성원에서 읽는 상태
    4.세컨더리 구성원에서 우선적으로 읽는 상태
    5.가까운 구성원에서 읽는 상태 ( 정보 변경 반영이 클라이언트마다 제각각이 될 우려가 있다.)


동기화 작동 방식
    오피로그 
        Mongodb 인스턴스 내에서 변경된 정보 내역을 저장한 특별한 Capped 컬렉션이다.
        db.oplog.rs.find().limit(1).pretty() // 조회
        
        //사진
    초기 동기화
        새로운 세컨더리를 추가했을 때 오피로그의 정보만으로 동기화를 수행할 수 없다면 '초기 동기화'를 수행하게 된다.

    ReadConcern 
        그냥 프라이머리에 기록된 정보를 가져올지, 아니면 대다수의 복제 세트에 기록된 정보를 가져올지 정할 수있다.
        local 단계 majority(여러 개의 도큐먼트를 작업하는 트랜잭션에 이 옵션을 사용하려면) 단계 linearizable 단계 (시간옵션을 설정 해주어야함)

        cursor.readConcern("linearizable").maxTimeMS(10000)
        //쿼리의 결과로 나온 커서 뒤에 readConcern이라는 명령어를 이용하면 읽어오는 정보에 관련 옵션을 붙일수있다.

        db.rating.aggregate(
            [{$match:{rating:{$lt:5}},
            readConcern:{level:"majority"}          
            }]
        )

        session =db.getMongo().startSession()
        session.startTransaction({readConcern:{lecel:"snapshot"}},
        WriteConcern:{w:"majority"}});
        session.commitTransaction();
        session.endSession();

    WriteConcern
        복제 세트에 장애가 발생했을 때 중요한 정보가 사라질지, 유지될지 결정할 수 있다.
        w : 복제 세트의 어느정도의 구성원에 쓰기 작업이 완료되어야 전체 쓰기 작업이 완료되었다고 판단할지 결정하는 옵션
        j : 쓰기 작업은 메모리에 변경 사항을 남겼다가 일정 주기로 디스크에 변경 사항을 기록 디스크 변경사항을 임시로 저장하는 작업을 저널링이라고 한다. 저널링은 장애가 발생해도 기록이 남기 때문에 복구가 가능하다.
        wtimeout : w 옵션에서 설정한 구성원들을 기다릴 수 있는 최대시간 , 시간이 지나면 에러 반환 이미 실행한 쓰기 작업 자체는 취소되지 않는다.

        db.rating.aggregate(
            [ {$match:{rating:{$lt:5}}}],{
                writeConcern:{w:"majority",j:t}
            }
        )

        ession =db.getMongo().startSession()
        session.startTransaction({readConcern:{lecel:"snapshot"}},
        WriteConcern:{w:"majority"}});
        session.commitTransaction();
        session.endSession();

    오피로그의 크기
        세컨더리가 동기화를 끝내지 못했는데 오피로그의 크기가 작아 동기화할 데이터가 지워진다면 가지고 있는 데이터를 모두 지우고 초기 동기화 과정을 가지게 된다.
        다행히 4.0 버전 이상부터는 대다수의 세컨더리에서 오피로그를 동기화하지 못했으면 프라이머리의 오피로그를 삭제하지 않는 기술이 탑재 되었다

        - 세컨더리를 잠깐 복제 세트에서 제외 했다가 다시 포함시키는 상황
        - 물리적으로 백업된 정보를 인스턴스에 덮어씌우고 세컨더리를 복제 세트에 추가하는 상황

        오피로그 크기 수정하기
        use local
        db.oplog.rs.stats().maxSize

        db.adminCommand({replSetResizeOplog:ture,size:16834})

    동기화 상태 확인
    rs.printSlaveReplicationInfo()