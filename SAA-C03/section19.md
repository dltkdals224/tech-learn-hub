# Section 19. 솔루션 설계자 관점의 서버리스 개요

## 목차

1. [서버리스 개요](#1-서버리스-개요)
2. [AWS Lambda](#2-aws-lambda)
3. [Lambda 고급 기능](#3-lambda-고급-기능)
4. [Amazon DynamoDB](#4-amazon-dynamodb)
5. [Amazon API Gateway](#5-amazon-api-gateway)
6. [AWS Step Functions](#6-aws-step-functions)
7. [Amazon Cognito](#7-amazon-cognito)

---

## 1. 서버리스 개요

### 1.1 서버리스란?

- **Serverless**: 개발자가 서버를 관리하거나 프로비전하지 않아도 되는 패러다임
- 초기에는 **FaaS(Function as a Service)** 개념 = AWS Lambda
- 현재는 관리형 데이터베이스, 스토리지, API 등 다양한 서비스로 확장

### 1.2 AWS 서버리스 서비스 목록

| 카테고리 | 서비스 |
|---|---|
| 컴퓨팅 | Lambda, Fargate |
| 데이터베이스 | DynamoDB, Aurora Serverless |
| 스토리지 | S3 |
| API | API Gateway |
| 메시징 | SNS, SQS, Kinesis Firehose |
| 인증 | Cognito |
| 오케스트레이션 | Step Functions |

> 💡 **시험 포인트**:
> - 서버리스 = 서버 관리 불필요, 자동 스케일링
> - Lambda는 대표적인 FaaS 서비스
> - DynamoDB, S3, Cognito 모두 서버리스 서비스에 포함
> - 시험에서 "서버리스 아키텍처" 요구 시 위 서비스 조합을 떠올릴 것

---

## 2. AWS Lambda

### 2.1 EC2 vs Lambda 비교

| 항목 | EC2 | Lambda |
|---|---|---|
| 서버 | 프로비전 필요 | 불필요 (가상 함수) |
| 실행 제한 | 무제한 | **최대 15분** |
| 확장 | 수동/ASG | **자동** |
| 비용 | 실행 시간 기준 | 호출 횟수 + 실행 시간 |

### 2.2 Lambda 주요 특징 및 이점

- **요금**: 요청 수 + 컴퓨팅 시간 기반 과금
- **프리 티어**: 월 100만 요청, 40만 GB-초 컴퓨팅
- **지원 언어**: Node.js, Python, Java, C#/.NET, Golang, Ruby, **Custom Runtime** (Rust 등)
- **Lambda Container Image**: Lambda Runtime API 구현 필요 (임의 Docker 이미지는 ECS/Fargate 권장)
- **RAM**: 최대 10GB까지 설정 가능 → RAM 증가 시 CPU 및 네트워크 성능도 함께 향상

### 2.3 Lambda 통합 서비스

API Gateway, Kinesis, DynamoDB, S3, CloudFront, CloudWatch Events/EventBridge, CloudWatch Logs, SNS, SQS, Cognito

### 2.4 동기식 호출 (Synchronous Invocations)

- 결과를 즉시 반환
- 에러 처리는 **클라이언트** 측에서 담당
- 호출 방법: CLI, SDK, API Gateway, **ALB**, CloudFront(Lambda@Edge), S3 Batch, Cognito, Step Functions

**ALB → Lambda 흐름:**

```
HTTP 요청 → ALB → JSON 이벤트 변환 → Lambda
Lambda 응답 → statusCode, headers, body, isBase64Encoded → ALB → HTTP 응답
```

- Lambda를 **타겟 그룹**으로 등록 필요
- **Multi-Header Values**: 동일 키의 query params/headers → 배열로 변환

### 2.5 비동기식 호출 (Asynchronous Invocations)

- **S3, SNS, CloudWatch Events/EventBridge**: 이벤트를 Event Queue에 적재 → Lambda가 처리
- 에러 시 재시도: **최대 3회** (1분, 2분 대기)
- 처리 로직의 **멱등성(idempotent)** 보장 필수
- **DLQ(Dead Letter Queue)**: SQS 또는 SNS로 실패 이벤트 처리

### 2.6 이벤트 소스 매핑 (Event Source Mapping)

- 대상 서비스: **Kinesis Data Streams, SQS & SQS FIFO, DynamoDB Streams**
- 소스에서 **폴링(polling)** 방식으로 레코드를 가져옴
- Lambda가 **이벤트 배치**와 함께 동기식으로 호출됨

### 2.7 Lambda Destinations

- **비동기 호출** 및 **이벤트 소스 매핑**의 결과를 대상 서비스로 전달
- 성공/실패 모두: SQS, SNS, Lambda, EventBridge bus로 라우팅
- **DLQ보다 권장** (더 높은 유연성)

> 💡 **시험 포인트**:
> - ALB → Lambda: 타겟 그룹 등록 필수
> - 비동기 호출 재시도 횟수: **3회**
> - Event Source Mapping 대상: Kinesis, SQS, DynamoDB Streams
> - Destinations vs DLQ: Destinations가 더 유연하여 권장

---

## 3. Lambda 고급 기능

### 3.1 Lambda 실행 역할 (IAM)

- Lambda 함수에 AWS 서비스 접근 권한 부여
- **Best Practice**: Lambda 함수당 하나의 IAM 역할 생성
- **리소스 기반 정책**: 다른 계정/서비스가 Lambda를 호출할 수 있도록 허용

### 3.2 환경 변수 (Environment Variables)

- Key/Value 쌍으로 설정
- **KMS**로 암호화 가능
- 코드 재배포 없이 함수 동작 조정 가능

### 3.3 로깅 및 모니터링

| 서비스 | 내용 |
|---|---|
| CloudWatch Logs | Lambda 실행 로그 저장 |
| CloudWatch Metrics | 호출 수, 실행 시간, 동시 실행 수, 에러, 성공률, 스로틀 |
| X-Ray | 분산 추적; **Active Tracing** 활성화 + X-Ray SDK 코드 삽입 필요 |

### 3.4 VPC 내 Lambda

```
기본 설정: AWS 소유 VPC (프라이빗 VPC 리소스 접근 불가)

프라이빗 VPC 배포 시:
VPC + 서브넷 + 보안 그룹 지정
→ Lambda가 서브넷에 ENI 생성
→ 인터넷 접근: 프라이빗 서브넷 → NAT Gateway → IGW
→ AWS 서비스 프라이빗 접근: VPC Endpoint 사용
```

### 3.5 함수 구성 (Configuration)

| 항목 | 범위/기본값 |
|---|---|
| RAM | 128MB ~ 10GB (64MB 단위) |
| 타임아웃 | 기본 3초, 최대 **900초 (15분)** |
| /tmp 디스크 | 512MB ~ 10GB |
| 실행 컨텍스트 | 재사용 가능한 임시 런타임 환경 |

- **초기화 코드** (DB 연결, SDK 클라이언트 등): **핸들러 함수 외부**에 배치 → 재사용으로 성능 향상

### 3.6 동시 실행 (Concurrency)

- 계정당 최대 **1,000개** 동시 실행 (소프트 한도)
- **예약 동시성(Reserved Concurrency)**: 특정 함수의 최대 동시 실행 수 보장; 초과 시 스로틀
- **프로비전된 동시성(Provisioned Concurrency)**: 함수 호출 전 사전 준비 → **콜드 스타트** 제거
- **콜드 스타트**: 새 인스턴스 생성 시 초기화 코드 실행 → 첫 요청 지연 발생

### 3.7 Lambda Layers

- 여러 함수에 공통 코드/의존성 재사용
- **Custom Runtime** 지원 (C++, Rust 등)
- 함수 의존성 외부화 → 배포 패키지 크기 감소

### 3.8 Lambda 한도 요약 (리전별)

| 항목 | 한도 |
|---|---|
| 메모리 | 128MB - 10GB |
| 최대 실행 시간 | **15분** |
| 환경 변수 | 4KB |
| /tmp 디스크 | 512MB - 10GB |
| 동시 실행 | 1,000 |
| 배포 패키지 | 50MB(압축), 250MB(비압축) |
| 컨테이너 이미지 | 10GB |

### 3.9 Lambda@Edge

- CloudFront **엣지 로케이션**에 전 세계적으로 배포
- CloudFront 요청/응답을 사용자 요청에 가깝게 처리

> 💡 **시험 포인트**:
> - 콜드 스타트 해결: **Provisioned Concurrency**
> - VPC 내 Lambda의 인터넷 접근: **NAT Gateway** 필수
> - Lambda 최대 실행 시간: **15분 (900초)**
> - 초기화 코드(DB 연결 등)는 핸들러 외부에 배치하여 재사용

---

## 4. Amazon DynamoDB

### 4.1 개요

- **완전관리형** 고가용성 NoSQL DB (멀티 AZ 복제)
- 관계형 DB 아님; 조인 미지원
- 대규모 워크로드 지원: 초당 수백만 요청, 수조 개 행, 수백 TB 스토리지
- **한 자릿수 밀리초** 지연 시간
- **IAM**으로 보안/인증/관리
- 저비용, 자동 스케일링

### 4.2 기본 개념

- **테이블**: 생성 시 기본 키 결정; 항목 수 무제한; 최대 항목 크기 **400KB**
- **속성(Attributes)**: 컬럼; 나중에 추가 가능; null 허용
- **데이터 타입**:
  - 스칼라: String, Number, Binary, Boolean, Null
  - 문서: List, Map
  - 집합: String Set, Number Set, Binary Set
- **기본 키 옵션**:
  - **파티션 키만 (HASH)**: 고유해야 함
  - **파티션 키 + 정렬 키 (HASH + RANGE)**: 조합이 고유해야 함

### 4.3 읽기/쓰기 용량 모드

| 항목 | Provisioned | On-Demand |
|---|---|---|
| 처리량 | RCU/WCU 미리 지정 | 자동 스케일링 |
| 비용 | 낮음 (예측 가능한 워크로드) | 높음 (예측 불가한 워크로드) |
| 스로틀링 | 초과 시 발생 | 없음 |

- **RCU (Read Capacity Unit)**: 강력한 일관성 읽기 1회/초 (4KB), 또는 최종적 일관성 읽기 2회/초 (4KB)
- **WCU (Write Capacity Unit)**: 쓰기 1회/초 (1KB)
- **버스트 용량(Burst Capacity)**: 일시적 초과 용량 허용
- Provisioned 모드에서 **Auto Scaling** 사용 가능

### 4.4 DynamoDB Accelerator (DAX)

- DynamoDB용 **인메모리 캐시**
- **마이크로초** 지연 시간 (밀리초 → 마이크로초)
- 기본 TTL: **5분**
- 애플리케이션 로직 변경 불필요 (호환 API)
- 개발: 단일 노드; 운영: **3+ AZ 멀티 노드**

| 비교 | DAX | ElastiCache |
|---|---|---|
| 용도 | 개별 객체 캐시, 쿼리/스캔 캐시 | 집계 결과 캐시 |
| 변경 | 앱 코드 변경 불필요 | 앱 코드 변경 필요 |

### 4.5 DynamoDB Streams

- 테이블의 항목 수준 변경사항을 **순서 있는 스트림**으로 제공
- 스트림 레코드 처리: Kinesis Data Streams 전송, Lambda 읽기, KCL 앱 읽기
- 데이터 보존: **24시간**
- **스트림 유형**:
  - `KEYS_ONLY`: 변경된 항목의 키 속성만
  - `NEW_IMAGE`: 변경 후 전체 항목
  - `OLD_IMAGE`: 변경 전 전체 항목
  - `NEW_AND_OLD_IMAGES`: 변경 전/후 전체 항목
- **활용**: 실시간 변경 반응, 파생 테이블 삽입, 교차 리전 복제, Lambda 트리거

### 4.6 글로벌 테이블 (Global Tables)

- **멀티 리전, 멀티 액티브** DynamoDB 테이블
- **Active-Active 복제**: 모든 리전에서 읽기/쓰기 가능
- 사전 요건: **DynamoDB Streams** 활성화 필수
- 활용: 글로벌 저지연, DR(재해 복구)

### 4.7 TTL (Time To Live)

- 만료 타임스탬프 이후 항목 자동 삭제
- 추가 비용 없음; 삭제 시 WCU 소비 안 함
- TTL 속성(epoch 타임스탬프) 설정 → DynamoDB가 현재 시간과 비교하여 처리

### 4.8 백업

| 방식 | 특징 |
|---|---|
| **PITR** (Point-in-time Recovery) | 선택적 활성화; 최근 **35일** 연속 백업; 임의 시점 복원; 새 테이블 생성 |
| **온디맨드 백업** | 장기 보존용 전체 백업; 성능 영향 없음; 수동 생성/삭제; 교차 리전 복사 가능 |

### 4.9 S3 연동

- **S3로 내보내기** (PITR 활성화 필요):
  - 최근 35일 내 데이터 내보내기
  - RCU 소비 없음
  - DynamoDB JSON 또는 ION 형식
- **S3에서 가져오기**:
  - CSV, DynamoDB JSON, ION 지원
  - WCU 소비 없음
  - 새 테이블 생성
  - 가져오기 오류는 CloudWatch Logs에 기록

> 💡 **시험 포인트**:
> - DynamoDB Streams 보존 기간: **24시간**
> - Global Tables 사전 요건: **DynamoDB Streams 활성화**
> - DAX: 앱 코드 변경 없이 마이크로초 캐싱
> - PITR: 최대 **35일** 이내 임의 시점 복원 가능

---

## 5. Amazon API Gateway

### 5.1 주요 기능

- **AWS Lambda + API Gateway** = 인프라 관리 불필요
- **WebSocket 프로토콜** 지원
- API 버전 관리 (v1, v2)
- 환경 분리 (dev, test, prod)
- 보안 (인증/인가)
- **API 키 생성**, 요청 스로틀링
- Swagger/OpenAPI 가져오기
- 요청/응답 변환 및 검증
- SDK 및 API 스펙 자동 생성
- **API 응답 캐싱**

### 5.2 통합 유형

| 통합 유형 | 설명 |
|---|---|
| Lambda Function | Lambda 호출; 쉬운 REST API 백엔드 구성 |
| HTTP | 온프레미스 HTTP API, ALB 등 HTTP 엔드포인트 노출; 속도 제한·캐싱·사용자 인증·API 키 추가 |
| AWS Service | AWS 서비스를 API로 노출 (Step Functions 시작, SQS 메시지 게시 등) |

### 5.3 엔드포인트 유형

| 유형 | 설명 |
|---|---|
| **Edge-Optimized** (기본) | 글로벌 클라이언트; CloudFront 엣지를 통해 요청 라우팅; API Gateway는 단일 리전 |
| **Regional** | 동일 리전 클라이언트; 수동으로 CloudFront 결합 가능 |
| **Private** | VPC Interface Endpoint(ENI)를 통해서만 접근; 리소스 정책으로 접근 제어 |

### 5.4 보안

- **사용자 인증**: IAM 역할, Cognito, **Custom Authorizer (Lambda)**
- 커스텀 도메인 이름 + **HTTPS via ACM**
- Edge-Optimized: 인증서 반드시 **us-east-1** 리전에 있어야 함
- Regional: 같은 리전에 인증서 필요
- **Route 53**에서 CNAME 또는 A-alias 레코드 설정

> 💡 **시험 포인트**:
> - API Gateway + Lambda = 완전 서버리스 REST API
> - Private 엔드포인트: VPC Interface Endpoint 필수
> - Edge-Optimized ACM 인증서: **반드시 us-east-1**
> - Custom Authorizer = Lambda Authorizer (토큰 기반 인증)

---

## 6. AWS Step Functions

### 6.1 개요

- 워크플로우를 **상태 머신(State Machine)** 으로 모델링 (JSON 정의)
- Lambda, ECS, EC2, SQS, DynamoDB 등 오케스트레이션
- 순서 실행, 병렬 실행, 조건, 타임아웃, **에러 처리** 지원
- 최대 실행 기간: **1년**
- **사람 승인(Human Approval)** 단계 구현 가능

### 6.2 활용 사례

- 주문 처리 워크플로우
- 데이터 처리 파이프라인
- ETL 자동화
- 인프라 자동화

> 💡 **시험 포인트**:
> - Step Functions 최대 실행 시간: **1년**
> - 복잡한 Lambda 체이닝 → Step Functions으로 대체
> - 사람 승인 단계 포함 가능
> - 시각적 워크플로우 모니터링 제공

---

## 7. Amazon Cognito

### 7.1 개요

웹/모바일 앱 사용자에게 **아이덴티티(Identity)** 제공

### 7.2 Cognito User Pools (CUP)

- 앱 사용자를 위한 **로그인 기능** 제공
- **서버리스 사용자 데이터베이스**
- 지원 기능: 사용자명+비밀번호, MFA, 이메일/전화 인증, 소셜 로그인 (Facebook/Google/SAML/OpenID)
- **JWT(JSON Web Token)** 반환
- **API Gateway** 및 **ALB**와 통합하여 인증 처리

### 7.3 Cognito Identity Pools (Federated Identity)

- 사용자에게 **AWS 자격 증명** 제공 → AWS 서비스 직접 접근
- 인증 방법: Cognito User Pool, 소셜 IdP, SAML, OpenID
- **임시 AWS 자격 증명** 반환 (STS 통해)
- 사용자가 S3/DynamoDB에 직접 접근 가능 (IAM 정책의 **policy variables**로 사용자별 권한 설정)

### 7.4 User Pool vs Identity Pool 비교

| 항목 | User Pool | Identity Pool |
|---|---|---|
| 기능 | 사용자 DB, 인증 | AWS 자격 증명 발급 |
| 사용처 | API Gateway / ALB 인증 | AWS 서비스 직접 접근 |
| 반환값 | **JWT** | **AWS 임시 자격 증명** |

> 💡 **시험 포인트**:
> - Cognito User Pool: 사용자 인증 → JWT 반환
> - Cognito Identity Pool: AWS 자격 증명(임시) 발급 → S3, DynamoDB 직접 접근
> - User Pool + Identity Pool 조합: 인증 후 AWS 서비스 직접 접근 패턴
> - ALB와 통합 가능한 것: **User Pool**

---

← [Section 18. AWS의 컨테이너: ECS, Fargate, ECR 및 EKS](section18.md) | [Section 20. 서버리스 솔루션 아키텍처 토론](section20.md) →
