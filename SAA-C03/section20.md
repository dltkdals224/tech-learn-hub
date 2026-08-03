# Section 20. 서버리스 솔루션 아키텍처 토론

## 목차

1. [Mobile Application: MyTodoList](#1-mobile-application-mytodolist)
2. [Website: MyBlog.com](#2-website-myblogcom)
3. [Micro Services 아키텍처](#3-micro-services-아키텍처)
4. [Distributing Paid Content](#4-distributing-paid-content)
5. [Big Data Ingestion Pipeline](#5-big-data-ingestion-pipeline)
6. [서버리스 아키텍처 요약](#6-서버리스-아키텍처-요약)

---

## 1. Mobile Application: MyTodoList

### 1.1 요구사항

- HTTPS REST API 제공
- **서버리스** 구조
- 사용자가 S3와 직접 상호작용 가능
- 관리형 서버리스 인증
- 서버리스 DB에 읽기/쓰기
- DynamoDB 응답 **캐싱** 지원

### 1.2 아키텍처 설계

```
모바일 클라이언트
  │
  ├─[REST HTTPS]→ API Gateway
  │                   │
  │               [인증]↔ Cognito User Pool
  │                   │
  │               [호출]→ Lambda → DynamoDB (읽기/쓰기)
  │                              └→ ElastiCache (캐싱)
  │
  └─[인증]→ Cognito Identity Pool → [임시 AWS 자격 증명]
                                        │
                                        └→ S3 (직접 접근)
```

### 1.3 구성 요소별 역할

| 구성 요소 | 역할 |
|---|---|
| **API Gateway** | REST HTTPS 엔드포인트 제공 |
| **Cognito User Pool** | 사용자 인증 (JWT 발급) |
| **Lambda** | 비즈니스 로직 처리 |
| **DynamoDB** | 서버리스 NoSQL 데이터 저장 |
| **ElastiCache** | DynamoDB 응답 캐싱 |
| **Cognito Identity Pool** | S3 직접 접근용 임시 AWS 자격 증명 발급 |
| **S3** | 사용자 파일 저장 |

### 1.4 핵심 포인트

- **API Gateway + Lambda**: 서버리스 REST API의 핵심 조합
- **Cognito User Pool**: 사용자 인증 처리
- **DynamoDB**: 서버리스 데이터베이스
- **ElastiCache**: DynamoDB 읽기 캐싱으로 성능 향상
- **Cognito Identity Pool + IAM**: S3 직접 접근 패턴

> 💡 **시험 포인트**:
> - 모바일 앱에서 S3 직접 업로드: **Cognito Identity Pool** → 임시 자격 증명 사용
> - DynamoDB 캐싱: ElastiCache (집계 결과) 또는 DAX (개별 항목)
> - 서버리스 REST API = API Gateway + Lambda + DynamoDB 조합
> - Cognito User Pool(인증) ≠ Identity Pool(AWS 자격 증명 발급)

---

## 2. Website: MyBlog.com

### 2.1 요구사항

- **글로벌 스케일** 지원
- 대부분 읽기 작업
- **정적 웹사이트** 제공
- 블로그 포스트는 S3에 저장
- 새 포스트 **안전하게 업로드**
- 인증된 사용자만 서버리스 REST API 사용

### 2.2 아키텍처 설계

```
정적 콘텐츠 흐름:
사용자 → Route 53 → CloudFront (엣지 캐싱)
                         └→ S3 (OAI/OAC로 보호)

REST API 흐름:
인증된 사용자 → Cognito User Pool (JWT)
             → API Gateway → Lambda → DynamoDB Global Tables

이미지 업로드 흐름:
사용자 → CloudFront → S3 (Pre-signed URL 또는 OAC)
```

### 2.3 구성 요소별 역할

| 구성 요소 | 역할 |
|---|---|
| **S3** | 정적 웹사이트 파일, 블로그 포스트, 이미지 저장 |
| **CloudFront** | 글로벌 콘텐츠 배포 (엣지 캐싱) |
| **OAI/OAC** | CloudFront만 S3에 접근 가능하도록 제한 |
| **Route 53** | DNS 관리 |
| **Cognito User Pool** | 사용자 인증 |
| **API Gateway** | REST API 엔드포인트 |
| **Lambda** | 비즈니스 로직 |
| **DynamoDB Global Tables** | 멀티 리전 복제 데이터 저장 |

### 2.4 핵심 포인트

- **CloudFront + S3**: EC2 없이 정적 콘텐츠 글로벌 배포
- **OAI/OAC**: S3 버킷을 CloudFront를 통해서만 접근 가능하게 보호
- **DynamoDB Global Tables**: 전 세계 읽기 성능 최적화
- **Pre-signed URL**: 안전한 S3 업로드 처리

> 💡 **시험 포인트**:
> - 정적 웹사이트 글로벌 배포: **S3 + CloudFront**
> - S3 직접 접근 차단: **OAI(Origin Access Identity)** 또는 **OAC(Origin Access Control)**
> - DynamoDB 글로벌 읽기 성능: **Global Tables** (Active-Active)
> - 안전한 S3 업로드: **Pre-signed URL** 사용

---

## 3. Micro Services 아키텍처

### 3.1 과제

- 수백 개의 서비스가 서로 상호작용
- 서비스 간 통신 방식 결정 (동기 vs 비동기)
- 분산 서비스 디버깅 복잡성

### 3.2 통신 패턴

| 패턴 | 구성 | 특징 |
|---|---|---|
| **동기식** | API Gateway → Lambda (또는 ECS) | 즉각적 응답 필요 시 |
| **비동기식** | SQS → Lambda | 결합도 낮춤, 버퍼링 |
| **이벤트 기반** | Kinesis → Lambda | 스트리밍 데이터 처리 |

### 3.3 자유 설계 원칙

- 각 마이크로서비스를 필요에 따라 자유롭게 설계
- 서비스별 **독립적 개발, 스케일링, 배포** 가능
- 서비스 검색(Service Discovery): 로드 밸런서, DNS, **API Gateway**

```
마이크로서비스 예시 구조:

서비스 A (서버리스)           서비스 B (컨테이너)
API Gateway → Lambda    ←→   ECS / Fargate
     ↓                            ↓
  DynamoDB                     RDS/Aurora
     
       ↕ SQS/SNS (비동기 통신)
       
서비스 C (서버 기반)
EC2 + ALB
     ↓
  ElastiCache
```

### 3.4 마이크로서비스 구현 시 고려사항

- **동기 통신**: API Gateway 사용
- **비동기 통신**: SQS/SNS 사용
- 각 서비스는 서버리스 또는 EC2/ECS 자유 선택
- **X-Ray**: 분산 추적으로 서비스 간 흐름 모니터링

> 💡 **시험 포인트**:
> - 마이크로서비스 동기 통신: **API Gateway**
> - 마이크로서비스 비동기 통신: **SQS/SNS**
> - 분산 추적 및 디버깅: **X-Ray**
> - 각 서비스는 서버리스·컨테이너·EC2 중 독립적으로 선택 가능

---

## 4. Distributing Paid Content

### 4.1 요구사항

- 동영상 온라인 판매
- **프리미엄 사용자만** 다운로드 가능
- 프리미엄 사용자 DB 관리
- **글로벌 배포**

### 4.2 아키텍처 설계

```
1. 사용자 → Cognito User Pool (로그인)
               ↓
2. API Gateway → Lambda → DynamoDB (멤버십 확인)
               ↓ (프리미엄 확인 완료)
3. Lambda → CloudFront Signed URL 생성 → 클라이언트에 반환
               ↓
4. 클라이언트 → Signed URL로 CloudFront 접근 → S3 (동영상)

* S3에는 OAC 설정 → CloudFront를 통해서만 접근 가능
```

### 4.3 구성 요소별 역할

| 구성 요소 | 역할 |
|---|---|
| **Cognito User Pool** | 프리미엄 사용자 인증 |
| **API Gateway** | REST API 엔드포인트 |
| **Lambda** | 멤버십 검증 + Signed URL 생성 |
| **DynamoDB** | 프리미엄 멤버십 정보 저장 |
| **CloudFront Signed URL** | 프리미엄 콘텐츠 한시적 접근 허용 |
| **S3** | 동영상 파일 저장 |
| **OAC** | S3 직접 접근 차단 |

### 4.4 핵심 포인트

- **CloudFront Signed URLs**: 프리미엄 콘텐츠 보호의 핵심
- **Cognito + Lambda + DynamoDB**: 멤버십 인증 체계
- **OAC on S3**: S3 직접 접근 차단 → CloudFront 경유 강제

> 💡 **시험 포인트**:
> - 유료 콘텐츠 보호: **CloudFront Signed URL** (S3 Pre-signed URL과 구분)
> - Signed URL: 개별 파일 접근 제어 / Signed Cookie: 여러 파일 접근 제어
> - S3 직접 접근 방지: OAC(신규) 또는 OAI(구) 설정 필수
> - CloudFront Signed URL vs S3 Pre-signed URL: CDN 캐싱 여부 차이

---

## 5. Big Data Ingestion Pipeline

### 5.1 요구사항

- **완전 서버리스** 수집 파이프라인
- **실시간** 데이터 수집
- 데이터 변환 처리
- SQL로 변환된 데이터 쿼리 가능
- S3에 리포트 저장
- 데이터 웨어하우스로 적재
- **시각화 대시보드** 제공

### 5.2 아키텍처 설계

```
[수집 단계]
IoT 디바이스 → IoT Core → Kinesis Data Streams
                                    ↓
[저장 단계]              Kinesis Data Firehose → S3 (수집 버킷)
                                    ↓
[변환 단계]              S3 이벤트 트리거 → Lambda (데이터 변환)
                                    ↓
                         Kinesis Data Firehose → S3 (리포팅 버킷)
                                    ↓
[분석 단계]              Athena (서버리스 SQL 쿼리)
                                    ↓
                         S3 (쿼리 결과) → QuickSight (시각화)
                                    ↓
[웨어하우스]             Redshift Spectrum (웨어하우스 쿼리)
```

### 5.3 구성 요소별 역할

| 구성 요소 | 역할 |
|---|---|
| **IoT Core** | IoT 디바이스 데이터 수집 |
| **Kinesis Data Streams** | 실시간 스트리밍 데이터 처리 |
| **Kinesis Data Firehose** | 근실시간(near-real-time) 데이터 S3 적재 |
| **Lambda** | Firehose 내 데이터 변환 |
| **S3** | 수집 버킷 / 리포팅 버킷 |
| **Athena** | 서버리스 SQL (S3 데이터 쿼리) |
| **QuickSight** | BI 시각화 대시보드 |
| **Redshift Spectrum** | 데이터 웨어하우스 쿼리 |

### 5.4 핵심 포인트

- **IoT Core**: IoT 디바이스 데이터의 진입점
- **Kinesis Data Streams**: 실시간 스트리밍
- **Kinesis Firehose**: 근실시간 데이터 로딩 (완전 관리형)
- **Athena**: S3에 저장된 데이터를 서버리스 SQL로 직접 쿼리
- **QuickSight**: 서버리스 BI 시각화

> 💡 **시험 포인트**:
> - IoT 데이터 수집: **IoT Core → Kinesis Data Streams**
> - 실시간 스트리밍: Kinesis Data Streams / 근실시간 적재: **Kinesis Firehose**
> - S3 데이터 서버리스 SQL 분석: **Athena**
> - Athena + QuickSight 조합: 서버리스 분석 대시보드의 정석 패턴

---

## 6. 서버리스 아키텍처 요약

### 6.1 서비스-요구사항 매핑 테이블

| 요구사항 | 서비스 |
|---|---|
| REST API | **API Gateway + Lambda** |
| 사용자 인증 | **Cognito User Pool** |
| AWS 자격 증명 발급 | **Cognito Identity Pool** |
| 서버리스 NoSQL DB | **DynamoDB** |
| 서버리스 관계형 DB | **Aurora Serverless** |
| DynamoDB 캐싱 (개별 항목) | **DAX** |
| 일반 캐싱 (집계 결과) | **ElastiCache** |
| 정적 콘텐츠 배포 | **S3 + CloudFront** |
| 실시간 스트리밍 수집 | **Kinesis Data Streams** |
| 데이터 저장소 로드 | **Kinesis Firehose** |
| SQL 분석 (S3) | **Athena** |
| BI 시각화 | **QuickSight** |
| 워크플로우 오케스트레이션 | **Step Functions** |
| S3 직접 접근 | **Cognito Identity Pool** |
| 유료 콘텐츠 보호 | **CloudFront Signed URL** |
| IoT 데이터 수집 | **IoT Core** |
| 분산 추적 | **X-Ray** |

### 6.2 아키텍처 패턴별 핵심 조합

**서버리스 REST API:**
```
API Gateway → Lambda → DynamoDB
                    └→ ElastiCache (선택적 캐싱)
```

**서버리스 인증 + S3 직접 접근:**
```
Cognito User Pool (인증) → JWT
Cognito Identity Pool → 임시 AWS 자격 증명 → S3
```

**서버리스 빅데이터 분석:**
```
Kinesis → Firehose → S3 → Athena → QuickSight
```

**전역 정적 웹사이트:**
```
Route 53 → CloudFront → S3 (OAC)
```

> 💡 **시험 포인트**:
> - 서버리스 = 인프라 관리 없음 + 자동 스케일링 + 사용량 기반 과금
> - 아키텍처 문제에서 "서버리스" 키워드: API GW, Lambda, DynamoDB, Cognito, S3 조합 우선 고려
> - DAX vs ElastiCache: DynamoDB 전용 캐시는 **DAX**, 범용 캐시는 **ElastiCache**
> - 실시간(real-time) vs 근실시간(near-real-time): Streams vs Firehose 구분 필수

---

← [Section 19. 솔루션 설계자 관점의 서버리스 개요](section19.md) | [Section 21. AWS의 데이터베이스](section21.md) →
