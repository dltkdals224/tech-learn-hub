# Section 27. 네트워킹 - VPC

## 목차

1. [VPC 기초](#1-vpc-기초)
2. [서브넷과 인터넷 게이트웨이](#2-서브넷과-인터넷-게이트웨이)
3. [NAT Gateway](#3-nat-gateway)
4. [보안 그룹 vs NACL](#4-보안-그룹-vs-nacl)
5. [VPC 피어링](#5-vpc-피어링)
6. [VPC 엔드포인트](#6-vpc-엔드포인트)
7. [VPC Flow Logs](#7-vpc-flow-logs)
8. [Site-to-Site VPN](#8-site-to-site-vpn)
9. [Direct Connect](#9-direct-connect)
10. [Transit Gateway](#10-transit-gateway)
11. [Traffic Mirroring](#11-traffic-mirroring)
12. [IPv6 및 Egress-Only IGW](#12-ipv6-및-egress-only-igw)
13. [네트워킹 비용](#13-네트워킹-비용)
14. [AWS Network Firewall](#14-aws-network-firewall)

---

## 1. VPC 기초

### 개요

- **VPC (Virtual Private Cloud)**: 리전 범위(region-scoped)의 가상 사설 네트워크
- 리전당 최대 **5개** VPC (soft limit; 요청 시 증가 가능)
- VPC당 최대 **5개** CIDR 블록 할당 가능
  - 최소 크기: **/28** (16개 IP)
  - 최대 크기: **/16** (65,536개 IP)

### 프라이빗 IPv4 범위

| CIDR 블록 | 범위 |
|---|---|
| `10.0.0.0/8` | 10.0.0.0 ~ 10.255.255.255 |
| `172.16.0.0/12` | 172.16.0.0 ~ 172.31.255.255 |
| `192.168.0.0/16` | 192.168.0.0 ~ 192.168.255.255 |

- VPC CIDR는 다른 네트워크(온프레미스, 타 VPC)와 **겹치면 안 됨**

> 💡 **시험 포인트**:
> - VPC는 리전 단위 리소스; 서브넷은 AZ 단위 리소스
> - CIDR 최소 /28, 최대 /16 암기
> - 리전당 기본 5개 VPC (soft limit)
> - 프라이빗 IP 범위 3개 암기: 10.x, 172.16-31.x, 192.168.x

---

## 2. 서브넷과 인터넷 게이트웨이

### 서브넷 (Subnet)

- 특정 **AZ에 종속**; AZ당 하나 이상의 서브넷 생성 가능
- **퍼블릭 서브넷**: 인터넷 접근 가능 (IGW 경유)
- **프라이빗 서브넷**: 인터넷 직접 접근 불가

**AWS가 예약하는 5개 IP 주소** (예: `10.0.0.0/24` 서브넷):

| IP 주소 | 용도 |
|---|---|
| `10.0.0.0` | 네트워크 주소 |
| `10.0.0.1` | VPC 라우터 |
| `10.0.0.2` | AWS DNS |
| `10.0.0.3` | 미래 예약 |
| `10.0.0.255` | 브로드캐스트 (AWS 미지원) |

- `/29` 서브넷: 8개 IP - 5개 예약 = **실사용 가능 3개**

### 인터넷 게이트웨이 (IGW)

- VPC 리소스가 인터넷에 연결될 수 있도록 허용
- 수평 확장, **고가용성(HA)**, 이중화 자동 지원
- **VPC당 1개**만 연결 가능
- IGW 단독으로는 인터넷 접근 불가; **라우팅 테이블 설정 필수**

### 라우팅 테이블 (Route Table)

| 서브넷 유형 | 라우팅 설정 |
|---|---|
| 퍼블릭 서브넷 | `0.0.0.0/0` → IGW |
| 프라이빗 서브넷 | 인터넷 경로 없음 (또는 → NAT Gateway) |

> 💡 **시험 포인트**:
> - 서브넷당 예약 IP 5개; /29이면 실사용 3개
> - IGW는 VPC당 1개; 라우팅 테이블 없이는 동작 안 함
> - 퍼블릭 서브넷 = 0.0.0.0/0 → IGW 라우팅 존재하는 서브넷
> - IGW는 수평 확장·HA·이중화 자동 제공

---

## 3. NAT Gateway

### 개요

- **AWS 관리형**; AZ 내에서 고가용성(HA)
- 프라이빗 서브넷 인스턴스가 인터넷에 **아웃바운드** 연결 가능; **인바운드 차단**
- **퍼블릭 서브넷에 생성**; **Elastic IP** 필요
- 보안 그룹 불필요
- 요금: 시간당 요금 + 처리 데이터량 기반
- **같은 서브넷의 EC2 인스턴스는 사용 불가**

### 고가용성 (Multi-AZ)

- NAT GW는 **단일 AZ 내에서만** 복원력 제공
- 멀티 AZ 장애 허용(fault tolerance): **AZ마다 NAT GW 1개씩** 생성

```
[AZ-A]                    [AZ-B]
Public Subnet             Public Subnet
  NAT GW (EIP-A)            NAT GW (EIP-B)
      ↑                          ↑
Private Subnet (A)        Private Subnet (B)
```

### NAT Instance (레거시)

| 항목 | NAT Gateway | NAT Instance |
|---|---|---|
| 관리 주체 | AWS 관리형 | 직접 관리 EC2 |
| 가용성 | AZ 내 HA | 직접 구성 필요 |
| 확장성 | 자동 확장 | 수동 확장 |
| 보안 그룹 | 불필요 | 필요 |
| Source/Dest Check | 해당 없음 | **반드시 비활성화** |
| 권장 여부 | 권장 | 비권장 (레거시) |

> 💡 **시험 포인트**:
> - NAT GW는 퍼블릭 서브넷에 생성, Elastic IP 연결 필수
> - 멀티 AZ HA: AZ마다 NAT GW 별도 생성
> - NAT Instance는 Source/Destination Check 비활성화 필수
> - 동일 서브넷 EC2는 NAT GW 사용 불가

---

## 4. 보안 그룹 vs NACL

### 비교표

| 항목 | 보안 그룹 (Security Group) | NACL |
|---|---|---|
| 적용 레벨 | **인스턴스** 레벨 | **서브넷** 레벨 |
| 규칙 유형 | **Allow 규칙만** | Allow + **Deny 규칙** |
| Stateful | **Yes** (리턴 트래픽 자동 허용) | **No** (인/아웃바운드 각각 평가) |
| 규칙 평가 | 모든 규칙 평가 | **번호 순서대로** (first match wins) |
| 기본 적용 | 명시적 연결 필요 | **서브넷의 모든 인스턴스에 자동** |

### NACL 세부 사항

- 서브넷당 **NACL 1개** 적용
- 기본 NACL: 모든 인/아웃바운드 **허용**
- 규칙 번호: **1 ~ 32766**; 번호가 낮을수록 **높은 우선순위**
- 주요 사용 사례: **서브넷 레벨에서 특정 IP 차단**

**임시 포트 (Ephemeral Ports)**:
- 클라이언트는 1024 ~ 65535 범위의 포트를 임시로 선택
- NACL에서 **아웃바운드 임시 포트 허용 규칙** 별도 필요
- OS별 범위: Linux(32768-60999), Windows(49152-65535)

> 💡 **시험 포인트**:
> - NACL = Deny 규칙 가능, Stateless; SG = Allow만, Stateful
> - NACL 규칙은 번호 순서로 평가 (낮은 번호 우선)
> - IP 차단이 필요하면 NACL 사용 (SG는 Deny 불가)
> - NACL은 Stateless이므로 임시 포트 아웃바운드 허용 필요

---

## 5. VPC 피어링

### 개요

- 두 VPC를 **AWS 내부 네트워크**로 프라이빗하게 연결
- CIDR 블록이 **겹치면 불가**
- **비전이적(NOT transitive)**: A↔B + B↔C ≠ A↔C (A↔C 별도 생성 필요)
- **크로스 계정** 및 **크로스 리전** 피어링 가능
- 각 VPC 서브넷의 **라우팅 테이블 모두 업데이트** 필요

```
VPC A ←──── Peering ────→ VPC B ←──── Peering ────→ VPC C
  ↑                                                    ↑
  └──────────── 별도 Peering 생성 필요 ────────────────┘
```

> 💡 **시험 포인트**:
> - VPC 피어링은 비전이적; 삼각 구조면 3개 피어링 필요
> - CIDR 중복 시 피어링 불가
> - 크로스 계정/리전 피어링 가능
> - 양쪽 VPC 라우팅 테이블 모두 업데이트 필수

---

## 6. VPC 엔드포인트

### 개요

- 인터넷, NAT, IGW 없이 **AWS 서비스에 프라이빗하게** 연결
- 트래픽이 AWS 내부 네트워크를 통해서만 이동

### 엔드포인트 유형 비교

| 항목 | Interface Endpoint | Gateway Endpoint |
|---|---|---|
| 구성 방식 | 서브넷에 **ENI (프라이빗 IP)** 프로비저닝 | **라우팅 테이블**에 게이트웨이 추가 |
| 지원 서비스 | **대부분의 AWS 서비스** | **S3, DynamoDB만** |
| 비용 | 유료 | **무료** |
| 기술 기반 | **AWS PrivateLink** | 라우팅 기반 |

**선택 기준**:
- S3 / DynamoDB → **Gateway Endpoint** (무료 + 설정 간단)
- 그 외 서비스 → **Interface Endpoint**

> 💡 **시험 포인트**:
> - Gateway Endpoint: S3, DynamoDB만 지원; 무료
> - Interface Endpoint: 대부분 서비스 지원; 유료; PrivateLink 기반
> - 엔드포인트 사용 시 인터넷 경로 불필요 (보안 강화)
> - 시험에서 "프라이빗하게 S3 접근" → Gateway Endpoint

---

## 7. VPC Flow Logs

### 개요

- VPC, 서브넷, 네트워크 인터페이스의 **IP 트래픽 정보 캡처**
- 연결 문제 모니터링 및 트러블슈팅에 활용

### 주요 특성

- **캡처 대상**: VPC 전체 / 서브넷 / 개별 ENI
- **저장 대상**: S3, CloudWatch Logs, Kinesis Data Firehose
- AWS 관리형 서비스 트래픽도 캡처: ELB, RDS, ElastiCache, Redshift, WorkSpaces, NAT GW, Transit Gateway
- S3에 저장 시 **Athena**로 쿼리 분석 가능
- CloudWatch Logs에 저장 시 **Logs Insights**로 분석

### Flow Log 주요 필드

| 필드 | 설명 |
|---|---|
| `srcaddr` / `dstaddr` | 출발지 / 목적지 IP |
| `srcport` / `dstport` | 출발지 / 목적지 포트 |
| `protocol` | 프로토콜 번호 |
| `packets` / `bytes` | 패킷 수 / 바이트 수 |
| `action` | **ACCEPT** 또는 **REJECT** |
| `log-status` | OK / NODATA / SKIPDATA |

> 💡 **시험 포인트**:
> - Flow Logs action: ACCEPT/REJECT → 보안 그룹·NACL 디버깅에 활용
> - 저장 위치: S3(Athena), CloudWatch Logs(Insights), Kinesis Firehose
> - AWS 관리형 서비스 트래픽도 포함
> - Flow Logs로 실시간 패킷 내용은 확인 불가 (메타데이터만)

---

## 8. Site-to-Site VPN

### 개요

- 온프레미스 네트워크와 AWS VPC를 **인터넷을 통해 암호화** 연결
- **Virtual Private Gateway (VGW)**: AWS 측 VPN 집선 장치; VPC에 연결
- **Customer Gateway (CGW)**: 고객 측 소프트웨어/장치

### 연결 설정

```
[온프레미스]                    [AWS VPC]
Customer Gateway ──── VPN ──── Virtual Private Gateway
    (CGW)           (암호화)         (VGW)
```

- 라우팅 테이블에 **Route Propagation 활성화** 필요
- **Static routing**: 경로 수동 정의
- **Dynamic routing**: BGP 프로토콜 활용

### AWS VPN CloudHub

- 단일 **VGW에 여러 CGW 연결**
- **Hub-and-Spoke** 모델로 VPN 연결 구성
- 저비용 멀티 사이트 HA 연결 솔루션
- 사이트 간 통신도 VGW를 통해 가능

```
          [VGW]
         / | \
       /   |   \
[CGW1] [CGW2] [CGW3]
(서울)  (부산)  (대전)
```

> 💡 **시험 포인트**:
> - VGW: AWS 측; CGW: 고객 측; 둘 다 있어야 VPN 연결 가능
> - Route Propagation 활성화 필수
> - CloudHub: 여러 사이트를 VGW 하나로 연결하는 hub-and-spoke
> - Site-to-Site VPN은 인터넷 경유 (DX와 차이점)

---

## 9. Direct Connect

### 개요

- 온프레미스에서 AWS로의 **전용 프라이빗 물리 연결**
- 특징: 빠름, 안정적, 보안성 높음, 대용량 데이터 전송 비용 절감
- IPv4 + IPv6 지원
- **설치 기간: 1개월 이상** 소요

### 연결 유형

| 유형 | 대역폭 | 특징 |
|---|---|---|
| **Dedicated** | 1, 10, 100 Gbps | 전용 물리 이더넷 포트 (DX 로케이션) |
| **Hosted** | 50 Mbps ~ 10 Gbps | 용량 공유; 온디맨드 용량 변경 가능 |

### Virtual Interface (VIF)

| VIF 유형 | 연결 대상 |
|---|---|
| **Private VIF** | VPC 내 리소스 (VGW 또는 DXGW 경유) |
| **Public VIF** | AWS 퍼블릭 서비스 (S3, Glacier 등) |
| **Transit VIF** | Transit Gateway 연결 |

### Direct Connect Gateway (DXGW)

- **단일 DX 연결**로 **여러 리전의 여러 VPC** 연결 가능
- 크로스 리전 VPC 연결 시 필수

### 복원력 설계

| 구성 | 설명 |
|---|---|
| DX + Site-to-Site VPN (백업) | DX 장애 시 VPN으로 페일오버; 비용 절감 |
| DX + DX (고복원력) | 두 개 DX 연결로 최고 가용성 확보; 비용 높음 |

> 💡 **시험 포인트**:
> - DX = 프라이빗 물리 전용선; 설치 1개월+ 소요
> - Private VIF → VPC; Public VIF → S3 등 퍼블릭 서비스
> - DXGW: 단일 DX로 멀티 리전 VPC 연결
> - VPN을 DX 백업으로 구성 가능 (저비용 복원력)

---

## 10. Transit Gateway

### 개요

- 수천 개의 VPC와 온프레미스를 **전이적(transitive)**으로 연결
- **Hub-and-Spoke** 아키텍처
- **리전 리소스**; 크로스 리전 작동 가능
- **RAM (Resource Access Manager)**으로 계정 간 공유

### 주요 기능

- **라우팅 테이블**로 VPC 간 통신 제어 (어떤 VPC가 어디와 통신할지)
- Direct Connect, Site-to-Site VPN, VPC 피어링과 연동
- **IP Multicast 지원** (다른 AWS 서비스는 미지원)

### ECMP (Equal-Cost Multi-Path)

- 여러 VPN 터널을 동시에 사용하여 **VPN 대역폭 증가**
- Site-to-Site VPN과 함께 사용

```
온프레미스                Transit Gateway
    ├── VPN 터널 1 ──────→ [TGW]──── VPC A
    └── VPN 터널 2 ──────→     └──── VPC B
         (ECMP)                └──── VPC C
```

> 💡 **시험 포인트**:
> - Transit Gateway = 전이적 연결 (VPC 피어링은 비전이적)
> - RAM으로 크로스 계정 공유
> - IP Multicast는 Transit Gateway만 지원
> - ECMP: 다수 VPN 터널로 대역폭 증가

---

## 11. Traffic Mirroring

### 개요

- VPC 내 네트워크 트래픽을 **캡처 및 검사**
- 트래픽을 보안 어플라이언스로 라우팅

### 구성 요소

| 항목 | 설명 |
|---|---|
| **Source** | 트래픽 캡처 대상 ENI |
| **Target** | 트래픽 수신 ENI 또는 NLB |

### 주요 사용 사례

- 콘텐츠 검사 (Content Inspection)
- 위협 모니터링 (Threat Monitoring)
- 네트워크 트러블슈팅

> 💡 **시험 포인트**:
> - Traffic Mirroring: 트래픽 복사본을 보안 어플라이언스로 전달
> - Source는 ENI; Target은 ENI 또는 NLB
> - Flow Logs와 차이: Flow Logs는 메타데이터, Mirroring은 실제 패킷 복사

---

## 12. IPv6 및 Egress-Only IGW

### IPv6

- IPv6 주소는 **모두 퍼블릭** (프라이빗 IPv4와 같은 개념 없음)
- VPC에서 IPv6 활성화 시 **듀얼 스택 모드** (IPv4 + IPv6 동시 사용)
- EC2 인스턴스: IPv4 비활성화 불가; IPv6는 선택적 추가

### Egress-Only Internet Gateway

- **IPv6 전용 NAT Gateway**와 동일한 역할
- IPv6 아웃바운드 트래픽만 허용; 인바운드 차단
- **Stateful** (NAT GW와 동일)
- 라우팅 테이블에 `::/0 → Egress-Only IGW` 추가 필요

| 항목 | NAT Gateway | Egress-Only IGW |
|---|---|---|
| IP 버전 | IPv4 | **IPv6** |
| 방향 | 아웃바운드만 | 아웃바운드만 |
| Stateful | Yes | Yes |

> 💡 **시험 포인트**:
> - IPv6는 모두 퍼블릭 주소; 프라이빗 IPv6 없음
> - Egress-Only IGW = IPv6용 NAT Gateway
> - IPv6 아웃바운드만 허용 → Egress-Only IGW
> - 듀얼 스택: VPC에서 IPv4와 IPv6 동시 사용

---

## 13. 네트워킹 비용

### 데이터 전송 비용 요약

| 전송 경로 | 비용 |
|---|---|
| 동일 AZ, 프라이빗 IP | **무료** |
| 다른 AZ, 프라이빗 IP | **$0.01/GB** |
| 다른 AZ, 퍼블릭/Elastic IP | **$0.02/GB** |
| 크로스 리전 | **$0.02/GB** |

### 비용 절감 전략

- **프라이빗 IP 사용**: 퍼블릭 IP 대비 50% 절감
- **같은 AZ 배치**: 최대 절감 (무료)
- VPC 엔드포인트 사용: NAT GW 데이터 처리 비용 절감
- Direct Connect 사용: 대용량 데이터 전송 비용 절감

> 💡 **시험 포인트**:
> - 같은 AZ 프라이빗 IP: 무료
> - 다른 AZ: 프라이빗 $0.01, 퍼블릭 $0.02
> - 크로스 리전: $0.02/GB
> - 비용 최적화 = 프라이빗 IP + 같은 AZ 우선

---

## 14. AWS Network Firewall

### 개요

- VPC 전체를 **Layer 3 ~ Layer 7** 수준에서 보호
- AWS 관리형 방화벽 서비스

### 적용 트래픽 유형

- VPC ↔ VPC
- 아웃바운드 (인터넷행)
- 인바운드 (인터넷 발)
- Direct Connect / VPN 경유 트래픽

### 주요 기능

| 기능 | 설명 |
|---|---|
| 수천 개 규칙 지원 | IP/포트, 프로토콜 필터링 |
| Active Flow Inspection | 트래픽 흐름 실시간 검사 |
| URL / IP / 도메인 필터링 | Layer 7 수준 제어 |
| 중앙 관리 | **AWS Firewall Manager**로 크로스 계정 관리 |

### 로그 저장

- S3, CloudWatch, Kinesis Data Firehose로 전송

> 💡 **시험 포인트**:
> - Network Firewall: VPC 레벨 L3-L7 방화벽
> - WAF(L7 HTTP), SG(인스턴스), NACL(서브넷)과 역할 구분
> - Firewall Manager: 크로스 계정 중앙 관리
> - 로그: S3 / CloudWatch / Kinesis Firehose

---

← [Section 26. AWS 보안 및 암호화: KMS, SSM Parameter Store, CloudHSM, Shield, WAF](section26.md) | [Section 28. 재해 복구 및 마이그레이션](section28.md) →
