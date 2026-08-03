# Section 32. 시험 준비 + 연습 시험 - AWS Certified SAA

## 목차
1. [시험 개요](#1-시험-개요)
2. [시험 전략](#2-시험-전략)
3. [핵심 암기 포인트](#3-핵심-암기-포인트)
4. [자주 출제되는 패턴](#4-자주-출제되는-패턴)

---

## 1. 시험 개요

- **시험 코드**: SAA-C03 (AWS Certified Solutions Architect – Associate)
- **문항 수**: 65문제
- **시험 시간**: 130분
- **문제 유형**:
  - Multiple Choice (단일 정답 선택)
  - Multiple Response (복수 정답 선택, 보통 2~3개)
- **합격 점수**: 약 **720 / 1000**
- **유효 기간**: 3년

### 4개 도메인 및 출제 비율

| 도메인 | 내용 | 비율 |
|---|---|---|
| **Domain 1** | 탄력적 아키텍처 설계 | **30%** |
| **Domain 2** | 고성능 아키텍처 설계 | **28%** |
| **Domain 3** | 안전한 애플리케이션 및 아키텍처 설계 | **24%** |
| **Domain 4** | 비용 최적화 아키텍처 설계 | **18%** |

> 💡 **시험 포인트**:
> - Domain 1 (탄력성) + Domain 2 (고성능) = 전체의 **58%** → 최우선 학습
> - Domain 3 (보안) = 24% → IAM, KMS, VPC 보안 집중
> - Domain 4 (비용) = 18% → 구매 옵션, 스토리지 라이프사이클 집중

---

## 2. 시험 전략

### 문제 풀이 접근법

1. **전체 문제 먼저 읽기** — 답 보기 전에 질문 완전히 이해
2. **핵심 키워드 식별**:
   - `"most cost-effective"` → 비용 최적화 답 선택
   - `"highest availability"` → Multi-AZ, 멀티 리전
   - `"least operational overhead"` → 관리형/서버리스 서비스
   - `"serverless"` → Lambda, Fargate, Aurora Serverless, DynamoDB
3. **명백히 틀린 보기 먼저 제거** (보통 2개는 빠르게 제거 가능)
4. **남은 2개에서 AWS 권장 접근법** 선택

### AWS가 선호하는 답변 패턴

- **관리형 서비스** 우선 (운영 부담 최소화)
- **서버리스** 우선 (확장성, 가용성 자동)
- **단일 장애점(SPOF) 제거** (Multi-AZ, 다중화)
- **비용 효율적** 솔루션

### 주요 키워드별 정답 패턴

| 키워드 | 정답 방향 |
|---|---|
| **비용 절감** | Spot > Reserved > On-Demand; 서버리스가 가변 워크로드에 최저 비용 |
| **고가용성** | Multi-AZ, 멀티 리전, Auto Scaling, ELB |
| **성능 향상** | ElastiCache/DAX (캐시), CloudFront (정적), Read Replicas (DB 읽기) |
| **보안 강화** | IAM 최소 권한, KMS, Secrets Manager, WAF, Shield |
| **운영 단순화** | CloudFormation, 관리형 서비스, 서버리스 |

---

## 3. 핵심 암기 포인트

### 스토리지 서비스 비교

| 서비스 | 타입 | 주요 특징 | 시험 포인트 |
|---|---|---|---|
| **EBS** | Block | 단일 AZ; gp2/gp3/io1/io2/st1/sc1 | EC2에 연결; AZ 내 복제 |
| **EFS** | File (NFS) | Multi-AZ; 서버리스; 사용량 과금 | 공유 파일 시스템; Linux |
| **FSx for Windows** | File (SMB) | Windows 네이티브; AD 통합 | Windows EC2 공유 스토리지 |
| **FSx for Lustre** | File | HPC; ML; S3 직접 연동 | 고성능 병렬 파일 시스템 |
| **S3** | Object | 무제한; 최대 5TB/객체; 11 9s 내구성 | 정적 웹, 백업, 데이터 레이크 |
| **Instance Store** | Block | 임시(Ephemeral); 최고 IOPS | EC2 재시작 시 데이터 소실 |

---

### 데이터베이스 서비스 선택 가이드

| 요구 패턴 | 선택 서비스 |
|---|---|
| SQL / OLTP | **RDS** / **Aurora** |
| NoSQL 키-값 | **DynamoDB** |
| NoSQL 도큐먼트 (MongoDB 호환) | **DocumentDB** |
| 인메모리 캐시 | **ElastiCache** (Redis/Memcached) |
| 그래프 관계 데이터 | **Neptune** |
| 시계열 데이터 | **Timestream** |
| OLAP / 데이터 웨어하우스 | **Redshift** |
| 변경 불가 원장(Ledger) | **QLDB** |

> 💡 **시험 포인트**:
> - Aurora = RDS 호환 + 최대 5배 성능 + 자동 스토리지 확장
> - DynamoDB = 완전 서버리스 NoSQL; DAX로 마이크로초 캐싱
> - ElastiCache Redis vs Memcached: Redis = 영속성, 복제; Memcached = 단순 캐시, 멀티스레드

---

### "Least Operational Overhead" 패턴

관리형/서버리스 서비스 우선 선택:
- **컴퓨팅**: Lambda, **Fargate** (컨테이너 서버리스)
- **데이터베이스**: Aurora Serverless, **DynamoDB**
- **스트리밍**: Kinesis Data Firehose (완전 관리형)
- **IaC**: CloudFormation (인프라 자동화)
- **스케일링**: Auto Scaling (자동 용량 관리)

---

### "High Availability" 패턴

| 레벨 | 서비스 | 구성 |
|---|---|---|
| **인스턴스** | EC2 | ASG min=1 + Multi-AZ |
| **데이터베이스** | RDS Multi-AZ / Aurora | 자동 페일오버 |
| **캐시** | ElastiCache Multi-AZ | 자동 페일오버 |
| **네트워크** | NAT Gateway | AZ별 별도 구성 |
| **DNS** | Route 53 | Health Check + Failover |
| **글로벌** | Route 53 + CloudFront | 멀티 리전 |

**글로벌 HA 아키텍처:**
- Route 53 **지리적 위치(Geolocation)** / **지연 시간(Latency)** 라우팅
- **DynamoDB Global Tables** — 멀티 리전 복제
- **Aurora Global Database** — 1개 주 리전 + 최대 5개 읽기 전용 리전

---

### "Security" 패턴

| 요구사항 | 서비스 |
|---|---|
| **저장 시 암호화** | **KMS** |
| **전송 중 암호화** | HTTPS / TLS (ACM 인증서) |
| **최소 권한 접근** | **IAM 역할(Roles)** |
| **시크릿/자격 증명** | **Secrets Manager** (자동 교체) / SSM Parameter Store |
| **인스턴스 수준 방화벽** | **보안 그룹(SG)** |
| **서브넷 수준 방화벽** | **NACL** |
| **L7 WAF** | **AWS WAF** |
| **DDoS 보호** | **Shield Standard** (무료) / **Shield Advanced** |

> 💡 **시험 포인트**:
> - Secrets Manager vs SSM Parameter Store: SM=자동 교체, SSM=무료 기본
> - SG는 Stateful (응답 자동 허용); NACL은 Stateless (인/아웃 모두 명시)
> - WAF는 CloudFront, ALB, API Gateway 앞에 배치 가능

---

## 4. 자주 출제되는 패턴

### 비용 최적화 핵심 전략

- **Spot Instances**: 내결함성 배치 작업 / Stateless 워크로드
- **Reserved Instances / Savings Plans**: 안정적 워크로드 (1년 or 3년)
- **S3 Lifecycle 정책**: 오래된 데이터 → S3 Standard-IA → S3 Glacier
- **Rightsizing**: AWS Compute Optimizer 권장 사항 활용
- **미사용 리소스 삭제**: EBS 스냅샷, 미연결 EIP, 유휴 EC2

---

### 잘 틀리는 문제 유형 총정리

**DNS / Route 53:**
- `CNAME` vs `Alias`: **루트 도메인**(Zone Apex)에는 반드시 **Alias** 사용 (CNAME 불가)
- Alias는 ELB, CloudFront, S3, API GW 등 AWS 리소스만 가리킬 수 있음

**S3:**
- S3 복제(Replication): 소스/대상 버킷 모두 **버전 관리 활성화** 필수
- 복제는 **신규 객체만** 해당 → 기존 객체는 **S3 Batch Replication** 사용
- 교차 리전 복제(CRR) vs 동일 리전 복제(SRR) 구분

**SQS:**
- **SQS FIFO** vs **SQS Standard**: FIFO = 순서 보장 + 중복 제거 (초당 300TPS 제한)
- Visibility Timeout: 처리 중 메시지가 다시 보이지 않는 시간

**CloudWatch / 모니터링:**
- **EC2 RAM 사용량**: 기본 CloudWatch 지표 **아님** → CloudWatch 에이전트 설치 + 커스텀 지표 필요
- EC2 기본 지표: CPU, Network I/O, Disk I/O

**네트워크:**
- **NAT Gateway**: **퍼블릭 서브넷**에 위치, **Elastic IP** 연결 (프라이빗 서브넷 아님)
- **EFS vs EBS**: EFS = 공유 멀티 AZ NFS; EBS = 단일 AZ 블록 스토리지

**DynamoDB:**
- **DynamoDB Global Tables** 사전 조건: **DynamoDB Streams 활성화** 필수

**Lambda / VPC:**
- Lambda가 VPC 내에 있을 때 **인터넷 접근** → **NAT Gateway** 필요
- Lambda가 VPC 내에서 AWS 서비스 접근 → **VPC Endpoints** (프라이빗, 비용 효율적)

**CloudFront:**
- CloudFront 앞에 WAF를 배치하면 CloudFront가 WAF를 우회할 수 없음
- **오리진 액세스 제어(OAC)**: S3 버킷을 CloudFront 통해서만 접근 가능하도록 제한

> 💡 **시험 포인트**:
> - "루트 도메인 → AWS 리소스 연결" = **Route 53 Alias 레코드**
> - "기존 S3 객체 다른 리전으로 복사" = **S3 Batch Replication**
> - "EC2 메모리 모니터링" = CloudWatch Agent + 커스텀 지표
> - "프라이빗 서브넷 EC2 → 인터넷" = NAT Gateway (퍼블릭 서브넷)

---

← [Section 31. 백서 및 아키텍처 - AWS 공인 솔루션스 아키텍트 어소시에이트](section31.md) | [Section 33. 완강을 축하합니다! - AWS Certified Solutions Architect Associate](section33.md) →
