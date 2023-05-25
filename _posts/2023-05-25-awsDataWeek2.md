---
layout: post
author: Bruce Lee
title: AWS Data Week Day 3
---

## 👨‍🎓 1. AWS Redshift<br/><br/>
- 고객 니즈에는 비용 효율적으로 대용량 쿼리를 빠르게 사용할 수 있는 부분이 있었으며 특정 시간대에 몰리는 트래픽을 감당할 수 있는 구조가 필요했다
- Federated Query 를 사용하여 현재 운영 중인 DB 와 S3 등 다른 데이터 소스 간에 마이그레이션 없이 데이터 활용이 가능해졌다
- Redshift data sharing 서비스를 사용하면 서로 다른 클러스터 간에 데이터를 옮기거나 복사하지 않고 조회할 수 있는 서비스이다
  <br/><br/>

## 👨‍🎓 2. AWS Stream<br/><br/>
- kinesis 는 오픈소스 Apache Flink 기반의 SQL, Python 및 Scala 를 사용한 스트림 처리이다
- RDS 에 CDC 로 MSK Connect 를 걸어서 MSK 로 옮기고 해당 MSK 에서 트리거링 하는 형태로도 활용 가능하다
- On-demand 를 사용하여 샤드를 트래픽 집중 시간대에 사용하는 방식도 가능하다
  <br/><br/>
## 👨‍🎓 3. AWS OpenSearch<br/><br/>
- 분산형 검색엔진, 노드를 여러대 둘수록 성능이 올라가는 검색엔진
- 유즈케이스는 1) 전문검색 (full text search) 2) 로그검색
- Observability
  - 커스텀 Observability app 을 생성하여 총체적은 시스템 health 나 Trace group 의 뷰를 바탕으로 Latency, Error rate 등을 파악할 수 있다
- PPL (Piped Processing Language) 쿼리를 이용하여 생성된 시각화인 Operation panel 을 제공한다
- AWS Distro for Open Telemetry 사용한 데모
- 한 곳에 로그를 모아서 guard duty 등과 결합하여 문제가 되는 소스 IP 혹은 인스턴스 ID 를 바탕으로 조회할 수 있다
- Open source : [SIEM](https://github.com/aws-samples/siem-on-amazon-opensearch-service)
- Serverless : 서울리전은 현재 나오지 않음
  - 클러스터를 만들거나 샤드를 관리할 필요가 없음
  - 인덱스들의 집합인 컬렉션을 사용하며 이를 엔드포인트로 하여 내부는 블랙박스로 돌아감
  - 로그면 시계열로 혹은 많은 검색어가 필요한 경우 전문검색으로 한다

