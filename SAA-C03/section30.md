# Section 30. 기타 서비스

## 목차
1. [AWS CloudFormation](#1-aws-cloudformation)
2. [Amazon SES](#2-amazon-ses-simple-email-service)
3. [Amazon Pinpoint](#3-amazon-pinpoint)
4. [SSM Session Manager](#4-ssm-session-manager)
5. [SSM Run Command](#5-ssm-run-command)
6. [SSM Patch Manager](#6-ssm-patch-manager)
7. [SSM Maintenance Windows](#7-ssm-maintenance-windows)
8. [SSM Automation](#8-ssm-automation)
9. [AWS Cost Explorer](#9-aws-cost-explorer)
10. [AWS Batch vs Lambda](#10-aws-batch-vs-lambda)
11. [Amazon AppFlow](#11-amazon-appflow)
12. [AWS Amplify](#12-aws-amplify)

---

## 1. AWS CloudFormation

### 개요
- **Infrastructure as Code (IaC)** for AWS
- **선언적(Declarative)**: 원하는 인프라를 선언 → CloudFormation이 생성 순서 결정 및 프로비저닝
- 수동 리소스 생성 불필요; 코드로 **버전 관리** 가능
- CloudFormation 자체는 **무료** (생성되는 리소스 비용만 부과)
- 템플릿 형식: **JSON** 또는 **YAML**

**주요 이점:**
- 템플릿 → S3 업로드 → CloudFormation 참조
- **스택(Stack)**: 템플릿으로 정의된 리소스 묶음
- 스택 삭제 → 리소스 일괄 삭제; 스택 재생성 → 동일 환경 재현
- **비용 절감 전략**: DEV 스택 오후 5시 삭제 → 오전 8시 재생성

---

### CloudFormation 핵심 구성 요소

| 구성 요소 | 설명 | 비고 |
|---|---|---|
| **Resources** | AWS 리소스 정의 (필수) | 224+ 리소스 타입 지원 |
| **Parameters** | 동적 입력값 | SSM Parameter Store/Secrets Manager 연동 가능 |
| **Mappings** | 정적 변수 | 예: 리전별 AMI ID 매핑 |
| **Outputs** | 내보내기 값 | 다른 스택에서 **크로스 스택 참조** 가능 |
| **Conditions** | 조건부 리소스 생성 | 파라미터 기반 if/else 로직 |
| **Metadata / Rules / Hooks** | 고급 설정 | — |

> 💡 **시험 포인트**:
> - Outputs + `Fn::ImportValue` → **크로스 스택 참조** 패턴
> - Mappings는 하드코딩 정적 값; Parameters는 배포 시 입력 동적 값
> - CloudFormation은 IaC의 AWS 네이티브 솔루션 → "least operational overhead"

---

### CloudFormation StackSets
- **여러 AWS 계정 + 여러 리전**에 걸쳐 스택 생성/업데이트/삭제
- 관리자 계정(Admin)이 StackSet 관리
- 신뢰된 계정(Trusted Accounts)이 스택 인스턴스 실행

### CloudFormation Drift 감지
- 리소스가 CloudFormation 외부에서 변경되었는지 탐지
- 콘솔: CloudFormation → 스택 선택 → **Detect Drift**
- 드리프트된 리소스 목록 및 변경 사항 상세 확인

> 💡 **시험 포인트**:
> - StackSets: 멀티 계정/멀티 리전 일괄 배포의 표준 방법
> - Drift Detection: IaC 거버넌스, 무단 변경 감지

---

## 2. Amazon SES (Simple Email Service)

- 이메일을 **안전하게, 글로벌 규모로** 발송
- **인바운드/아웃바운드** 이메일 모두 지원
- **평판 대시보드**, 성능 인사이트, 스팸 방지 피드백
- 활용: 마케팅 이메일, 트랜잭션 이메일, 대량 이메일

> 💡 **시험 포인트**:
> - SES = 대규모 이메일 발송 전용 서비스
> - SNS는 구독 기반 알림; SES는 이메일 전용 대량 발송
> - 인바운드 이메일 수신 후 S3/Lambda/SNS로 전달 가능

---

## 3. Amazon Pinpoint

- **확장 가능한 양방향(2-way) 마케팅 커뮤니케이션** 서비스
- 채널: **이메일, SMS, 푸시 알림, 음성, 인앱 메시징**
- 메시지 **세그먼트 및 개인화** 지원
- 마케팅 캠페인 효과 **분석 및 측정**

**Pinpoint vs SNS/SES 비교:**

| 서비스 | 특징 |
|---|---|
| **SNS** | 구독 기반 알림, 개발자 API 중심 |
| **SES** | 이메일 대량 발송, 트랜잭션 이메일 |
| **Pinpoint** | 메시지 템플릿 + 전달 스케줄 + 세그먼트 + 캠페인 관리 |

> 💡 **시험 포인트**:
> - Pinpoint = 마케터가 직접 캠페인 관리하는 **마케팅 플랫폼**
> - 다채널(멀티채널) 캠페인 관리가 필요하면 Pinpoint

---

## 4. SSM Session Manager

- SSH 키, 배스천 호스트, **포트 22 없이** EC2/온프레미스 서버에 안전한 셸 접속
- 세션 로그 → **S3** 또는 **CloudWatch Logs** 저장
- **Linux, macOS, Windows** 모두 지원
- 인바운드 포트 불필요 (포트 22 오픈 불필요)
- **IAM** 기반 접근 제어

**요구사항:**
- **SSM Agent** 설치 (Amazon Linux 2/Windows AMI에 기본 포함)
- EC2 인스턴스에 **SSM 권한이 있는 IAM 인스턴스 프로파일** 필요

```
사용자 (IAM 권한) → AWS Systems Manager → SSM Agent → EC2 인스턴스
                     (HTTPS/443 아웃바운드만 필요)
```

> 💡 **시험 포인트**:
> - "SSH 없이 EC2 접속" = **SSM Session Manager**
> - 보안 강화: 포트 22 완전 차단 가능
> - 감사(Audit): 모든 세션 CloudTrail + S3/CW Logs에 기록

---

## 5. SSM Run Command

- **여러 인스턴스에 스크립트/명령어 일괄 실행** (리소스 그룹/태그 기반)
- SSH 불필요; SSM Agent 활용
- **출력 대상**: 콘솔, S3, CloudWatch Logs
- SNS로 알림 발송 가능
- **보안**: IAM 권한 제어 + CloudTrail 감사

**활용 사례:**
- 패치 적용, 소프트웨어 설치, 대규모 스크립트 실행

> 💡 **시험 포인트**:
> - Run Command = SSH 없는 **원격 명령 실행**
> - 태그 기반으로 수백 개 인스턴스 동시 실행 가능
> - Patch Manager와 함께 패치 자동화에 활용

---

## 6. SSM Patch Manager

- 관리형 노드(EC2/온프레미스) **패치 자동화**
- OS 업데이트, 애플리케이션 업데이트, **보안 업데이트**
- 주문형(On-demand) 또는 **Maintenance Windows** 스케줄 패치
- 인스턴스 스캔 → **패치 컴플라이언스 리포트** 생성

> 💡 **시험 포인트**:
> - Patch Manager + Maintenance Windows = 예약 패치 자동화
> - 패치 준수 현황 리포트 → 감사/컴플라이언스 요구사항 충족

---

## 7. SSM Maintenance Windows

- 인스턴스에 대한 작업 수행 **일정(Schedule)** 정의
- 포함 요소: **스케줄, 기간(Duration), 등록 인스턴스, 등록 태스크**
- 활용: 패치 적용, 백업, AMI 생성 등 예약 작업

---

## 8. SSM Automation

- 일반적인 **유지보수/배포 태스크 자동화**
- **Automation Runbook (SSM Document)**: EC2 또는 AWS 리소스에 수행할 작업 정의

**트리거 방법:**
- 수동 실행
- **EventBridge** 규칙
- **AWS Config** 규칙 위반 시
- 스케줄 기반

**활용 사례:**
- 인스턴스 재시작, **AMI 생성**, EBS 스냅샷 생성

> 💡 **시험 포인트**:
> - SSM Automation = AWS 리소스 대상 **운영 자동화 워크플로우**
> - Runbook = 자동화 절차 정의 문서 (콘솔/CLI/API로 실행)

---

## 9. AWS Cost Explorer

- AWS **비용 및 사용량 시각화/관리** 도구
- **최대 12개월** 비용 예측
- **Savings Plan** 추천
- 세분화: **시간별 / 리소스 수준** 분석
- **태그** 기반 비용 그룹화

> 💡 **시험 포인트**:
> - Cost Explorer = 비용 분석 + 예측 + Savings Plan 추천
> - 태그 전략이 없으면 팀/프로젝트별 비용 분리 불가

---

## 10. AWS Batch vs Lambda

| 항목 | **Lambda** | **AWS Batch** |
|---|---|---|
| **실행 시간** | 최대 **15분** | 제한 없음 |
| **디스크 공간** | /tmp 512MB~10GB | EBS / Instance Store |
| **컨테이너** | Lambda 런타임 이미지 | **모든 Docker 이미지** |
| **관리 방식** | **Serverless** | Managed EC2 / Fargate |
| **주요 용도** | 이벤트 드리븐, 짧은 태스크 | 장기 배치 처리 |
| **스케일링** | 자동 (요청 기반) | 작업 큐 기반 자동 스케일링 |

> 💡 **시험 포인트**:
> - 실행 시간 **15분 초과** → Lambda 불가 → **Batch** 사용
> - 커스텀 Docker 이미지 필요 시 → Batch
> - 서버리스 + 짧은 이벤트 처리 → Lambda

---

## 11. Amazon AppFlow

- SaaS 앱과 AWS 간 데이터 전송을 위한 **완전 관리형 통합 서비스**
- 커스텀 통합 코드 불필요

**지원 소스:**
- Salesforce, SAP, Zendesk, Slack, ServiceNow 등

**지원 대상:**
- **S3**, **Redshift**, Snowflake, Salesforce

**전송 주기:**
- 스케줄 기반, 이벤트 드리븐, 주문형(On-demand)

**주요 기능:**
- 데이터 변환: 필터링, 유효성 검사
- 전송 중 + 저장 시 **암호화**

> 💡 **시험 포인트**:
> - AppFlow = SaaS ↔ AWS 데이터 파이프라인 (노코드/로우코드)
> - "Salesforce 데이터를 S3/Redshift로" → AppFlow

---

## 12. AWS Amplify

- 확장 가능한 **풀스택 웹/모바일 앱** 개발 및 배포 도구 세트
- "**웹/모바일을 위한 Elastic Beanstalk**"

**프론트엔드:**
- **호스팅**: S3 + CloudFront 기반 정적 웹 호스팅
- **CI/CD**: Git 연동 자동 빌드/배포

**백엔드 통합 서비스:**

| 기능 | AWS 서비스 |
|---|---|
| **인증(Auth)** | Cognito |
| **API** | AppSync (GraphQL) / API Gateway (REST) |
| **데이터베이스** | DynamoDB / Aurora Serverless |
| **스토리지** | S3 |
| **함수** | Lambda |

**Amplify Studio:**
- 시각적 인터페이스로 백엔드/프론트엔드 구성
- 비개발자도 앱 빌드 가능

> 💡 **시험 포인트**:
> - Amplify = 프론트엔드 + AWS 백엔드 통합 개발 플랫폼
> - 모바일/웹 앱의 인증, API, DB, 스토리지를 빠르게 구성
> - "웹/모바일 앱을 빠르게 배포" → Amplify

---

← [Section 29. 더 많은 솔루션 아키텍처](section29.md) | [Section 31. 백서 및 아키텍처 - AWS 공인 솔루션스 아키텍트 어소시에이트](section31.md) →
