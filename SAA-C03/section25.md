# Section 25. Identity and Access Management (IAM) - 고급

## 목차

1. [AWS Organizations](#1-aws-organizations)
2. [IAM 고급 정책](#2-iam-고급-정책)
3. [AWS IAM Identity Center](#3-aws-iam-identity-center)
4. [AWS Directory Services](#4-aws-directory-services)
5. [AWS Control Tower](#5-aws-control-tower)

---

## 1. AWS Organizations

### 개요

- **글로벌 서비스**: 여러 AWS 계정을 중앙에서 관리
- **Management account** (마스터 계정) + **Member accounts** 구성
- 멤버 계정은 하나의 조직에만 소속 가능
- **Consolidated Billing**: 단일 결제 수단; 볼륨 할인 가격 적용
- Reserved Instances 및 Savings Plans를 조직 전체 계정에서 공유 풀링
- 계정 생성 자동화를 위한 API 제공

---

### Organizational Units (OUs)

- 계정을 그룹으로 묶는 단위; OUs 내에 OUs 중첩 가능
- 구성 예시:

| 기준 | 예시 OUs |
|---|---|
| 비즈니스 단위 | Finance / HR / Dev |
| 환경 | Prod / Dev / Test |
| 프로젝트 | ProjectA / ProjectB |

---

### Service Control Policies (SCPs)

- OU 또는 개별 계정에 적용되는 IAM 정책
- 권한을 **제한(restrict)**; **Management account에는 적용되지 않음**
- 명시적 Allow가 반드시 필요 (루트로부터 상속된 Allow 없음)
- 상위 레벨의 Explicit Deny가 우선
- **주요 사용 사례:**
  - 특정 서비스 접근 제한
  - 조직 탈퇴 방지
  - 컴플라이언스 강제 적용

---

### SCP vs IAM 비교

| 항목 | SCP | IAM |
|---|---|---|
| 적용 범위 | 계정 수준 경계 | 사용자/역할 수준 |
| 권한 부여 | 불가 (제한만 가능) | 가능 |
| 적용 대상 | OU / 계정 | User / Role / Group |

- **실효 권한 = SCP ∩ IAM Policy** (교집합)

---

### 조직 정책 종류

- **Tag Policies**: 조직 전체 리소스의 태그 표준화
- **Backup Policies**: 조직 전체 백업 계획 정의
- **AI Services Opt-out Policies**: 모든 계정에서 AI 데이터 수집 거부

> 💡 **시험 포인트**:
> - SCP는 Management account에 적용되지 않음
> - 실효 권한은 SCP와 IAM 정책의 **교집합**
> - SCP는 권한을 부여하지 않고 **제한**만 함
> - Consolidated Billing으로 Reserved Instances를 조직 전체에서 공유 가능

---

## 2. IAM 고급 정책

### IAM Conditions

| Condition Key | 설명 |
|---|---|
| `aws:SourceIp` | 클라이언트 IP 기반 제한 |
| `aws:RequestedRegion` | 리전 기반 제한 |
| `ec2:ResourceTag` | 태그 기반 제한 |
| `aws:MultiFactorAuthPresent` | MFA 요구 |
| `aws:PrincipalOrgID` | AWS Organization 멤버 계정으로 접근 제한 |

---

### IAM Permission Boundaries

- IAM 엔티티가 가질 수 있는 **최대 권한 설정**
- **Users 및 Roles에 적용** (Groups에는 적용 불가)
- 정책이 더 많은 권한을 부여해도 Boundary가 실효 권한을 제한
- **사용 사례**: 개발자가 역할 생성은 가능하되, 자신의 권한을 상승시키지 못하도록 방지

---

### Policy Evaluation Logic (우선순위)

1. **Explicit Deny** (최우선)
2. **SCPs** (Organizations)
3. **Resource-based policies**
4. **IAM permission boundaries**
5. **Session policies** (AssumeRole)
6. **Identity-based policies**

---

### IAM Access Analyzer

- 계정/조직 외부와 공유된 리소스 탐색
- **Zone of Trust**: AWS Account 또는 Organization 단위
- **Findings 대상**: S3, KMS, SQS 등 외부 접근 리소스
- **Policy Validation**: 문법 오류 + 보안 경고 확인
- **Policy Generation**: CloudTrail 활동 기반 → 최소 권한 정책 자동 생성

> 💡 **시험 포인트**:
> - Permission Boundary는 Groups에는 적용 불가
> - Policy Evaluation: Explicit Deny → SCP → Resource-based → Boundary → Session → Identity 순서
> - `aws:PrincipalOrgID`로 조직 내 계정만 접근 허용 가능
> - Access Analyzer는 CloudTrail 기반으로 최소 권한 정책 생성 가능

---

## 3. AWS IAM Identity Center

### 개요

- 구 명칭: **"AWS SSO"**
- 조직 내 모든 AWS 계정 + 비즈니스 클라우드 앱 + SAML 2.0 앱 + EC2 Windows 인스턴스에 대한 **Single Sign-On**
- 하나의 로그인으로 모든 서비스 접근
- **Identity Provider 옵션:**
  - Built-in Identity Store
  - 3rd Party: Active Directory, OneLogin, Okta

---

### Multi-Account Permissions

- **Permission Sets**: IAM 정책 모음 단위로 여러 AWS 계정 접근 관리
- 할당 구조: `사용자/그룹` → `Permission Set` → `계정`
- 사용자는 대상 계정에서 Permission Set 역할을 Assume

---

### Attribute-Based Access Control (ABAC)

- 사용자 **속성** 기반 세밀한 권한 제어 (cost center, department, locale 등)
- **사용 사례**: 속성 기반으로 권한을 한 번만 정의; 신규 사용자/계정 추가 시 자동 적용

---

### Active Directory 연동 옵션

| 옵션 | 설명 |
|---|---|
| **AD Connector** | 온프레미스 AD로 프록시; 자격증명은 온프레미스에서 관리 |
| **AWS Managed Microsoft AD** | AWS에 AD 생성; 온프레미스 AD와 신뢰 관계 수립 |
| **Simple AD** | AD 호환 관리형 디렉터리; 온프레미스 연결 없음 |

> 💡 **시험 포인트**:
> - IAM Identity Center는 구 AWS SSO로 Single Sign-On 제공
> - ABAC는 속성 기반으로 권한을 정의하며 확장성이 높음
> - AD Connector는 온프레미스 AD로의 프록시이며 자체 디렉터리 없음
> - Permission Set은 IAM 정책 모음으로 계정 간 접근 관리

---

## 4. AWS Directory Services

### 서비스 비교

| 서비스 | 특징 |
|---|---|
| **AWS Managed Microsoft AD** | AWS에 실제 AD 생성; MFA 지원; 온프레미스 AD와 Trust 수립 가능 |
| **AD Connector** | 온프레미스 AD로의 게이트웨이(프록시); 사용자 온프레미스 관리; 캐싱 없음 |
| **Simple AD** | AD 호환 관리형 디렉터리; 저비용/소규모; 온프레미스 AD 연결 불가 |

---

### AWS Managed Microsoft AD 상세

- AWS에 자체 AD 생성; **MFA 지원**
- 온프레미스 AD와 **신뢰 관계(Trust)** 수립 가능
- 사용자는 양측 모두 존재; 양측 모두에서 인증 가능

---

### IAM Identity Center + Active Directory 연동 구성

```
IAM Identity Center
    ↔ AWS Managed Microsoft AD ↔ 온프레미스 AD (Two-way Trust)
    
또는

IAM Identity Center → AD Connector → 온프레미스 AD
```

> 💡 **시험 포인트**:
> - Simple AD는 온프레미스 AD와 연결 불가
> - AD Connector는 자체 디렉터리 없이 온프레미스 AD로 프록시
> - AWS Managed AD는 양방향 Trust로 온프레미스 AD와 연동
> - IAM Identity Center와 AD 연동 시 AD Connector 또는 AWS Managed AD 사용 가능

---

## 5. AWS Control Tower

### 개요

- 안전하고 규정 준수된 멀티 계정 AWS 환경을 쉽게 설정 및 거버넌스
- AWS Organizations 위에서 동작 (OUs 자동 생성)
- Organizations, 계정, 보안 정책 설정 자동화

---

### Guardrails

| 종류 | 구현 방식 | 예시 |
|---|---|---|
| **Preventive Guardrail** | SCPs 사용 | 특정 리전 접근 제한 |
| **Detective Guardrail** | AWS Config 사용 | 태그 없는 리소스 식별 |

---

### Account Factory

- 계정 프로비저닝 및 배포 자동화
- 최종 사용자를 위한 **셀프서비스 포털** 제공

> 💡 **시험 포인트**:
> - Control Tower는 AWS Organizations 위에서 동작
> - Preventive Guardrail = SCPs, Detective Guardrail = AWS Config
> - Account Factory로 계정 프로비저닝 자동화 및 셀프서비스 포털 제공
> - Control Tower는 멀티 계정 환경의 거버넌스 자동화에 사용

---

← [Section 24. AWS 모니터링 및 감사: CloudWatch, CloudTrail 및 Config](section24.md) | [Section 26. AWS 보안 및 암호화: KMS, SSM Parameter Store, CloudHSM, Shield, WAF](section26.md) →
