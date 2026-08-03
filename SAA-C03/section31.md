# Section 31. 백서 및 아키텍처 - AWS 공인 솔루션스 아키텍트 어소시에이트

## 목차
1. [Well-Architected Framework 개요](#1-well-architected-framework-개요)
2. [6가지 설계 원칙 (Pillars)](#2-6가지-설계-원칙-pillars)
3. [AWS Trusted Advisor](#3-aws-trusted-advisor)
4. [참고 자료](#4-참고-자료)

---

## 1. Well-Architected Framework 개요

- **안전하고, 고성능이며, 복원력 있고, 효율적인** 인프라 구축을 돕는 프레임워크
- **6개 기둥(Pillars)** 으로 구성
- 특정 감사(Audit) 도구가 아닌, **트레이드오프 평가** 방법론
- AWS Well-Architected Tool (콘솔): 질문-답변 방식으로 워크로드 평가

> 💡 **시험 포인트**:
> - Well-Architected Framework = 6 Pillars 암기 필수
> - 각 Pillar별 핵심 AWS 서비스 매핑 이해
> - "설계 원칙"과 "AWS 서비스" 연결 문제 자주 출제

---

## 2. 6가지 설계 원칙 (Pillars)

### Pillar 1. 운영 우수성 (Operational Excellence)

**설계 원칙:**
- **IaC(Infrastructure as Code)** 로 운영 수행
- 문서 어노테이션 (주석 형태의 운영 문서)
- 소규모의 **자주, 되돌릴 수 있는 변경** 수행
- 운영 절차 지속 개선
- **장애 예측** 및 대응 준비
- **실패에서 학습**

**관련 AWS 서비스:**

| 카테고리 | 서비스 |
|---|---|
| IaC | **CloudFormation** |
| CI/CD | **CodeBuild, CodePipeline, CodeDeploy** |
| 모니터링 | **CloudWatch, X-Ray** |
| 거버넌스 | **Config, CloudTrail** |

---

### Pillar 2. 보안 (Security)

**설계 원칙:**
- **강력한 자격 증명 기반** 구축 (IAM 최소 권한)
- **추적 가능성** 활성화 (모든 액션 로깅)
- **모든 계층에 보안** 적용 (네트워크, 앱, 데이터)
- 보안 모범 사례 **자동화**
- 전송 중 + 저장 시 **데이터 보호**
- 데이터에 사람 접근 최소화
- **보안 이벤트 대응** 준비

**관련 AWS 서비스:**

| 카테고리 | 서비스 |
|---|---|
| 자격 증명 | **IAM, STS** |
| 탐지 | **CloudTrail, GuardDuty, Config** |
| 인프라 보호 | **Shield, WAF, Inspector** |
| 데이터 보호 | **KMS, ACM, Macie** |
| 시크릿 관리 | **Secrets Manager** |

> 💡 **시험 포인트**:
> - 보안 Pillar의 핵심: IAM (최소 권한) + KMS (암호화) + CloudTrail (추적)
> - GuardDuty: 위협 탐지; Macie: S3 민감 데이터 탐지; Inspector: 취약점 스캔

---

### Pillar 3. 신뢰성 (Reliability)

**설계 원칙:**
- **자동 장애 복구** 설계
- 복구 절차 **테스트** (재해 복구 훈련)
- **수평 확장**으로 단일 장애점 제거
- 용량 추측 중단 → **자동 스케일링**
- **자동화**를 통한 변경 관리

**관련 AWS 서비스:**

| 카테고리 | 서비스 |
|---|---|
| 네트워크 기반 | **IAM, VPC** |
| 로드 밸런싱 | **ELB** |
| DNS 장애 조치 | **Route 53** |
| 자동 복구 | **ASG, Multi-AZ** |
| 백업 | **Backup** |
| IaC | **CloudFormation** |
| 한도 관리 | **Service Quotas** |

> 💡 **시험 포인트**:
> - 신뢰성 = Multi-AZ + Auto Scaling + ELB + Route 53 Health Check 조합
> - "자동 장애 복구" → ASG가 핵심

---

### Pillar 4. 성능 효율성 (Performance Efficiency)

**설계 원칙:**
- 고급 기술 **민주화** (관리형 서비스 활용)
- **몇 분 만에 글로벌** 배포
- **서버리스** 아키텍처 활용
- **실험** 자주 수행
- **기계적 공감(Mechanical Sympathy)**: 워크로드에 맞는 서비스 선택

**관련 AWS 서비스:**

| 카테고리 | 서비스 |
|---|---|
| 컴퓨팅 | **Auto Scaling, Lambda** |
| 스토리지 | **EBS, S3** |
| 데이터베이스 | **RDS, DynamoDB, ElastiCache** |
| 글로벌 배포 | **CloudFront** |
| 스트리밍 | **SQS, Kinesis** |

---

### Pillar 5. 비용 최적화 (Cost Optimization)

**설계 원칙:**
- **클라우드 재무 관리** 구현
- **소비 모델** 채택 (사용한 만큼 지불)
- **전체 효율성** 측정
- 차별화되지 않는 무거운 작업에 지출 중단 (관리형 서비스 활용)
- **지출 분석 및 귀속** (태그 기반 비용 추적)

**관련 AWS 서비스:**

| 카테고리 | 서비스 |
|---|---|
| 스토리지 최적화 | **S3 Glacier, S3 Lifecycle** |
| 컴퓨팅 비용 절감 | **Reserved Instances, Savings Plans, Spot Instances** |
| 비용 분석 | **Trusted Advisor, Cost Explorer, Budgets** |
| 사이즈 최적화 | **Right Sizing Recommendations** |

> 💡 **시험 포인트**:
> - 비용 최적화 = 올바른 구매 옵션 + 불필요 리소스 삭제 + 스토리지 라이프사이클
> - Spot > Reserved > On-Demand > Dedicated (비용 순)
> - Cost Explorer vs Budgets: Explorer=분석, Budgets=임계값 알림

---

### Pillar 6. 지속 가능성 (Sustainability)

- **환경 영향**, 특히 **에너지 효율**에 집중
- 2021년 추가된 6번째 Pillar

**설계 원칙:**
- **영향 이해** 및 측정
- **지속 가능성 목표** 수립
- **활용률 최대화** (유휴 리소스 제거)
- **관리형 서비스** 활용 (AWS의 효율적인 인프라 공유)
- **다운스트림 영향** 감소 (사용자 측 에너지 소비 감소)

**관련 AWS 서비스:**

| 카테고리 | 서비스 |
|---|---|
| 동적 스케일링 | **EC2 Auto Scaling** |
| 서버리스 | **Lambda, Fargate** |
| 읽기 부하 분산 | **RDS Read Replicas** |
| 효율적 컴퓨팅 | **Graviton2 인스턴스** |
| 스토리지 최적화 | **EFS-IA** |

> 💡 **시험 포인트**:
> - Graviton2: ARM 기반 프로세서 → 동일 성능 대비 더 낮은 에너지 소비
> - 서버리스 = 유휴 시간 없음 = 에너지 효율적
> - 지속 가능성 Pillar는 2021년 추가 (기존 5 Pillars에서 확장)

---

## 3. AWS Trusted Advisor

### 개요
- **설치 불필요** — AWS 계정 수준 자동 평가 도구
- AWS 계정 분석 → **6개 카테고리** 권장 사항 제공

### 6개 검사 카테고리

| 카테고리 | 예시 |
|---|---|
| **비용 최적화** | 사용률 낮은 EC2 인스턴스, 미사용 EBS 볼륨 |
| **성능** | 높은 활용률 EC2, CloudFront 최적화 |
| **보안** | 오픈 SG, 루트 계정 MFA 미설정 |
| **내결함성** | Multi-AZ 미설정, EBS 스냅샷 없음 |
| **서비스 한도** | 한도 초과 임박 리소스 |
| **운영 우수성** | — |

### 지원 플랜별 기능

| 플랜 | 검사 수 | 주요 기능 |
|---|---|---|
| **Basic / Developer** | **7개 핵심 검사** | S3 공개 버킷, 비제한 SG, IAM 최소 사용자 수, 루트 MFA, EBS/RDS 스냅샷, 서비스 한도, CloudFront 인증서 만료 |
| **Business / Enterprise** | **전체 검사** | AWS Support API, 주간 업데이트, 프로그래밍 방식 접근 |

> 💡 **시험 포인트**:
> - Basic 플랜 7개 핵심 검사 내용 암기
> - Business 이상만 전체 Trusted Advisor 검사 접근 가능
> - Trusted Advisor는 **실시간 권장 사항** 제공 (Config는 규칙 기반 컴플라이언스)

---

## 4. 참고 자료

**공식 문서 및 학습 자료:**
- [AWS Well-Architected Framework 백서](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture)
- [AWS Blog](https://aws.amazon.com/blogs)
- [AWS Quick Starts (레퍼런스 아키텍처)](https://aws.amazon.com/quickstart/)
- [AWS Solutions Library](https://aws.amazon.com/solutions/)

**시험 준비 자료:**
- AWS Skill Builder (공식 연습 문제)
- AWS 공식 연습 시험 (유료)
- Well-Architected Labs (실습)

---

← [Section 30. 기타 서비스](section30.md) | [Section 32. 시험 준비 + 연습 시험 - AWS Certified SAA](section32.md) →
