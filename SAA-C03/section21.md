# Section 21. AWS의 데이터베이스

## 목차

1. [DB 선택 가이드](#1-db-선택-가이드)
2. [Amazon RDS](#2-amazon-rds)
3. [Amazon Aurora](#3-amazon-aurora)
4. [Amazon ElastiCache](#4-amazon-elasticache)
5. [Amazon DynamoDB](#5-amazon-dynamodb)
6. [Amazon DocumentDB](#6-amazon-documentdb)
7. [Amazon Neptune](#7-amazon-neptune)
8. [Amazon Keyspaces](#8-amazon-keyspaces-for-apache-cassandra)
9. [Amazon QLDB](#9-amazon-qldb)
10. [Amazon Timestream](#10-amazon-timestream)

---

## 1. DB 선택 가이드

올바른 데이터베이스를 선택하기 위한 핵심 질문들:

- **워크로드 유형**: Read-heavy? Write-heavy? 균형 잡힌 워크로드? 처리량(Throughput) 요구사항은?
- **데이터 규모**: 데이터 양은? 보관 기간은? 평균 객체 크기는?
- **내구성**: **Durability** 수준? 데이터의 **Source of Truth**는?
- **성능 요구사항**: 지연 시간(Latency) 요구사항? 동시 접속자 수?
- **데이터 모델**: 조인(Joins) 필요? 반정형(Semi-structured) 데이터?
- **스키마 유형**: 강한 스키마(Strong schema)? 유연한 스키마(Flexible schema)? 리포팅/검색 기능 필요?
- **비용**: 라이선스 비용 고려

> 💡 **시험 포인트**:
> - 각 DB 서비스의 "대표 키워드"를 암기해야 한다 (예: 그래프 → Neptune, 원장 → QLDB)
> - OLTP vs OLAP 구분: RDS/Aurora = OLTP, Redshift = OLAP
> - 서버리스 여부: DynamoDB, Aurora Serverless, Keyspaces, QLDB, Timestream = 서버리스 가능
> - "MongoDB 호환" = DocumentDB, "Cassandra 호환" = Keyspaces

---

## 2. Amazon RDS

### 2.1 개요

- **관계형 데이터베이스(Relational DB)** 관리형 서비스
- 지원 엔진: **PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, Aurora**
- **OLTP(Online Transaction Processing)** 워크로드에 최적화

### 2.2 주요 특징

- **강한 스키마(Strong schema)** + SQL 지원
- **Multi-AZ**: 자동 장애 조치(Failover)로 고가용성 제공
- **Read Replicas**: 읽기 확장(Scaling reads)
- 자동 백업, 스냅샷, 패치 관리

### 2.3 RDS가 적합하지 않은 경우

| 시나리오 | 대안 |
|---|---|
| 비정형(Unstructured) 데이터 | DynamoDB, DocumentDB |
| 복잡한 분석/집계 쿼리 | Redshift (OLAP) |
| 그래프 관계 데이터 | Neptune |

> 💡 **시험 포인트**:
> - RDS = OLTP, Redshift = OLAP (분석 워크로드에는 Redshift 선택)
> - Read Replicas는 비동기 복제, Multi-AZ는 동기 복제
> - RDS에서는 SSH로 기저 EC2 인스턴스에 접근 불가

---

## 3. Amazon Aurora

### 3.1 개요

- AWS 고유의 클라우드 네이티브 관계형 DB
- **MySQL** 또는 **PostgreSQL** 호환
- MySQL 대비 **5배**, PostgreSQL 대비 **3배** 빠른 성능

### 3.2 스토리지 및 고가용성

- **자동 확장 스토리지**: 10GB → 최대 **128TB**
- 6개 사본(Copies)을 3개 AZ에 분산 저장
- 쓰기 시 4개, 읽기 시 3개 사본 필요

### 3.3 고급 기능

| 기능 | 설명 |
|---|---|
| **Aurora Global Database** | 다른 리전에 읽기 전용 복제본(최대 5개 리전) |
| **Multi-Master** | 여러 Writer 인스턴스로 쓰기 고가용성 |
| **Aurora Serverless** | 자동 시작/중지, 자동 스케일링 |
| **Aurora Machine Learning** | SQL → ML 예측 (SageMaker, Comprehend 연동) |

### 3.4 Aurora Serverless

- **비정기적/예측 불가능한 워크로드**에 적합
- 자동 시작/중지로 사용하지 않을 때 비용 절감
- 용량 자동 스케일링 (ACU: Aurora Capacity Units)

### 3.5 Global Aurora

- **읽기 복제본**: 최대 5개 리전에 보조 클러스터 배포
- **DR(재해복구)**: 다른 리전을 **1분 미만**에 프라이머리로 승격 가능
- **복제 지연**: 일반적으로 **1초 미만**

> 💡 **시험 포인트**:
> - Aurora Global DB = 복제 지연 < 1초, 리전 승격 < 1분 → RTO/RPO 최소화
> - Aurora Serverless = 간헐적·예측 불가 워크로드 키워드
> - Aurora Machine Learning = SageMaker + Comprehend 통해 SQL에서 ML 예측 가능
> - Aurora = 최대 15개 읽기 복제본 (RDS는 최대 5개)

---

## 4. Amazon ElastiCache

### 4.1 개요

- 관리형 **Redis** 또는 **Memcached** (인메모리 데이터베이스)
- **서브 밀리초(Sub-ms) 지연 시간**, 높은 처리량
- EC2 인스턴스 타입을 직접 프로비저닝 필요 (서버리스 아님)

### 4.2 Redis vs Memcached 비교

| 기능 | Redis | Memcached |
|---|---|---|
| **Multi-AZ** | Yes (자동 장애 조치) | No |
| **읽기 복제본** | Yes (읽기 확장) | No |
| **영속성(Persistent)** | Yes (AOF 영속성) | No |
| **백업/복구** | Yes | No |
| **Pub/Sub** | Yes | No |
| **데이터 구조** | Yes (Sorted Sets 등) | No |
| **멀티 스레드** | No | Yes |

### 4.3 선택 기준

**Redis 선택 시:**
- 영속적 캐시, 고가용성(HA) 필요
- 백업/복구 필요
- **Sorted Sets** (리더보드, 순위표)
- 세션(Session) 저장

**Memcached 선택 시:**
- 순수 캐싱(Pure caching)만 필요
- 멀티 스레드(Multi-threaded) 성능 필요
- HA/영속성 불필요

> 💡 **시험 포인트**:
> - Redis = HA + 영속성 + 백업 + Sorted Sets → "리더보드" 키워드면 Redis
> - Memcached = 멀티 스레드, 단순 캐싱, HA 불필요
> - ElastiCache = EC2 인스턴스 타입 프로비저닝 필요 (완전 서버리스 아님)
> - 세션 저장소로 ElastiCache Redis 또는 DynamoDB 사용 가능

---

## 5. Amazon DynamoDB

### 5.1 개요

- **서버리스 NoSQL** 데이터베이스, **밀리초(ms) 지연 시간**
- Key-Value 및 Document 데이터 모델
- 완전 관리형, 가용성 99.99%

### 5.2 용량 모드

| 모드 | 설명 | 적합한 경우 |
|---|---|---|
| **Provisioned** | RCU/WCU 사전 지정, Auto Scaling 가능 | 예측 가능한 워크로드 |
| **On-Demand** | 사용량 기반 자동 스케일링 | 예측 불가 워크로드 |

### 5.3 고급 기능

- **DynamoDB Accelerator (DAX)**: 인메모리 캐시, **마이크로초(μs) 지연 시간**
- **DynamoDB Streams**: 변경 이벤트 스트림 → 이벤트 기반 아키텍처
- **Global Tables**: **액티브-액티브(Active-Active)** 다중 리전 복제
- **TTL(Time To Live)**: 만료 시간 설정으로 자동 항목 삭제

### 5.4 제한 사항 및 사용 사례

- 최대 항목 크기: **400KB**
- 적합한 사용 사례: 서버리스 앱, 소형 문서, Key-Value 저장소, 게임 리더보드, IoT

> 💡 **시험 포인트**:
> - DynamoDB = 서버리스 + ms 지연, DAX = μs 지연 (캐시 레이어)
> - Global Tables = 액티브-액티브 (Aurora Global = 액티브-패시브 읽기)
> - ElastiCache 대안으로 세션 저장 시 TTL 활용
> - 최대 항목 크기 400KB 초과 → S3에 저장 후 참조 방식 사용

---

## 6. Amazon DocumentDB

### 6.1 개요

- **Aurora가 AWS 관리형 MySQL/PostgreSQL**이라면, **DocumentDB는 AWS 관리형 MongoDB**
- **MongoDB 호환** (JSON 데이터 모델)
- 완전 관리형, 고가용성, 자동 스케일링 스토리지

### 6.2 주요 특징

- **3개 AZ**에 복제 (Aurora와 유사한 아키텍처)
- 자동 스케일링 스토리지: **10GB 단위**, 최대 **64TB**
- MongoDB API, 드라이버, 도구 호환

### 6.3 사용 사례

- 콘텐츠 관리, 카탈로그, 사용자 프로파일
- **MongoDB 마이그레이션** 시나리오

> 💡 **시험 포인트**:
> - "MongoDB 호환" = DocumentDB (시험에서 가장 중요한 키워드)
> - DocumentDB ≠ Aurora: Aurora는 RDS 호환, DocumentDB는 NoSQL 문서 DB
> - 최대 64TB 자동 스케일링 (Aurora는 최대 128TB)

---

## 7. Amazon Neptune

### 7.1 개요

- 완전 관리형 **그래프 데이터베이스(Graph Database)**
- 고도로 연결된(Highly connected) 데이터셋 작업에 최적화
- **수십억 개의 관계(Relations)** 저장, **밀리초 지연 시간**

### 7.2 고가용성

- **3개 AZ**에 걸쳐 HA
- 최대 **15개 읽기 복제본**

### 7.3 사용 사례

| 사용 사례 | 설명 |
|---|---|
| **소셜 네트워크** | 친구 관계, 팔로우/팔로워 그래프 |
| **지식 그래프(Knowledge Graph)** | Wikipedia 스타일 데이터 |
| **사기 탐지(Fraud Detection)** | 이상 패턴 관계 분석 |
| **추천 엔진** | 구매/시청 패턴 기반 추천 |

> 💡 **시험 포인트**:
> - **그래프 데이터베이스** = Neptune (유일한 AWS 관리형 그래프 DB)
> - 소셜 네트워크, 사기 탐지, 추천 엔진 키워드 → Neptune
> - 3 AZ HA, 최대 15개 읽기 복제본

---

## 8. Amazon Keyspaces (for Apache Cassandra)

### 8.1 개요

- **Apache Cassandra 호환** 관리형 서비스
- 서버리스, 확장 가능, 고가용성
- **CQL(Cassandra Query Language)** 사용

### 8.2 주요 특징

- **On-Demand** 또는 **Provisioned** 용량 모드
- 저장 데이터 암호화(Encryption at rest)
- **PITR(Point-In-Time Recovery)**: 최대 **35일** 백업

### 8.3 사용 사례

- IoT 데이터, **시계열(Time-series) 데이터**
- 고용량 Cassandra 마이그레이션

> 💡 **시험 포인트**:
> - "Cassandra 호환" = Keyspaces (Apache Cassandra on AWS)
> - Wide column 데이터 모델, 시계열 데이터 적합
> - PITR 최대 35일 (DynamoDB도 동일)

---

## 9. Amazon QLDB

### 9.1 개요

- **QLDB = Quantum Ledger Database**
- **금융 트랜잭션(Financial Transactions)** 기록 전용 원장(Ledger) DB
- 완전 관리형, 서버리스, HA, 3개 AZ 복제

### 9.2 핵심 특징

- **불변(Immutable)**: 어떤 항목도 제거하거나 수정 불가
- **암호학적 검증(Cryptographically Verifiable)**: 데이터 변경 이력 검증 가능
- 일반 블록체인 프레임워크 대비 **2-3배 빠른 성능**

### 9.3 QLDB vs Amazon Managed Blockchain

| | QLDB | Amazon Managed Blockchain |
|---|---|---|
| **분산화** | 중앙 권한(Central authority) | 탈중앙화(Decentralization) |
| **사용 목적** | 단일 조직의 변경 불가 기록 | 다중 당사자 탈중앙화 원장 |
| **규정 준수** | 금융 규제 준수 적합 | 블록체인 네트워크 |

### 9.4 사용 사례

- 금융 트랜잭션, **공급망(Supply Chain)**, HR/급여(Payroll)

> 💡 **시험 포인트**:
> - QLDB = "불변 원장" + "금융 트랜잭션" 키워드
> - QLDB ≠ Managed Blockchain: QLDB는 중앙 권한, Blockchain은 탈중앙화
> - 데이터 변경 이력 감사(Audit) 필요 시 QLDB

---

## 10. Amazon Timestream

### 10.1 개요

- 완전 관리형, 빠르고 확장 가능한 서버리스 **시계열 데이터베이스(Time Series Database)**
- 관계형 DB 대비 **1,000배 빠름**, 시계열 쿼리에 최적화
- 하루 **수조 개(Trillions)의 이벤트** 저장 및 분석

### 10.2 스토리지 계층

| 계층 | 설명 |
|---|---|
| **메모리(Memory)** | 최근(Recent) 데이터 저장 |
| **비용 최적화 스토리지** | 과거(Historical) 데이터 저장 |

### 10.3 주요 특징

- 내장 **시계열 분석 함수(Time Series Analytics Functions)**
- 전송 중/저장 중 **암호화(Encryption)**
- 자동 스케일 업/다운

### 10.4 통합 서비스

```
AWS IoT → Timestream
Kinesis → Timestream
Prometheus → Timestream
Telegraf → Timestream
Timestream → Grafana (시각화)
```

### 10.5 사용 사례

- **IoT 앱**, 운영 애플리케이션, 실시간 분석(Real-time analytics)

> 💡 **시험 포인트**:
> - "시계열 데이터베이스" = Timestream (IoT 메트릭, 운영 데이터)
> - Keyspaces도 시계열 가능하지만 Cassandra 마이그레이션 목적; IoT/메트릭 = Timestream
> - Grafana와 통합으로 시각화 대시보드 구성 가능

---

## DB 비교 요약표

| DB | 타입 | 주요 특징 | 대표 키워드 |
|---|---|---|---|
| **RDS/Aurora** | 관계형 OLTP | SQL, Multi-AZ, Read Replicas | 트랜잭션, 관계형 |
| **ElastiCache** | In-memory | Sub-ms, Redis/Memcached | 캐시, 리더보드 |
| **DynamoDB** | NoSQL Key-Value | 서버리스, ms 지연 | Key-Value, 서버리스 |
| **DocumentDB** | NoSQL Document | MongoDB 호환 | MongoDB |
| **Neptune** | Graph | 소셜 네트워크, 사기 탐지 | 그래프, 관계 |
| **Keyspaces** | Wide Column | Cassandra 호환 | Cassandra |
| **QLDB** | Ledger | 불변, 금융 트랜잭션 | 원장, 감사 |
| **Timestream** | Time Series | IoT, 메트릭 | 시계열, IoT |
| **Redshift** | 관계형 OLAP | 데이터 웨어하우스, 분석 | 분석, 데이터 웨어하우스 |

---

← [Section 20. 서버리스 솔루션 아키텍처 토론](section20.md) | [Section 22. 데이터 & 분석](section22.md) →
