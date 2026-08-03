# Section 26. AWS 보안 및 암호화: KMS, SSM Parameter Store, CloudHSM, Shield, WAF

## 목차

1. [암호화 개요](#1-암호화-개요)
2. [AWS KMS](#2-aws-kms-key-management-service)
3. [SSM Parameter Store](#3-ssm-parameter-store)
4. [AWS Secrets Manager](#4-aws-secrets-manager)
5. [AWS Certificate Manager (ACM)](#5-aws-certificate-manager-acm)
6. [AWS WAF](#6-aws-waf-web-application-firewall)
7. [AWS Shield](#7-aws-shield)
8. [AWS Firewall Manager](#8-aws-firewall-manager)
9. [Amazon GuardDuty](#9-amazon-guardduty)
10. [Amazon Inspector](#10-amazon-inspector)
11. [Amazon Macie](#11-amazon-macie)
12. [보안 서비스 요약](#12-보안-서비스-요약)

---

## 1. 암호화 개요

### Encryption in Flight (SSL/TLS)

- 전송 전 데이터 암호화 → 수신 후 복호화
- **SSL 인증서**가 HTTPS 활성화
- **MITM(Man-In-The-Middle) 공격** 방어

---

### Server-side Encryption at Rest

- 서버가 데이터 수신 후 암호화 → 전송 전 복호화
- 암호화/복호화 키는 **KMS**에서 관리

---

### Client-side Encryption

- **클라이언트**가 직접 암호화; 서버는 복호화 불가
- **Envelope Encryption** 사용 가능

> 💡 **시험 포인트**:
> - Encryption in Flight = HTTPS/TLS, MITM 방어
> - Server-side = 서버가 암/복호화, KMS 키 관리
> - Client-side = 서버는 평문 데이터 미접근
> - Envelope Encryption은 Client-side 암호화에서 활용

---

## 2. AWS KMS (Key Management Service)

### 개요

- 암호화 키 관리; 대부분의 AWS 서비스와 통합
- **CloudTrail**을 통한 키 사용 감사 가능
- **절대 평문으로 시크릿 저장 금지**
- API 호출(SDK, CLI)을 통한 KMS 키 암호화 사용 가능

---

### Key Types 비교

| 항목 | AWS Managed Keys | Customer Managed Keys (CMK) | Customer-Owned Keys |
|---|---|---|---|
| 생성 | AWS 자동 생성 | 사용자 생성 | 사용자 소유/관리 |
| 비용 | 무료 | $1/month | CloudHSM에서 관리 |
| 로테이션 | 자동 (1년) | 수동 또는 자동 | 수동 |

---

### Key Material Origin

| 옵션 | 설명 |
|---|---|
| **KMS** | AWS가 KMS HSM 내에서 키 소재 생성/관리 |
| **External** | 사용자 직접 키 소재 가져오기 (수동 로테이션 필수) |
| **AWS CloudHSM** | 사용자 CloudHSM 클러스터에서 키 소재 생성 |

---

### Key Policies

- KMS 키 접근 제어 (S3 버킷 정책과 유사)
- **Default**: 루트 사용자 키 접근 가능; 키 정책 없으면 아무도 접근 불가
- **Custom**: 특정 사용자/역할 지정; **크로스 계정 접근** 부여 가능

---

### Multi-Region Keys

- 기본 키를 보조 리전에 복제
- **모든 리전에서 동일한 Key ID** 사용; 한 리전에서 암호화 → 다른 리전에서 복호화 가능
- 글로벌이 아님; 각 리전에서 독립적 키 (동일 키 소재만 공유)
- **사용 사례**: Active-Active 앱, DynamoDB Global Tables, Global Aurora, 동일 CMK로 S3 글로벌 복제

---

### S3 Replication + 암호화

| 암호화 유형 | 복제 여부 |
|---|---|
| Unencrypted / SSE-S3 | 기본 복제 |
| SSE-C | 복제 불가 |
| SSE-KMS | 대상 리전 키 지정 필요; S3 복제 역할에 키 사용 KMS Key Policy 권한 필요 |

---

### AMI 계정 간 공유 + KMS

```
1. Customer KMS Key로 암호화된 AMI 생성
2. 대상 계정과 AMI 공유
3. KMS Key Policy에 대상 계정 권한 부여
4. 대상 계정에서 AMI로 인스턴스 시작 (재암호화 시 자체 KMS 키 사용)
```

> 💡 **시험 포인트**:
> - SSE-KMS S3 복제 시 대상 리전 KMS 키 + 복제 역할 권한 모두 필요
> - Multi-Region Keys는 글로벌 키가 아닌 동일 키 소재의 독립 복사본
> - Key Policy 없으면 루트 포함 누구도 키 접근 불가
> - External Key Material은 수동 로테이션 필수

---

## 3. SSM Parameter Store

### 개요

- 설정값 및 시크릿을 위한 **보안 저장소**
- **서버리스**, 확장 가능, 내구성 높음, 간단한 SDK
- 선택적 KMS 암호화
- **버전 추적** 지원
- 보안: IAM | 알림: **EventBridge** | 연동: **CloudFormation**

---

### Standard vs Advanced Tier

| 항목 | Standard | Advanced |
|---|---|---|
| 최대 파라미터 수 | 10,000 | 100,000 |
| 최대 값 크기 | 4KB | 8KB |
| 파라미터 정책 | 미지원 | 지원 (만료, 알림) |
| 비용 | 무료 | $0.05/advanced param/month |

---

### Parameter Policies (Advanced 전용)

- **Expiration**: 파라미터 만료 날짜 설정
- **ExpirationNotification**: 만료 전 알림
- **NoChangeNotification**: 변경 없을 시 알림

> 💡 **시험 포인트**:
> - SSM Parameter Store는 Standard Tier 무료
> - Parameter Policies는 Advanced Tier 전용
> - EventBridge 연동으로 파라미터 변경 알림 가능
> - CloudFormation에서 SSM Parameter Store 값 참조 가능

---

## 4. AWS Secrets Manager

### 개요

- **시크릿 저장 전용** 신규 서비스
- X일마다 시크릿 **강제 로테이션** 기능
- 로테이션 시 Lambda를 이용한 시크릿 자동 생성
- **RDS 통합**: MySQL, PostgreSQL, Aurora
- **KMS 암호화 필수**
- 여러 AWS 리전에 시크릿 복제 (Primary/Replica 모델)
- 주요 용도: **RDS 연동** (비교: SSM Parameter Store = 범용)

---

### SSM Parameter Store vs Secrets Manager

| 항목 | SSM Parameter Store | Secrets Manager |
|---|---|---|
| 비용 | 무료 (Standard) | $0.40/secret/month + $0.05/10k API calls |
| 로테이션 | 수동 (Lambda 직접 작성) | 자동 (RDS 등 통합) |
| KMS | 선택적 | 필수 |
| CloudFormation | 지원 | 지원 |
| 주요 용도 | 범용 설정값/시크릿 | RDS/DB 시크릿 특화 |

> 💡 **시험 포인트**:
> - Secrets Manager는 KMS 암호화 필수
> - RDS 자격증명 자동 로테이션은 Secrets Manager 사용
> - SSM Parameter Store는 무료이지만 자동 로테이션 미지원
> - Secrets Manager는 멀티 리전 복제 지원

---

## 5. AWS Certificate Manager (ACM)

### 개요

- **TLS/SSL 인증서** 프로비저닝, 관리, 배포
- 퍼블릭 TLS 인증서 **무료**; 자동 갱신
- **통합 서비스**: Elastic Load Balancers, CloudFront, API Gateway
- **EC2에서 직접 사용 불가** (EC2는 인증서 추출 불가)

---

### ACM Private CA

- 관리형 **Private Certificate Authority (CA)**
- 내부 리소스용 프라이빗 TLS 인증서 발급
- **$400/month/CA**

> 💡 **시험 포인트**:
> - ACM 퍼블릭 인증서는 무료이며 자동 갱신
> - EC2 인스턴스에는 ACM 인증서 직접 사용 불가
> - ACM Private CA는 내부 서비스용 인증서 발급에 사용
> - ALB, CloudFront, API Gateway와 통합 가능

---

## 6. AWS WAF (Web Application Firewall)

### 개요

- 일반적인 웹 익스플로잇으로부터 웹 앱 보호 (**Layer 7 = HTTP**)
- **배포 대상**: ALB, API Gateway, CloudFront, AppSync, Cognito User Pool

---

### Web ACL (Access Control List) Rules

| 규칙 유형 | 설명 |
|---|---|
| **IP Sets** | 최대 10,000 IP; IP 기반 허용/차단 |
| **HTTP Headers/Body/URI** | SQL Injection, XSS 방어 |
| **Size Constraints** | 요청 크기 제한 |
| **Geo-match** | 특정 국가 차단 |
| **Rate-based Rules** | IP당 요청 수 카운트; DDoS 방어 |

---

### AWS Managed Rules

- 공통 위협에 대한 즉시 사용 가능한 규칙 (OWASP Top 10, 봇 등)

> 💡 **시험 포인트**:
> - WAF는 Layer 7(HTTP) 보호
> - Rate-based Rules로 DDoS 방어 가능
> - ALB, API Gateway, CloudFront, AppSync에 배포 가능
> - Geo-match로 특정 국가 IP 차단 가능

---

## 7. AWS Shield

### Shield Standard

- **무료**; 모든 AWS 고객에게 자동 적용
- 보호 대상: **SYN/UDP Floods**, Reflection 공격, Layer 3/4 공격

---

### Shield Advanced

- **$3,000/month/organization**
- 보호 대상: EC2, ELB, CloudFront, Global Accelerator, Route 53
- **24/7 AWS DDoS Response Team (DRT)** 접근 가능
- DDoS 발생 시 추가 비용 보호 (**Cost Protection**)
- WAF 규칙을 통한 **자동 L7 보호**

> 💡 **시험 포인트**:
> - Shield Standard는 무료이며 자동 적용
> - Shield Advanced는 DRT 24/7 지원 + 비용 보호 포함
> - Shield Advanced는 WAF와 통합하여 L7 자동 보호
> - Route 53, Global Accelerator도 Shield Advanced 보호 대상

---

## 8. AWS Firewall Manager

### 개요

- AWS Organization의 **모든 계정**에 걸쳐 보안 규칙 중앙 관리
- **보안 정책 종류:**
  - WAF rules (ALB, API GW, CloudFront)
  - Shield Advanced (ALB, CLB, NLB, EIP, CloudFront)
  - Security Groups (EC2, ALB, ENI)
  - Network Firewall (VPC 레벨)
  - Route 53 Resolver DNS Firewall
- 새로 생성된 리소스에 **자동 적용**

---

### WAF vs Shield vs Firewall Manager 비교

| 서비스 | 역할 |
|---|---|
| **WAF** | 특정 리소스에 Web ACL 규칙 정의 |
| **Shield Advanced** | DDoS 보호 + DRT + 비용 보호; WAF와 연동 |
| **Firewall Manager** | 조직 전체 계정에 WAF/Shield 규칙 중앙 관리 |

> 💡 **시험 포인트**:
> - Firewall Manager는 조직 전체 보안 정책 **중앙 관리**
> - 신규 리소스 생성 시 보안 정책 자동 적용
> - Shield Advanced + Firewall Manager 조합으로 조직 전체 DDoS 보호
> - WAF는 단일 리소스, Firewall Manager는 조직 범위

---

## 9. Amazon GuardDuty

### 개요

- ML 기반 **지능형 위협 탐지**로 AWS 계정 보호
- **입력 소스:**
  - CloudTrail Events/Logs
  - VPC Flow Logs
  - DNS Logs
  - EKS Audit Logs
  - RDS/Aurora 로그인 로그
  - EBS, Lambda, S3

---

### 주요 특징

- **EventBridge**를 통한 자동화 알림
- **암호화폐 공격** 전용 Finding 제공
- **원클릭 활성화**; 30일 무료 체험
- 소프트웨어 설치 불필요

> 💡 **시험 포인트**:
> - GuardDuty는 ML 기반 위협 탐지, 소프트웨어 설치 불필요
> - 암호화폐 마이닝 공격 전용 Finding 존재
> - EventBridge 연동으로 자동 대응 가능
> - VPC Flow Logs, CloudTrail, DNS Logs가 주요 입력 소스

---

## 10. Amazon Inspector

### 개요

- **자동화된 보안 평가** 서비스
- **평가 대상:**

| 대상 | 설명 |
|---|---|
| **EC2 인스턴스** | 에이전트 필요; OS 취약점, 의도치 않은 네트워크 접근 |
| **ECR 컨테이너 이미지** | 푸시 시 스캔 |
| **Lambda 함수** | 코드 + 패키지 의존성 취약점; 배포 시 스캔 |

---

### 주요 특징

- **Security Hub**에 결과 보고; **EventBridge**로 Findings 전송
- **지속적 스캔** (일회성 아님); 우선순위와 함께 리스크 점수 제공

> 💡 **시험 포인트**:
> - EC2 Inspector는 에이전트 설치 필요
> - ECR 이미지는 푸시 시 자동 스캔
> - Lambda는 배포 시 코드 및 패키지 취약점 스캔
> - Inspector는 지속적 스캔이며 Security Hub에 통합

---

## 11. Amazon Macie

### 개요

- ML을 활용한 완전 관리형 **데이터 보안 및 프라이버시** 서비스
- **S3 데이터 분석**으로 민감한 데이터 식별 및 알림
- **PII (Personally Identifiable Information)** 탐지 특화
- **EventBridge** 연동으로 알림 전송

> 💡 **시험 포인트**:
> - Macie는 S3 버킷 내 PII 데이터 탐지에 특화
> - ML 기반으로 민감 데이터 자동 분류
> - EventBridge 연동으로 자동화된 알림 및 대응 가능
> - S3 데이터 보안 감사 시나리오에서 Macie 선택

---

## 12. 보안 서비스 요약

| 서비스 | 목적 |
|---|---|
| **KMS** | 암호화 키 관리 |
| **SSM Parameter Store** | 설정값 및 시크릿 저장 (범용) |
| **Secrets Manager** | 시크릿 저장 + 자동 로테이션 (RDS 특화) |
| **ACM** | TLS/SSL 인증서 관리 |
| **WAF** | 웹 앱 Layer 7 보호 |
| **Shield Standard** | 무료 DDoS 기본 보호 |
| **Shield Advanced** | 고급 DDoS 보호 + DRT |
| **Firewall Manager** | 조직 전체 보안 정책 중앙 관리 |
| **GuardDuty** | 위협 탐지 (ML) |
| **Inspector** | 취약점 자동 평가 |
| **Macie** | S3 PII 감지 |

---

← [Section 25. Identity and Access Management (IAM) - 고급](section25.md) | [Section 27. 네트워킹 - VPC](section27.md) →
