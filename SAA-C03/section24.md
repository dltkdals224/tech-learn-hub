# Section 24. AWS 모니터링 및 감사: CloudWatch, CloudTrail 및 Config

## 목차

1. [Amazon CloudWatch Metrics](#1-amazon-cloudwatch-metrics)
2. [CloudWatch Logs](#2-cloudwatch-logs)
3. [CloudWatch Alarms](#3-cloudwatch-alarms)
4. [CloudWatch Events / Amazon EventBridge](#4-cloudwatch-events--amazon-eventbridge)
5. [CloudWatch Insights](#5-cloudwatch-insights)
6. [AWS CloudTrail](#6-aws-cloudtrail)
7. [AWS Config](#7-aws-config)

---

## 1. Amazon CloudWatch Metrics

### 1.1 개요

- AWS의 **모든 서비스에 대한 지표(Metrics)** 제공
- **Metric**: 모니터링할 변수 (CPU Utilization, NetworkIn 등)
- 메트릭은 **네임스페이스(Namespaces)** 에 속함; 메트릭 당 최대 **30개 차원(Dimensions)**
- 타임스탬프(Timestamps) 필수; **대시보드(Dashboards)** 제공

### 1.2 EC2 모니터링

| 항목 | 기본 모니터링 | 세부 모니터링 |
|---|---|---|
| 수집 주기 | **5분** | **1분** (추가 비용) |
| 활성화 | 기본 제공 | 별도 활성화 필요 |

**기본 제공 EC2 메트릭:**
- CPU 사용률, 네트워크 인/아웃, 디스크 읽기/쓰기
- **RAM 사용률은 기본 제공 안 됨** → 커스텀 메트릭 필요

### 1.3 커스텀 메트릭 (Custom Metrics)

- 직접 메트릭 푸시 가능: RAM, 디스크, 커스텀 앱 지표
- **표준 해상도(Standard Resolution)**: 1분
- **고해상도(High Resolution)**: **1초** (StorageResolution=1)

> 💡 **시험 포인트**:
> - EC2 기본 메트릭에 **RAM 포함 안 됨** → CloudWatch Agent + 커스텀 메트릭 필요
> - 세부 모니터링(Detailed Monitoring) = 1분 간격, 추가 비용
> - 커스텀 메트릭 고해상도 = 최소 1초 단위
> - 메트릭은 네임스페이스 단위로 분리됨

---

## 2. CloudWatch Logs

### 2.1 구조

- **Log Groups**: 애플리케이션을 나타내는 이름
- **Log Streams**: 앱 인스턴스, 로그 파일, 컨테이너 단위 스트림
- **만료 정책(Expiration Policy)**: 만료 없음 ~ 1일 ~ 10년 설정 가능

### 2.2 로그 목적지 (Destinations)

| 목적지 | 방식 |
|---|---|
| **S3** | 배치 내보내기(Batch Export) |
| **Kinesis Data Streams** | 실시간 스트리밍 |
| **Kinesis Firehose** | 실시간 스트리밍 |
| **Lambda** | 실시간 처리 |
| **OpenSearch** | 실시간 검색/분석 |

- 기본 **암호화(Encrypted)**, **KMS** 키 사용 가능

### 2.3 로그 소스 (Log Sources)

- SDK, **CloudWatch Agent**, **CloudWatch Unified Agent**
- Elastic Beanstalk, ECS(컨테이너), **Lambda (자동)**
- API Gateway, **CloudTrail (필터링)**, Route 53 (DNS 쿼리)

### 2.4 CloudWatch Logs Insights

- 로그 데이터 검색 및 분석; 내장 쿼리 제공
- 여러 계정/로그 그룹 동시 쿼리
- **실시간 아님 (Not Real-time)**

### 2.5 Metric Filters (메트릭 필터)

- CloudWatch Logs에 필터 표현식 적용 (예: 로그에서 "ERROR" 탐색)
- 로그 필터 → **메트릭 생성** → **CloudWatch 알람 생성** 연계 가능

### 2.6 Subscription Filters (구독 필터)

- **Kinesis Data Streams, Firehose, Lambda**로 실시간 피드 전달
- **크로스 계정(Cross-Account)**: 구독 필터 → Kinesis Data Streams (대상 계정)

### 2.7 CloudWatch Agent

| 에이전트 | 수집 대상 |
|---|---|
| **CloudWatch Logs Agent** (구버전) | 로그만 |
| **CloudWatch Unified Agent** (신버전) | 시스템 레벨 메트릭(CPU, 디스크, RAM, netstat, 프로세스) + 로그 |

- EC2 기본 메트릭에 **RAM/디스크 미포함** → Unified Agent 설치 필요
- EC2 및 **온프레미스** 서버 모두 지원

> 💡 **시험 포인트**:
> - RAM/Disk 지표 수집 → **CloudWatch Unified Agent** 설치 필요
> - S3 내보내기는 배치(Batch) 방식 → 실시간 아님
> - Logs Insights = 쿼리 분석 도구, 실시간 아님
> - 구독 필터 = 실시간 스트리밍 (Kinesis/Lambda)

---

## 3. CloudWatch Alarms

### 3.1 개요

- 모든 메트릭 기반으로 **알림(Notifications) 트리거**
- **알람 상태**:
  - `OK`: 정상
  - `INSUFFICIENT_DATA`: 데이터 부족
  - `ALARM`: 임계값 초과
- **Period**: 메트릭 평가 기간 (10초, 30초, 60초의 배수)

### 3.2 알람 타겟 (Alarm Targets)

| 타겟 | 동작 |
|---|---|
| **EC2 액션** | 중지(Stop), 종료(Terminate), 재부팅(Reboot), 복구(Recover) |
| **EC2 Auto Scaling** | 스케일 인/아웃 |
| **SNS** | 알림 → Lambda, SQS, 이메일 등 연계 |

### 3.3 Composite Alarms (복합 알람)

- 여러 알람 상태를 **AND/OR 조건**으로 모니터링
- **알람 노이즈(Alarm Noise) 감소** 효과

### 3.4 EC2 인스턴스 복구 (Instance Recovery)

- `StatusCheckFailed_System` 알람 트리거 → EC2 Recover
- 복구 후 유지: **동일 프라이빗/퍼블릭/Elastic IP, 메타데이터, 배치 그룹**
- **전용 호스트(Dedicated Hosts)** 에 적합

### 3.5 기타

- **Billing Alarms**: **us-east-1** 리전에서만 생성 가능; 전체 비용 기준
- **CloudWatch Logs Metric Filters에서 알람 생성** 가능

> 💡 **시험 포인트**:
> - Billing Alarm은 **반드시 us-east-1**에서만 생성
> - Composite Alarms = AND/OR 조건으로 알람 노이즈 감소
> - EC2 Recovery = StatusCheckFailed_System 알람, IP/메타데이터 유지
> - Logs Metric Filter → CloudWatch Alarm 연계 패턴 기억

---

## 4. CloudWatch Events / Amazon EventBridge

### 4.1 Amazon EventBridge 개요

- **CloudWatch Events의 발전된 버전(Evolution)**

### 4.2 이벤트 버스 (Event Buses)

| 버스 종류 | 설명 |
|---|---|
| **Default Event Bus** | AWS 서비스 이벤트 |
| **Partner Event Bus** | 외부 파트너 이벤트 (Zendesk, DataDog, Segment, Auth0) |
| **Custom Event Bus** | 사용자 정의 애플리케이션 이벤트 |

### 4.3 Schema Registry (스키마 레지스트리)

- EventBridge가 버스의 이벤트 분석 → **스키마 자동 추론(Infer Schema)**
- 언어별 **코드 바인딩(Code Bindings)** 제공
- 스키마 **버전 관리(Versioning)** 지원

### 4.4 리소스 기반 정책 (Resource-based Policy)

- 특정 이벤트 버스에 대한 권한 관리
- 다른 AWS 계정/리전의 이벤트 **허용/거부**
- 모든 이벤트를 **단일 AWS 계정/리전으로 집계** 가능

### 4.5 주요 기능

- **이벤트 패턴 매칭(Event Pattern Matching)**
- **크론 표현식(Cron Expressions)** 으로 스케줄링
- **이벤트 아카이브(Archive)** → **이벤트 재생(Replay)**: 디버깅, 재처리
- **콘텐츠 필터링(Content Filtering)**

> 💡 **시험 포인트**:
> - EventBridge = CloudWatch Events의 발전형, 파트너/커스텀 버스 추가
> - 이벤트 아카이브 + 재생 기능 = 디버깅/재처리에 활용
> - Schema Registry = 이벤트 구조 자동 추론 + 코드 바인딩
> - 크로스 계정 이벤트 집계는 리소스 기반 정책으로 제어

---

## 5. CloudWatch Insights

### 5.1 CloudWatch Container Insights

- **컨테이너**에서 메트릭 + 로그 수집, 집계, 요약
- 지원 대상: **ECS, EKS, EC2의 Kubernetes, Fargate**
- Kubernetes 환경에서는 **컨테이너화된 CloudWatch Agent** 사용

### 5.2 CloudWatch Lambda Insights

- Lambda 함수 모니터링 및 트러블슈팅
- 시스템 레벨 메트릭 수집: CPU 시간, 메모리, 디스크, 네트워크
- 진단 정보: **콜드 스타트(Cold Starts)**, Lambda 워커 셧다운
- **Lambda Layer**로 제공

### 5.3 CloudWatch Contributor Insights

- 로그 데이터 분석 → 시계열 생성 → **상위 N 기여자(Top-N Contributors)** 표시
- **Heavy Hitters** 탐지 (시스템에 영향을 주는 주요 발생원)
- 활용: **VPC 로그**, **DNS 로그** 분석
- AWS 생성 모든 로그와 호환

### 5.4 CloudWatch Application Insights

- 모니터링 대상 애플리케이션 + 관련 AWS 서비스의 **자동화된 대시보드(Automated Dashboards)**
- 지원 애플리케이션: Java, .NET, 데이터베이스(RDS, DynamoDB, S3), ECS, Lambda, SNS, SQS, API GW, Kinesis
- **Amazon SageMaker** 기반으로 동작
- **MTTR(Mean Time To Repair) 단축**

> 💡 **시험 포인트**:
> - Container Insights = ECS/EKS/Fargate/Kubernetes 컨테이너 모니터링
> - Lambda Insights = Lambda Layer 형태 제공, 콜드 스타트 진단
> - Contributor Insights = Top-N Heavy Hitter 탐지 (VPC/DNS 로그)
> - Application Insights = SageMaker 기반 자동 대시보드, MTTR 감소

---

## 6. AWS CloudTrail

### 6.1 개요

- AWS 계정의 **거버넌스(Governance), 컴플라이언스(Compliance), 감사(Audit)** 제공
- **기본 활성화(Enabled by default)**
- 콘솔, SDK, CLI, AWS 서비스를 통한 이벤트/API 호출 이력 기록
- 로그를 **CloudWatch Logs** 또는 **S3**로 전송 가능
- 기본: **전체 리전(All Regions)** 적용 (단일 리전 옵션 가능)
- **리소스가 삭제되면 → CloudTrail 먼저 확인**

### 6.2 이벤트 유형

| 이벤트 유형 | 기본 로깅 | 설명 |
|---|---|---|
| **Management Events** | O (기본) | AWS 리소스 작업 (읽기/쓰기 분리 가능) |
| **Data Events** | X (기본 아님) | S3 객체 수준 작업, Lambda 호출 (대용량) |
| **Insights Events** | X (별도 활성화) | 비정상 활동 감지 (Write 이벤트 기준선 → 이상 감지) |

**Insights Events 흐름:**
```
Write Management Events 기준선 분석 → 이상 감지
→ CloudTrail / S3 / CloudWatch Events 전송
```

### 6.3 보존 기간 및 장기 분석

- CloudTrail 이벤트 기본 보존: **90일**
- 장기 보존 → **S3 저장 + Athena 쿼리 분석**

### 6.4 CloudTrail Lake

- CloudTrail 이벤트를 위한 **관리형 데이터 레이크(Managed Data Lake)**
- 여러 계정/리전에 걸쳐 쿼리 및 분석 가능
- 보존 기간: 기본 **7년**, 최대 **7년** (구성 가능)

> 💡 **시험 포인트**:
> - CloudTrail = 기본 활성화, API 호출 감사 (누가, 언제, 무엇을)
> - Data Events(S3 객체 수준, Lambda) = **기본 로깅 안 됨**, 별도 활성화
> - Insights Events = 비정상 활동 자동 탐지
> - 90일 초과 보존 → **S3 + Athena** 조합

---

## 7. AWS Config

### 7.1 개요

- AWS 리소스의 **컴플라이언스 감사 및 기록(Auditing and Recording Compliance)**
- 시간에 따른 **구성 변경 이력(Configuration Changes)** 기록
- **리전 단위 서비스** (여러 리전/계정으로 집계 가능)
- 데이터는 **S3 저장**, **Athena 쿼리** 분석

**답할 수 있는 질문 예시:**
- "보안 그룹에 무제한 SSH 접근이 있는가?"
- "S3 버킷에 퍼블릭 접근이 허용되는가?"
- "ALB 구성이 언제 어떻게 변경되었는가?"

### 7.2 Config Rules (구성 규칙)

- **AWS 관리형 규칙(Managed Rules)**: 75개 이상 사전 정의
- **커스텀 규칙(Custom Rules)**: Lambda 함수로 정의
- 평가 시점: **구성 변경 시** 또는 **주기적 간격(Periodic)**
- **변경 방지(Deny) 기능 없음** → 감사(Audit) 및 평가(Assess)만 수행

### 7.3 Config Remediation (자동 교정)

- **SSM Automation Documents** 를 사용하여 비준수 리소스 **자동 교정(Auto-Remediation)**
- 교정 후에도 비준수 시 **Remediation Retries** 지원

### 7.4 Config Notifications (알림)

- **EventBridge**: 비준수 리소스에 대한 알림 트리거
- **SNS**: 모든 구성 변경 + 컴플라이언스 상태 알림

### 7.5 CloudWatch vs CloudTrail vs Config 비교

| 항목 | CloudWatch | CloudTrail | Config |
|---|---|---|---|
| **목적** | 성능 지표, 로그, 알람 | API 호출 감사 | 리소스 구성 변경 추적 |
| **활용 예시** | CPU 80% 알람 | DeleteSecurityGroup 호출 기록 | SG 규칙 변경 이력 |
| **알람/교정** | Yes (알람) | No | Yes (SSM 자동화) |
| **기본 활성화** | Yes | Yes | 별도 활성화 |

> 💡 **시험 포인트**:
> - Config = **변경 이력 추적 + 컴플라이언스 평가** (변경 방지 불가)
> - 비준수 자동 교정 → **SSM Automation Documents**
> - "언제 구성이 바뀌었나?" → Config / "누가 API 호출했나?" → CloudTrail
> - Config는 리전 단위, 크로스 리전/계정 집계 가능

---

← [Section 23. 머신 러닝](section23.md) | [Section 25. Identity and Access Management (IAM) - 고급](section25.md) →
