# Section 9. AWS 기초: RDS + Aurora + ElastiCache

## 목차
1. [Amazon RDS 개요](#1-amazon-rds-개요)
2. [RDS Storage Auto Scaling](#2-rds-storage-auto-scaling)
3. [RDS Read Replicas](#3-rds-read-replicas)
4. [RDS Multi AZ (재해 복구)](#4-rds-multi-az-재해-복구)
5. [RDS Custom](#5-rds-custom)
6. [RDS 백업](#6-rds-백업)
7. [Amazon Aurora](#7-amazon-aurora)
8. [Aurora 고급 기능](#8-aurora-고급-기능)
9. [Aurora 백업 및 복원](#9-aurora-백업-및-복원)
10. [RDS & Aurora 보안](#10-rds--aurora-보안)
11. [Amazon RDS Proxy](#11-amazon-rds-proxy)
12. [Amazon ElastiCache](#12-amazon-elasticache)

---

## 1. Amazon RDS 개요

**RDS(Relational Database Service)**는 SQL을 쿼리 언어로 사용하는 데이터베이스를 위한 완전 관리형 서비스다.

### 지원 엔진

- PostgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- IBM DB2
- **Aurora** (AWS 독점 데이터베이스)

### EC2에 직접 DB를 설치하는 것 대비 RDS의 장점

| 기능 | 설명 |
|------|------|
| 자동 프로비저닝, OS 패치 | AWS가 처리 |
| 지속적 백업 + Point in Time Restore | 특정 시점으로 복원 가능 |
| 모니터링 대시보드 | 기본 제공 |
| Read Replicas | 읽기 성능 향상 |
| Multi AZ 설정 | 재해 복구(DR) |
| 유지보수 윈도우 | 업그레이드 스케줄 관리 |
| 스케일링 (수직/수평) | 용이한 확장 |
| EBS 기반 스토리지 | gp2 또는 io1 |

**단점:** RDS 인스턴스에 SSH 접속 **불가** (RDS Custom 제외)

> 💡 **시험 포인트**:
> - RDS는 **완전 관리형** → SSH 접속 불가
> - Point in Time Restore는 최근 **5분 전**까지 복원 가능 (트랜잭션 로그 5분마다 백업)

---

## 2. RDS Storage Auto Scaling

**RDS Storage Auto Scaling**은 DB 스토리지가 부족해지면 자동으로 스토리지를 늘려준다.

**동작 조건 (3가지 모두 만족 시 자동 확장):**
1. 여유 스토리지가 할당 스토리지의 **10% 미만**
2. Low-storage 상태가 **5분 이상** 지속
3. 마지막 수정으로부터 **6시간 경과**

**설정 필요:**
- **Maximum Storage Threshold** (최대 스토리지 한도) 지정 필수
- 예측 불가능한 워크로드에 유용
- 모든 RDS 엔진 지원

> 💡 **시험 포인트**:
> - Storage Auto Scaling은 **Maximum Storage Threshold** 설정 필수
> - 수동으로 스토리지를 늘리는 작업 없이 자동 처리

---

## 3. RDS Read Replicas

**Read Replica**는 읽기 트래픽을 분산하기 위해 주 DB의 복제본을 만드는 기능이다.

**주요 특성:**
- 최대 **15개** Read Replica 생성 가능
- **Same AZ, Cross AZ, Cross Region** 모두 가능
- 복제 방식: **ASYNC** (비동기) → Eventual Consistency (최종 일관성)
- Replica를 독립적인 DB로 **승격(Promote)** 가능
- 애플리케이션이 Read Replica를 사용하려면 **커넥션 문자열 업데이트** 필요

### Read Replica 사용 사례

운영 DB에 분석/리포팅 쿼리를 실행해야 할 때 Read Replica를 생성하여 분석 애플리케이션이 Replica에서 읽도록 함 → 운영 DB 영향 없음

- Read Replica는 **SELECT(읽기)만** 가능 (INSERT, UPDATE, DELETE 불가)

### Read Replica 네트워크 비용

| 상황 | 비용 |
|------|------|
| **같은 리전** 내 다른 AZ | **무료** |
| **Cross-Region** | 데이터 전송 **요금 발생** |

> 💡 **시험 포인트**:
> - Read Replica = **ASYNC** 복제 → Eventual Consistency
> - Multi-AZ = **SYNC** 복제 → 강한 일관성
> - 같은 리전 내 AZ 간 Read Replica 데이터 전송 = **무료**
> - Cross-Region Read Replica = **추가 비용**

---

## 4. RDS Multi AZ (재해 복구)

**RDS Multi AZ**는 재해 복구(DR)를 위한 기능이다. Standby 인스턴스를 다른 AZ에 유지한다.

**주요 특성:**
- 복제 방식: **SYNC** (동기)
- **하나의 DNS 이름** → 자동 페일오버 (앱 코드 변경 없음)
- AZ 장애, 네트워크 장애, 인스턴스/스토리지 장애 시 자동 전환
- **스케일링 목적으로는 사용 불가** (Standby는 읽기/쓰기 불가)
- Read Replica를 Multi AZ로 구성하여 DR도 함께 확보 가능

### Single-AZ → Multi-AZ 전환

**Zero Downtime** 전환 가능 (DB 중지 불필요):
1. 스냅샷 생성
2. 새 AZ에서 스냅샷으로 Standby DB 복원
3. 두 DB 간 동기화 설정

> 💡 **시험 포인트**:
> - Multi AZ = **DR 목적**, **SYNC** 복제, **Zero Downtime** 전환
> - Multi AZ Standby는 **읽기/쓰기 불가** → 스케일링 목적 사용 불가
> - "고가용성" 시나리오 → Multi AZ / "읽기 성능 향상" → Read Replica

---

## 5. RDS Custom

**RDS Custom**은 Oracle과 Microsoft SQL Server DB에서 OS 및 DB에 대한 직접 접근이 가능한 관리형 서비스다.

| 구분 | RDS | RDS Custom |
|------|-----|------------|
| OS 접근 | 불가 | 가능 (SSH / SSM) |
| DB 커스터마이징 | 제한적 | 패치, 설정, 네이티브 기능 활성화 가능 |
| 자동화 | 전체 | 커스터마이징 시 Automation Mode 비활성화 필요 |
| 지원 엔진 | 다수 | Oracle, MS SQL Server만 |

**주의:** 커스터마이징 전 반드시 **DB 스냅샷 생성** 권장

> 💡 **시험 포인트**:
> - RDS Custom = Oracle, MS SQL Server + **OS 수준 접근** 필요 시
> - 커스터마이징 시 **Automation Mode 비활성화** 후 진행

---

## 6. RDS 백업

### 자동 백업 (Automated Backups)

- 백업 윈도우 기간 중 **매일 전체 백업**
- 트랜잭션 로그: **5분마다** 백업
- → 최근 5분 전까지 **Point in Time 복원** 가능
- 보관 기간: **1~35일** (0으로 설정 시 자동 백업 비활성화)

### 수동 DB 스냅샷

- 사용자가 직접 트리거
- **원하는 기간만큼 무제한** 보관 가능

**팁:** RDS를 장기간 중지해도 스토리지 비용 계속 발생 → 오랫동안 사용 안 할 경우 스냅샷 후 삭제 권장

> 💡 **시험 포인트**:
> - 자동 백업 최대 보관: **35일**
> - 수동 스냅샷: **무제한** 보관
> - Aurora 자동 백업은 **비활성화 불가** (RDS는 0으로 설정 시 비활성화 가능)

---

## 7. Amazon Aurora

**Aurora**는 AWS가 개발한 독점 데이터베이스로, MySQL과 PostgreSQL 드라이버와 호환된다.

### Aurora 주요 특징

| 항목 | 내용 |
|------|------|
| 호환성 | MySQL(5x 성능 향상), PostgreSQL(3x 성능 향상) |
| 스토리지 | 10GB 단위로 자동 확장, 최대 **256 TB** |
| Read Replica | 최대 **15개**, MySQL보다 빠른 복제 (sub 10ms 지연) |
| 페일오버 | **30초 미만** 자동 페일오버 |
| 고가용성 | 기본 내장 (HA native) |
| 비용 | RDS보다 약 **20% 비쌈**, 하지만 효율적 |

### Aurora 고가용성 및 읽기 스케일링

**3개 AZ에 6개 데이터 복사본 유지:**
- 쓰기: 6개 중 **4개** 필요
- 읽기: 6개 중 **3개** 필요
- **Self-healing**: Peer-to-peer 복제로 데이터 자가 복구
- 스토리지가 수백 개 볼륨에 **스트라이핑**되어 분산

**Aurora DB Cluster 구조:**
- **Writer Endpoint**: 마스터를 가리킴 (쓰기 전용)
- **Reader Endpoint**: Connection Load Balancing으로 모든 Read Replica에 분산

> 💡 **시험 포인트**:
> - Aurora는 **6개 복사본 / 3개 AZ** → 자가 복구
> - 페일오버 **30초 이내** (RDS보다 빠름)
> - Writer Endpoint → 마스터, Reader Endpoint → Read Replica 로드밸런싱

---

## 8. Aurora 고급 기능

### Aurora Replicas Auto Scaling

Read 트래픽이 증가하면 Reader Endpoint가 자동으로 확장된 Replica까지 포함하도록 업데이트된다.

### Aurora Custom Endpoints

특정 Aurora 인스턴스들을 묶어 Custom Endpoint를 정의할 수 있다.

- 예: 고성능 인스턴스(db.r5.2xlarge)를 분석 쿼리 전용 Endpoint로 지정
- Custom Endpoint 정의 후에는 일반 Reader Endpoint를 거의 사용하지 않게 됨

### Aurora Serverless

- **자동 DB 인스턴스화 및 Auto-Scaling** (실제 사용량 기반)
- 간헐적, 예측 불가능한 워크로드에 적합
- 용량 계획 불필요
- **초당 과금** (비용 효율적)
- Proxy Fleet이 클라이언트와 Aurora 인스턴스 사이를 관리

### Global Aurora

| 옵션 | 설명 |
|------|------|
| **Cross Region Read Replicas** | 재해 복구 + 간단한 설정 |
| **Aurora Global Database** (권장) | 1개 Primary 리전(읽기/쓰기) + 최대 10개 Secondary 리전(읽기 전용) |

**Aurora Global Database 특징:**
- Secondary 리전 복제 지연: **1초 미만**
- 리전당 최대 **16개** Read Replica
- 다른 리전을 DR 리전으로 승격: RTO **1분 미만**
- 전형적인 Cross-Region 복제: **1초 미만**

### Aurora Machine Learning

SQL 쿼리를 통해 ML 기반 예측을 애플리케이션에 통합할 수 있다.

- 지원 서비스: **Amazon SageMaker** (모든 ML 모델), **Amazon Comprehend** (감성 분석)
- ML 전문 지식 불필요
- 활용 사례: 사기 탐지, 광고 타겟팅, 감성 분석, 상품 추천

### Babelfish for Aurora PostgreSQL

Aurora PostgreSQL이 MS SQL Server 명령(T-SQL)을 이해할 수 있게 해주는 기능. SQL Server 기반 앱을 Aurora PostgreSQL로 마이그레이션 시 코드 변경 최소화 가능.

> 💡 **시험 포인트**:
> - Aurora Global Database: Secondary 리전 복제 지연 **< 1초**, DR 승격 RTO **< 1분**
> - Aurora Serverless: 간헐적/예측 불가능 워크로드 → 초당 과금
> - Custom Endpoint: 특정 인스턴스 그룹에 분석 쿼리 분리

---

## 9. Aurora 백업 및 복원

### Aurora 백업

| 유형 | 보관 기간 | 비활성화 |
|------|----------|---------|
| 자동 백업 | 1~35일 | **불가** |
| 수동 스냅샷 | 무제한 | - |

### RDS & Aurora 복원 옵션

- 백업/스냅샷 복원 시 **새 DB가 생성**됨

**MySQL RDS를 S3에서 복원:**
1. 온프레미스 DB 백업 생성
2. Amazon S3에 저장
3. S3 백업에서 새 RDS 인스턴스 복원

**MySQL Aurora 클러스터를 S3에서 복원:**
1. **Percona XtraBackup**으로 온프레미스 DB 백업
2. S3에 저장
3. S3에서 새 Aurora 클러스터 복원

### Aurora Database Cloning

기존 Aurora DB 클러스터에서 새 클러스터를 빠르게 복제.

- **Copy-on-Write 프로토콜** 사용: 초기에는 동일한 데이터 볼륨 공유 → 변경 시에만 새 스토리지 할당
- 스냅샷 & 복원보다 빠르고 비용 효율적
- 프로덕션 DB로부터 스테이징 DB를 만들 때 유용 (프로덕션 영향 없음)

> 💡 **시험 포인트**:
> - Aurora 자동 백업은 **비활성화 불가** (RDS는 가능)
> - Aurora Cloning = **Copy-on-Write**, 스냅샷보다 빠름
> - S3에서 MySQL Aurora 복원 시 **Percona XtraBackup** 사용

---

## 10. RDS & Aurora 보안

| 보안 항목 | 설명 |
|---------|------|
| **저장 데이터 암호화** | KMS 사용, 시작 시점에 정의 / 마스터 미암호화 시 Replica도 암호화 불가 / 암호화하려면 스냅샷 → 암호화 복원 |
| **전송 중 암호화** | TLS 기본 지원, AWS TLS 루트 인증서 사용 |
| **IAM 인증** | 사용자명/비밀번호 대신 IAM 역할로 DB 연결 |
| **보안 그룹** | 네트워크 접근 제어 |
| **SSH 접속** | **불가** (RDS Custom만 가능) |
| **감사 로그** | 활성화 가능, CloudWatch Logs로 전송하여 장기 보존 |

> 💡 **시험 포인트**:
> - RDS/Aurora는 기본적으로 **SSH 불가** (RDS Custom 예외)
> - 비암호화 DB 암호화: **스냅샷 → 암호화 복원** 경로 사용
> - IAM Authentication: 비밀번호 없이 IAM 역할로 DB 접속 가능

---

## 11. Amazon RDS Proxy

**RDS Proxy**는 RDS/Aurora를 위한 완전 관리형 DB 프록시다.

**주요 기능:**
- 앱이 DB 연결을 **풀링(pooling)** 하고 공유하게 함
- DB 리소스(CPU, RAM) 부하 감소, 오픈 연결 최소화
- **Serverless, Auto-scaling, Multi-AZ** (고가용성)
- RDS & Aurora 페일오버 시간을 최대 **66%** 단축
- 코드 변경 없이 대부분의 앱에서 사용 가능
- **IAM 인증 강제** 가능, 자격 증명은 **AWS Secrets Manager**에 저장
- **VPC 내에서만 접근 가능** (퍼블릭 인터넷 접근 불가)

**지원 엔진:**
- RDS: MySQL, PostgreSQL, MariaDB, MS SQL Server
- Aurora: MySQL, PostgreSQL

**주요 사용 사례:** Lambda 함수가 RDS에 연결할 때 Connection Pooling으로 연결 폭증 방지

> 💡 **시험 포인트**:
> - RDS Proxy는 **Lambda + RDS** 조합에서 연결 폭증 문제 해결책
> - 페일오버 시간 **66% 감소**
> - **퍼블릭 액세스 불가** → VPC 내에서만 사용 가능

---

## 12. Amazon ElastiCache

**ElastiCache**는 관리형 **Redis** 또는 **Memcached** 인메모리 데이터베이스 서비스다. 초고성능, 낮은 지연시간의 캐시를 제공한다.

**특징:**
- 읽기 집약적 워크로드의 DB 부하 감소
- 애플리케이션을 Stateless로 만들 수 있음 (세션 데이터 저장)
- AWS가 OS 유지보수, 패치, 최적화, 설정, 모니터링, 장애 복구, 백업 처리
- **애플리케이션 코드 변경이 필요** (캐시를 활용하려면 코드 수정 필수)

### 솔루션 아키텍처 패턴

**DB Cache 패턴:**
```
앱 → ElastiCache 조회 → Cache Hit: 바로 반환
                     → Cache Miss: RDS에서 조회 → ElastiCache에 저장 → 반환
```
- Cache Invalidation 전략 필요 (오래된 데이터 방지)

**User Session Store 패턴:**
- 사용자가 로그인 → 세션 데이터를 ElastiCache에 저장
- 다른 인스턴스로 요청이 가도 ElastiCache에서 세션 조회 → 로그인 상태 유지

### Redis vs Memcached 비교

| 항목 | Redis | Memcached |
|------|-------|-----------|
| Multi-AZ | ✅ Auto-Failover | ❌ |
| Read Replicas | ✅ (읽기 스케일링 + HA) | ❌ |
| 데이터 내구성 | ✅ AOF persistence | ❌ (비영구적) |
| 백업/복원 | ✅ | Serverless만 지원 |
| 자료구조 | Sorted Sets, Sets 등 | 단순 Key-Value |
| 멀티스레드 | ❌ | ✅ (고처리량 sharding) |
| Sharding | ❌ | ✅ Multi-node sharding |

### ElastiCache 보안

| 항목 | 설명 |
|------|------|
| **IAM 인증** | Redis만 지원, AWS API 수준 보안 |
| **Redis AUTH** | 클러스터 생성 시 password/token 설정 가능, SSL in-flight 암호화 지원 |
| **Memcached** | SASL 기반 인증 지원 |

### ElastiCache 캐싱 패턴

| 패턴 | 설명 | 특징 |
|------|------|------|
| **Lazy Loading** | 읽은 데이터만 캐시 | 캐시가 오래될 수 있음 (stale data) |
| **Write Through** | DB 쓰기 시 캐시도 업데이트 | 최신 데이터 유지, 쓰기 비용 증가 |
| **Session Store** | TTL 기능으로 임시 세션 데이터 저장 | Stateless 앱 구현 |

### Redis Use Case: Gaming Leaderboards

- **Redis Sorted Sets**: 고유성(uniqueness) + 원소 순서(ordering) 보장
- 새 원소 추가 시 실시간 순위 계산 후 올바른 순서로 삽입
- 게임 리더보드와 같이 실시간 랭킹이 필요한 경우에 최적

> 💡 **시험 포인트**:
> - ElastiCache는 **애플리케이션 코드 변경 필수**
> - Redis: **HA, Persistence, Sorted Sets** → 세션, 리더보드, 고가용성 캐시
> - Memcached: **Sharding, 멀티스레드** → 단순 대용량 캐시
> - Lazy Loading은 Cache Miss 시 DB 조회 → 첫 요청 느림 (cold start)
> - Write Through는 항상 최신 데이터 → 쓰기 두 번 (DB + Cache)

---

> **이전 섹션**: [Section 8. 고가용성 및 스케일링성: ELB 및 ASG](section8.md)
> **다음 섹션**: [Section 10. Route 53](section10.md)
