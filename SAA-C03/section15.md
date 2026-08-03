# Section 15. CloudFront 및 AWS 글로벌 액셀러레이터

## 목차

1. [CloudFront 개요](#1-cloudfront-개요)
2. [CloudFront Origins](#2-cloudfront-origins)
3. [CloudFront 캐싱](#3-cloudfront-캐싱)
4. [CloudFront 보안](#4-cloudfront-보안)
5. [CloudFront 고급 기능](#5-cloudfront-고급-기능)
6. [AWS Global Accelerator](#6-aws-global-accelerator)

---

## 1. CloudFront 개요

**CloudFront**는 AWS의 **CDN(Content Delivery Network)** 서비스로, 전 세계 엣지 로케이션에 콘텐츠를 캐싱하여 읽기 성능을 향상시킨다.

- 전 세계 **216개 이상의 PoP(Points of Presence)** 엣지 로케이션 보유
- **AWS Shield**, **AWS WAF**와 통합되어 **DDoS 방어** 기능 내장
- 사용자는 가장 가까운 엣지 로케이션에서 콘텐츠를 받아 레이턴시 감소

> 💡 **시험 포인트**:
> - CloudFront는 글로벌 CDN이며 엣지 캐싱을 통한 읽기 성능 향상이 핵심
> - Shield와 WAF와 통합 → DDoS 방어 자동 적용
> - 216+ PoP 숫자는 시험에 직접 출제되지 않으나 "글로벌 분산"의 근거로 이해
> - 정적 콘텐츠 전 세계 배포에 최적화된 서비스

---

## 2. CloudFront Origins

CloudFront가 콘텐츠를 가져오는 **원본(Origin)** 유형은 세 가지다.

### 2-1. S3 버킷

- 파일을 배포하고 엣지에서 캐싱
- **OAC(Origin Access Control)** 으로 S3 버킷 보안 강화 — CloudFront를 통해서만 접근 허용
- CloudFront를 통해 S3로 파일 **업로드**도 가능 (단순 다운로드만이 아님)

### 2-2. VPC Origin

- **프라이빗 VPC 서브넷** 내 애플리케이션 서버를 오리진으로 사용
- Private ALB, Private NLB, Private EC2 인스턴스 지원
- VPC 외부 노출 없이 CloudFront를 통해 콘텐츠 전달 가능

### 2-3. Custom Origin (HTTP)

- **S3 정적 웹사이트**: 버킷을 S3 웹사이트로 먼저 활성화해야 함
- **Public ALB**, 온프레미스 HTTP 서버 등 공개된 HTTP 백엔드 모두 지원

### 2-4. 동작 흐름

```
Client
  ↓ 요청
CloudFront Edge Location
  ↓ 캐시 확인
  ├─ HIT  → 즉시 응답 (TTL 내 캐시 데이터)
  └─ MISS → Origin으로 포워딩 → 응답 받아 엣지에 캐싱 후 클라이언트에 전달
```

### 2-5. CloudFront vs S3 Cross-Region Replication

| 항목 | CloudFront | S3 CRR |
|---|---|---|
| 용도 | 전 세계 엣지 캐싱 | 특정 리전에 복제 |
| TTL | 있음 (TTL 내 캐시 유지) | 거의 실시간 |
| 읽기 전용 여부 | 아니오 (업로드 가능) | 예 |
| 적합한 케이스 | 정적 콘텐츠 전 세계 배포 | 동적 콘텐츠, 소수 리전 저지연 |

> 💡 **시험 포인트**:
> - OAC는 OAI(Origin Access Identity)를 대체하는 최신 방식 — 시험에서 OAC가 정답
> - S3 정적 웹사이트를 오리진으로 쓰려면 반드시 S3 웹사이트 기능을 먼저 활성화
> - CloudFront는 정적 + 전 세계 배포, S3 CRR은 동적 + 소수 특정 리전 저지연에 적합
> - VPC Origin으로 프라이빗 ALB/NLB/EC2도 오리진으로 사용 가능

---

## 3. CloudFront 캐싱

### 3-1. 캐시 기준

캐시는 다음 세 가지 요소를 기반으로 구분된다:

- **Headers** (요청 헤더)
- **Session Cookies**
- **Query String Parameters**

### 3-2. TTL 및 캐시 무효화

- **TTL**: 0 ~ 1년 범위; `Cache-Control` 또는 `Expires` 헤더로 설정
- **Cache Invalidation**: `CreateInvalidation` API 호출
  - 특정 경로 무효화: `/images/*`, `/index.html` 등
  - 모든 파일: `/*`

### 3-3. 캐시 히트율 최대화 전략

- **정적 배포 / 동적 배포 분리**: 정적 콘텐츠는 별도 CloudFront 배포로 분리
- 불필요한 헤더/쿠키/쿼리스트링을 캐시 키에서 제외

### 3-4. Cache Behaviors

- URL 경로 패턴별로 **다른 캐시 설정** 적용 가능
- 기본(Default) 동작: `*` 패턴
- 특정 경로 오버라이드 예시:

| 경로 패턴 | 동작 |
|---|---|
| `/api/*` | 캐싱 비활성화, ALB로 포워딩 |
| `/images/*` | S3로 포워딩, 긴 TTL 적용 |
| `*` (기본) | S3로 포워딩, 기본 TTL |

> 💡 **시험 포인트**:
> - 캐시 키는 헤더/쿠키/쿼리스트링 조합으로 구성 — 너무 많이 포함하면 캐시 히트율 저하
> - Cache Invalidation은 추가 비용 발생 — 즉각적인 콘텐츠 갱신 필요 시 사용
> - Cache Behaviors로 `/api/*`는 캐싱 없이 동적 처리, 나머지는 캐싱 적용 패턴이 자주 출제
> - TTL 0 설정 시 매 요청마다 오리진 확인 (캐싱 사실상 비활성화)

---

## 4. CloudFront 보안

### 4-1. Geo Restriction (지리적 제한)

- **Allowlist**: 허용된 특정 국가만 접근 가능
- **Blocklist**: 특정 국가의 접근 차단
- 국가 판별: 3rd party **Geo-IP 데이터베이스** 활용
- 사용 사례: 저작권법 준수, 콘텐츠 라이선스 지역 제한

### 4-2. HTTPS 설정

**Viewer Protocol Policy** (클라이언트 ↔ CloudFront):

| 옵션 | 설명 |
|---|---|
| HTTP and HTTPS | 둘 다 허용 |
| Redirect HTTP to HTTPS | HTTP 요청을 HTTPS로 리다이렉트 |
| HTTPS Only | HTTPS만 허용 |

**Origin Protocol Policy** (CloudFront ↔ Origin):

- HTTP Only
- Match Viewer (클라이언트와 동일한 프로토콜 사용)
- HTTPS Only

### 4-3. Origin Access Control (OAC)

- S3 버킷 오리진 보호: **CloudFront를 통해서만** S3 접근 허용 (직접 S3 URL 접근 차단)
- S3 버킷 정책에 CloudFront 서비스 프린시플만 허용하도록 설정
- 기존 **OAI(Origin Access Identity)를 대체**하는 최신 방식

### 4-4. CloudFront Signed URLs / Signed Cookies

프리미엄/유료 콘텐츠를 **특정 사용자에게만** 배포할 때 사용.

| 항목 | Signed URL | Signed Cookie |
|---|---|---|
| 접근 범위 | 개별 파일 1개 | 여러 파일 |
| 사용 사례 | 특정 단일 파일 접근 | 다수 파일 패키지 접근 |

**Signed URL/Cookie 포함 정보:**
- URL 만료 시간
- 허용된 IP 범위
- 신뢰할 수 있는 서명자 (CloudFront 키 페어를 가진 AWS 계정)

**만료 시간 가이드:**
- 공유 콘텐츠 (영화 등): 수 분 단위
- 프라이빗 콘텐츠 (구독 서비스 등): 수 년 단위

**CloudFront Signed URL vs S3 Pre-Signed URL:**

| 항목 | CloudFront Signed URL | S3 Pre-Signed URL |
|---|---|---|
| 접근 경로 | CloudFront 경유만 가능 | S3 직접 접근 |
| IAM 연동 | CloudFront 키 페어 | 생성자의 IAM 권한 사용 |
| CloudFront 기능 활용 | 가능 (캐싱, 지역 제한 등) | 불가 |
| 유효 기간 | 설정 가능 | 제한적 |

### 4-5. Field-Level Encryption

- POST 요청의 특정 필드를 **엣지에서 비대칭 암호화 (공개 키)**
- 최대 **10개 필드** 암호화 지원
- 민감한 정보 (신용카드 번호 등)를 오리진 도달 전 추가 보호

> 💡 **시험 포인트**:
> - Signed URL = 1개 파일, Signed Cookie = 다수 파일 — 구분 필수
> - CloudFront Signed URL은 CloudFront 경유만 허용, S3 Pre-Signed URL은 S3 직접 접근
> - OAC가 OAI를 대체 — 시험에서 "S3 오리진 보호"는 OAC가 정답
> - Geo Restriction은 IP 기반 Geo-IP DB 사용 (100% 정확하지 않음)

---

## 5. CloudFront 고급 기능

### 5-1. Price Classes (가격 클래스)

비용과 성능 간 트레이드오프 조정:

| 클래스 | 포함 리전 | 특징 |
|---|---|---|
| All | 전체 엣지 로케이션 | 최고 성능, 최고 비용 |
| Price Class 200 | 대부분 리전 (고비용 일부 제외) | 중간 |
| Price Class 100 | 최저 비용 리전만 (북미, 유럽) | 최저 비용 |

### 5-2. Multiple Origins & Origin Groups

- **Multiple Origins**: URL 경로 패턴에 따라 다른 오리진으로 라우팅
  - 예: `/api/*` → ALB, `/*` → S3
- **Origin Groups**: **Primary + Secondary 오리진** 구성
  - Primary 오리진 장애 시 Secondary로 자동 **Failover**
  - 고가용성(HA) 오리진 구성에 사용

### 5-3. CloudFront Functions

- **JavaScript** 기반 경량 함수
- **Sub-millisecond** 시작 시간, **수백만 req/s** 처리 가능
- CloudFront 내에서 네이티브로 관리

**지원 이벤트:**
- **Viewer Request** (클라이언트 → CloudFront)
- **Viewer Response** (CloudFront → 클라이언트)

**사용 사례:**
- 캐시 키 정규화 (쿼리스트링 정렬)
- 요청 헤더/URL 조작
- 요청 인증/인가 (JWT 검증 등)

### 5-4. Lambda@Edge

- **Node.js 또는 Python** Lambda 함수
- **us-east-1**에서 작성 후 전체 리전에 자동 배포
- **수천 req/s** 처리 (CloudFront Functions 대비 낮은 확장성)

**지원 이벤트:**
- Viewer Request / Viewer Response
- **Origin Request** / **Origin Response** (CloudFront Functions과의 차이점)

**사용 사례:**
- 실행 시간이 필요한 복잡한 로직 (ms ~ 10초)
- CPU/메모리 집약적 처리
- 서드파티 라이브러리 사용
- 네트워크 접근, 파일 시스템 접근 필요

### 5-5. CloudFront Functions vs Lambda@Edge 비교

| 항목 | CloudFront Functions | Lambda@Edge |
|---|---|---|
| 언어 | JavaScript | Node.js, Python |
| 확장성 | 수백만 req/s | 수천 req/s |
| 실행 시간 | < 1ms | 1ms ~ 10초 |
| 네트워크 접근 | 불가 | 가능 |
| 이벤트 트리거 | Viewer req/res만 | Viewer + Origin req/res |
| 관리 위치 | CloudFront 네이티브 | us-east-1 Lambda |
| 비용 | 더 저렴 | 상대적으로 비쌈 |

> 💡 **시험 포인트**:
> - Lambda@Edge만 Origin Request/Response 이벤트 처리 가능 — 핵심 구분 포인트
> - 네트워크 접근, 파일 시스템, 서드파티 라이브러리 → Lambda@Edge
> - 간단한 헤더 조작, 캐시 키 정규화 → CloudFront Functions (빠르고 저렴)
> - Origin Groups의 Failover 기능 = CloudFront 레벨 HA 구성

---

## 6. AWS Global Accelerator

### 6-1. 문제와 해결책

**문제:** 전 세계 사용자가 애플리케이션에 접근할 때, 퍼블릭 인터넷을 통한 **다수의 네트워크 홉(hop)** 으로 인해 지연 및 불안정 발생

**해결책:** AWS 내부 프라이빗 네트워크를 활용하여 트래픽을 최적 경로로 라우팅

### 6-2. 동작 방식

```
사용자
  ↓
가장 가까운 AWS 엣지 로케이션 (Anycast IP 수신)
  ↓ AWS 내부 프라이빗 네트워크
애플리케이션 (Elastic IP / EC2 / ALB / NLB)
```

- **2개의 Anycast IP** 생성 → 전 세계 어디서든 동일 IP로 접근
- **Anycast IP**: 여러 서버가 동일 IP를 공유, 가장 가까운 서버로 자동 라우팅

### 6-3. 지원 엔드포인트

- Elastic IP
- EC2 인스턴스
- ALB (퍼블릭 또는 프라이빗)
- NLB (퍼블릭 또는 프라이빗)

### 6-4. 주요 특징

| 특징 | 설명 |
|---|---|
| 일관된 성능 | 클라이언트 캐시 문제 없음 (고정 Static IP) |
| 지능적 라우팅 | 가장 낮은 레이턴시 엔드포인트로 자동 라우팅 |
| 빠른 리전 Failover | 헬스체크 기반, 수초 내 장애 감지 및 전환 |
| 보안 | 2개 IP만 화이트리스트 등록; Shield로 DDoS 보호 |
| 헬스체크 | 내장 헬스체크 → DR(재해복구) 시나리오에 적합 |

### 6-5. CloudFront vs Global Accelerator

| 항목 | CloudFront | Global Accelerator |
|---|---|---|
| 핵심 기능 | 콘텐츠 캐싱 | TCP/UDP 트래픽 가속 |
| 프로토콜 | HTTP/HTTPS | TCP, UDP (비-HTTP 포함) |
| IP 유형 | 동적 (변경 가능) | **정적 Anycast IP** |
| 캐싱 | 있음 | **없음** |
| 게임 / IoT / VoIP | 부적합 | 적합 |
| 적합한 케이스 | 정적/동적 웹 콘텐츠 배포 | 비-HTTP 프로토콜, IP 화이트리스팅 필요 시 |

> 💡 **시험 포인트**:
> - Global Accelerator = **정적 Anycast IP 2개** + AWS 내부망 경유 → 캐싱 없음
> - 게임(UDP), IoT, VoIP 등 **비-HTTP** 트래픽 가속 → Global Accelerator
> - IP 화이트리스팅이 필요한 경우 → 고정 IP인 Global Accelerator
> - CloudFront는 캐싱 중심, Global Accelerator는 네트워크 경로 최적화 중심

---

← [Section 14. 아마존 S3 보안](section14.md) | [Section 16. AWS 스토리지 추가 기능](section16.md) →
