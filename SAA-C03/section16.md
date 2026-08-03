# Section 16. AWS 스토리지 추가 기능

## 목차

1. [AWS Snow Family](#1-aws-snow-family)
2. [Amazon FSx](#2-amazon-fsx)
3. [AWS Storage Gateway](#3-aws-storage-gateway)
4. [AWS Transfer Family](#4-aws-transfer-family)
5. [AWS DataSync](#5-aws-datasync)

---

## 1. AWS Snow Family

네트워크가 느리거나 대용량 데이터를 AWS로 이전할 때 사용하는 **오프라인 물리적 데이터 마이그레이션 장치** 시리즈.

### 1-1. 디바이스 종류

**Snowcone:**
- 소형, 경량 휴대용 장치
- 용량: **8TB HDD** 또는 **14TB SSD**
- 혹독한 환경에서도 작동 (Rugged)
- **DataSync 사전 설치** — 네트워크 연결 시 자동 동기화 가능
- 사용 사례: 엣지 컴퓨팅, 소규모 데이터 수집, 열악한 현장

**Snowball Edge:**
- 물리적 데이터 운송 장치, **TB ~ PB** 규모 마이그레이션

| 모델 | 스토리지 | 컴퓨팅 | 특징 |
|---|---|---|---|
| Storage Optimized | 80TB HDD + 1TB SSD | 40 vCPU | 대용량 데이터 이전 중심 |
| Compute Optimized | 28TB NVMe SSD | 104 vCPU, GPU 선택 | 엣지 ML, 미디어 트랜스코딩 |

- 사용 사례: 대규모 데이터 마이그레이션, 엣지 컴퓨팅, 로컬 스토리지

**Snowmobile:**
- 실제 **트럭** 형태의 초대형 데이터 이전 서비스
- **>100PB** 데이터 이전 가능
- **10PB 이상**이면 Snowball보다 Snowmobile이 효율적

### 1-2. 디바이스 비교

| 디바이스 | 용량 | 사용 환경 |
|---|---|---|
| Snowcone | 8~14TB | 소규모, 열악한 현장 |
| Snowball Edge | 28~80TB | 중~대규모 마이그레이션 |
| Snowmobile | ~100PB | 초대규모 데이터센터 이전 |

### 1-3. 사용 프로세스

```
1. AWS 콘솔에서 Snow 디바이스 주문
2. Snowball 클라이언트 / OpsHub 설치
3. 로컬 서버에 데이터 복사
4. 디바이스 AWS로 반송
5. AWS가 S3로 데이터 업로드
6. 디바이스 보안 삭제 (Secure Wipe)
```

### 1-4. 엣지 컴퓨팅 (Edge Computing)

- Snowcone / Snowball Edge에서 **EC2 인스턴스** 및 **Lambda 함수** 실행 가능
- 네트워크 미연결 환경에서 전처리, ML 추론, 미디어 트랜스코딩
- **장기 배포 옵션**: 1년 또는 3년 약정 가격으로 지속 운용

### 1-5. AWS OpsHub

- Snow Family 디바이스 관리용 **GUI 애플리케이션**
- 파일 전송, 인스턴스 관리, 모니터링 기능 제공

### 1-6. Snow Family → Glacier 주의사항

- Snow Family로 **Glacier에 직접 임포트 불가**
- 순서: **Snow → S3 임포트 → S3 수명 주기 정책으로 Glacier 전환**

> 💡 **시험 포인트**:
> - 10PB 초과 마이그레이션 → Snowmobile, 이하 → Snowball Edge
> - Snow → Glacier 직접 불가 → S3 거쳐서 수명 주기 정책으로 이동
> - Snowcone에는 DataSync 사전 설치 → 네트워크 연결 시 자동 전송 가능
> - Snowball Edge Compute Optimized = GPU 옵션 → ML/미디어 처리 적합

---

## 2. Amazon FSx

특정 워크로드를 위한 **완전 관리형 고성능 파일 시스템** 서비스.

### 2-1. FSx for Windows File Server

- **Windows 네이티브 공유 파일 시스템** 완전 관리형
- 지원 프로토콜: **SMB**, **Windows NTFS**
- **Microsoft Active Directory** 통합, ACL, 사용자 쿼터 지원
- **Linux EC2에서도 마운트 가능** (SMB 프로토콜 경유)
- **Multi-AZ** 배포로 고가용성 구성
- 데이터 **일 1회 S3 자동 백업**

**사용 사례:**
- Windows 애플리케이션 파일 공유
- Microsoft SharePoint, MS SQL Server, IIS

### 2-2. FSx for Lustre

- **HPC(High Performance Computing)** 를 위한 병렬 분산 파일 시스템
- **Lustre** = Linux + Cluster 의 합성어
- 성능: **수백 GB/s 처리량**, **수백만 IOPS**, **서브 밀리초 레이턴시**

**사용 사례:**
- 머신러닝, 빅데이터 분석, 금융 모델링
- 비디오 처리, EDA(전자 설계 자동화)

**S3 연동:**
- S3를 파일 시스템처럼 읽기 가능
- 처리 결과를 S3로 다시 저장 가능

**온프레미스 접근:**
- VPN 또는 Direct Connect를 통해 온프레미스에서 FSx for Lustre 사용 가능

### 2-3. FSx for Lustre 배포 옵션

| 항목 | Scratch (스크래치) | Persistent (영구) |
|---|---|---|
| 용도 | 임시 저장, 단기 처리 | 장기 저장, 지속적 사용 |
| 비용 | 낮음 (비용 최적화) | 상대적으로 높음 |
| 복제 | 없음 | 동일 AZ 내 복제 |
| 장애 복구 | 없음 | 서버 장애 시 수분 내 자동 복구 |

### 2-4. FSx for NetApp ONTAP

- AWS 관리형 **NetApp ONTAP** 파일 시스템
- 지원 프로토콜: **NFS, SMB, iSCSI** (멀티 프로토콜)
- **Multi-AZ** 고가용성
- **자동 스토리지 확장/축소**
- **중복 제거(De-duplication)**, **압축**, **즉시 클로닝** 지원
- 기존 온프레미스 ONTAP 워크로드의 AWS 마이그레이션에 적합

### 2-5. FSx for OpenZFS

- AWS 관리형 **OpenZFS** 파일 시스템
- 지원 프로토콜: **NFS** (v3, v4, v4.1, v4.2)
- 성능: 최대 **100만 IOPS**, **< 0.5ms 레이턴시**
- **즉시 Point-in-Time 클로닝** 지원
- 기존 ZFS 워크로드를 AWS로 이전할 때 사용

### 2-6. FSx 요약 비교

| FSx 유형 | 프로토콜 | 주요 사용 사례 |
|---|---|---|
| Windows File Server | SMB, NTFS | Windows 앱, AD 통합, SharePoint |
| Lustre | Lustre | HPC, ML, 대규모 분석 |
| NetApp ONTAP | NFS, SMB, iSCSI | 온프레미스 ONTAP 마이그레이션 |
| OpenZFS | NFS | ZFS 워크로드 AWS 이전 |

> 💡 **시험 포인트**:
> - HPC, ML, 분석 워크로드 → FSx for Lustre
> - Windows Active Directory 통합 → FSx for Windows File Server
> - 멀티 프로토콜(NFS+SMB+iSCSI) → FSx for NetApp ONTAP
> - FSx for Lustre Scratch = 임시/단기, Persistent = 장기/HA

---

## 3. AWS Storage Gateway

**온프레미스와 AWS 클라우드를 연결**하는 하이브리드 스토리지 서비스.

### 3-1. 주요 사용 사례

- 재해 복구 (Disaster Recovery)
- 백업 및 복구
- 계층화 스토리지 (Tiered Storage)
- 온프레미스 캐시 및 저지연 접근

### 3-2. 게이트웨이 유형

| 타입 | 프로토콜 | 백엔드 스토리지 |
|---|---|---|
| S3 File Gateway | NFS / SMB | S3 (Standard, IA, One Zone-IA, Intelligent-Tiering) |
| FSx File Gateway | SMB | FSx for Windows File Server |
| Volume Gateway | iSCSI | S3 + EBS 스냅샷 |
| Tape Gateway | iSCSI VTL | S3 및 Glacier |

### 3-3. S3 File Gateway

- 설정된 S3 버킷을 **NFS/SMB**로 파일 시스템처럼 접근
- **최근 사용 데이터를 로컬에 캐싱** → 저지연 접근
- 버킷별 **IAM 역할**로 접근 제어
- SMB 프로토콜: **Active Directory 통합**으로 Windows 인증 지원

### 3-4. Volume Gateway

**두 가지 모드:**

- **Cached Volumes (캐시 볼륨)**:
  - 기본 데이터: S3에 저장
  - 자주 접근하는 데이터만 로컬 캐싱
  - S3에 EBS 스냅샷으로 백업

- **Stored Volumes (저장 볼륨)**:
  - 기본 데이터: 온프레미스에 저장 (낮은 레이턴시)
  - S3로 **비동기 백업** (EBS 스냅샷)

### 3-5. Tape Gateway

- **가상 테이프 라이브러리(VTL)** 를 S3/Glacier로 구현
- 기존 테이프 기반 백업 애플리케이션과 **호환** (iSCSI 인터페이스)
- 물리적 테이프 인프라를 교체하면서 기존 소프트웨어 그대로 사용 가능

### 3-6. FSx File Gateway

- 온프레미스에서 **SMB**를 통해 FSx for Windows File Server에 접근
- 자주 접근하는 데이터를 **로컬 캐시**에 유지
- Windows 네이티브 기능 (AD, DFS, NTFS ACL) 그대로 활용

### 3-7. Storage Gateway Hardware Appliance

- 가상화 인프라가 없는 사이트를 위해 **amazon.com에서 물리 하드웨어 어플라이언스 구매 가능**
- File / Volume / Tape Gateway 모두 지원
- 소형 데이터센터, 지점 등에 적합

> 💡 **시험 포인트**:
> - 온프레미스 ↔ S3/EFS/Glacier 하이브리드 연결 = Storage Gateway
> - Volume Gateway Cached = S3 주 저장, 로컬 캐시 / Stored = 로컬 주 저장, S3 백업
> - 기존 테이프 백업 소프트웨어 유지하며 클라우드 이전 → Tape Gateway
> - 가상화 없는 온프레미스 → Hardware Appliance 구매 옵션

---

## 4. AWS Transfer Family

S3 또는 EFS로 **FTP 프로토콜 기반 파일 전송**을 완전 관리형으로 제공.

### 4-1. 지원 프로토콜

| 프로토콜 | 설명 |
|---|---|
| **FTP** | 평문 파일 전송 (비암호화) |
| **FTPS** | FTP over SSL (암호화) |
| **SFTP** | SSH 기반 보안 파일 전송 |

### 4-2. 특징

- 완전 관리형 인프라: **스케일링**, **신뢰성**, **Multi-AZ HA** 자동 처리
- **비용**: 프로비저닝된 엔드포인트 시간당 요금 + 데이터 전송량(GB) 요금

### 4-3. 인증 방식

- Transfer Family 내 사용자 자격증명 직접 관리
- 외부 디렉토리 통합:
  - **Microsoft Active Directory**
  - **LDAP**
  - **Okta**
  - **Amazon Cognito**

> 💡 **시험 포인트**:
> - FTP/FTPS/SFTP로 S3 또는 EFS에 파일 전송 → Transfer Family
> - 기존 FTP 기반 워크플로우를 클라우드로 이전할 때 코드 변경 없이 적용 가능
> - Multi-AZ 고가용성 기본 제공
> - Okta, Cognito 등 외부 IdP 통합으로 사용자 인증 가능

---

## 5. AWS DataSync

온프레미스 또는 다른 클라우드에서 AWS로, 혹은 **AWS 서비스 간 대량 데이터 동기화/이전** 서비스.

### 5-1. 데이터 이동 경로

**온프레미스/클라우드 → AWS (DataSync Agent 필요):**

| 소스 | 프로토콜 |
|---|---|
| 온프레미스 NAS / 파일 서버 | NFS, SMB |
| HDFS (Hadoop) | HDFS |
| 기타 클라우드 스토리지 | S3 API |

**AWS 서비스 간 이동 (Agent 불필요):**

- Amazon S3 ↔ Amazon EFS
- Amazon S3 ↔ Amazon FSx (Windows, Lustre, NetApp ONTAP, OpenZFS)
- Amazon EFS ↔ Amazon FSx

### 5-2. 주요 특징

| 특징 | 설명 |
|---|---|
| 스케줄링 | 시간별, 일별, 주별 실행 가능 |
| 메타데이터 보존 | NFS POSIX 권한, SMB ACL, 타임스탬프 유지 |
| 대역폭 제한 | 에이전트당 최대 **10 Gbps**, 사용자 지정 제한 설정 가능 |
| 보안 | 전송 중 암호화, 저장 시 암호화 |

### 5-3. DataSync vs Storage Gateway

| 항목 | DataSync | Storage Gateway |
|---|---|---|
| 주요 목적 | 일회성/주기적 대량 이전 | 지속적 하이브리드 연결 |
| 연속 복제 | 비적합 | 적합 (Cached/Stored 볼륨) |
| 온프레미스 통합 | 임시/마이그레이션 | 영구적 온프레미스 연결 |

### 5-4. 사용 사례

- 온프레미스 NFS/SMB 데이터를 S3/EFS/FSx로 **마이그레이션**
- 정기적인 데이터 동기화 (일 1회 배치 등)
- 데이터센터 폐기(Decommission) 시 AWS로 전체 이전
- AWS 서비스 간 데이터 재배치

> 💡 **시험 포인트**:
> - DataSync는 지속적 실시간 복제가 아닌 **주기적 동기화/마이그레이션** 도구
> - 파일 권한/메타데이터 보존이 필요한 마이그레이션 → DataSync
> - 온프레미스에서 DataSync 사용 시 **DataSync Agent** 설치 필요
> - AWS 서비스 간 이동 시에는 Agent 불필요 — S3↔EFS↔FSx 간 직접 동기화

---

← [Section 15. CloudFront 및 AWS 글로벌 액셀러레이터](section15.md) | [Section 17. 디커플링 애플리케이션: SQS, SNS, Kinesis, Active MQ](section17.md) →
