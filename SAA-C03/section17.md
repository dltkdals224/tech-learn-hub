# Section 17. 디커플링 애플리케이션: SQS, SNS, Kinesis, Active MQ

## 목차

1. [SQS 개요](#1-sqs-개요)
2. [SQS 고급 기능](#2-sqs-고급-기능)
3. [SQS 보안](#3-sqs-보안)
4. [Amazon SNS](#4-amazon-sns)
5. [SNS + SQS Fan Out](#5-sns--sqs-fan-out)
6. [Amazon Kinesis](#6-amazon-kinesis)
7. [Amazon MQ](#7-amazon-mq)

---

## 1. SQS 개요

### Amazon SQS – Standard Queue

- **Oldest offering** (10년+), fully managed, serverless, 애플리케이션 **디커플링**에 활용
- **무제한 처리량**, 큐 내 메시지 수 제한 없음
- **기본 보존 기간**: 4일, 최대 14일
- **낮은 지연**: 게시/수신 < 10ms
- **메시지 크기**: 최대 256KB
- **At-least-once delivery** (중복 메시지 가능)
- **Best-effort ordering** (순서 보장 없음)

### Producing Messages

- Producer가 **PutMessage** API로 SQS에 메시지 전송
- 메시지는 Consumer가 삭제할 때까지 큐에 유지

### Consuming Messages

- Consumer (EC2, Lambda, ECS): SQS **폴링** (최대 10개 메시지 동시 수신)
- 메시지 처리 후 **DeleteMessage API**로 삭제
- 다수의 Consumer가 서로 다른 메시지를 동시에 수신 가능

### SQS with Auto Scaling Group

- EC2 인스턴스가 ASG로 구성되어 SQS 폴링
- **CloudWatch Metric**: Queue Length (`ApproximateNumberOfMessages`)
- CloudWatch Alarm → 큐 깊이 기반 **ASG 스케일링** 트리거

### 디커플링 활용 사례

| 구성 요소 | 역할 |
|-----------|------|
| 프론트엔드 ASG | 비디오 업로드 → S3 저장 + SQS 메시지 전송 |
| 백엔드 ASG | SQS 폴링 → 비디오 처리 → S3 저장 |
| CloudWatch | 큐 깊이 모니터링 → 백엔드 ASG 자동 스케일링 |

> 💡 **시험 포인트**:
> - SQS는 **at-least-once delivery** → 중복 처리 가능성 항상 고려
> - 메시지 크기 최대 **256KB** 초과 시 SQS Extended Client (S3 활용) 필요
> - `ApproximateNumberOfMessages` 지표로 ASG 스케일링 연동 가능
> - Standard Queue는 순서 보장 없음; 순서 필요 시 **FIFO Queue** 사용

---

## 2. SQS 고급 기능

### Message Visibility Timeout

- Consumer가 메시지를 폴링하면 **다른 Consumer에게 비가시** 처리 (기본 30초)
- Consumer는 타임아웃 내에 처리 + 삭제 완료해야 함
- 타임아웃 내 삭제 안 되면 → 메시지 **다시 가시화** (중복 처리 발생)
- **ChangeMessageVisibility API**로 타임아웃 연장 가능
- 너무 길면 → 재처리 지연; 너무 짧으면 → 중복 발생

### Long Polling

- 큐가 비어 있을 때 메시지 도착 대기 (API 호출 횟수 감소)
- **WaitTimeSeconds**: 1~20초
- Short Polling 대비 **권장 방식**
- 큐 레벨 또는 API 레벨(`ReceiveMessage WaitTimeSeconds`)에서 설정 가능

### FIFO Queue

- **First In First Out** (순서 보장 큐)
- **Exactly-once processing** (중복 제거)
- 처리량 제한:
  - 배칭 없음: **300 msg/s**
  - 배칭 사용: **3,000 msg/s**
- 중복 제거 방식:
  - **Content-based**: SHA-256 해싱
  - **ID-based**: 5분 간격 중복 제거 ID
- **Message Groups**: 같은 그룹 → 순서 보장; 다른 그룹 → 병렬 소비 가능
- 큐 이름은 반드시 **`.fifo`** 로 종료

### Dead Letter Queue (DLQ)

- Consumer가 메시지 처리 실패 시 큐로 반환
- **MaximumReceives** 임계값 초과 → **DLQ**로 이동
- Standard 큐의 DLQ는 Standard; FIFO 큐의 DLQ는 **FIFO**
- 디버깅 용도로 활용
- **DLQ Redrive**: 문제 해결 후 DLQ에서 소스 큐로 메시지 재이동
- DLQ 보존 기간: **14일** 권장

### Delay Queue

- 메시지를 최대 **15분** 지연 (기본값 0)
- 메시지별 오버라이드 가능 (`DelaySeconds` 파라미터)

### SQS Extended Client

- 256KB 초과 메시지 처리 → **SQS Extended Client Library (Java)** 활용
- 실제 페이로드는 **S3**에 저장; SQS에는 메타데이터 메시지만 전송

> 💡 **시험 포인트**:
> - **Visibility Timeout** 내 처리 실패 → 중복 처리; `ChangeMessageVisibility`로 연장
> - **Long Polling** (WaitTimeSeconds 1~20s) → 비용 절감 및 지연 감소
> - FIFO Queue 이름은 반드시 **`.fifo`** 접미사 필요
> - DLQ에서 재처리 가능한 **Redrive** 기능 기억

---

## 3. SQS 보안

### 암호화

| 유형 | 방식 |
|------|------|
| 전송 중 암호화 | **HTTPS** |
| 저장 중 암호화 | **KMS keys** |
| 클라이언트 측 | 직접 암호화/복호화 |

### 접근 제어

- **IAM 정책**: SQS API 접근 규제
- **SQS Access Policies** (S3 버킷 정책과 유사):
  - **Cross-account** SQS 접근
  - SNS, S3 등 타 서비스의 SQS 쓰기 허용

> 💡 **시험 포인트**:
> - Cross-account SQS 접근은 **SQS Access Policy** 필요 (IAM만으로 불충분)
> - SNS → SQS 쓰기 허용도 **SQS Access Policy**로 설정
> - KMS 암호화 사용 시 CMK 키 정책도 함께 관리 필요

---

## 4. Amazon SNS

- **하나의 메시지를 다수의 수신자에게** 전달 (pub/sub 모델)
- **Event Producer** → SNS Topic 발행 → 모든 **Subscriber** 수신
- 토픽당 최대 **12,500,000개** 구독
- 최대 **100,000개** 토픽

### Subscribers (구독자 유형)

- **SQS**, HTTP/HTTPS, **Lambda**, **Kinesis Firehose**, Emails, SMS/Mobile Notifications

### Publishing (발행 방법)

| 방법 | 절차 |
|------|------|
| Topic Publish (SDK) | 토픽 생성 → 구독 생성 → 발행 |
| Direct Publish (모바일 SDK) | 플랫폼 앱 생성 → 플랫폼 엔드포인트 생성 → 발행 |

### 보안

- HTTPS 전송 중 암호화
- KMS 저장 중 암호화
- **IAM 정책** + **SNS Access Policies**

### SNS FIFO

- **Message Group ID**로 순서 보장, 중복 제거
- 구독자는 **SQS FIFO 큐만** 가능
- 처리량 제한 있음

### SNS Message Filtering

- JSON 정책으로 구독별 메시지 **필터링**
- 필터 없음 → 모든 메시지 수신

> 💡 **시험 포인트**:
> - SNS는 **push** 방식; SQS는 **pull(폴링)** 방식
> - SNS FIFO의 구독자는 **SQS FIFO만** 가능
> - **Message Filtering**으로 구독자별 수신 메시지 분류 가능
> - SNS 자체는 메시지를 **저장하지 않음** (전달 실패 시 유실 가능)

---

## 5. SNS + SQS Fan Out

### 문제 및 해결

- **문제**: 하나의 이벤트를 여러 SQS 큐에 전달
- **해결**: **SNS Topic** → 여러 SQS 큐가 구독

### 특징

- **완전 디커플링**, 데이터 유실 없음
- SQS: 데이터 영속성, 지연 처리, 재시도 가능
- 추후 SQS 큐 추가 용이
- **Cross-region delivery**: 다른 리전의 SQS에도 전달 가능

### 활용 사례

| 사례 | 구성 |
|------|------|
| S3 Events → 다수 SQS | S3 Event → SNS → 여러 SQS (S3는 이벤트 유형+접두사당 하나의 알림만 허용) |
| SNS → S3 (Firehose 경유) | SNS → **Kinesis Data Firehose** → S3 (또는 KDF 지원 대상) |

> 💡 **시험 포인트**:
> - S3 이벤트를 **여러 대상**에 전달할 때 → **SNS Fan Out 패턴** 활용
> - S3는 동일 이벤트 유형+접두사 조합에 **단일 EventNotification**만 지원
> - SNS → Kinesis Data Firehose → S3/Redshift 연결 가능

---

## 6. Amazon Kinesis

실시간 스트리밍 데이터를 대규모로 수집·처리·저장하는 플랫폼

### Kinesis Data Streams

- 데이터 스트림 수집, 처리, 저장
- **샤드(Shard)**: 사용 전 프로비저닝 필요 (1~N개)
- 샤드당: 입력 **1 MB/s**, 출력 **2 MB/s**
- **보존 기간**: 기본 24시간, 최대 365일
- **데이터 리플레이(재처리)** 가능
- **불변 데이터** (삭제 불가)
- 샤드 내 레코드 **순서 보장**

| 역할 | 도구 |
|------|------|
| Producer | AWS SDK, **Kinesis Producer Library (KPL)**, Kinesis Agent |
| Consumer | **Kinesis Consumer Library (KCL)**, SDK; 관리형: Lambda, Kinesis Data Firehose, Kinesis Data Analytics |

### 용량 모드 비교

| 항목 | Provisioned (프로비전) | On-Demand |
|------|----------------------|-----------|
| 샤드 관리 | 수동 | 자동 |
| 처리량 | 프로비전된 샤드 기준 | 기본 4개 샤드; 최고 처리량에 따라 자동 조정 |
| 비용 | 프로비전된 샤드 시간 과금 | 스트림 시간 + 데이터 in/out GB 과금 |

### Kinesis Data Firehose

- 데이터 스트림을 **데이터 저장소에 로드** (완전 관리형, 서버리스)
- 지원 목적지:
  - **S3**, **Redshift** (S3 COPY 경유), **OpenSearch**
  - 3rd Party: Datadog, Splunk
  - HTTP 엔드포인트
- **Near real-time** (버퍼: 1~900초, 또는 1MB 이상)
- 자동 스케일링
- **데이터 저장 없음**, 리플레이 불가
- 지원 형식: CSV, JSON, Parquet, ORC, 압축

### Kinesis Data Streams vs Firehose 비교

| 항목 | Data Streams | Firehose |
|------|-------------|---------|
| 용도 | 스트리밍 수집 | 스트림 로드 (데이터 저장소) |
| 관리 | 직접 프로그래밍 | 완전 관리형 |
| 리플레이 | 가능 (1~365일) | 불가 |
| 지연 | 실시간 (~200ms) | Near real-time |

### Kinesis Data Analytics (for SQL)

- Kinesis 스트림에 **SQL로 실시간 분석**
- 관리형, 자동 스케일링
- **소비량 기반 과금**
- 출력: Kinesis Data Streams, Kinesis Data Firehose

### Kinesis Video Streams

- 연결 디바이스로부터 **비디오 스트림** 수집·처리·저장

### SQS vs Kinesis 순서 보장 비교

| 서비스 | 순서 보장 방식 |
|--------|--------------|
| Kinesis | **Partition Key** → 동일 키 → 동일 샤드 → 샤드 내 순서 보장 |
| SQS Standard | 순서 보장 없음 |
| SQS FIFO (GroupID 없음) | Consumer 1개, 전체 순서 보장 |
| SQS FIFO (GroupID 사용) | GroupID별 순서 보장, 병렬 Consumer 가능 |

> 💡 **시험 포인트**:
> - Kinesis Data Streams: **리플레이 가능**, 데이터 **불변**, 샤드 내 순서 보장
> - Firehose: **Near real-time**, 리플레이 **불가**, 완전 관리형
> - 동일 Partition Key → **동일 샤드** → 순서 보장 (예: 사용자 ID를 Partition Key로)
> - Firehose는 **데이터 저장소** 적재 목적; Streams는 **실시간 처리** 목적

---

## 7. Amazon MQ

- 온프레미스 앱의 **개방형 프로토콜** 마이그레이션용 (MQTT, AMQP, STOMP, Openwire, WSS)
- **Amazon MQ** = 관리형 **Apache ActiveMQ** 또는 **RabbitMQ**
- SQS/SNS 대비 **스케일링 제한** (프로비전된 용량)
- 서버에서 실행, **Multi-AZ 장애 조치** 가능
- **큐와 토픽 기능** 모두 지원

### 선택 기준

| 상황 | 선택 |
|------|------|
| 신규 애플리케이션 | **SQS / SNS** |
| 기존 MQ 프로토콜 사용 앱 마이그레이션 | **Amazon MQ** |
| 재설계 없이 마이그레이션 원할 때 | **Amazon MQ** |

> 💡 **시험 포인트**:
> - Amazon MQ는 **기존 온프레미스 메시징 시스템 마이그레이션** 시나리오에서 선택
> - MQTT, AMQP 등 프로토콜 언급 → **Amazon MQ** 선택 신호
> - SQS/SNS만큼 **확장성이 높지 않음** (provisioned)
> - **Multi-AZ 장애 조치** 지원으로 고가용성 구성 가능

---

← [Section 16. AWS 스토리지 추가 기능](section16.md) | [Section 18. AWS의 컨테이너: ECS, Fargate, ECR 및 EKS](section18.md) →
