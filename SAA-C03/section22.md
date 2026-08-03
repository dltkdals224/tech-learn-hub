# Section 22. 데이터 & 분석

## 목차

1. [Amazon Athena](#1-amazon-athena)
2. [Amazon Redshift](#2-amazon-redshift)
3. [Amazon OpenSearch](#3-amazon-opensearch)
4. [Amazon EMR](#4-amazon-emr)
5. [Amazon QuickSight](#5-amazon-quicksight)
6. [AWS Glue](#6-aws-glue)
7. [AWS Lake Formation](#7-aws-lake-formation)
8. [Kinesis Data Analytics](#8-kinesis-data-analytics)
9. [Amazon MSK](#9-amazon-msk)
10. [Big Data Ingestion Pipeline](#10-big-data-ingestion-pipeline-서버리스-통합-예시)

---

## 1. Amazon Athena

### 1.1 개요

- **서버리스(Serverless)** 쿼리 서비스로 S3 데이터를 SQL로 분석 (Presto 기반)
- 인프라 관리 불필요, 완전 관리형
- 지원 포맷: **CSV, JSON, ORC, Avro, Parquet**
- 가격: 스캔 데이터 **TB당 $5**
- **Amazon QuickSight**와 함께 리포팅/대시보드 구성

### 1.2 주요 사용 사례

| 사용 사례 | 설명 |
|---|---|
| **일회성 쿼리** | 임시 데이터 분석 |
| **비즈니스 인텔리전스** | BI 리포팅, 대시보드 |
| **로그 분석** | VPC Flow Logs, ELB Logs, CloudTrail |

### 1.3 성능 향상 방법

- **컬럼 기반 포맷** (Parquet, ORC) 사용 → 스캔 감소 → 비용 절감
  - Glue를 사용해 데이터를 Parquet/ORC로 변환 가능
- **데이터 압축**: bzip2, gzip, lz4, snappy, zlib, zstd
- **S3 파티셔닝**: 경로로 데이터셋 파티션 (예: `s3://bucket/year=1991/month=1/day=1/`)
- **대용량 파일** 사용 (>128MB) → 오버헤드 감소

### 1.4 Federated Query (연합 쿼리)

```
S3 이외의 모든 데이터 소스에 쿼리 가능
→ Data Source Connectors (Lambda 함수) 사용

지원 소스:
  - ElastiCache, DynamoDB, RDS
  - 온프레미스(On-premises) DB

결과는 다시 S3에 저장
```

> 💡 **시험 포인트**:
> - Athena = 서버리스 + S3 SQL 쿼리 + TB당 $5
> - 비용 절감 = 컬럼 포맷(Parquet/ORC) + 압축 + 파티셔닝
> - Federated Query = Lambda Data Source Connector 사용
> - CloudTrail 로그 분석 → Athena 정답

---

## 2. Amazon Redshift

### 2.1 개요

- PostgreSQL 기반이지만 **OLTP가 아닌 OLAP(Online Analytical Processing)**
- **컬럼 기반(Column-based)** 스토리지 → 분석 쿼리에 최적화
- 기존 데이터 웨어하우스 대비 **10배 향상된 성능**, 페타바이트(PB) 규모 확장
- **MPP(Massively Parallel Processing)** 지원
- SQL 인터페이스, BI 도구 통합 (QuickSight, Tableau)

### 2.2 Athena vs Redshift

| | Athena | Redshift |
|---|---|---|
| **타입** | 서버리스 | 프로비저닝 클러스터 |
| **적합 쿼리** | 임시/단순 쿼리 | 복잡한 쿼리, 조인, 집계 |
| **성능** | 쿼리별 비용 | 인덱스로 더 빠름 |
| **데이터 위치** | S3 | 클러스터 내 + Spectrum |

### 2.3 클러스터 구조

- **Leader Node**: 쿼리 계획(Query Planning), 결과 집계
- **Compute Nodes**: 실제 쿼리 실행
- 노드 타입: `dc2` (Dense Compute), `ra3` (Managed Storage)

### 2.4 스냅샷 & DR

- **Multi-AZ**: 일부 클러스터 타입에서 지원
- **스냅샷**: S3에 저장되는 시점 기반 백업, 증분(Incremental) 방식
  - 자동: 8시간마다 또는 5GB 변경마다 (보존 기간 설정 가능)
  - 수동: 삭제할 때까지 유지
- **Cross-Region Snapshot Copy**: DR을 위한 다른 리전 자동 복사

### 2.5 데이터 로딩 방법

```
1. Kinesis Firehose → Redshift
2. S3 COPY 명령 → Redshift
   (인터넷 또는 Enhanced VPC Routing 사용)
3. EC2 JDBC 드라이버 → Redshift
   (대용량 쓰기에 적합)
```

### 2.6 Redshift Spectrum

- **클러스터 없이 S3 데이터 직접 쿼리** (단, Redshift 클러스터 필요)
- 쿼리를 수천 개의 Spectrum 노드에 제출
- S3를 데이터 스토어로 사용하면서 Redshift SQL 활용

> 💡 **시험 포인트**:
> - Redshift = OLAP, 데이터 웨어하우스, 컬럼 기반 스토리지
> - Redshift Spectrum = S3 데이터를 Redshift SQL로 쿼리 (데이터 로딩 불필요)
> - Cross-Region Snapshot = DR 전략
> - COPY 명령으로 S3에서 Redshift로 대용량 로딩

---

## 3. Amazon OpenSearch

### 3.1 개요

- **Amazon Elasticsearch**의 후속 서비스
- **어떤 필드도 검색 가능**, 부분 일치(Partial matches) 지원
- DynamoDB 쿼리를 보완(Complement)하는 검색 레이어 역할
- 관리형 클러스터 또는 **서버리스** 모드
- 보안: **Cognito + IAM, KMS, TLS**

### 3.2 일반적인 통합 패턴

**DynamoDB + OpenSearch 패턴:**

```
DynamoDB Table
    ↓ DynamoDB Streams
    ↓ Lambda
    ↓ OpenSearch (검색 인덱스)

사용자 API:
  - OpenSearch에서 검색
  - 결과 ID로 DynamoDB에서 항목 조회
```

**CloudWatch Logs + OpenSearch 패턴:**

```
CloudWatch Logs
    ↓ Subscription Filter
    ├─ Lambda (실시간) → OpenSearch
    └─ Kinesis Firehose → OpenSearch
```

**데이터 수집 소스:**
- Kinesis Firehose, IoT, CloudWatch Logs, 커스텀 앱

> 💡 **시험 포인트**:
> - OpenSearch = DynamoDB 부분 검색 보완용 (DynamoDB는 기본키 조회만 효율적)
> - 전문 검색(Full-text search), 부분 일치 → OpenSearch
> - DynamoDB Streams → Lambda → OpenSearch 패턴 자주 출제
> - Cognito + IAM으로 접근 제어

---

## 4. Amazon EMR

### 4.1 개요

- **EMR = Elastic MapReduce**
- 데이터 처리/ML/웹 인덱싱/빅데이터를 위한 **Hadoop 클러스터** 생성
- 클러스터: 수백 개의 EC2 인스턴스
- 자동 스케일링 + **스팟 인스턴스(Spot Instances)** 지원

### 4.2 지원 프레임워크

- **Hadoop, Spark, HBase, Presto, Flink**

### 4.3 노드 타입

| 노드 | 역할 | 지속성 |
|---|---|---|
| **Master** | 클러스터 관리, 작업 조율 | 장기 실행 |
| **Core** | 데이터 저장 + 작업 실행 | 장기 실행 |
| **Task** | 작업 실행만 (저장 없음) | 임시 (주로 Spot) |

### 4.4 사용 사례

- **데이터 처리**, 머신 러닝, 웹 인덱싱, 빅데이터 워크로드

> 💡 **시험 포인트**:
> - EMR = Hadoop 클러스터 관리형 서비스 (Spark, Hive 포함)
> - Task 노드 = Spot 인스턴스로 비용 절감 (데이터 저장 안 함)
> - Core 노드 = On-Demand 유지 (데이터 저장)
> - Glue vs EMR: Glue = 서버리스 ETL, EMR = 완전 제어 대규모 처리

---

## 5. Amazon QuickSight

### 5.1 개요

- 서버리스 **ML 기반 BI(Business Intelligence)** 대시보드 서비스
- 빠른 속도, 자동 스케일링, 임베드 가능
- **SPICE 엔진**: 인메모리 연산 (QuickSight에 데이터 임포트 시)

### 5.2 통합 데이터 소스

```
RDS, Aurora, Athena, Redshift, S3
OpenSearch, Timestream, IoT Analytics
```

### 5.3 주요 기능

| 기능 | 설명 |
|---|---|
| **SPICE** | 인메모리 연산 엔진으로 빠른 쿼리 |
| **CLS(Column-Level Security)** | Enterprise Edition에서 컬럼 수준 보안 |
| **사용자/그룹** | Standard = 사용자, Enterprise = 그룹 |
| **대시보드 공유** | 분석(Analysis) 스냅샷을 퍼블리싱 |

### 5.4 사용 사례

- 비즈니스 분석, 시각화, 임시(Ad-hoc) 분석, 데이터 인사이트

> 💡 **시험 포인트**:
> - QuickSight = 서버리스 BI 대시보드 (Tableau의 AWS 버전)
> - SPICE = 인메모리 캐싱으로 빠른 대시보드 응답
> - CLS(Column-Level Security) = Enterprise Edition 전용
> - Athena + QuickSight = 서버리스 분석 + 시각화 조합

---

## 6. AWS Glue

### 6.1 개요

- 관리형 **ETL(Extract, Transform, Load)** 서비스
- 분석을 위한 데이터 준비/변환에 활용
- 완전 **서버리스(Serverless)**

### 6.2 핵심 기능

| 기능 | 설명 |
|---|---|
| **Glue Data Catalog** | 데이터셋 메타데이터 카탈로그; Glue Crawlers가 자동 채움 |
| **Glue Job Bookmarks** | 이미 처리된 데이터 재처리 방지 |
| **Glue Elastic Views** | SQL로 다수 데이터 스토어를 결합/복제 (가상 테이블) |
| **Glue DataBrew** | 코드 없이 데이터 정제 및 정규화 |
| **Glue Studio** | 시각적 ETL 작업 생성 UI |
| **Glue Streaming ETL** | 스트림 처리 (Kinesis, Kafka, MSK) |

### 6.3 Glue Data Catalog 역할

```
Glue Crawlers가 S3, RDS 등에서 스키마 자동 탐색
    ↓
Glue Data Catalog에 메타데이터 저장
    ↓
Athena, EMR, Redshift Spectrum이 스키마 검색(Discovery)에 활용
```

### 6.4 ETL 도구 비교

| 도구 | 특징 | 적합한 상황 |
|---|---|---|
| **Glue** | 서버리스, 관리형 ETL | 일반 ETL 파이프라인 |
| **EMR** | 완전 클러스터 제어 | 대규모 복잡 변환 |
| **Lambda** | 실시간/단순 변환 | 소규모 이벤트 기반 |

> 💡 **시험 포인트**:
> - Glue = 서버리스 ETL, Data Catalog = Athena/EMR/Redshift Spectrum에서 사용
> - Glue Crawlers = 자동 스키마 검색 및 카탈로그 등록
> - S3 데이터를 Parquet으로 변환할 때 Glue ETL 사용
> - Glue Job Bookmarks = 중복 처리 방지

---

## 7. AWS Lake Formation

### 7.1 개요

- **데이터 레이크(Data Lake)** = 분석 목적의 모든 데이터를 한 곳에 통합
- Lake Formation: 데이터 레이크를 **수개월이 아닌 수일 만에** 구축 가능한 완전 관리형 서비스
- 데이터 탐색, 정제, 변환, 수집 자동화
- **AWS Glue** 위에 구축됨

### 7.2 주요 기능

- 복잡한 수동 작업 자동화 (수집, 정제, 이동, 카탈로깅)
- S3, RDS, 관계형/NoSQL DB용 **블루프린트(Blueprints)** 제공
- **세분화된 접근 제어 (Fine-grained Access Control)**: 행(Row) 및 열(Column) 수준 보안

### 7.3 중앙화된 권한 관리

**Lake Formation 이전:**

```
S3 접근 → IAM 정책
Glue 접근 → IAM 정책
Athena 접근 → IAM 정책
QuickSight 접근 → IAM 정책
(각 서비스별 개별 관리)
```

**Lake Formation 도입 후:**

```
모든 데이터 레이크 접근 → Lake Formation 단일 지점에서 관리
(Row/Column 수준 세분화 제어 가능)
```

> 💡 **시험 포인트**:
> - Lake Formation = 데이터 레이크 + 중앙화된 권한 관리 (행/열 수준)
> - Glue 위에 구축: Lake Formation이 Glue를 사용해 ETL 수행
> - "여러 서비스의 데이터 접근을 한 곳에서 관리" → Lake Formation
> - Fine-grained Access Control = 행/열 수준 보안 → Lake Formation의 핵심 차별점

---

## 8. Kinesis Data Analytics

### 8.1 SQL 애플리케이션용

- **Kinesis Data Streams** 및 **Kinesis Firehose**에서 SQL로 실시간 분석
- S3의 참조 데이터(Reference Data)를 추가해 스트리밍 데이터 보강
- 완전 관리형, 자동 스케일링, 소비율 기반 과금
- 출력: **Kinesis Data Streams, Firehose**

### 8.2 Apache Flink용

- Java, Scala, SQL로 스트리밍 데이터 처리/분석
- 관리형 클러스터에서 Apache Flink 앱 실행
- 소스: **Kinesis Data Streams, Amazon MSK**
- **Firehose에서 직접 읽기 불가** (SQL 애플리케이션 사용)
- SQL 애플리케이션보다 강력, **상태 기반 연산(Stateful computations)** 지원

### 8.3 SQL vs Flink 비교

| | SQL 애플리케이션 | Apache Flink |
|---|---|---|
| **언어** | SQL | Java, Scala, SQL |
| **소스** | Kinesis Streams + Firehose | Kinesis Streams + MSK |
| **복잡성** | 단순 | 복잡, 상태 기반 |
| **서버 관리** | 완전 서버리스 | 관리형 클러스터 |

> 💡 **시험 포인트**:
> - Flink = Firehose에서 직접 읽기 불가 (Kinesis Streams 또는 MSK에서만)
> - SQL Application = Kinesis Streams + Firehose 모두 소스로 사용 가능
> - 복잡한 스트림 처리, 상태 기반 연산 → Flink 선택
> - Kinesis Data Analytics = 완전 관리형, 서버 프로비저닝 불필요

---

## 9. Amazon MSK

### 9.1 개요

- **MSK = Managed Streaming for Apache Kafka**
- Kinesis의 대안; AWS에서의 Kafka
- 완전 관리형 Apache Kafka 클러스터
- Kafka 장애 자동 복구, 클러스터 생성/업데이트/삭제 관리
- **EBS 볼륨**으로 데이터 영속 저장
- **MSK Serverless**: 용량 관리 없이 Kafka 실행

### 9.2 Kinesis Data Streams vs Amazon MSK 비교

| 항목 | Kinesis Data Streams | Amazon MSK |
|---|---|---|
| **메시지 크기** | 1MB | 기본 1MB (최대 10MB 설정 가능) |
| **스트림 확장** | 샤드 분할/병합 | Kafka 파티션 추가만 가능 (삭제 불가) |
| **전송 중 보안** | TLS in-flight | Plaintext 또는 TLS in-flight |
| **저장 중 보안** | KMS at-rest | KMS at-rest |
| **주요 소비자** | Kinesis SDK, Lambda, Firehose, Analytics | Kafka consumers, Lambda, Glue, Flink, IoT Analytics |

### 9.3 MSK 사용 사례

- 기존 Kafka 워크로드의 AWS 마이그레이션
- 높은 처리량, 낮은 지연 시간의 이벤트 스트리밍
- Flink와 통합한 복잡한 스트림 처리

> 💡 **시험 포인트**:
> - MSK = Kafka 호환, Kinesis = AWS 고유 스트리밍 (둘 다 스트리밍 서비스)
> - MSK는 파티션 추가만 가능 (축소 불가) vs Kinesis는 샤드 분할/병합 가능
> - MSK 메시지 크기 = 기본 1MB, 최대 10MB까지 설정 가능
> - "Kafka 마이그레이션" = MSK 정답

---

## 10. Big Data Ingestion Pipeline (서버리스 통합 예시)

### 10.1 아키텍처 패턴

서버리스 빅데이터 수집 파이프라인의 대표적인 구성:

```
IoT 디바이스
    ↓
AWS IoT Core (디바이스 메시지 수집)
    ↓
Amazon Kinesis Data Streams (실시간 스트리밍)
    ↓
Amazon Kinesis Firehose (데이터 전달)
    ↓
Amazon S3 (원시 데이터 수집 버킷)
    ↓
Amazon SQS (알림 큐)
    ↓
AWS Lambda (트리거)
    ↓
Amazon Athena (S3 데이터 SQL 쿼리)
    ↓
Amazon S3 (리포팅 버킷)
    ↓
Amazon QuickSight (시각화 대시보드)
또는
Amazon Redshift (추가 분석)
```

### 10.2 각 컴포넌트 역할

| 서비스 | 역할 |
|---|---|
| **IoT Core** | IoT 디바이스 데이터 수집 |
| **Kinesis Data Streams** | 실시간 데이터 스트리밍 |
| **Kinesis Firehose** | S3로 데이터 전달 (버퍼링) |
| **S3 (수집 버킷)** | 원시 데이터 저장소 |
| **SQS + Lambda** | 이벤트 기반 처리 트리거 |
| **Athena** | 서버리스 SQL 쿼리 |
| **S3 (리포팅 버킷)** | 처리된 결과 저장 |
| **QuickSight/Redshift** | 시각화 또는 심층 분석 |

> 💡 **시험 포인트**:
> - 전체 파이프라인이 서버리스: IoT Core + Kinesis + S3 + Lambda + Athena + QuickSight
> - Firehose = S3 직접 전달, Data Streams = 실시간 소비자 필요
> - SQS → Lambda = 이벤트 기반 아키텍처의 표준 패턴
> - Athena 결과를 S3에 저장 후 QuickSight로 시각화하는 흐름 암기

---

## 데이터 & 분석 서비스 요약

| 서비스 | 타입 | 핵심 특징 | 대표 키워드 |
|---|---|---|---|
| **Athena** | 서버리스 쿼리 | S3 SQL, TB당 $5 | 서버리스 SQL, 로그 분석 |
| **Redshift** | 데이터 웨어하우스 | OLAP, MPP, 컬럼 기반 | 데이터 웨어하우스, OLAP |
| **OpenSearch** | 검색/분석 | 전문 검색, 부분 일치 | 전문 검색, Elasticsearch |
| **EMR** | 빅데이터 클러스터 | Hadoop/Spark | 빅데이터, Hadoop |
| **QuickSight** | BI 대시보드 | 서버리스, SPICE | BI, 대시보드 |
| **Glue** | ETL | 서버리스 ETL, Data Catalog | ETL, 데이터 변환 |
| **Lake Formation** | 데이터 레이크 | 중앙화된 권한, Row/Column 보안 | 데이터 레이크 |
| **Kinesis Analytics** | 스트림 분석 | SQL/Flink 실시간 분석 | 실시간 스트림 분석 |
| **MSK** | 관리형 Kafka | Kafka 호환 | Kafka, 이벤트 스트리밍 |

---

← [Section 21. AWS의 데이터베이스](section21.md) | [Section 23. 머신 러닝](section23.md) →
