# Section 8. 고가용성 및 스케일링성: ELB 및 ASG

## 목차
1. [스케일링과 고가용성 개념](#1-스케일링과-고가용성-개념)
2. [Elastic Load Balancer (ELB)](#2-elastic-load-balancer-elb)
3. [CLB - Classic Load Balancer](#3-clb---classic-load-balancer)
4. [ALB - Application Load Balancer](#4-alb---application-load-balancer)
5. [NLB - Network Load Balancer](#5-nlb---network-load-balancer)
6. [GWLB - Gateway Load Balancer](#6-gwlb---gateway-load-balancer)
7. [Sticky Sessions (고정 세션)](#7-sticky-sessions-고정-세션)
8. [Cross-Zone Load Balancing](#8-cross-zone-load-balancing)
9. [SSL/TLS와 SNI](#9-ssltls와-sni)
10. [Connection Draining](#10-connection-draining)
11. [Auto Scaling Group (ASG)](#11-auto-scaling-group-asg)
12. [ASG 스케일링 정책](#12-asg-스케일링-정책)

---

## 1. 스케일링과 고가용성 개념

### 스케일링 (Scalability)

| 유형 | 설명 | EC2 예시 |
|------|------|---------|
| **수직 확장 (Vertical Scaling)** | 인스턴스 사이즈를 키움 (Scale Up/Down) | t2.micro → t2.large |
| **수평 확장 (Horizontal Scaling)** | 인스턴스 수를 늘림 (Scale Out/In) | ASG + Load Balancer |

- EC2 수직 확장 범위: t2.nano (0.5 GB RAM, 1 vCPU) ~ u-12tb1.metal (12.3 TB RAM, 448 vCPU)

### 고가용성 (High Availability)

**고가용성**이란 2개 이상의 AZ에서 애플리케이션을 실행하는 것을 의미한다. 단일 데이터센터 장애에도 서비스가 유지된다.

| 유형 | 설명 | 예시 |
|------|------|------|
| **Passive HA** | 장애 시 자동 전환 | RDS Multi-AZ |
| **Active HA** | 여러 인스턴스가 동시에 서비스 | 수평 확장 + Load Balancer |

> 💡 **시험 포인트**:
> - 수직 확장 = 인스턴스 타입 변경 (다운타임 발생 가능)
> - 수평 확장 = 인스턴스 수 증가 (다운타임 없음)
> - 고가용성은 반드시 **2개 이상의 AZ** 필요

---

## 2. Elastic Load Balancer (ELB)

**로드 밸런서**는 트래픽을 여러 서버(EC2 등)로 분산시키는 서버다. AWS의 **ELB(Elastic Load Balancer)**는 완전 관리형 서비스로, AWS가 업그레이드/유지보수/고가용성을 처리한다.

### 로드 밸런서 사용 이유
- 단일 진입점 (DNS) 제공
- 장애 인스턴스를 원활하게 처리 (Health Check)
- SSL 종료 (HTTPS → HTTP 변환)
- 쿠키 기반 세션 고정
- 트래픽 분리 (Public ↔ Private)

### 헬스 체크 (Health Check)
- 특정 포트와 경로로 주기적 확인 (예: `/health`, 포트 4567)
- HTTP 200 응답 → 정상(Healthy)
- 비정상 인스턴스로의 트래픽 전송 중단

### AWS 로드 밸런서 4종

| 유형 | 버전 | 레이어 | 프로토콜 | 특징 |
|------|------|--------|---------|------|
| **CLB** | v1 (2009) | L4 + L7 | TCP, HTTP, HTTPS | 구형, 지원 중단 예정 |
| **ALB** | v2 (2016) | L7 | HTTP, HTTPS, WebSocket | 고급 라우팅 규칙 |
| **NLB** | v2 (2017) | L4 | TCP, UDP | 초고성능, 정적 IP |
| **GWLB** | 2020 | L3 | IP Packets (GENEVE 6081) | 3rd party 네트워크 어플라이언스 |

### 로드 밸런서 보안 그룹

```
사용자 → [HTTP/HTTPS 허용] → ELB → [ELB SG에서만 허용] → EC2
```
- ELB: 0.0.0.0/0에서 HTTP(80)/HTTPS(443) 허용
- EC2: ELB의 보안 그룹에서만 트래픽 허용 (직접 접근 차단)

> 💡 **시험 포인트**:
> - 최신 권장: **ALB**(L7) 또는 **NLB**(L4)
> - CLB는 **구형(deprecated)** → 시험에서 레거시 맥락으로 등장
> - EC2 보안 그룹은 ELB SG를 소스로 설정하여 직접 접근 차단

---

## 3. CLB - Classic Load Balancer

- **Layer 4 + Layer 7** 지원 (TCP, HTTP, HTTPS)
- SSL 인증서는 **1개**만 지원
- 현재는 **지원 중단(deprecated)** 상태

> 💡 **시험 포인트**: CLB는 구형 LB. SNI 미지원, 인증서 1개 한정.

---

## 4. ALB - Application Load Balancer

**Layer 7(HTTP)** 전용 로드 밸런서. URL, 호스트명, 쿼리스트링 등을 기반으로 고급 라우팅이 가능하다.

### ALB 라우팅 규칙

| 라우팅 기준 | 예시 |
|-----------|------|
| **경로(Path)** | `/users` → 인스턴스 A, `/search` → 인스턴스 B |
| **호스트명(Hostname)** | `one.example.com` / `other.example.com` |
| **쿼리스트링(Query String)** | `?Platform=Mobile` → 모바일 타겟 그룹 |
| **HTTP 헤더** | 특정 헤더 값에 따라 라우팅 |

### ALB Target Groups (대상 그룹)

- EC2 인스턴스 (ASG에 의해 관리 가능)
- ECS 태스크
- Lambda 함수
- Private IP 주소

### 알아두면 좋은 것 (Good to Know)

- **클라이언트 실제 IP**: `X-Forwarded-For` 헤더에 포함
- **포트**: `X-Forwarded-Port`
- **프로토콜**: `X-Forwarded-Proto`
- 마이크로서비스, 컨테이너 기반 앱에 이상적
- SNI 지원 → 다수의 SSL 인증서 동시 사용 가능

> 💡 **시험 포인트**:
> - ALB는 **L7** → HTTP 기반 고급 라우팅 (CLB는 단순 TCP 라우팅)
> - 클라이언트 원본 IP 확인: `X-Forwarded-For` 헤더
> - Lambda, ECS, 컨테이너를 Target Group으로 설정 가능

---

## 5. NLB - Network Load Balancer

**Layer 4(TCP/UDP)** 로드 밸런서. 초고성능과 초저지연이 특징이다.

**주요 특징:**
- 초당 수백만 건의 요청 처리
- 지연시간 ~100ms (ALB ~400ms 대비 매우 낮음)
- **AZ당 1개의 정적 IP** 제공 (Elastic IP 할당 가능)
- 3개 고정 IP를 허용해야 하는 애플리케이션에 적합

### NLB Target Groups

- EC2 인스턴스
- 특정 IP 주소 (Private IP)
- **ALB** (NLB → ALB 체이닝 가능)

> 💡 **시험 포인트**:
> - "고정 IP가 필요한 로드밸런서" → **NLB** (Elastic IP 할당 가능)
> - "극한의 성능, TCP/UDP" → **NLB**
> - NLB는 ALB를 Target Group으로 설정 가능 (NLB 고정 IP + ALB HTTP 라우팅)

---

## 6. GWLB - Gateway Load Balancer

**Layer 3(IP 패킷)** 로드 밸런서. 모든 트래픽을 **3rd party 네트워크 가상 어플라이언스**를 통해 라우팅한다.

**사용 목적:** 방화벽, IDS/IPS, 심층 패킷 검사, 네트워크 트래픽 수정

**두 가지 기능을 동시에 수행:**
1. **Transparent Network Gateway**: 단일 진입/출구 (모든 트래픽 통과)
2. **Load Balancer**: 가상 어플라이언스들로 트래픽 분산

**프로토콜:** GENEVE, 포트 **6081**

### GWLB Target Groups

- EC2 인스턴스 (방화벽 등 실행)
- Private IP 주소

> 💡 **시험 포인트**:
> - "트래픽 검사/분석을 위한 3rd party 어플라이언스" → **GWLB**
> - GENEVE 프로토콜, 포트 **6081**
> - 트래픽이 어플라이언스를 거친 후 최종 목적지로 전달

---

## 7. Sticky Sessions (고정 세션)

**Sticky Sessions(세션 고정)**은 동일한 클라이언트가 항상 동일한 인스턴스로 연결되도록 하는 기능이다.

- CLB, ALB, NLB에서 지원
- 쿠키(Cookie)를 사용하여 클라이언트를 특정 인스턴스에 고정
- 세션 데이터를 잃지 않아야 하는 상황에 유용
- **단점**: 특정 인스턴스에 부하가 집중될 수 있음

### 쿠키 종류

| 종류 | 세부 유형 | 이름 | 생성자 | 만료 |
|------|---------|------|--------|------|
| **Application-based** | 커스텀 쿠키 | 애플리케이션이 지정 | 대상(Target) | 앱이 설정 |
| **Application-based** | 애플리케이션 쿠키 | AWSALBAPP | LB | LB가 설정 |
| **Duration-based** | 기간 기반 쿠키 | AWSALB (ALB) / AWSELB (CLB) | LB | LB가 설정 |

**사용 불가 이름**: AWSALB, AWSALBAPP, AWSALBTG (AWS 예약명)

> 💡 **시험 포인트**:
> - Sticky Session 쿠키 이름: ALB는 **AWSALB**, CLB는 **AWSELB**
> - 커스텀 쿠키 이름에 AWS 예약 이름 사용 불가
> - Sticky Session으로 특정 인스턴스에 부하 쏠림 발생 가능

---

## 8. Cross-Zone Load Balancing

**Cross-Zone Load Balancing**은 AZ에 관계없이 모든 인스턴스에 균등하게 트래픽을 분산하는 기능이다.

```
[Cross-Zone 활성화]                 [Cross-Zone 비활성화]
AZ-A: 인스턴스 2개                 AZ-A: 인스턴스 2개
AZ-B: 인스턴스 8개                 AZ-B: 인스턴스 8개
→ 전체 10개 인스턴스에 10%씩 분산    → AZ-A는 50%/50%, AZ-B는 12.5%씩
```

| LB 유형 | 기본값 | AZ간 데이터 요금 |
|---------|--------|----------------|
| **ALB** | 활성화 | 무료 |
| **NLB / GWLB** | 비활성화 | 활성화 시 요금 발생 |
| **CLB** | 비활성화 | 활성화 시 무료 |

> 💡 **시험 포인트**:
> - ALB는 Cross-Zone이 **기본 활성화**, AZ 간 데이터 이동 **무료**
> - NLB/GWLB는 **기본 비활성화**, 활성화 시 **요금 발생**
> - CLB는 기본 비활성화지만 활성화해도 **추가 요금 없음**

---

## 9. SSL/TLS와 SNI

### SSL/TLS 기본

- **SSL(Secure Sockets Layer) / TLS(Transport Layer Security)**: 전송 중 암호화를 위한 인증서
- X.509 인증서를 사용
- **ACM(AWS Certificate Manager)**을 통해 인증서 관리
- HTTPS 리스너에 기본 인증서 지정 필수

### SNI (Server Name Indication)

**SNI**는 TLS 핸드셰이크 시 클라이언트가 접속할 호스트명을 서버에 알려주는 프로토콜 확장이다. 하나의 서버(LB)에서 **여러 SSL 인증서**를 사용할 수 있게 해준다.

| LB 유형 | SNI 지원 | 인증서 개수 |
|---------|----------|------------|
| **ALB** | ✅ | 여러 개 |
| **NLB** | ✅ | 여러 개 |
| **CLB** | ❌ | 1개만 |
| **CloudFront** | ✅ | 여러 개 |

### ELB SSL 인증서 요약

| LB | 지원 |
|----|------|
| CLB | SSL 인증서 1개, 다중 도메인 시 CLB 여러 개 필요 |
| ALB | SNI로 다수의 인증서 지원, 리스너별 인증서 설정 |
| NLB | SNI로 다수의 인증서 지원 |

> 💡 **시험 포인트**:
> - SNI는 **ALB, NLB, CloudFront**에서 지원 (**CLB 미지원**)
> - CLB에서 여러 SSL 도메인 → CLB를 여러 개 사용
> - ALB/NLB는 SNI 덕분에 하나의 LB로 여러 HTTPS 도메인 처리 가능

---

## 10. Connection Draining

인스턴스가 로드 밸런서에서 등록 해제(de-registering)될 때 기존 연결을 안전하게 완료할 수 있는 시간을 부여한다.

| 명칭 | LB 유형 |
|------|---------|
| **Connection Draining** | CLB |
| **Deregistration Delay** | ALB / NLB |

**기본값**: 300초 (설정 범위: 1~3600초)
- 0으로 설정 시 즉시 연결 종료
- 수명이 짧은 요청: 낮은 값 권장
- 수명이 긴 요청(대용량 업로드 등): 높은 값 권장

> 💡 **시험 포인트**:
> - Connection Draining 중에는 신규 요청이 해당 인스턴스로 전달되지 않음
> - 기본 **300초**, 0이면 즉시 종료

---

## 11. Auto Scaling Group (ASG)

**ASG(Auto Scaling Group)**은 트래픽 변화에 따라 EC2 인스턴스 수를 자동으로 조정한다.

**핵심 기능:**
- Scale Out: 트래픽 증가 시 인스턴스 추가
- Scale In: 트래픽 감소 시 인스턴스 제거
- 최소/최대/원하는(desired) 용량 설정
- ELB에 새 인스턴스 자동 등록
- 비정상 인스턴스 자동 종료 후 재생성 (ELB Health Check 연동)
- **ASG 자체는 무료** (생성된 EC2 인스턴스 비용만 발생)

### Launch Template (시작 템플릿)

ASG가 새 인스턴스를 시작할 때 사용하는 설정 템플릿이다. (구형 "Launch Configuration"은 deprecated)

포함 항목:
- AMI + 인스턴스 유형
- EC2 User Data
- EBS 볼륨
- 보안 그룹
- SSH 키 페어
- IAM 역할
- 네트워크 및 서브넷 정보
- 로드 밸런서 정보

### ASG + Load Balancer

ELB가 EC2 인스턴스의 헬스체크를 수행하고, ASG에 의해 생성된 인스턴스는 자동으로 ELB에 등록된다.

> 💡 **시험 포인트**:
> - ASG는 ELB Health Check와 연동하여 비정상 인스턴스 자동 교체
> - **Launch Template** 사용 (Launch Configuration은 deprecated)
> - ASG 자체는 **무료**, EC2 비용만 발생

---

## 12. ASG 스케일링 정책

### Dynamic Scaling (동적 스케일링)

| 정책 | 설명 | 예시 |
|------|------|------|
| **Target Tracking Scaling** | 특정 지표의 목표값 유지 | 평균 CPU 40% 유지 |
| **Simple / Step Scaling** | CloudWatch 알람 기반 스케일링 | CPU > 70% → 인스턴스 2개 추가, CPU < 30% → 1개 제거 |

### Scheduled Scaling (예약 스케일링)

알려진 사용 패턴을 기반으로 미리 스케일링 설정
- 예: 금요일 오후 5시에 최소 용량을 10으로 증가

### Predictive Scaling (예측 스케일링)

ML로 과거 부하를 분석해 미래 부하를 예측하고 사전에 스케일링 액션을 예약한다.

```
과거 부하 분석 → 미래 부하 예측 → 스케일링 액션 미리 스케줄링
```

### 스케일링에 좋은 지표 (Good Metrics)

| 지표 | 설명 |
|------|------|
| **CPUUtilization** | 전체 ASG 인스턴스의 평균 CPU 사용률 |
| **RequestCountPerTarget** | 인스턴스당 요청 수를 일정하게 유지 |
| **Average Network In/Out** | 네트워크 바인딩 애플리케이션 |
| **Custom Metric** | CloudWatch로 푸시한 사용자 지정 지표 |

### Scaling Cooldown (스케일링 쿨다운)

스케일링 활동 이후 **쿨다운 기간(기본 300초)** 동안 ASG는 추가 인스턴스를 시작하거나 종료하지 않는다. 메트릭이 안정화될 시간을 준다.

**팁**: Ready-to-use AMI를 사용하면 인스턴스 설정 시간 단축 → 쿨다운 기간 줄일 수 있음

> 💡 **시험 포인트**:
> - 스케일링 정책 3종: **Target Tracking / Simple,Step / Scheduled** + Predictive
> - 쿨다운 기본 **300초**, 쿨다운 중 추가 스케일링 없음
> - 빠른 스케일링 응답이 필요하면 **AMI를 사전 구성**하여 쿨다운 단축

---

> **이전 섹션**: [Section 7. EC2 인스턴스 스토리지](section7.md)
> **다음 섹션**: [Section 9. AWS 기초: RDS + Aurora + ElastiCache](section9.md)
