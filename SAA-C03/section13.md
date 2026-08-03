# Section 13. 고급 Amazon S3

## 목차

1. [S3 스토리지 클래스](#1-s3-스토리지-클래스)
2. [S3 수명 주기 정책](#2-s3-수명-주기-정책-lifecycle-rules)
3. [S3 Analytics](#3-s3-analytics)
4. [S3 이벤트 알림](#4-s3-이벤트-알림)
5. [S3 성능 최적화](#5-s3-성능-최적화)
6. [S3 Select & Glacier Select](#6-s3-select--glacier-select)
7. [S3 배치 작업](#7-s3-배치-작업-batch-operations)

---

## 1. S3 스토리지 클래스

### 클래스 비교표

| 스토리지 클래스 | 가용성 | 최소 저장 기간 | 용도 |
|---|---|---|---|
| **S3 Standard - General Purpose** | 99.99% | 없음 | 자주 액세스하는 데이터 |
| **S3 Standard-IA** | 99.9% | 30일 | 덜 자주 접근, 빠른 검색 |
| **S3 One Zone-IA** | 99.5% | 30일 | 단일 AZ, 재생성 가능 데이터 |
| **S3 Glacier Instant Retrieval** | 99.9% | 90일 | 분기 1회 액세스, ms 단위 검색 |
| **S3 Glacier Flexible Retrieval** | 99.99% | 90일 | 아카이브; Expedited(1-5분), Standard(3-5시간), Bulk(5-12시간) |
| **S3 Glacier Deep Archive** | 99.99% | 180일 | 장기 보관; Standard(12시간), Bulk(48시간) |
| **S3 Intelligent-Tiering** | 99.9% | 없음 | 액세스 패턴 불명확 시 자동 티어 이동 |

### S3 Intelligent-Tiering 세부 티어

| 티어 | 조건 |
|---|---|
| **Frequent Access** (기본) | 최근 액세스된 객체 |
| **Infrequent Access** | 30일 이상 미액세스 |
| **Archive Instant Access** | 90일 이상 미액세스 |
| **Archive Access** (선택) | 90~700일 이상 |
| **Deep Archive Access** (선택) | 180~700일 이상 |

- 클래스 간 이동: **수동** 또는 **수명 주기 규칙**으로 자동화 가능

> 💡 **시험 포인트**:
> - **One Zone-IA**는 단일 AZ 저장 → AZ 장애 시 데이터 손실 가능. 재생성 가능한 데이터에만 사용.
> - **Glacier Instant Retrieval**은 밀리초 검색 지원 → 분기 1회 정도 접근하는 데이터에 적합.
> - **Glacier Deep Archive**의 최소 저장 기간은 **180일**, 복원 시간은 최대 **48시간**.
> - **Intelligent-Tiering**은 모니터링 비용이 발생하지만 검색 비용은 없다.

---

## 2. S3 수명 주기 정책 (Lifecycle Rules)

### Transition Actions (전환 작업)

객체를 X일 후 다른 스토리지 클래스로 이동:

```
Standard
  → (60일 후) Standard-IA
    → (180일 후) Glacier
      → (365일 후) Glacier Deep Archive
```

### Expiration Actions (만료 작업)

| 사용 사례 | 설정 |
|---|---|
| 오래된 액세스 로그 삭제 | 365일 후 삭제 |
| 구 버전 삭제 (버전 관리 활성화 시) | N일 후 이전 버전 삭제 |
| 불완전한 멀티파트 업로드 정리 | X일 후 미완료 업로드 삭제 |

### 규칙 적용 범위

- **특정 접두사(Prefix)**: `s3://mybucket/mp3/*`
- **특정 객체 태그**: `Department = Finance`

> 💡 **시험 포인트**:
> - **불완전한 멀티파트 업로드**도 수명 주기 규칙으로 정리 가능 → 비용 절감.
> - 수명 주기 정책에서 클래스 전환 시 **최소 저장 기간**보다 짧게 설정 불가.

---

## 3. S3 Analytics

- **S3 Storage Class Analysis**: 스토리지 클래스 전환 시점 권장 사항 제공
- **Standard → Standard-IA** 전환 권장 분석 지원
- **지원하지 않는 클래스**: One Zone-IA, Glacier
- 보고서는 **매일 업데이트**; 초기 권장 사항 확인까지 **24~48시간** 소요
- 수명 주기 규칙 생성을 위한 **첫 번째 분석 단계**로 활용

> 💡 **시험 포인트**:
> - S3 Analytics는 **Standard/Standard-IA 간 전환**만 분석한다.
> - Glacier 전환 권장은 제공하지 않는다.

---

## 4. S3 이벤트 알림

### 이벤트 유형

- `S3:ObjectCreated`
- `S3:ObjectRemoved`
- `S3:ObjectRestore`
- `S3:Replication`

### 이벤트 필터링

- 객체 이름 패턴 필터 가능 (예: `*.jpg`)

### 이벤트 대상 (Destinations)

| 대상 | 특징 |
|---|---|
| **SNS Topic** | 알림 팬아웃 |
| **SQS Queue** | 큐 기반 처리 |
| **Lambda Function** | 서버리스 처리 |
| **EventBridge** | 고급 필터링 + 다수 대상 + 아카이브/재생 |

### EventBridge 활용

- **모든 S3 이벤트** → EventBridge로 전송
- EventBridge에서 **18개 이상의 AWS 서비스**로 라우팅 가능
- 고급 필터링: 메타데이터, 객체 크기, 이름 등 기반
- 이벤트 **아카이브 및 재생(Replay)** 지원

- 이벤트 전달 시간: 일반적으로 **몇 초**, 최대 수 분 소요

> 💡 **시험 포인트**:
> - **EventBridge**를 사용하면 단일 S3 이벤트를 여러 대상으로 동시에 라우팅 가능.
> - 단순 트리거(Lambda 자동 실행 등)에는 직접 연결(SNS/SQS/Lambda)을 사용한다.

---

## 5. S3 성능 최적화

### 기본 성능 (Baseline Performance)

- **자동 스케일링**: 높은 요청 속도에 자동 대응; 지연 시간 **100~200ms**
- 접두사(Prefix)당 요청 한도:
  - **3,500 PUT/COPY/POST/DELETE** /초
  - **5,500 GET/HEAD** /초
- 버킷 내 **접두사 수 제한 없음** → 접두사를 분산하면 성능 향상

### Multi-Part Upload (멀티파트 업로드)

```
파일 분할 → 각 파트 병렬 업로드 → S3에서 자동 조합
```

- **100MB 초과** 시 권장
- **5GB 초과** 시 필수
- 업로드 **병렬화** → 최대 대역폭 활용

### S3 Transfer Acceleration (전송 가속화)

```
클라이언트 → AWS 엣지 로케이션 (빠른 업로드)
                ↓
           AWS 내부 사설망
                ↓
          S3 버킷 (목적지 리전)
```

- 원거리 전송 속도 향상
- **멀티파트 업로드**와 함께 사용 가능

### S3 Byte-Range Fetches (범위 조회)

- GET 요청을 **범위 단위로 분할**하여 병렬 처리
- 오류 발생 시 **해당 범위만 재시도** → 복원력 향상
- 파일 일부만 검색 가능 (예: 파일 헤더의 첫 50바이트)

> 💡 **시험 포인트**:
> - **Transfer Acceleration**은 업로드 속도 향상 (엣지 로케이션 활용).
> - **Byte-Range Fetches**는 다운로드 병렬화 및 부분 조회에 사용.
> - 접두사를 다양하게 분산하면 GET/PUT 처리량을 선형으로 늘릴 수 있다.
> - 멀티파트 업로드는 **5GB 초과 필수**, **100MB 초과 권장** — 시험에 자주 출제.

---

## 6. S3 Select & Glacier Select

- **SQL 쿼리**로 서버 측 필터링 수행 → 필요한 데이터만 검색
- 행(Row) & 열(Column) 기반 단순 SQL 필터 지원

### 성능 이점

| 지표 | 효과 |
|---|---|
| **속도** | 최대 **400% 빠름** |
| **비용** | 최대 **80% 저렴** |
| **네트워크** | 전송 데이터량 감소 |
| **CPU** | 클라이언트 처리 비용 감소 |

```sql
-- 예시: S3 Select로 CSV에서 특정 조건 필터링
SELECT * FROM S3Object WHERE Age > 30
```

> 💡 **시험 포인트**:
> - S3 Select는 **서버 측 필터링** → 전체 객체를 다운로드하지 않는다.
> - Glacier Select도 동일한 원리로 Glacier 내 아카이브 데이터에 적용 가능.

---

## 7. S3 배치 작업 (Batch Operations)

### 개요

기존 S3 객체에 대한 **대규모 일괄 작업** 수행

### 주요 사용 사례

- 버킷 간 객체 복사
- 암호화되지 않은 객체 **일괄 암호화**
- ACL / 태그 수정
- Glacier에서 객체 복원
- 각 객체에 **Lambda 함수 호출**

### 작업 구성 요소

```
S3 Inventory (객체 목록 생성)
      ↓
S3 Select (목록 필터링)
      ↓
S3 Batch Operations (작업 실행)
      ↓
 - 자동 재시도
 - 진행률 추적
 - 알림 전송
 - 완료 보고서 생성
```

### S3 Batch Operations 워크플로우

1. **S3 Inventory**로 객체 목록 생성
2. **S3 Select**로 대상 객체 필터링
3. **S3 Batch Operations**에 작업 전달 및 실행

> 💡 **시험 포인트**:
> - 기존 객체 복제(CRR 활성화 이전 객체)에는 **S3 Batch Replication** 사용.
> - Batch Operations는 **재시도, 진행률 추적, 보고서 생성**을 자동으로 관리한다.
> - S3 Inventory → S3 Select → S3 Batch Operations의 **파이프라인 흐름**을 기억할 것.

---

← [Section 12. Amazon S3 소개](section12.md) | [Section 14. 아마존 S3 보안](section14.md) →
