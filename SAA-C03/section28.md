# Section 28. 재해 복구 및 마이그레이션

## 목차

1. [재해 복구 전략](#1-재해-복구-전략)
2. [DMS (Database Migration Service)](#2-dms-database-migration-service)
3. [RDS & Aurora 마이그레이션](#3-rds--aurora-마이그레이션)
4. [On-Premises 전략](#4-on-premises-전략)
5. [AWS Backup](#5-aws-backup)
6. [AWS Application Discovery Service](#6-aws-application-discovery-service)
7. [AWS Application Migration Service (MGN)](#7-aws-application-migration-service-mgn)
8. [VMware Cloud on AWS](#8-vmware-cloud-on-aws)

---

## 1. 재해 복구 전략

### RPO vs RTO

- **RPO (Recovery Point Objective)**: 허용 가능한 **데이터 손실 범위** (마지막 백업 ~ 재해 발생 시점)
- **RTO (Recovery Time Objective)**: 허용 가능한 **복구 시간** (재해 발생 ~ 시스템 복구 완료)

```
[마지막 백업]──── RPO ────[재해 발생]──── RTO ────[복구 완료]
      ↑                        ↑                       ↑
  스냅샷 시점              장애 시점               서비스 재개
```

- RPO가 낮을수록 → 데이터 손실 적음 → 더 자주 백업/복제 필요
- RTO가 낮을수록 → 복구 빠름 → 더 많은 인프라 준비 필요 (비용 증가)

### DR 전략 비교

| 전략 | RTO | 비용 | 설명 |
|---|---|---|---|
| **Backup & Restore** | 높음 (수 시간) | 낮음 | 정기 스냅샷 백업; 복구 시 인프라 프로비저닝 |
| **Pilot Light** | 중간 | 낮음 | 핵심 서비스(DB 등)만 최소 규모로 항상 실행; 장애 시 확장 |
| **Warm Standby** | 낮음 | 중간 | 축소된 전체 시스템 항상 실행; 장애 시 스케일 업 |
| **Multi-Site / Hot Site** | 매우 낮음 | 높음 | 프로덕션과 동일 규모로 DR 환경 항상 운영 |

### 전략별 구현 팁

**Backup & Restore:**
- 스냅샷(EBS, RDS, Redshift) 정기 생성
- S3에 크로스 리전 복제
- CloudFormation으로 인프라 코드화

**Pilot Light:**
- 데이터를 DR 리전에 실시간 복제
- 최소한의 EC2 인스턴스만 대기
- 장애 시 Route 53 DNS 전환 + 인스턴스 확장

**Warm Standby:**
- Route 53 페일오버 활성화
- 최소 용량 ASG로 축소 운영
- 장애 시 ASG 스케일 업

**Hot Site (Multi-Site):**
- Route 53 Active-Active 구성
- 두 리전 모두 전체 프로덕션 용량 운영
- Chaos Engineering으로 장애 시나리오 정기 테스트

> 💡 **시험 포인트**:
> - RPO = 데이터 손실 허용 범위; RTO = 복구 시간 목표
> - 비용 vs 복구 속도 트레이드오프: Backup&Restore(저비용/느림) → Hot Site(고비용/빠름)
> - Pilot Light: DB 등 핵심만 실행; Warm Standby: 전체 시스템 소규모 실행
> - Multi-Site = Active-Active; 최고 가용성, 최고 비용

---

## 2. DMS (Database Migration Service)

### 개요

- 데이터베이스를 AWS로 **빠르고 안전하게 마이그레이션**
- **마이그레이션 중 소스 DB 계속 사용 가능** (다운타임 최소화)
- 소스 지원: Oracle, MS SQL, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2
- 타겟 지원: 대부분의 AWS DB 서비스

### 마이그레이션 유형

| 유형 | 설명 | 도구 |
|---|---|---|
| **Homogeneous** | 동일 DB 엔진 (Oracle → Oracle, MySQL → MySQL) | DMS만 사용 |
| **Heterogeneous** | 다른 DB 엔진 (Oracle → Aurora, SQL Server → MySQL) | **SCT + DMS** |

**Schema Conversion Tool (SCT)**:
- 소스 DB 스키마 + 코드(뷰, 프로시저 등)를 타겟 엔진 형식으로 변환
- 이기종 마이그레이션 시 DMS 이전에 **반드시 SCT 먼저 실행**

### DMS 소스 및 타겟

**소스:**
- 온프레미스 / EC2의 DB
- Azure SQL DB
- RDS (Aurora 포함)

**타겟:**
- 온프레미스 / EC2 / RDS
- DynamoDB, S3
- OpenSearch, Kinesis Data Streams
- DocumentDB, Redshift, Neptune

### Multi-AZ DMS

- 다른 AZ에 **스탠바이 레플리카** 자동 유지
- I/O 동결 없이 데이터 이중화
- 장애 시 자동 페일오버

### 지속적 복제 (CDC)

- **CDC (Change Data Capture)**: 소스 DB의 변경사항을 지속적으로 타겟에 복제
- 다운타임 제로(Zero Downtime) 마이그레이션 가능

```
[소스 DB] ──── 초기 전체 로드 ────→ [DMS 복제 인스턴스] ──→ [타겟 DB]
    └──────── CDC 지속 복제 ────────────────────────────────┘
```

> 💡 **시험 포인트**:
> - 이기종 마이그레이션: SCT(스키마 변환) → DMS(데이터 이동) 순서
> - 동종 마이그레이션: DMS만 사용
> - CDC로 다운타임 없이 지속 복제 가능
> - 마이그레이션 중 소스 DB 운영 가능

---

## 3. RDS & Aurora 마이그레이션

### RDS MySQL → Aurora MySQL

| 방법 | 다운타임 | 특징 |
|---|---|---|
| DB 스냅샷 → Aurora 클러스터로 복원 | 있음 | 간단하지만 다운타임 발생 |
| RDS MySQL에서 **Aurora Read Replica** 생성 → 프로모션 | 최소 | 복제 지연(lag)=0 확인 후 프로모션; 소규모 다운타임 |

### 외부 MySQL → Aurora MySQL

| 방법 | 권장도 | 설명 |
|---|---|---|
| **Percona XtraBackup → S3 → Aurora** | 권장 | 다운타임 최소; S3를 중간 스토리지로 활용 |
| `mysqldump` → Aurora | 비권장 | 느림; 대용량 DB에 부적합 |

### 외부 PostgreSQL → Aurora PostgreSQL

1. PostgreSQL DB를 S3에 백업
2. Aurora에서 `aws_s3` 확장(Extension)을 사용하여 S3에서 임포트
3. 지속적 복제 필요 시 DMS 활용

### DynamoDB 마이그레이션

- 타 계정으로 DynamoDB 테이블 이동: **DMS** 또는 **AWS Data Pipeline** 사용

> 💡 **시험 포인트**:
> - RDS → Aurora: Read Replica 방식이 다운타임 최소
> - 외부 MySQL → Aurora: Percona XtraBackup → S3 경유가 최선
> - PostgreSQL → Aurora: aws_s3 Extension + S3 활용
> - 이기종 DB 지속 복제: DMS + CDC

---

## 4. On-Premises 전략

### VM Import/Export

- 온프레미스 **VM을 AMI로 임포트** → EC2로 마이그레이션
- EC2 인스턴스를 온프레미스 VM으로 **엑스포트** 가능
- 주요 사용 사례:
  - 온프레미스 VM을 DR 저장소로 AWS에 보관
  - 기존 애플리케이션 클라우드 리프트(Lift)

### AWS Application Discovery Service

- 온프레미스 서버 정보를 수집하여 **마이그레이션 계획 수립** 지원
- 서버 활용률 데이터 + **의존성 매핑** 제공

**발견 모드:**

| 모드 | 방법 | 수집 데이터 |
|---|---|---|
| **Agentless Discovery** | VMware vCenter 커넥터 | VM 인벤토리, 설정, 성능 이력 |
| **Agent-based Discovery** | 서버에 에이전트 설치 | 네트워크 의존성, 프로세스, 성능 데이터 |

- 수집 데이터는 **AWS Migration Hub**에서 통합 조회

### AWS Migration Hub

- 마이그레이션 **중앙 추적 및 계획** 플랫폼
- 통합 서비스: Application Discovery Service, Application Migration Service (MGN)
- DMS, SMS(Server Migration Service), MGN 상태 통합 집계

> 💡 **시험 포인트**:
> - VM Import/Export: 온프레미스 VM ↔ AWS AMI/EC2
> - Application Discovery Service: 에이전트리스(VMware) vs 에이전트 기반
> - Migration Hub: 마이그레이션 상태 중앙 집중 추적
> - 에이전트 기반이 더 상세한 정보 수집 (네트워크 의존성 포함)

---

## 5. AWS Backup

### 개요

- **완전 관리형 중앙 집중식** 백업 서비스
- 여러 AWS 서비스의 백업을 단일 콘솔에서 관리

### 지원 서비스

- EC2, EBS, S3
- RDS / Aurora, DynamoDB, DocumentDB, Neptune
- EFS, FSx (Windows File Server, Lustre)
- Storage Gateway (Volume Gateway)

### 주요 기능

| 기능 | 설명 |
|---|---|
| 백업 플랜 | 빈도, 보존 기간 설정 |
| 태그 기반 정책 | 태그로 백업 대상 자동 지정 |
| 크로스 리전 백업 | 다른 리전으로 백업 복사 |
| 크로스 계정 백업 | 다른 계정으로 백업 복사 |
| **Backup Vault Lock** | **WORM 정책**; 루트 계정도 삭제 불가; 컴플라이언스 강화 |

**Backup Vault Lock:**
- Write Once Read Many (WORM) 정책 적용
- 활성화 이후 누구도 (루트 계정 포함) 백업 삭제/수정 불가
- 규제 준수 환경에서 필수

> 💡 **시험 포인트**:
> - AWS Backup: 여러 서비스 백업을 중앙 관리
> - Backup Vault Lock = WORM; 루트도 삭제 불가 → 컴플라이언스
> - 크로스 리전 + 크로스 계정 백업 지원
> - 태그 기반 백업 정책으로 자동화 가능

---

## 6. AWS Application Discovery Service

### 개요

- 온프레미스 인프라 분석을 통해 **마이그레이션 계획 최적화**
- 서버 사양, 성능, 네트워크 의존성 데이터 수집

### 발견 방식 상세

**Agentless Discovery (VMware 환경):**
- OVA 형식의 커넥터를 vCenter에 배포
- VM 인벤토리, 구성, 성능 이력 수집
- 에이전트 설치 없음 → 빠른 배포

**Agent-based Discovery (물리 서버 / 비VMware):**
- 각 서버에 Discovery Agent 설치
- 수집 항목: 시스템 설정, 성능, 실행 프로세스, **네트워크 연결 및 의존성**
- 더 상세한 데이터 제공

### Migration Hub 통합

```
[Application Discovery Service]
        ↓ 수집 데이터 전송
[AWS Migration Hub]
        ↓ 통합 추적
[DMS / MGN / SMS]
```

> 💡 **시험 포인트**:
> - Agentless: VMware 전용; 에이전트 불필요
> - Agent-based: 모든 서버 지원; 네트워크 의존성 수집 가능
> - Migration Hub에서 발견 데이터 + 마이그레이션 상태 통합 관리

---

## 7. AWS Application Migration Service (MGN)

### 개요

- 물리 서버, 가상 서버, 클라우드 서버를 **AWS 네이티브로 리프트-앤-시프트**
- 구 서비스명: CloudEndure Migration의 후속

### 동작 방식

1. 소스 서버에 **AWS Replication Agent** 설치
2. **블록 레벨 지속 복제** (EBS로 복제)
3. **컷오버(Cutover)** 창: 수 분 이내
4. 비즈니스 중단 없이 마이그레이션 완료

```
[소스 서버 (온프레미스/VM/타 클라우드)]
        ↓ Replication Agent
[AWS MGN 스테이징 영역 (저비용 EC2 + EBS)]
        ↓ 검증 완료 후 컷오버
[프로덕션 EC2 인스턴스]
```

### 지원 범위

- 다양한 플랫폼: VMware, Hyper-V, 물리 서버
- 다양한 OS: Windows, Linux
- 다양한 DB: MySQL, SQL Server, Oracle 등

> 💡 **시험 포인트**:
> - MGN = Lift-and-Shift 자동화 서비스
> - 블록 레벨 복제로 최소 다운타임 컷오버 (분 단위)
> - DMS는 DB 마이그레이션; MGN은 서버 전체 마이그레이션 (역할 구분)
> - 에이전트 설치 → 지속 복제 → 컷오버 순서

---

## 8. VMware Cloud on AWS

### 개요

- **VMware Cloud Foundation을 AWS 인프라에서 실행**
- 온프레미스 VMware 데이터센터를 AWS로 확장

### 주요 사용 사례

- VMware vSphere 워크로드를 AWS로 마이그레이션 (재아키텍처 없이)
- 온프레미스 데이터센터 용량 부족 시 AWS로 **Stretch Cluster**
- VMware 환경에서 **AWS 네이티브 서비스** 활용
- DR 목적: 온프레미스 VMware 워크로드 → AWS VMware Cloud 페일오버

### 특징

- AWS 인프라에서 실행되지만 **VMware 툴/기술 그대로 사용**
- vSphere, vSAN, NSX 등 VMware 스택 지원
- AWS 서비스(S3, RDS 등)와 직접 통합 가능

> 💡 **시험 포인트**:
> - VMware Cloud on AWS: 재아키텍처 없이 VMware 워크로드 이전
> - Stretch Cluster: 온프레미스와 AWS 동시 운영으로 용량 확장
> - VM Import/Export와 차이: VMware Cloud는 VMware 환경 유지; VM Import는 EC2로 변환

---

## 마이그레이션 서비스 요약

| 목적 | 서비스 |
|---|---|
| **DB 마이그레이션** | DMS + SCT (이기종) |
| **서버 리프트-앤-시프트** | AWS MGN (Application Migration Service) |
| **온프레미스 서버 분석** | Application Discovery Service |
| **마이그레이션 중앙 추적** | Migration Hub |
| **전사 백업 관리** | AWS Backup |
| **VMware 워크로드 유지** | VMware Cloud on AWS |
| **VM → AMI 변환** | VM Import/Export |

---

← [Section 27. 네트워킹 - VPC](section27.md) | [Section 29. 더 많은 솔루션 아키텍처](section29.md) →
