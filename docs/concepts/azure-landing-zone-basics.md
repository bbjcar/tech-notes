# Azure Landing Zone 기초 — 구조와 핵심 용어 27개

> 분류: 개념 정리 · 작성 기준: 2026-11-26 (Microsoft Learn CAF 확인) · 관련 제품: Azure, Cloud Adoption Framework, ExpressRoute, Azure Firewall, Application Gateway

!!! abstract "한 줄 요약"
    **Landing Zone**은 Azure를 회사 표준으로 쓰기 위해 네트워크·보안·계정·정책·로그를 **미리 깔아 두는 기본 설계도**입니다.
    교육 예시를 한 문장으로 줄이면 이렇습니다.
    **"회사와 Azure를 ExpressRoute로 연결하고, Hub의 Firewall/UTM으로 보안을 통합 관리하며, 업무 시스템은 Spoke VNet에 두고, 웹 서비스는 Application Gateway WAF로 보호한다."**

## 1. Q. Landing Zone이 정확히 뭔가요?

집을 짓기 전에 도로를 내고 전기·수도를 연결하고 경비실을 세우는 것과 같습니다. Microsoft Learn(Cloud Adoption Framework)의 정의는 **여러 구독으로 이루어진 Azure 환경을 거버넌스·보안·확장 관점에서 관리하기 위한 검증된 아키텍처**입니다.

| 구성 요소 | 하는 일 | 누가 관리하나 |
|---|---|---|
| **플랫폼 랜딩 존** (Platform landing zone) | 관리 그룹 계층, 공통 네트워크·보안 모니터링·ID·관리 서비스, 정책(가드레일) | 플랫폼 팀 (보통 테넌트당 1개) |
| **워크로드 랜딩 존** (Workload landing zone, 예전 이름: Application landing zone) | 업무 시스템 하나가 쓰는 개발·테스트·운영 환경(구독) | 워크로드(업무) 팀 |

- **Workload** = 하나의 업무 목적을 위해 함께 동작하는 리소스·코드·데이터·AI 모델 묶음입니다. 예: ERP, MES, 그룹웨어, RPA, 홈페이지.
- 워크로드 구독은 관리 그룹(Online / Corp / Local)에 배치되어 **Azure Policy를 상속**합니다. 그래서 새 시스템을 올려도 같은 보안 기준이 자동으로 적용됩니다.
- 구축 방법은 두 가지입니다. **Accelerator**(Microsoft가 제공하는 IaC 템플릿)로 배포하거나, 직접 설계(**Custom build**)합니다. 대부분은 Accelerator가 가장 빠릅니다.

## 2. 전체 구조 한눈에 보기

```text
① 회사 → Azure 연결          ② Hub: 공통 서비스
회사                          Hub VNet
 ↓ ExpressRoute (전용회선)     ├─ Firewall (Azure Firewall / UTM)
ExpressRoute Gateway          ├─ ExpressRoute Gateway
 ↓                            ├─ DNS
Azure                         └─ Logging

③ Spoke: 업무 시스템          ④ 웹 서비스 보호
Hub                           사용자 (인터넷)
 ├─ ERP  (Spoke VNet)          ↓ HTTPS
 ├─ MES  (Spoke VNet)         Application Gateway + WAF
 ├─ SCM  (Spoke VNet)          ↓
 └─ RPA  (Spoke VNet)         Web Server
```

**공식 참조 아키텍처(Hub-Spoke)와 맞춰 보면** 교육의 Hub는 **Connectivity 구독**에 해당합니다.

| 공식 참조 아키텍처의 구독 | 들어가는 것 | 교육 용어와의 연결 |
|---|---|---|
| Connectivity | Hub VNet, Azure Firewall, VPN/ExpressRoute Gateway, Azure DNS, DDoS Protection | Hub, Firewall, ExpressRoute Gateway, DNS |
| Management · Security | Log Analytics 작업 영역, Microsoft Sentinel, 대시보드 | Logging, Log Analytics, Sentinel |
| Identity | 도메인 서비스 VNet, Recovery Services 자격 증명 모음 | Entra ID, (온프레미스 AD 연동) |
| Workload landing zone | 업무 시스템 VNet(Spoke)·리소스 그룹·리소스 | Spoke, Workload |

모든 구독에 공통으로 붙는 도구도 있습니다. Policy·Role 할당, Cost Management, Defender for Cloud, Network Watcher, Update Manager 등입니다.

## 3. 반드시 기억할 핵심 키워드 5개

| 키워드 | 한 줄 의미 |
|---|---|
| **Landing Zone** | Azure 표준 구축 방법론 (기본 설계도) |
| **Hub-Spoke** | 중앙(Hub)에 공통 서비스, 바깥(Spoke)에 업무 시스템을 두는 네트워크 구조 |
| **ExpressRoute** | 회사 ↔ Azure 전용선 |
| **Firewall / UTM** | 네트워크 출입 통제 |
| **WAF** | 웹 공격 방어 |

## 4. 핵심 용어집 (27개)

### 기본 개념

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 1 | Landing Zone | Azure를 기업 환경에서 표준화해 쓰기 위한 기본 설계도 | 네트워크·보안·계정·정책·로그를 미리 설계 |
| 2 | Workload | 업무 시스템 하나 | ERP, MES, 그룹웨어, RPA, 홈페이지 |

### 네트워크

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 3 | VNet (Virtual Network) | Azure 안의 사설 네트워크 | 회사 내부망 = VNet (Web / App / DB) |
| 4 | Subnet | VNet 내부를 나눈 영역 | 건물을 영업부·개발부·회의실로 나누듯 AG / VM / DB Subnet |
| 5 | Hub | 공통 서비스를 모아 두는 중앙 네트워크 | Firewall, ExpressRoute, DNS, Logging 배치 |
| 6 | Spoke | 실제 업무 시스템이 들어가는 네트워크 | ERP, MES, SCM, RPA 각각이 Spoke |
| 7 | Hub-Spoke | 중앙 집중형 네트워크 구조 | 보안 관리가 쉽고, 운영 비용이 줄고, 정책을 통일할 수 있음 |

### 온프레미스 연결

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 8 | ExpressRoute | 회사 ↔ Azure 전용선 | 인터넷 대신 전용회선 → 빠르고 안정적이며 보안성이 높음 |
| 9 | ExpressRoute Gateway | Azure 쪽에서 ExpressRoute를 받는 관문 | 회사 → ExpressRoute → Gateway → Azure |

### 보안

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 10 | Firewall | 네트워크 출입 통제 장비 | 건물 경비실. 443 허용, 80·1234 차단 |
| 11 | UTM | 여러 보안 기능을 묶은 통합 보안 장비 | Firewall + IPS + URL Filtering + 악성코드 탐지. 예: Fortigate, Palo Alto, Check Point |
| 12 | IPS | 침입 탐지 및 차단 | 악성 공격 발견 → 자동 차단 |
| 13 | URL Filtering | 특정 사이트 접속 차단 | 토렌트·SNS 사이트 차단 |

### 웹 보안

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 14 | Application Gateway | Azure의 웹(L7) 로드밸런서 | HTTPS 처리, 부하 분산, URL 라우팅 |
| 15 | WAF | 웹 공격 전용 방화벽 | SQL Injection, XSS 등 OWASP Top 10 차단 |
| 16 | SSL Offload | HTTPS 암호화 처리를 서버 대신 Gateway가 수행 | 사용자 →(HTTPS)→ Gateway →(HTTP)→ 웹서버. 서버 부하 감소 |
| 17 | HTTPS Redirect | HTTP 접속을 HTTPS로 자동 전환 | `http://` → `https://` |

### Azure 보안

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 18 | NSG (Network Security Group) | Subnet 또는 NIC에 붙이는 포트·IP 기반 허용/차단 규칙 | 443 허용, 3389(RDP)·22(SSH) 차단 |
| 19 | Azure Firewall | Microsoft가 제공하는 관리형 클라우드 방화벽 | 보통 Hub에 두고 전체 트래픽을 중앙 통제 |

!!! tip "NSG vs Azure Firewall"
    NSG는 **각 Subnet 앞의 출입문 규칙**(L3/L4)이고, Azure Firewall은 **Hub에 있는 중앙 경비실**(FQDN 필터, 위협 인텔리전스, 중앙 로그)입니다. 보통 둘 다 함께 씁니다.

### 인증 · DNS

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 20 | Microsoft Entra ID (구 Azure AD) | Azure·M365 계정 관리 시스템 | 사용자 → Entra ID 로그인 → Azure 접근 |
| 21 | DNS | 이름을 IP로 바꾸는 전화번호부 | `www.example.com` → `203.0.113.10` |
| 22 | Private DNS | 회사(VNet) 내부용 DNS | 외부에서는 조회 불가 |

### 운영 · 모니터링

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 23 | Log Analytics | Azure 로그 저장·분석소 | WAF, VM, Azure 활동 로그 저장 |
| 24 | Microsoft Sentinel | 클라우드 SIEM (보안 관제) | 보안 로그 수집, 이상 징후 탐지, 공격 분석 |
| 25 | Defender for Cloud | Azure 보안 상태 점검 서비스 | VM에 공인 IP가 있거나 포트가 열려 있으면 경고 |

### 고가용성

| # | 용어 | 한 줄 설명 | 쉽게 말하면 / 예시 |
|---|---|---|---|
| 26 | Availability Zone | 같은 지역 안의 서로 다른 데이터센터 | 한 Zone에 장애가 나도 서비스 유지 |
| 27 | Multi-Zone | 서비스를 여러 Zone에 나눠 배치 | App Server를 Zone 1·2·3에 분산 |

## 5. 정리 퀴즈

??? question "Q1. 회사 사무실과 Azure를 인터넷이 아닌 전용회선으로 연결하는 서비스는?"
    **ExpressRoute**. Azure 쪽에서는 ExpressRoute Gateway가 받습니다.

??? question "Q2. Hub-Spoke에서 ERP·MES 같은 업무 시스템은 어디에 두나요?"
    **Spoke VNet**. Hub에는 Firewall, Gateway, DNS, 로그 같은 공통 서비스를 둡니다.

??? question "Q3. 공식 문서에서 랜딩 존을 이루는 두 구성 요소는?"
    **플랫폼 랜딩 존**(공통 기반·정책, 플랫폼 팀)과 **워크로드 랜딩 존**(업무 시스템별 환경, 워크로드 팀)입니다.

## 참고 자료

- [What is an Azure landing zone? — Cloud Adoption Framework (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/)
- [Hub-spoke network topology in Azure — Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/hub-spoke)
- [What is Azure ExpressRoute? — Microsoft Learn](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-introduction)
- [What is Azure Web Application Firewall on Application Gateway? — Microsoft Learn](https://learn.microsoft.com/en-us/azure/web-application-firewall/ag/ag-overview)
