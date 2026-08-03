# Section 29. 더 많은 솔루션 아키텍처

## 목차
1. [이벤트 처리 패턴](#1-이벤트-처리-패턴)
2. [고성능 컴퓨팅 (HPC)](#2-고성능-컴퓨팅-hpc)
3. [고가용성 및 고성능 아키텍처](#3-고가용성-및-고성능-아키텍처)

---

## 1. 이벤트 처리 패턴

### Lambda, SNS, SQS 조합

| 패턴 | 동작 방식 | 실패 처리 |
|---|---|---|
| **SQS + Lambda** | Lambda가 SQS를 폴링 | 실패 시 메시지 SQS 복귀 → maxReceives 초과 → **DLQ** |
| **SQS FIFO + Lambda** | 메시지 그룹별 Lambda 스케일링 | 정확히 1번 처리 (exactly-once) |
| **SNS + Lambda** | 비동기 호출 | 실패 시 3회 재시도 → **DLQ (Lambda 측)** |
| **Fan-out** | SNS → 여러 SQS 큐 | 분리(decouple) + 병렬 처리 |

> 💡 **시험 포인트**:
> - SQS + Lambda 실패 시 DLQ는 **SQS** 쪽에 설정
> - SNS + Lambda 실패 시 DLQ는 **Lambda** 쪽에 설정
> - Fan-out 패턴: SNS 1개 → SQS 여러 개로 메시지 복제/분산 처리
> - SQS FIFO는 메시지 그룹 ID 기반 순서 보장 + 중복 제거

---

### S3 이벤트 알림 (S3 Event Notifications)

**지원 이벤트 유형:**
- `S3:ObjectCreated` — 객체 생성
- `S3:ObjectRemoved` — 객체 삭제
- `S3:ObjectRestore` — Glacier 복원
- `S3:Replication` — 복제 실패/완료

**전송 대상:**
- **SNS** — 팬아웃, 이메일 알림
- **SQS** — 큐잉, 비동기 처리
- **Lambda** — 즉각 처리
- **EventBridge** — 고급 라우팅 (추천)

**주요 특성:**
- 일반적으로 수 초 내 전달 (최대 1분 이상 걸릴 수 있음)
- **버전 관리(Versioning) 활성화** 필요 → 모든 이벤트 전달 보장
- **EventBridge**: 고급 필터링, 여러 대상, **아카이브/재생(replay)** 기능

> 💡 **시험 포인트**:
> - 모든 S3 이벤트를 확실히 캡처하려면 **Versioning 활성화** 필수
> - EventBridge는 SNS/SQS/Lambda 외에도 Step Functions, Kinesis 등 18개+ 대상 지원
> - 기존 SNS/SQS/Lambda 방식 대비 EventBridge가 더 유연 (필터링, 아카이브)

---

### EventBridge – API 호출 인터셉트

**동작 흐름:**
```
API 호출 → CloudTrail 기록 → EventBridge 규칙 매칭 → 알림/액션
```

**활용 예시:**
- DynamoDB 테이블 삭제 감지 → SNS 알림 발송
- IAM 정책 변경 감지 → Slack 알림
- EC2 인스턴스 종료 감지 → 자동 복구 트리거

> 💡 **시험 포인트**:
> - **CloudTrail + EventBridge** 조합은 모든 AWS API 호출 모니터링의 핵심
> - 실시간 보안 알림 아키텍처 단골 출제 패턴

---

### API Gateway – 데이터 순서 보장

**Kinesis Data Streams 활용:**
- **파티션 키(Partition Key)** 설정: 동일 사용자 요청 → 동일 샤드 → **순서 보장**
- 예: `user_id`를 파티션 키로 사용
- 데이터 재생(replay) 가능

```
API Gateway → Kinesis Data Streams (partition_key=user_id) → Lambda/Consumer
                        ↓
              동일 user_id → 동일 샤드 → 순서 보장
```

---

## 2. 고성능 컴퓨팅 (HPC)

### HPC 활용 사례

클라우드 기반 HPC를 활용하는 분야:
- **유전체학(Genomics)** — DNA 서열 분석
- **전산 화학(Computational Chemistry)** — 분자 시뮬레이션
- **금융 리스크 모델링** — 몬테카를로 시뮬레이션
- **기상 예측** — 수치 기상 모델
- **머신러닝/딥러닝** — 대규모 모델 학습
- **자율주행** — 시뮬레이션 및 학습

---

### 데이터 관리 및 전송

| 서비스 | 용도 | 특징 |
|---|---|---|
| **Direct Connect** | GB/TB 데이터 클라우드 전송 | 전용 네트워크, 안정적 대역폭 |
| **Snowball/Snowmobile** | PB 규모 대용량 데이터 이전 | 물리적 장치 |
| **DataSync** | 온프레미스 → S3/EFS/FSx | 자동화, 스케줄링 |

---

### 컴퓨팅 및 네트워킹

**인스턴스 유형:**
- GPU 최적화: `p3`, `p4`, `g4` 계열 (딥러닝, 영상처리)
- CPU 최적화: 고클럭 범용 컴퓨팅
- **Spot Instances** / **Spot Fleets**: 비용 절감, 자동 스케일링

**배치 그룹 (Placement Groups):**
- **클러스터(Cluster)**: 동일 AZ + 동일 랙 → **최저 지연/최고 대역폭** → HPC 최적
- Spread: 고가용성 우선
- Partition: Hadoop/Kafka 분산 처리

**향상된 네트워킹 (Enhanced Networking / SR-IOV):**
- 더 높은 대역폭, 더 높은 PPS, 더 낮은 지연

| 어댑터 | 최대 속도 | 비고 |
|---|---|---|
| **ENA** (Elastic Network Adapter) | **100 Gbps** | 최신 권장 |
| Intel VF Interface | 10 Gbps | 레거시 |

**Elastic Fabric Adapter (EFA):**
- ENA의 개선 버전, **HPC 전용**
- **Linux 전용** (Windows 미지원)
- 노드 간 통신(inter-node communication) 최적화
- **MPI(Message Passing Interface)** 표준 활용
- **OS 커널 우회** → 초저지연 달성

> 💡 **시험 포인트**:
> - HPC 네트워킹 = **EFA + Cluster Placement Group** 조합
> - EFA는 **Linux만** 지원
> - ENA는 일반 향상 네트워킹; EFA는 HPC 특화 (OS bypass + MPI)
> - 클러스터 배치 그룹: 낮은 지연 + 높은 대역폭, 단 단일 AZ

---

### 스토리지

**인스턴스 연결 스토리지:**

| 스토리지 | 최대 IOPS | 특징 |
|---|---|---|
| **EBS io2 Block Express** | 256,000 IOPS | 영구 스토리지 |
| **Instance Store** | 수백만 IOPS | EC2 생명주기와 연결; 임시 |

**네트워크 스토리지:**
- **S3**: 객체 스토리지, 무제한
- **EFS**: 공유 파일 시스템 (NFS), Multi-AZ
- **Amazon FSx for Lustre**: HPC 특화 고성능 파일 시스템; S3 연동

---

### 자동화 및 오케스트레이션

**AWS Batch:**
- **멀티노드 병렬 작업** 지원
- EC2 / Spot 인스턴스 자동 스케줄링
- 사용 사례: ML 학습, EDA(전자 설계 자동화), 유전체학

**AWS ParallelCluster:**
- 오픈소스 HPC 클러스터 관리 도구
- **텍스트 파일(설정 파일)** 로 클러스터 구성
- VPC, 서브넷, 클러스터 타입, 인스턴스 타입 자동 생성
- **EFA 지원** → 고성능 노드 간 통신

> 💡 **시험 포인트**:
> - FSx for Lustre: HPC + S3 직접 연동 → 빠른 데이터 입출력
> - AWS Batch: 장기 배치 작업 (Lambda의 15분 제한 없음)
> - ParallelCluster: 텍스트 파일 기반 HPC 클러스터 자동 프로비저닝

---

## 3. 고가용성 및 고성능 아키텍처

### EC2 인스턴스 고가용성 패턴

**Elastic IP 활용:**
```
Primary EC2 (실패) → Elastic IP 재할당 → Standby EC2
```
- 스크립트/Lambda로 자동 IP 재할당

**Auto Scaling Group 활용:**
```
ASG (min=1, max=1, desired=1)
  └─ 인스턴스 실패 시 자동으로 새 인스턴스 시작
  └─ User Data로 애플리케이션 자동 구성/시작
```

**Route 53 DNS 장애 조치:**
- Health Check + DNS Failover → Active/Passive 구성
- 주 인스턴스 장애 시 대기 인스턴스로 자동 전환

---

### 고성능 아키텍처 패턴

**웹 계층 (Stateless):**
- **ELB + ASG** → 수평 확장
- 세션 데이터: **ElastiCache** (Redis) 또는 **DynamoDB**
- 정적 콘텐츠: **CloudFront + S3**

**데이터베이스 계층:**
- **RDS Multi-AZ** 또는 **Aurora** → 고가용성
- **읽기 복제본(Read Replicas)** → 읽기 트래픽 분산
- **ElastiCache / DAX** → DB 앞단 캐싱

**캐싱 계층별 서비스:**

| 계층 | 서비스 | 용도 |
|---|---|---|
| 엣지 (Edge) | **CloudFront** | 정적 콘텐츠, 글로벌 배포 |
| 앱 계층 | **ElastiCache / DAX** | 동적 쿼리 결과 캐싱 |
| DB 계층 | **RDS Read Replicas** | DB 읽기 트래픽 분산 |

**비동기 처리 패턴:**
- **SQS / SNS / Kinesis** → 컴포넌트 분리(decoupling)
- **Lambda / ECS** → 이벤트 기반 처리
- 버스트 트래픽 흡수: SQS 큐 버퍼링 → 백엔드 보호

> 💡 **시험 포인트**:
> - Stateless 웹 계층 = ELB + ASG + ElastiCache/DynamoDB (세션)
> - 고가용성 DB = RDS Multi-AZ (자동 페일오버) + Read Replicas (읽기 확장)
> - 비동기/분리 아키텍처 = SQS + Lambda/ECS 조합
> - 글로벌 HA = Route 53 + CloudFront + Multi-Region 배포

---

← [Section 28. 재해 복구 및 마이그레이션](section28.md) | [Section 30. 기타 서비스](section30.md) →
