# Section 18. AWS의 컨테이너: ECS, Fargate, ECR 및 EKS

## 목차

1. [Docker 개요](#1-docker-개요)
2. [Amazon ECS](#2-amazon-ecs)
3. [Amazon Fargate](#3-amazon-fargate)
4. [Amazon ECR](#4-amazon-ecr)
5. [Amazon EKS](#5-amazon-eks)

---

## 1. Docker 개요

- **Docker** = 애플리케이션을 컨테이너에 배포하는 소프트웨어 개발 플랫폼
- 컨테이너는 **어떤 OS, 어떤 머신**에서도 실행 가능
- 예측 가능한 동작, 유지보수·배포 간소화
- 언어·OS·기술 무관하게 동작

### Docker Images (이미지 저장소)

| 저장소 | 특징 |
|--------|------|
| **Docker Hub** | 공개 저장소 |
| **Amazon ECR** | 프라이빗/퍼블릭 저장소 (AWS 관리형) |

### Docker vs Virtual Machines

| 항목 | VM | Docker |
|------|----|----|
| OS | 각 VM마다 전체 OS 복사본 | 호스트 OS 커널 공유 |
| 밀도 | 낮음 | 하나의 EC2 인스턴스에 다수 컨테이너 실행 가능 |
| 기동 속도 | 느림 | 빠름 |

> 💡 **시험 포인트**:
> - Docker 이미지는 **ECR**(프라이빗) 또는 **Docker Hub**(퍼블릭)에 저장
> - Docker 컨테이너는 **호스트 OS 커널 공유** → VM보다 가볍고 빠름
> - AWS에서 컨테이너 실행 = **ECS Task** 실행

---

## 2. Amazon ECS

### EC2 Launch Type

- AWS에서 Docker 컨테이너 실행 = **ECS Cluster**에서 **ECS Tasks** 실행
- **EC2 Launch Type**: EC2 인스턴스를 직접 **프로비저닝 및 관리** 필요
- 각 EC2 인스턴스에 **ECS Agent** 실행 필수 → ECS Cluster 등록
- AWS가 컨테이너 시작/중지 관리

### Fargate Launch Type

- 관리할 EC2 인스턴스 없음 (**서버리스**)
- **Task Definition** 생성 → AWS가 CPU/RAM 기반으로 ECS Task 실행
- 스케일링: Task 수 증가만으로 처리

### IAM Roles for ECS

| 역할 유형 | 적용 대상 | 용도 |
|-----------|-----------|------|
| **EC2 Instance Profile** | EC2 Launch Type 전용 | ECS Agent 사용; ECS·CloudWatch Logs·ECR·Secrets Manager API 호출 |
| **ECS Task Role** | 각 Task | Task별 특정 역할 부여; Task Definition에 정의; 서비스별 다른 역할 사용 |

### Load Balancer Integration

| LB 유형 | 권장 여부 | 특징 |
|---------|-----------|------|
| **ALB** | 권장 | 대부분의 사용 사례 지원 |
| **NLB** | 고처리량/고성능 | AWS PrivateLink 연동 시 활용 |
| **ELB Classic** | 비권장 | - |

### ECS Data Volumes (EFS)

- ECS Task에 **EFS 파일 시스템 마운트**
- EC2 및 Fargate Launch Type **모두 지원**
- **Multi-AZ** 공유 데이터 가능
- **Fargate + EFS** = 서버리스 + 영구 공유 스토리지
- 활용: 영구적인 Multi-AZ 공유 스토리지가 필요한 경우
- 주의: **S3는 파일 시스템으로 마운트 불가**

### ECS Service Auto Scaling

- 원하는 Task 수 자동 증가/감소
- 스케일링 지표:
  - **ECS Service 평균 CPU 사용률**
  - **ECS Service 평균 메모리 사용률**
  - **ALB Request Count per Target**
- 스케일링 방식:

| 방식 | 설명 |
|------|------|
| **Target Tracking** | CloudWatch 지표 목표값 기반 |
| **Step Scaling** | CloudWatch Alarm 기반 |
| **Scheduled Scaling** | 예측 가능한 변화에 대한 예약 |

- **ECS Service Auto Scaling ≠ EC2 Auto Scaling** (별개 개념)

### EC2 Launch Type 스케일링

| 방식 | 설명 |
|------|------|
| **Auto Scaling Group** | CPU 기반 EC2 인스턴스 추가 |
| **ECS Cluster Capacity Provider** | ECS Task를 위한 EC2 자동 프로비저닝+스케일링 (권장; ASG와 연동) |

### ECS Rolling Updates

- 업데이트 중 시작/중지할 Task 수 제어
- **Min/Max Healthy Percentage** 설정으로 롤링 범위 조정

### ECS Task Definitions

- ECS가 Docker 컨테이너 실행 방법을 담은 **JSON 메타데이터**
- 포함 정보:
  - 이미지 이름, 포트 바인딩, 메모리/CPU
  - 환경 변수, 네트워킹 정보
  - **IAM 역할**, 로깅 설정
- Task Definition당 최대 **10개 컨테이너**

| 유형 | 포트 매핑 방식 |
|------|--------------|
| **EC2 Launch Type** | **Dynamic Host Port Mapping** → ALB가 자동으로 올바른 포트 탐색 (ALB 필요, CLB 불가) |
| **Fargate** | 각 Task에 고유 Private IP; **컨테이너 포트만 정의** |

### ECS 환경 변수

| 유형 | 저장 위치 | 용도 |
|------|-----------|------|
| **Hardcoded** | Task Definition | 일반 URL 등 |
| **SSM Parameter Store** | AWS SSM | 민감 데이터 (API 키 등) |
| **Secrets Manager** | AWS Secrets Manager | 민감 데이터 (DB 패스워드 등) |
| **Environment Files (bulk)** | **S3** | 대량 환경 변수 |

### ECS Data Volumes (Bind Mounts)

- 동일 **Task Definition** 내 다수 컨테이너 간 데이터 공유
- EC2 및 Fargate 모두 지원

| Launch Type | 스토리지 |
|-------------|---------|
| **EC2** | EC2 인스턴스 스토리지 사용 |
| **Fargate** | 임시 스토리지 (20~200 GiB) |

- 활용: **사이드카 컨테이너** (메트릭/로그 수집)

### ECS Task Placement (EC2 Launch Type 전용)

- ECS가 CPU·메모리·포트 제약 기반으로 Task 배치 위치 결정

| 전략/제약 | 유형 | 설명 |
|-----------|------|------|
| **Binpack** | 전략 | 최소 CPU/메모리 사용 인스턴스에 배치 → 비용 절감 |
| **Random** | 전략 | 무작위 배치 |
| **Spread** | 전략 | AZ·인스턴스 등 기준으로 분산 배치 |
| **distinctInstance** | 제약 | 각 Task를 서로 다른 인스턴스에 배치 |
| **memberOf** | 제약 | 표현식 기반 조건 지정 |

- 전략과 제약은 **조합 가능**

> 💡 **시험 포인트**:
> - **ECS Task Role** = Task별 권한; **EC2 Instance Profile** = ECS Agent 권한 (EC2 전용)
> - Fargate + EFS = **서버리스 + 영구 공유 스토리지** 조합 기억
> - EC2 Launch Type의 Dynamic Port Mapping → **ALB** 필요 (CLB 불가)
> - **ECS Cluster Capacity Provider** = EC2 인스턴스 자동 프로비저닝 권장 방식

---

## 3. Amazon Fargate

- **서버리스 컨테이너 관리**: EC2 인스턴스 관리 불필요
- AWS가 컴퓨팅 인프라 관리
- Task별 CPU/메모리만 지정
- 각 Fargate Task에 **고유 ENI (Elastic Network Interface)** 할당
- 각 Task에 **고유 Private IP** 부여

> 💡 **시험 포인트**:
> - Fargate = **인프라 관리 Zero** 서버리스 컨테이너
> - 각 Task → 별도 ENI → 별도 Private IP (보안 그룹 Task 단위 적용 가능)
> - Fargate + EFS 조합 → **완전 서버리스 + 영구 스토리지** 아키텍처

---

## 4. Amazon ECR

- **Elastic Container Registry**: AWS에서 Docker 이미지 저장·관리
- **프라이빗** 및 **퍼블릭** 레지스트리 지원 (Amazon ECR Public Gallery)
- ECS와 통합, **S3 기반** 백엔드
- **IAM**으로 접근 제어 (권한 오류 → IAM 정책 확인)
- 지원 기능:
  - 이미지 **취약점 스캐닝**
  - 버전 관리, **이미지 태그**
  - **Lifecycle Policies** (이미지 자동 정리)

> 💡 **시험 포인트**:
> - ECR 권한 오류 → **IAM 정책** 확인
> - ECR 이미지는 **S3에 저장** (내부적으로)
> - **Lifecycle Policy**로 오래된 이미지 자동 삭제 가능
> - ECR Public Gallery = 퍼블릭 컨테이너 이미지 공유 가능

---

## 5. Amazon EKS

### Elastic Kubernetes Service

- AWS에서 **관리형 Kubernetes** 실행
- ECS의 오픈소스 대안 (다른 API)
- **EC2** (워커 노드 배포) 및 **Fargate** (서버리스 파드) 지원
- 활용: 온프레미스 Kubernetes 사용 중 → AWS 마이그레이션 또는 멀티 클라우드

### Node Types (노드 유형)

| 유형 | 관리 주체 | 특징 |
|------|-----------|------|
| **Managed Node Groups** | AWS | EC2 노드 생성·관리; EKS가 관리하는 ASG 소속; 온디맨드/스팟 인스턴스 지원 |
| **Self-Managed Nodes** | 사용자 | 수동 생성 후 EKS 클러스터에 등록; 사전 빌드된 AMI 사용 가능 |
| **AWS Fargate** | AWS | 노드 관리 불필요; 완전 서버리스 |

### EKS Data Volumes

- **StorageClass** 매니페스트 필요
- **Container Storage Interface (CSI)** 호환 드라이버 필요

| 스토리지 | 비고 |
|----------|------|
| **EBS** | EC2 노드용 블록 스토리지 |
| **EFS** | Fargate 지원; Multi-AZ 공유 파일 스토리지 |
| **FSx for Lustre** | 고성능 컴퓨팅 워크로드 |
| **FSx for NetApp ONTAP** | 엔터프라이즈 파일 스토리지 |

> 💡 **시험 포인트**:
> - EKS = **Kubernetes 기반**; ECS = **AWS 고유** 컨테이너 오케스트레이션
> - 온프레미스 Kubernetes → AWS 마이그레이션 시 **EKS** 선택
> - EKS + Fargate = **완전 서버리스 Kubernetes**
> - EKS 스토리지는 **CSI 드라이버** 필요; EFS는 Fargate에서도 사용 가능

---

← [Section 17. 디커플링 애플리케이션: SQS, SNS, Kinesis, Active MQ](section17.md) | [Section 19. 솔루션 설계자 관점의 서버리스 개요](section19.md) →
