# Section 11. 클래식 솔루션 아키텍처 토론

## 목차
1. [소개](#1-소개)
2. [WhatIsTheTime.com (Stateless Web App)](#2-whatisthetimecom-stateless-web-app)
3. [MyClothes.com (Stateful Web App with Shopping Cart)](#3-myclothescom-stateful-web-app-with-shopping-cart)
4. [MyWordPress.com (Stateful with Image Storage)](#4-mywordpresscom-stateful-with-image-storage)
5. [애플리케이션 빠른 구동 방법](#5-애플리케이션-빠른-구동-방법)
6. [Elastic Beanstalk](#6-elastic-beanstalk)

---

## 1. 소개

이 섹션에서는 시험에 자주 등장하는 **5가지 클래식 솔루션 아키텍처** 케이스 스터디를 다룬다. 각 아키텍처의 **진화 과정**과 그 이유를 이해하는 것이 핵심이다.

- WhatIsTheTime.com: Stateless 웹 앱 스케일링
- MyClothes.com: 쇼핑 카트를 가진 Stateful 웹 앱
- MyWordPress.com: 이미지 저장이 필요한 Stateful 앱
- 애플리케이션 빠른 구동 방법
- Elastic Beanstalk

> 💡 **시험 포인트**:
> - 각 아키텍처가 왜 그렇게 진화했는지 **이유**를 이해하면 시험 시나리오 문제 대응 가능
> - 고가용성(HA), 확장성(Scalability), Stateless 설계가 핵심 키워드

---

## 2. WhatIsTheTime.com (Stateless Web App)

현재 시각을 알려주는 단순한 웹 앱의 아키텍처 진화 과정이다. 상태 저장이 불필요하다.

### 아키텍처 진화 단계

| 단계 | 구성 | 문제점 |
|------|------|--------|
| **1단계** | EC2 1대 + Elastic IP | 인스턴스 장애 시 다운타임 발생 |
| **2단계** | Route 53 + TTL | DNS 페일오버 가능, 하지만 여전히 단일 인스턴스 |
| **3단계** | 수직 스케일링 (M5 large) | 업그레이드 중 **다운타임** 발생 |
| **4단계** | 다수 EC2 + Route 53 (EIP 제거) | 수평 확장, 하지만 TTL 동안 구형 IP 캐시 문제 |
| **5단계** | **ELB** (헬스 체크) + EC2 Private Subnet | ELB가 헬스 체크로 정상 인스턴스만 라우팅 |
| **6단계** | **Auto Scaling Group** 추가 | 수요에 따라 자동 인스턴스 조정 |
| **7단계** | **Multi-AZ** (ELB + ASG) | 진정한 고가용성 달성 |

### 핵심 교훈

- **Elastic IP는 확장성이 없다** → ELB + Route 53 Alias로 대체
- ELB: **Public Subnet** 배치, EC2: **Private Subnet** 배치
- **ASG**가 수요 급증에 대응하고 비용을 최적화

> 💡 **시험 포인트**:
> - Stateless 앱 고가용성 → **Multi-AZ ASG + ELB + Route 53 Alias**
> - ELB는 헬스 체크로 비정상 인스턴스를 자동 제외
> - EC2를 Private Subnet에 배치하면 직접 인터넷 노출 없이 보안 강화

---

## 3. MyClothes.com (Stateful Web App with Shopping Cart)

쇼핑 카트 상태를 유지해야 하는 e-커머스 사이트 아키텍처 진화 과정이다.

### 문제: 세션 유지

ELB + 다수 EC2 환경에서 각 요청이 다른 EC2로 이동 → **쇼핑 카트 데이터 소실**

### 해결 방법 비교

| 방법 | 설명 | 문제점 |
|------|------|--------|
| **ELB Stickiness (고정 세션)** | 동일 사용자 요청을 같은 EC2로 고정 | 해당 EC2 장애 시 세션 소실, 불균등 부하 |
| **User Cookies** | 쇼핑 카트 데이터를 클라이언트 쿠키에 저장 | 쿠키 크기 **4KB 제한**, 보안 위험 (조작 가능) |
| **Server Session + ElastiCache** | 세션 ID만 쿠키에 저장, 실제 데이터는 ElastiCache | **최선의 방법**, HTTP 요청 overhead 최소화 |

### 최종 권장 아키텍처

1. **ELB** + Multi-AZ EC2 (ASG)
2. **ElastiCache**: 세션 데이터 저장 + DB 쿼리 캐싱
3. **RDS**: 사용자 데이터 저장, **Read Replica**로 읽기 확장
4. **Multi-AZ RDS + Multi-AZ ElastiCache** → HA 확보

### 보안 그룹 구성

| 컴포넌트 | 허용 트래픽 |
|---------|-----------|
| **ELB** | 인터넷(0.0.0.0/0) 80/443 허용 |
| **EC2** | ELB 보안 그룹에서만 허용 |
| **ElastiCache** | EC2 보안 그룹에서만 허용 |
| **RDS** | EC2 보안 그룹에서만 허용 |

> 💡 **시험 포인트**:
> - **Stateless 앱 = 세션을 외부에 저장** (ElastiCache 또는 DynamoDB)
> - ELB Stickiness → HA 약화; User Cookies → 보안 위험; **ElastiCache 서버 세션 = 정답**
> - ElastiCache는 세션 저장 + DB 쿼리 캐싱 두 가지 역할 동시 수행 가능

---

## 4. MyWordPress.com (Stateful with Image Storage)

사용자가 업로드한 이미지를 저장해야 하는 WordPress 블로그 아키텍처다.

### 문제: 이미지 공유

이미지를 한 EC2의 **EBS 볼륨**에 저장 → 다른 인스턴스에서 해당 이미지에 접근 **불가**

### 해결 방법

| 저장소 | 설명 | 적합 여부 |
|--------|------|---------|
| **EBS** | 단일 EC2 연결, 다른 인스턴스 공유 불가 | 단일 인스턴스 환경만 |
| **EFS (Elastic File System)** | **NFS 기반 공유 파일 시스템**, 다수 EC2 동시 접근 가능 | **다중 인스턴스 환경** |

### 최종 아키텍처

- **EFS** + **ENI(Elastic Network Interface)** → 모든 AZ의 EC2가 동일 파일 시스템 공유
- **Aurora Multi-AZ** → 데이터베이스 고가용성
- Multi-AZ ASG + ELB

> 💡 **시험 포인트**:
> - **다수 EC2 간 파일 공유 = EFS** (EBS는 단일 인스턴스 전용)
> - EFS는 ENI를 통해 각 AZ에서 마운트
> - WordPress처럼 공유 스토리지가 필요한 CMS → EFS 선택

---

## 5. 애플리케이션 빠른 구동 방법

인스턴스/리소스 생성 후 애플리케이션을 빠르게 준비하는 방법이다.

### EC2 인스턴스

| 방법 | 설명 | 특징 |
|------|------|------|
| **Golden AMI** | 앱·OS 의존성을 사전 설치한 AMI 생성 후 즉시 런치 | 가장 빠른 부팅, 정적 구성에 최적 |
| **Bootstrap with User Data** | 인스턴스 시작 시 User Data 스크립트로 동적 구성 | 느리지만 유연한 구성 가능 |
| **Hybrid** | Golden AMI + User Data 조합 | Elastic Beanstalk 방식 |

### RDS

- **스냅샷 복원** → 스크립트 실행 없이 즉시 데이터가 있는 DB 준비
- 초기 데이터 세팅이 된 스냅샷에서 복원하면 훨씬 빠름

### EBS

- **스냅샷 복원** → 사전 포맷/데이터가 있는 볼륨 즉시 사용
- 마운트 후 바로 사용 가능 (포맷 불필요)

> 💡 **시험 포인트**:
> - 빠른 EC2 구동 → **Golden AMI** (User Data보다 훨씬 빠름)
> - RDS/EBS 빠른 준비 → **스냅샷 복원**
> - Elastic Beanstalk는 **Golden AMI + User Data 하이브리드** 방식 사용

---

## 6. Elastic Beanstalk

**Elastic Beanstalk**는 AWS에 애플리케이션을 배포하는 **개발자 중심의 관리형 서비스**다.

- AWS가 EC2, ELB, ASG, RDS 등을 **자동으로 프로비저닝 및 구성**
- 개발자는 **애플리케이션 코드**에만 집중
- 서비스 자체는 **무료** (사용하는 기반 리소스 비용만 지불)

### 구성 요소

| 구성 요소 | 설명 |
|---------|------|
| **Application** | Beanstalk 컴포넌트들의 모음 (환경, 버전, 설정) |
| **Application Version** | 애플리케이션 코드의 반복(iteration) |
| **Environment** | 특정 앱 버전을 실행하는 AWS 리소스 모음 (한 번에 하나의 버전) |

### Environment Tier (환경 티어)

| 티어 | 구성 | 사용 사례 |
|------|------|---------|
| **Web Server Environment** | ELB + ASG + EC2 | 웹 애플리케이션 서빙 |
| **Worker Environment** | SQS Queue + ASG + EC2 | 백그라운드 작업 처리 |

- 하나의 앱에 **여러 환경** 구성 가능: dev, test, prod 등
- Web Server ↔ Worker 환경은 **SQS로 연동** 가능

### 지원 플랫폼

Go, Java SE, Java with Tomcat, .NET Core on Linux, .NET on Windows Server, Node.js, PHP, Python, Ruby, Packer Builder, Single Container Docker, Multi-Container Docker, Preconfigured Docker

### 배포 모드

| 모드 | 구성 | 적합 환경 |
|------|------|---------|
| **Single Instance** | EC2 1대 + Elastic IP + RDS | 개발(dev) 환경, 비용 절감 |
| **High Availability with Load Balancer** | Multi-AZ ASG + ELB + RDS Multi-AZ | 운영(prod) 환경, 고가용성 |

> 💡 **시험 포인트**:
> - Elastic Beanstalk = **개발자는 코드만**, 인프라는 AWS가 관리
> - **Web Server Tier** vs **Worker Tier**: HTTP 요청 처리 vs SQS 메시지 처리
> - Beanstalk 자체는 **무료**, 기반 리소스(EC2, RDS 등)만 과금
> - 다양한 언어/플랫폼 지원, **Docker**도 지원

---

← [Section 10. Route 53](section10.md) | [Section 12. Amazon S3 소개](section12.md) →
