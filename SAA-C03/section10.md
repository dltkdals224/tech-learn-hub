# Section 10. Route 53

## 목차
1. [DNS 기초](#1-dns-기초)
2. [Amazon Route 53 개요](#2-amazon-route-53-개요)
3. [Route 53 Records](#3-route-53-records)
4. [Hosted Zones](#4-hosted-zones)
5. [Records TTL](#5-records-ttl)
6. [CNAME vs Alias](#6-cname-vs-alias)
7. [라우팅 정책 (Routing Policies)](#7-라우팅-정책-routing-policies)
8. [Health Checks](#8-health-checks)
9. [Domain Registrar vs DNS Service](#9-domain-registrar-vs-dns-service)
10. [Hybrid DNS & Resolver Endpoints](#10-hybrid-dns--resolver-endpoints)

---

## 1. DNS 기초

**DNS(Domain Name System)**는 호스트명을 IP 주소로 변환하는 시스템이다.

### 주요 용어

| 용어 | 설명 |
|------|------|
| **Domain Registrar** | 도메인 이름을 등록하는 서비스 (Route 53, GoDaddy 등) |
| **DNS Records** | A, AAAA, CNAME, NS, MX 등 DNS 레코드 유형 |
| **Zone File** | 모든 DNS 레코드를 포함하는 파일 |
| **Name Server** | DNS 쿼리를 처리하는 서버 |
| **TLD (Top Level Domain)** | `.com`, `.us`, `.in`, `.gov` 등 |
| **SLD (Second Level Domain)** | `amazon.com`, `google.com` 등 |

### DNS 동작 원리

```
Web Browser
    → Local DNS Server (ISP 제공)
        → Root DNS Server (.com NS 반환)
            → TLD DNS Server (amazon.com NS 반환)
                → SLD DNS Server (IP 주소 반환)
    ← IP 주소 수신 → 웹 서버 접속
```

> 💡 **시험 포인트**:
> - DNS 계층: Root → TLD → SLD 순서로 쿼리
> - Local DNS Server가 응답을 캐시하여 이후 쿼리를 빠르게 처리

---

## 2. Amazon Route 53 개요

**Route 53**은 AWS가 제공하는 고가용성·고확장성 **완전 관리형 Authoritative DNS** 서비스다.

**핵심 특징:**
- **Authoritative DNS**: 고객이 직접 DNS 레코드를 업데이트 가능
- **Domain Registrar** 기능도 제공 (도메인 직접 등록 가능)
- **100% 가용성 SLA**: AWS에서 유일하게 100% SLA를 보장하는 서비스
- 리소스 **헬스 체크** 기능 내장
- 포트 53을 사용하는 것에서 이름이 유래

> 💡 **시험 포인트**:
> - Route 53은 AWS 서비스 중 **유일하게 100% 가용성 SLA** 보장
> - Authoritative DNS = 고객이 DNS 레코드를 직접 제어 가능

---

## 3. Route 53 Records

각 레코드는 특정 도메인에 대한 트래픽을 어떻게 라우팅할지 정의한다.

### 주요 레코드 유형

| 레코드 타입 | 설명 |
|------------|------|
| **A** | 호스트명 → IPv4 주소 |
| **AAAA** | 호스트명 → IPv6 주소 |
| **CNAME** | 호스트명 → 다른 호스트명 (루트 도메인 사용 불가) |
| **NS** | Hosted Zone의 Name Server |

### 기타 레코드 유형
CAA, DS, MX, NAPTR, PTR, SOA, TXT, SPF, SRV

> 💡 **시험 포인트**:
> - **CNAME**은 루트 도메인(Zone Apex)에 사용 **불가** → `example.com`에 CNAME 설정 불가
> - **A/AAAA** 레코드는 IPv4/IPv6 구분

---

## 4. Hosted Zones

**Hosted Zone**은 도메인과 서브도메인에 대한 트래픽을 어떻게 라우팅할지 정의하는 레코드들의 컨테이너다.

| 유형 | 설명 |
|------|------|
| **Public Hosted Zone** | 인터넷 트래픽 라우팅 (공개 도메인) |
| **Private Hosted Zone** | **VPC 내부** 트래픽 라우팅 (내부 도메인) |

**비용:** Hosted Zone당 **$0.50/월**

> 💡 **시험 포인트**:
> - Private Hosted Zone은 **VPC와 연결** 필요 → VPC 내부에서만 해석 가능
> - Public + Private 동일 도메인 설정 가능 → VPC 내부는 Private, 외부는 Public 응답

---

## 5. Records TTL

**TTL(Time To Live)**은 클라이언트가 DNS 레코드를 캐시하는 시간이다.

| TTL 설정 | 트래픽 | 유연성 | 비용 |
|---------|--------|--------|------|
| **High TTL** (24시간) | Route 53 트래픽 감소 | 레코드 변경 반영 느림 | 저렴 |
| **Low TTL** (60초) | Route 53 트래픽 증가 | 레코드 변경 즉시 반영 | 비쌈 |

**주의:**
- TTL은 모든 DNS 레코드에 **필수** 설정 항목
- **Alias 레코드**는 TTL 설정 불필요 (예외)

> 💡 **시험 포인트**:
> - 레코드 변경 전 TTL을 낮게 설정 → 변경 후 다시 높게 복구하는 패턴 권장
> - **Alias 레코드**는 TTL을 Route 53이 자동 관리

---

## 6. CNAME vs Alias

| 항목 | CNAME | Alias |
|------|-------|-------|
| 대상 | 다른 호스트명 | **AWS 리소스** |
| 루트 도메인 사용 | **불가** (non-root only) | **가능** (root + non-root) |
| 비용 | 쿼리 비용 발생 | **무료** |
| 헬스 체크 | 없음 | **네이티브** 지원 |
| TTL | 설정 필요 | Route 53이 자동 관리 |

### Alias 레코드 대상 (AWS 리소스)

- **Elastic Load Balancer (ELB)**
- **CloudFront** 배포
- **API Gateway**
- **Elastic Beanstalk** 환경
- **S3 웹사이트** (S3 버킷 자체는 불가)
- **VPC Interface Endpoints**
- **Global Accelerator**
- **같은 Hosted Zone 내 Route 53 레코드**

> 💡 **시험 포인트**:
> - **EC2 DNS 이름에는 Alias 설정 불가**
> - `example.com` (Zone Apex) → **Alias만** 사용 가능 (CNAME 불가)
> - ELB 앞에 Route 53 연결 시 반드시 **Alias** 사용

---

## 7. 라우팅 정책 (Routing Policies)

Route 53이 DNS 쿼리에 어떻게 응답할지 결정하는 정책이다. (트래픽 자체를 라우팅하는 것이 아님 — DNS 응답만 처리)

### Simple (단순)

- 단일 또는 다수 리소스로 라우팅
- **다중 값**: 클라이언트가 랜덤으로 하나 선택
- 헬스 체크 **연결 불가**

### Weighted (가중치)

- 각 리소스에 **가중치(%)** 를 지정하여 트래픽 비율 제어
- **weight=0**: 해당 리소스로 트래픽 중단
- **모든 weight=0**: 동등하게 분산
- 헬스 체크 지원
- 사용 사례: 리전 간 로드밸런싱, 새 버전 A/B 테스트

### Latency-based (지연 시간 기반)

- 사용자와 AWS 리전 간 **지연 시간이 가장 낮은** 리소스로 라우팅
- 지연 시간은 사용자 → AWS 리전 트래픽 기반 측정
- 헬스 체크 지원

### Failover (Active-Passive)

- **Primary** 리소스에 헬스 체크 → 실패 시 **Secondary**로 자동 전환
- Primary 헬스 체크는 **필수**

### Geolocation (지리적 위치)

- 사용자의 **실제 위치** 기반 라우팅 (대륙, 국가, 미국 주 단위)
- **가장 구체적인** 위치 설정이 우선 적용
- **Default 레코드** 반드시 생성 (매칭되는 위치 없을 때)
- 사용 사례: 콘텐츠 현지화, 지역별 콘텐츠 제한

### Geoproximity (지리적 근접성)

- 리소스의 **지리적 위치** 기반 + **Bias(편향값)** 로 트래픽 이동
- **Bias 확장(+1~+99)**: 더 많은 트래픽 유도
- **Bias 축소(-1~-99)**: 트래픽 감소
- 리소스 유형: AWS 리전 또는 위도/경도(비AWS 리소스)
- **Route 53 Traffic Flow** 기능 필요

### IP-based (IP 기반)

- 클라이언트의 **IP 주소(CIDR)** 기반 라우팅
- CIDR 목록과 해당 엔드포인트 매핑 제공
- 사용 사례: 성능 최적화, 네트워크 비용 절감

### Multi-Value (다중 값)

- 다수 리소스로 라우팅, **헬스 체크** 지원
- 최대 **8개** 정상 레코드 반환
- **ELB의 대체 수단이 아님** (단순 다중 응답)

> 💡 **시험 포인트**:
> - **Geolocation** vs **Latency**: 위치 기반 vs 실제 지연 시간 기반 — 헷갈리기 쉬움
> - **Geoproximity** = Bias 조정으로 트래픽 이동 → **Traffic Flow 필수**
> - **Multi-Value ≠ ELB**: 클라이언트 측 로드밸런싱, ELB 대체 불가
> - Simple 정책은 **헬스 체크 불가**, Failover는 Primary에 **헬스 체크 필수**

---

## 8. Health Checks

Route 53 헬스 체크는 **공개 리소스**의 상태를 모니터링한다.

### 헬스 체크 유형 3가지

#### 1) 엔드포인트 직접 모니터링

| 항목 | 내용 |
|------|------|
| 전 세계 헬스 체커 수 | **15개** |
| 정상 판단 기준 | 18% 이상 정상 응답 시 Healthy |
| 기본 인터벌 | **30초** (Fast: 10초, 추가 비용) |
| 지원 프로토콜 | HTTP, HTTPS, TCP |
| 성공 기준 | 2xx/3xx 응답 코드 |
| 텍스트 매칭 | 응답 본문 첫 **5,120 바이트** 검사 가능 |

**주의:** 헬스 체커가 접근할 수 있도록 **라우터/방화벽이 Route 53 IP 허용** 필요

#### 2) Calculated Health Checks

- **최대 256개** 하위 헬스 체크를 **OR, AND, NOT**으로 조합
- 몇 개가 통과해야 전체 정상으로 판단할지 설정 가능
- 용도: 유지보수 중 웹사이트를 다운시키지 않고 헬스 체크 실패 방지

#### 3) CloudWatch Alarm 모니터링

- **Private Hosted Zone** 리소스 헬스 체크에 사용
- 헬스 체커는 VPC 외부에 있어 프라이빗 엔드포인트 접근 **불가**
- 해결 방법: **CloudWatch 지표 생성 → CloudWatch Alarm 설정 → Alarm을 헬스 체크로 모니터링**

> 💡 **시험 포인트**:
> - Private 리소스 헬스 체크 → **CloudWatch Metric + Alarm** 경유
> - 헬스 체커 15개 중 **18% 이상** 정상이면 Healthy
> - Calculated Health Check: 자식 헬스 체크 **최대 256개** 조합 가능

---

## 9. Domain Registrar vs DNS Service

도메인 등록 기관과 DNS 서비스는 **별도로** 사용할 수 있다.

**GoDaddy에서 도메인 구매 + Route 53을 DNS 서비스로 사용하는 방법:**

1. Route 53에서 **Public Hosted Zone** 생성
2. Route 53에서 제공하는 **NS 레코드** 확인
3. GoDaddy 등 등록 기관의 설정에서 **Custom NS 서버**를 Route 53 NS로 업데이트

> 💡 **시험 포인트**:
> - 도메인을 어디서 구매하든 **Route 53을 DNS 서비스로 사용 가능**
> - 핵심: 등록 기관의 NS 레코드를 **Route 53 NS로 교체**

---

## 10. Hybrid DNS & Resolver Endpoints

온프레미스와 AWS VPC 간 DNS 쿼리를 해결하기 위한 기능이다.

| 엔드포인트 유형 | 방향 | 설명 |
|--------------|------|------|
| **Inbound Endpoint** | 온프레미스 → AWS | 온프레미스 DNS가 Route 53으로 쿼리 전달 (AWS 내부 도메인 해석) |
| **Outbound Endpoint** | AWS → 온프레미스 | Route 53 Resolver가 조건부로 온프레미스 DNS 서버로 쿼리 전달 |

**사용 사례:**
- 온프레미스 서버가 AWS의 Private Hosted Zone 레코드를 해석해야 할 때 → **Inbound**
- AWS EC2가 온프레미스 내부 도메인(`corp.internal`)을 해석해야 할 때 → **Outbound**

> 💡 **시험 포인트**:
> - Inbound: 온프레미스 → Route 53 (AWS 도메인 해석)
> - Outbound: Route 53 → 온프레미스 DNS (온프레미스 도메인 해석)
> - Resolver Endpoints는 **ENI**로 VPC에 배치되며 Direct Connect / VPN 연결 필요

---

← [Section 9. AWS 기초: RDS + Aurora + ElastiCache](section9.md) | [Section 11. 클래식 솔루션 아키텍처 토론](section11.md) →
