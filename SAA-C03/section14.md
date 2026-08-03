# Section 14. 아마존 S3 보안

## 목차

1. [SSE 암호화 (Server-Side Encryption)](#1-sse-암호화-server-side-encryption)
2. [전송 중 암호화 (SSL/TLS)](#2-전송-중-암호화-ssltls)
3. [강제 암호화](#3-강제-암호화)
4. [CORS](#4-cors)
5. [MFA Delete](#5-mfa-delete)
6. [S3 Access Logs](#6-s3-access-logs)
7. [Pre-Signed URLs](#7-pre-signed-urls)
8. [S3 Glacier Vault Lock](#8-s3-glacier-vault-lock)
9. [S3 Object Lock](#9-s3-object-lock)
10. [S3 Access Points](#10-s3-access-points)
11. [S3 Object Lambda](#11-s3-object-lambda)

---

## 1. SSE 암호화 (Server-Side Encryption)

S3의 서버 측 암호화(SSE)는 4가지 방식이 있으며, 각 방식은 키 관리 주체와 동작 방식이 다르다.

### SSE-S3 (기본값)

- **키 관리 주체**: AWS S3
- **암호화 알고리즘**: AES-256
- **요청 헤더**: `"x-amz-server-side-encryption": "AES256"`
- 신규 버킷/객체에 **기본으로 활성화**되어 있음

### SSE-KMS

- **키 관리 주체**: AWS **KMS (Key Management Service)**
- **장점**: 사용자가 직접 키를 제어할 수 있고, **CloudTrail**로 감사 추적 가능
- **요청 헤더**: `"x-amz-server-side-encryption": "aws:kms"`
- **제한 사항**:
  - 업로드 시 `GenerateDataKey`, 다운로드 시 `Decrypt` KMS API 호출 발생
  - KMS 쿼터 제한 존재 (리전별 5,500 / 10,000 / 30,000 req/s)
  - 트래픽이 많을 경우 **스로틀링** 발생 가능
  - **S3 Bucket Key** 사용 시 KMS API 호출 횟수를 줄일 수 있음

### SSE-C

- **키 관리 주체**: 고객 (AWS 외부에서 완전 관리)
- S3는 암호화 키를 **저장하지 않음**
- **HTTPS 필수** (필수 요건)
- 모든 요청의 HTTP 헤더에 키를 직접 제공해야 함
- **AWS 콘솔 사용 불가** → AWS CLI 또는 SDK 사용 필요

### 클라이언트 측 암호화 (Client-Side Encryption)

- 클라이언트가 업로드 **전에 직접 암호화**
- 클라이언트가 다운로드 **후에 직접 복호화**
- 키와 암호화 사이클 전체를 고객이 완전히 관리

> 💡 **시험 포인트**:
> - SSE-S3는 **기본값**이며 헤더는 `AES256`, SSE-KMS 헤더는 `aws:kms`로 구분한다.
> - SSE-C는 반드시 **HTTPS**가 필요하며 콘솔 사용이 불가하다.
> - SSE-KMS 사용 시 KMS 쿼터 초과로 스로틀링이 발생할 수 있고, 이를 완화하기 위해 **S3 Bucket Key**를 사용한다.
> - SSE-KMS는 CloudTrail 감사 추적이 가능하다는 점이 핵심 이점이다.

---

## 2. 전송 중 암호화 (SSL/TLS)

- S3는 두 가지 엔드포인트를 제공한다:
  - **HTTP 엔드포인트**: 암호화되지 않음
  - **HTTPS 엔드포인트**: 전송 중 암호화 (encryption in flight)
- HTTPS 사용을 권장하며, **SSE-C의 경우 HTTPS 필수**
- 대부분의 클라이언트는 기본적으로 HTTPS를 사용함

> 💡 **시험 포인트**:
> - SSE-C는 전송 중 암호화(HTTPS)가 **강제** 요건이다.
> - HTTP와 HTTPS 모두 S3가 노출하지만, 보안을 위해 HTTPS 사용을 강제하려면 버킷 정책을 사용한다.

---

## 3. 강제 암호화

### HTTPS 강제 (aws:SecureTransport)

버킷 정책에서 `Deny` + 조건을 통해 HTTP 요청을 차단하고 HTTPS만 허용한다.

```json
{
  "Condition": {
    "Bool": {
      "aws:SecureTransport": "false"
    }
  }
}
```

### SSE-KMS 암호화 강제

버킷 정책에서 PUT 요청 시 `"s3:x-amz-server-side-encryption": "aws:kms"` 헤더가 없으면 **Deny** 처리.

### SSE-C 암호화 강제

버킷 정책에서 PUT 요청 시 `"s3:x-amz-server-side-encryption-customer-algorithm"` 헤더가 없으면 **Deny** 처리.

### 주의 사항

- **버킷 정책은 기본 암호화(Default Encryption)보다 먼저 평가된다.**
- SSE-S3는 신규 객체에 자동 적용(기본 암호화)되지만, SSE-KMS 또는 SSE-C를 강제하려면 버킷 정책에서 특정 암호화 헤더가 없는 API 호출을 거부해야 한다.

> 💡 **시험 포인트**:
> - `aws:SecureTransport: false` 조건으로 HTTP 요청을 **Deny**하여 HTTPS를 강제한다.
> - **버킷 정책 → 기본 암호화** 순서로 평가된다는 점을 기억한다.
> - 특정 암호화 방식을 강제하려면 헤더 조건을 포함한 Deny 정책을 사용한다.

---

## 4. CORS

### CORS 개념

- **Cross-Origin Resource Sharing**: 교차 출처 리소스 공유
- **Origin** = 스킴(프로토콜) + 호스트(도메인) + 포트
- 웹 브라우저 보안 정책: 허용되지 않은 경우 **크로스 오리진 요청을 차단**
- **프리플라이트(Preflight) 요청**: 브라우저가 CORS 헤더를 얻기 위해 먼저 `OPTIONS` 요청을 전송
- 응답 헤더: `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`

### Amazon S3 CORS

- 클라이언트가 S3 버킷에 크로스 오리진 요청을 할 경우, 해당 버킷에 **올바른 CORS 헤더를 활성화**해야 함
- 특정 오리진 또는 `*`(모든 오리진) 허용 가능
- **예시**: `my-bucket-html`(정적 웹사이트)이 `my-bucket-assets`에서 이미지를 로드하는 경우 → `my-bucket-assets`에 CORS 활성화 필요

> 💡 **시험 포인트**:
> - CORS 헤더는 **요청을 받는** 버킷(리소스 버킷)에 설정해야 한다.
> - S3 정적 웹호스팅에서 다른 버킷의 리소스를 참조하는 시나리오에서 자주 출제된다.

---

## 5. MFA Delete

- **MFA가 필요한 작업**:
  - 객체 버전 **영구 삭제**
  - 버킷의 버전 관리 **일시 중단**
- **MFA가 필요하지 않은 작업**:
  - 버전 관리 활성화
  - 삭제된 버전 목록 조회
- **버전 관리(Versioning)가 반드시 활성화**되어 있어야 MFA Delete 사용 가능
- **버킷 소유자(루트 계정)만** MFA Delete 활성화/비활성화 가능
- **AWS CLI를 통해서만** 활성화 가능 (콘솔 불가)

> 💡 **시험 포인트**:
> - MFA Delete는 **루트 계정**만 활성화할 수 있고, **콘솔이 아닌 CLI**를 통해서만 설정한다.
> - 버전 관리가 활성화되어 있어야 MFA Delete를 사용할 수 있다.
> - 버전 영구 삭제와 버전 관리 일시 중단만 MFA 대상임을 기억한다.

---

## 6. S3 Access Logs

- **목적**: 감사를 위해 S3 버킷에 대한 모든 접근 기록
- 모든 요청(모든 계정, 승인/거부 불문) → **별도의 S3 버킷**에 로그 저장
- **Amazon Athena** 또는 데이터 분석 도구로 분석 가능
- 대상 로깅 버킷은 **모니터링 버킷과 동일한 AWS 리전**에 있어야 함

### 주의 사항

> ⚠️ **로깅 버킷을 모니터링 버킷과 동일하게 설정하면 안 된다.**
> 로깅 루프가 발생하여 버킷 용량이 **지수적으로 증가**한다.

> 💡 **시험 포인트**:
> - 로깅 버킷과 모니터링 버킷을 **동일하게 설정하면 안 된다** — 로깅 루프 문제가 자주 출제된다.
> - 로그 분석 도구로 **Athena** 사용이 대표적이다.

---

## 7. Pre-Signed URLs

- S3 콘솔, AWS CLI, 또는 SDK를 통해 생성

### URL 만료 시간

| 생성 방법 | 만료 시간 범위 |
|---|---|
| S3 콘솔 | 1분 ~ 720분 (12시간) |
| AWS CLI | `--expires-in` 파라미터 (기본 3600초, 최대 604800초 = 168시간) |

- Pre-Signed URL을 받은 사용자는 **URL을 생성한 사용자의 권한**을 상속받음 (GET/PUT)

### 사용 사례

- 로그인한 사용자에게 프리미엄 비디오 **다운로드** 허용
- 수시로 변경되는 사용자 목록에 파일 다운로드 허용
- 특정 위치에 파일을 **일시적으로 업로드**할 수 있도록 허용

> 💡 **시험 포인트**:
> - Pre-Signed URL은 **생성자의 권한**을 그대로 사용한다.
> - CLI 최대 만료 시간은 **604800초(168시간)**이다.
> - 일시적인 접근 허용 시나리오에서 Pre-Signed URL이 정답인 경우가 많다.

---

## 8. S3 Glacier Vault Lock

- **WORM (Write Once Read Many)** 모델 채택
- **Vault Lock Policy** 생성 → 정책을 잠금(lock)하여 이후 수정/삭제 불가
- 컴플라이언스 요건 및 데이터 보존 규정 준수에 활용

> 💡 **시험 포인트**:
> - Vault Lock Policy는 한 번 잠기면 **변경도 삭제도 불가**하다.
> - WORM 모델이 언급되면 Glacier Vault Lock 또는 S3 Object Lock을 떠올린다.

---

## 9. S3 Object Lock

- **버전 관리(Versioning) 활성화 필수**
- **WORM (Write Once Read Many)** 모델 채택
- 지정된 기간 동안 객체 버전 삭제를 차단

### 보존 모드 (Retention Modes)

| 모드 | 설명 |
|---|---|
| **Compliance** | root user 포함 **누구도** 덮어쓰기/삭제 불가; retention mode/period 변경 불가 |
| **Governance** | **특별 권한을 가진 일부 사용자**만 retention/delete 가능; 대부분의 사용자는 불가 |

### Retention Period (보존 기간)

- 고정된 기간 동안 객체를 보호 (기간 **연장 가능**)

### Legal Hold (법적 보류)

- 보존 기간과 **독립적**으로 무기한 보호
- `s3:PutObjectLegalHold` IAM 권한을 통해 자유롭게 배치/제거 가능

> 💡 **시험 포인트**:
> - **Compliance 모드**는 root 포함 누구도 변경 불가 → 가장 엄격한 규정 준수용.
> - **Governance 모드**는 특별 권한 사용자는 예외적으로 변경 가능.
> - **Legal Hold**는 보존 기간과 무관하게 독립적으로 설정/해제된다.
> - Object Lock 사용 전 **버전 관리 활성화**가 전제 조건이다.

---

## 10. S3 Access Points

- 다수의 사용자/그룹이 서로 다른 접근 권한이 필요한 경우 S3 **보안 관리를 단순화**
- 각 **Access Point**는 고유의 DNS 이름(인터넷 또는 VPC 오리진)과 접근 포인트 정책(버킷 정책과 유사)을 가짐

### 사용 예시

| 액세스 포인트 | 대상 사용자 | 권한 |
|---|---|---|
| Finance AP | Finance 팀 | `/finance` 접두사 R/W |
| Sales AP | Sales 팀 | `/sales` 접두사 R/W |
| Analytics AP | Analytics 팀 | 버킷 전체 R |

- 버킷 정책 관리를 **대폭 단순화**할 수 있음

### VPC Origin Access Points

- Access Point를 **VPC 내부에서만** 접근 가능하도록 정의
- **VPC 엔드포인트**(Gateway 또는 Interface) 생성 필요
- VPC 엔드포인트 정책에서 대상 버킷 **및** Access Point 모두에 대한 접근을 허용해야 함

> 💡 **시험 포인트**:
> - Access Point는 복잡한 버킷 정책을 단순화하는 용도로 사용된다.
> - VPC Origin Access Point는 VPC 엔드포인트가 반드시 필요하며, 엔드포인트 정책도 함께 설정해야 한다.

---

## 11. S3 Object Lambda

- **AWS Lambda 함수**를 사용하여 객체가 **검색되기 전에** 변환 처리
- 구성 요소: S3 버킷 1개 + **S3 Access Point** + **S3 Object Lambda Access Point**

### 사용 사례

- 분석/비운영 환경을 위한 **PII(개인 식별 정보) 리댁팅(삭제/마스킹)**
- 데이터 포맷 변환 (예: XML → JSON)
- 이미지 **동적 리사이징** 또는 **워터마크** 추가

> 💡 **시험 포인트**:
> - S3 Object Lambda는 **별도의 버킷 없이** 람다로 객체를 동적 변환한다.
> - PII 마스킹, 포맷 변환, 이미지 처리 시나리오에서 S3 Object Lambda가 정답이다.
> - S3 Access Point와 S3 Object Lambda Access Point 두 가지가 모두 필요하다.

---

← [Section 13. 고급 Amazon S3](section13.md) | [Section 15. CloudFront 및 AWS 글로벌 액셀러레이터](section15.md) →
