---
layout: post
author: Bruce Lee
title: AWS Data Week Day 2
---

## 👨‍🎓 1. AWS NOSQL<br/><br/>
- 데이터 특성에 맞게 join 과 같은 관계형 디자인해야 하거나 트랜잭션을 유지해야하는 경우에는 NOSQL 이 알맞지 않다
- Aurora 의 경우, write 레플리케이션과 달리 read 레플리케이션은 계속해서 업그레이드 될 수 있다
- Mongo DB 와 같은 Document DB 는 장점도 있지만 최근 서버가 늘어나면서 복잡해지는 문제가 발생하고 있다
- ElastiCache (레디스) 와 같은 서비스는 인메모리지만 리스타트 시 데이터가 휘발되지 않도록 만든 서비스
- Neptune (Graph 형 DB)
- Time series 의 경우 데이터가 들어온 시간을 이용하여 중복한 데이터를 제거하고 데이터 변경만 있을 때 저장하는 DB 이다
  <br/><br/>
## 👨‍🎓 2. AWS DynamoDB<br/><br/>
- DynamoDB 의 경우 A1 (partition key) 를 사용하여 한번에 Item (데이터)를 찾아가고 A2 (sort key) 를 사용하여 B tree 구조로 데이터를 가져온다.
- 관계형 데이터와 달리 detail 과 같은 정보도 하나의 key 로 구성한다.
- GSI (Global Second Index) 의 장점은 파티션 키를 바꿀 수 있다는 것이다.
- 데이터 안정성을 위한 3 Copy 복제 : 3 AZ 에 복사
- 파티션 키는 유일한 값이 많은 키가 좋다 (10GB 제한)
  <br/><br/>
## 👨‍🎓 3. AWS ElastiCache<br/><br/>
- 관리형 REDIS 또는 MEMCACHED 호환의 IN-MEMORY 데이터 저장소이나 리스타트 후에도 메모리 휘발은 없으나 Write 에 약간 더 소요
- 분산시키는 개념인 Shards 를 3 AZ 에 3 Shards 로 복사해놓아서 read 레플리케이션을 통해 가용성을 높였다
- Neptune : 특정 지역에 갔을 때 해당 지역과 관련된 추천을 해주는 것과 같은 관계를 가질 때, 해당 그래프 DB 서비스를 사용할 수 있다.
- 넵튠을 사용하면 Self join 으로 반복으로 인한 퍼포먼스 저하가 생기지 않는다.
  <br/><br/>
## 👨‍🎓 4. AWS Timestream<br/><br/>
- 현재 Seoul 리전에는 없으며 Ingestion / Storage / Query layer 로 구성되어있다.
- Ingestion layer 는 인메모리를 사용하여 빠르게 데이터를 소화하며 AWS SDK 를 이용해 자바 등의 언어를 지원한다.
  <br/><br/>

## 5. QLDB - 블록체인 활용하여 데이터 변경 이력 보장<br/><br/>

## 👨‍🎓 6. AWS DynamoDB Deep Dive<br/><br/>
- 티어 0 서비스는 내 서비스 위에 다른 서비스가 올라갈 수 있는 것들을 의미한다.
- 서치바 혹은 책을 클릭하는 경우 나오는 정보, Best Seller 의 기준이나 Cart 등의 경우는 관계형 DB 만으로 구성하기 힘들다.
- DynamoDB 에 Stream 을 걸어서 특정 이벤트를 버퍼로 쌓아뒀다가 Async 하게 Elastisearch Service 에 데이터를 넣거나 ElastiCache for Redis 에 +- 하여 랭킹하는데 간단하게 이용 가능하다.
- 최근 Elastic 즉, 글로벌 트래픽 변화를 안정적으로 감당하기 위한 확장성이 중요하다. (EX. 스냅챗)
- DynamoDB Accelerator 는 서울 리전에 없으나 현재 서버리스가 아니기 때문에 비추천
- 리퀘스트에 따라 비용이 들며 비용최적화를 지원하고 있다. (인스턴스 대신 파티션이 있다)
<br/>

### DynamoDB 트렌드 (사용하면 좋은 경우)
- No Ops (글로벌 서비스로 version 이 없는 개념으로 key - value 로 이루어지므로 운영이 불필요)
- Write intensive (IoT 모니터링 등과 유저 활동 로깅, 히스토리나 채팅, 걸음 수 등 쓰기 트래픽이 갑자기 늘어나는 경우)
- Consistent latency (광고와 같이 느리면 누락되어 버리거나 세션, 유저 프로필, 상품 상세 정보 등에서 균일한 시간이 필요한 경우)
- 디즈니 플러스와 같은 첫 화면에서 주로 사용 (글로벌 레플리케이션을 이용해서 다른 국가에 갔을 때도 일정하게 시간 소요)
- 틴더의 경우 MongoDB - DynamoDB 간 온라인 마이그레이션 사례이다.
- lyft 는 차량의 이동 트래킹에 사용한다 (즉, 특정 도메인에 국한되지 않고 워크로드에 따라 사용하면 된다)
- 시계열 데이터를 저장해야 하는 경우가 아니면 굳이 테이블을 여러개로 만들 이유가 없다
- Data type 은 Primary key 로 사용할 수 있는 String, Number 2개 있다고 생각하는 것이 좋다
- read write 총 10개 API 로 통신한다
- 일반적으로는 GSI 를 사용하는데, LSI 는 강한 일관성을 갖는다 (하지만 테이블 생성 시 생성해야 하며 삭제 불가)
- Get Item 의 경우 3 노드 중 2개를 쓰고 시작하기 때문에 2/3 확률로 최신 데이터를 볼 수 있으나 일관성이 필요한 경우 프라이머리로 조회해야 한다
- Auto Admin
- 사용최적화를 통해 WriteCapacityUnit, ReadCapacityUnit 을 사용하거나 예약 용량(Reserved Capacity) 을 사용하여 특정 트래픽 집중 시간에 비용 최적화가 가능하다
- Entity, pk, sk, Attributes 싱글 테이블 디자인
- 성능 테스트 등이 필요한 경우 On-demand mode 에서 Provisioning mode 로 전환하여 비용 최적화
- 쓰로틀링 max 수치를 커버하기 위한 페일오버 리트라이 전략이 필요
- TTL (Time To live) : 배치를 별도로 사용하여 삭제하고 그 양을 조절하는 것보다 TTL 을 걸어놓는 것이 좋다
  - RCU, WCU 감소 및 스토리지 사이즈 조절 하는데 효율적
- Reserved Capacity (약정 계약을 통해 할인 받을 수 있음)
  1) 수집 (리전별 최근 3~6개월 사용량 수집 필요)
  2) 예측 (1년 총 사용량 예측 필요, 월별 MIN to MAX 값 도출 및 차이 분석)
  3) 산정 (리전별 구매 비율 산정 필요)
<br/>
### Standard-IA 테이블 클래스
- 스토리지 비용 비율이 90% 이상인 경우 테이블 클래스를 변경하여 Standard IA 클래스로 변경 시 비용 절감 가능
- 워크로드가 증가하는 경우 해당 클래스 변경했을 때 비용이 더 나올 수 있기 때문에 워크로드에 따른 기준 수립 필요
- Cost Explorer 참고 가능
- 대규모 테이블 위주로 반영해보는 것이 좋을 수 있음
  <br/><br/>
## 👨‍🎓 7. Document DB<br/><br/>
- Sharding 이 가능하며 Secondary Index 를 불편하지 않게 사용할 수 있는 Database
- Document Databases 는 JSON 과 호환이 잘되며 비정규화 데이터 모델로 Schemaless 하여 변경이 용이
- 하지만 RDBMS 에 비해 IO 가 큰 폭으로 증가할 수 있으며 (1 row 당 46 byte 로 약 3~4 배) DB Buffer 에도 상대적으로 적은 수의 document 가 존재하여 성능상 취약해질 가능성이 크다
- DocumentStore 의 Modeling Patten 에서 통채로 데이터 모델을 가지고 있는 Embedded Data Model 은 특수한 상황 (같은 도큐먼트에 대해 다른 update 작업이 발생하여 Lock 발생으로 인한 성능 저하 발생 등) 에서 Reference Data Model 을 사용할 수 는 있으나 Document DB 를 사용하는 것이 맞는지 생각해보아야 한다.
- MongoDB 에서 비정상 종료 시 복구에 사용되는 Journal Checkoint 발생 시 dirty page 를 sync 로 인한 성능 저하가 발생하나 DocumentDB 에서는 이를 IO 성능에 영향을 주지 않게 조정 가능하다
- Buffer pool (Shared pool) 은 Mongo DB 의 특성 상 OS 에게 Disk IO 를 의뢰하는 구조 상 튜닝포인트로 적합하지 않다
- Buffer pool 을 정리하는 Eviction 작업을 모니터링하는 것이 좋다
- Sharding key 를 기준으로 청크 단위로 데이터가 샤드 간 분리되며 mongos 사용 시에는 메모리가 중요하다
- 샤드 하나 대상으로 하는 Target Query 와 전체를 대상으로 하는 BroadCast Query 를 잘 구분하여 쿼리를 짜야 장애가 발생하지 않는다
- Task Executor Pool : mongos (cluster) 가 요청하는 샤드 쿼리 요청에 대해 연결되는 connection pool
- Getmore, Aggregation, count() 등 쿼리에서는 기본 Cursor 가 호출되며 별도의 OS heap 영역을 기본 16mb 기본 할당받는다
- Large size 의 경우, Centreal Heap 에서 할당받아야 하기 때문에 Spin Lock Wait 이 발생하며 이는 해당 메모리 할당을 위해 대기하고 있는 상태이다
- 해당 Lock 등에서 대기할 경우 Read & Write Ticket 을 사용한 채로 대기를 하게 되는데 Server 별로 기본 128개씩 이기 때문에 모두 사용하게 될 경우 Ticket 할당이 될때까지 query wait 이 걸린다
- Aggregation 쿼리 생성 시 $match 를 파이프라인 최상단에 배치하도록 하여 (where 조건절 처럼) 쿼리 튜닝 필요
- Match 조건에는 반드시 적절한 index 전략이 필요하다
  <br/><br/>
## 👨‍🎓 8. AWS Neptune<br/><br/>
- Graph Data Model : 객체간 관계를 기반으로 연결된 데이터를 그래프 형식으로 저장
- Property Graph 는 유연성을 초점으로 지원하며 RDF 는 형식을 조금 더 강조하는 형태로 지원되고 있다
- 따라서 RDF 는 시스템 간 knowledge graph 를 중심으로 Property Graph 는 어플리케이션 관점에서 관리되고 있다
- Vertex 와 Edge 기반으로 데이터를 구성
- SNS 등에서 팔로잉과 팔로워를 연결하고 팔로잉 하는 사람의 친구를 추천해주거나 구매 상품을 추천해주는 등의 관계로도 사용됨
- 계층적 구조로 되어있는 경우에 활용하기 용이하다
- 그래프 데이터 모델링은 비즈니스 프로세스가 아니라 관계 자체를 저장한다
  - 즉 RDB 는 관계를 보이지않는 메타데이터의 join 으로 표현하는데에 비해 그래프 DB 는 해당 관계 자체를 저장하기 때문에 효율적으로 찾아갈 수 있다
  - 관계 자체를 저장하기 때문에 좀 더 엄격한 스키마를 기반으로 데이터들의 관계를 이해하기 쉽다
- 오픈서치, SageMaker 등과 데이터 호환이 용이하다
  <br/><br/>
## 👨‍🎓 9. AWS Elasticache for redis<br/><br/>
- Redis data structures : String, Hash, List, Set, Sorted Set, Geospatial, HyperLogLog ...
- ziplist 에서 listpack 으로 오픈소스 레디스 변경사항 있음
- 특정 빈번한 aggregate 쿼리를 MD5 Hash 를 사용하여 쿼리문 자체를 캐싱해놓고 바로 return 하는 형태로 구현
