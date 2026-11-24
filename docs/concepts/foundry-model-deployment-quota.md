# Foundry 모델 배포·쿼터·배포 유형 — TPM은 속도 상한, 과금은 실제 사용량

> 분류: 개념 정리 · 작성 기준: 2026-11-24 (Microsoft Learn 확인) · 관련 제품: Microsoft Foundry Models, Copilot Studio, Microsoft 365 Copilot

!!! abstract "한 줄 요약"
    **모델 배포**는 Foundry 리소스 안에서 특정 모델을 특정 조건(배포 유형·처리량)으로 쓰겠다고 만드는 Azure 리소스입니다.
    M365 Copilot과 Copilot Studio 기본 모델은 **MS가 관리하는 모델**이라 내 Foundry 배포가 필요 없습니다.
    Standard 계열 배포의 **쿼터(TPM)는 처리 속도 상한**이고, **과금은 실제 쓴 토큰만큼(종량제)** 입니다.

## 1. Q. Foundry에서 모델을 배포하지 않으면 M365 Copilot·Copilot Studio에서 모델을 못 쓰나요?

**아니요. 플랫폼마다 모델 관리 주체가 다릅니다.**

| 플랫폼 | 모델은 누가 관리하나 | 내 Foundry 모델 배포 필요 여부 |
|---|---|---|
| Microsoft 365 Copilot | Microsoft (관리자는 정책으로 사용 가능 모델만 제어) | 불필요 |
| Copilot Studio (기본 모델) | Microsoft 관리 모델 | 불필요 |
| Copilot Studio에서 **Foundry 모델을 직접 연결** | 내 Foundry 리소스 | **필요** — 프롬프트 도구에서 배포 이름·기본 모델 이름·chat completions 엔드포인트로 연결 |
| Copilot Studio → Foundry 에이전트 호출 | Foundry 에이전트가 쓰는 내 배포 | **필요** (Foundry 쪽) |
| Foundry 에이전트·앱 | 내 Foundry 리소스 | **필요** — 단, 아래 instant access 예외 |

```text
[M365 Copilot / Copilot Studio 기본]
사용자 → Copilot 서비스 → Microsoft 관리 모델 인프라          (내 Azure 구독과 무관)

[Foundry]
사용자 → 에이전트/앱 → 내 Foundry 리소스 → 내 모델 배포 → 추론   (내 구독에 과금)
```

!!! tip "배포 없이 바로 호출: instant access (preview)"
    지원 모델은 배포를 만들지 않고 **모델 이름으로 바로 호출**할 수 있습니다. 이 경우 별도 배포·TPM 할당 없이 공유(글로벌) 쿼터를 쓰고 사용량만큼 과금됩니다. 프로토타입·신규 모델 체험용으로 적합합니다.

## 2. Q. 배포할 때 쿼터를 바로 정해야 하나요? 종량제(PAYG)는 안 되나요?

**둘은 반대 개념이 아니라 서로 다른 축입니다.** Standard 계열은 배포 시 처리량(TPM)을 정하지만, 그건 **선결제가 아니라 속도 상한**이고 과금은 종량제입니다.

| 축 | 의미 | 예 |
|---|---|---|
| **쿼터 (TPM, Tokens Per Minute)** | 이 배포가 **분당 최대 얼마나 빨리** 쓸 수 있나 | 200K TPM |
| **과금** | 이번 달 **실제로 얼마나** 썼나 | 입력 30M + 출력 5M 토큰 |
| **배포 유형** | **어디서**, 어떤 용량 풀로 처리하나 | Global Standard |

```text
구독의 모델별 쿼터 풀 (예: 특정 모델 · Global Standard · 1,000K TPM)
   ├─ 배포 A  300K TPM 할당   ← 속도 상한
   ├─ 배포 B  200K TPM 할당
   └─ 남은 쿼터 500K TPM      ← 새 배포에 할당 가능

월말 청구 = 배포 A·B에서 실제 처리한 토큰 수 × 토큰 단가
```

| 방식 | 사전 배포 | 처리량 확보 | 과금 기준 |
|---|---|---|---|
| Standard 계열 (Global / Data Zone / Standard) | 필요 | TPM 할당 (상한) | **실제 토큰 사용량** |
| Provisioned 계열 | 필요 | **PTU로 전용 처리량 예약** | 예약한 용량 |
| Batch 계열 | 필요 | 비동기 (24시간 내 처리) | 사용량, **약 50% 할인** |
| Instant access (preview) | 불필요 | 공유 쿼터 | 실제 토큰 사용량 |

- 쿼터가 부족하면 Foundry의 쿼터 관리 화면에서 증설을 요청합니다.
- 할당한 TPM을 다 못 써도 비용이 나가지 않습니다. 반대로 Provisioned는 안 써도 예약 용량만큼 나갑니다.

## 3. 배포 유형 전체 비교

| 배포 유형 | SKU 코드 | 추론 처리 위치 | 과금 | 적합한 경우 |
|---|---|---|---|---|
| **Global Standard** | `GlobalStandard` | 모든 Azure 지역 | 토큰 종량제 | **기본 시작점**. 신규 모델 최우선, 최저가, 최고 기본 쿼터 |
| Global Provisioned | `GlobalProvisionedManaged` | 모든 Azure 지역 | PTU 예약 | 예측 가능한 대량 처리 |
| Global Batch | `GlobalBatch` | 모든 Azure 지역 | 50% 할인, 24시간 | 대량 비동기 작업 |
| **Data Zone Standard** | `DataZoneStandard` | 데이터 존 내 (US / EU / APAC) | 토큰 종량제 | 데이터 존 단위 규정 준수 |
| Data Zone Provisioned | `DataZoneProvisionedManaged` | 데이터 존 내 | PTU 예약 | 데이터 존 + 처리량 보장 |
| Data Zone Batch | `DataZoneBatch` | 데이터 존 내 | 50% 할인 | 데이터 존 + 대량 비동기 |
| **Standard** | `Standard` | 지정한 Azure 지리(geography) 내 | 토큰 종량제 | 지리 단위 규정 준수, 소량 |
| Regional Provisioned | `ProvisionedManaged` | 지정한 Azure 지리 내 | PTU 예약 | 지리 규정 + 처리량 보장 |
| Developer | `DeveloperTier` | 모든 Azure 지역 | 토큰 종량제 | 파인튜닝 모델 평가 전용 (24시간 수명, SLA 없음) |

!!! info "모든 모델이 모든 유형을 지원하지는 않습니다"
    새 배포 유형은 **Global → Data Zone → 지리 기반** 순서로 열리고, 지리 기반은 가장 늦게 오며 제공 시점이 보장되지 않습니다. 모델·지역별 지원 여부는 매번 확인해야 합니다.

## 4. Q. Standard와 Global Standard의 차이는?

**핵심은 "추론을 어느 지역의 컴퓨팅에서 처리해도 되느냐"** 입니다. 둘 다 토큰 종량제이고 전용 GPU를 예약하지 않는다는 점은 같습니다.

| 구분 | Global Standard | Standard |
|---|---|---|
| 추론 처리 위치 | 모델이 배포된 **모든 Azure 지역** | **지정한 Azure 지리 안** (운영상 그 지리 내 다른 지역 가능) |
| 저장 데이터(at rest) 위치 | 지정한 지리에 유지 | 지정한 지리에 유지 |
| 용량 풀 | 글로벌 | 지리 단위 |
| 기본 쿼터 | 가장 높음 | 상대적으로 낮음 |
| 신규 모델 제공 | **가장 먼저** | 가장 늦음 (시점 미보장) |
| 가격 | 가장 낮음 | 상대적으로 높을 수 있음 |
| 처리량 보장 | Best-effort | Best-effort |
| 고르는 기준 | 일반 워크로드 | **데이터 처리 위치 요건**이 있을 때 |

```text
처리 범위가 넓은 순서

Global Standard      전 세계 Azure 지역
      ▼
Data Zone Standard   US / EU / APAC 같은 데이터 존
      ▼
Standard             지정한 Azure 지리
```

- 예: Foundry 리소스가 Korea Central에 있어도 **Global Standard 배포의 추론은 다른 나라 지역에서 처리될 수 있습니다.** 업로드 파일 등 저장 데이터는 지정 지리에 남습니다.
- Global·Data Zone Standard는 **1차 라우팅 지역에 장애가 나면 그 지역으로 가던 트래픽이 영향을 받습니다.** 업무 연속성이 중요하면 BCDR 가이드를 따로 확인합니다.

## 5. Q. "Best-effort"와 SLA는 어떻게 이해하면 되나요?

| 용어 | 의미 |
|---|---|
| **SLA** (Service Level Agreement) | 공급자와 고객이 합의한 **서비스 수준 협약**. 보통 가용성(가동률), 미달 시 보상 등을 정함 |
| **Best-effort** | 전용 처리량을 보장하지 않고, **그때 가용한 용량 안에서 최대한 처리**하는 방식 |

- 공식 문서 기준으로 **Standard 계열은 best-effort**, **Provisioned 계열은 처리량 보장과 더 낮은 지연 편차**를 제공합니다. **Developer 배포는 SLA가 없습니다.**
- 주의: Standard 계열이 best-effort라는 건 **처리량·지연 시간 특성**에 대한 이야기이지 **"SLA가 없다"는 뜻이 아닙니다.** 가동률 SLA는 배포 유형별로 Azure OpenAI SLA 문서에 따로 정의되어 있습니다.
- 대량 트래픽에서 지연 편차가 문제라면 Provisioned 계열을 검토합니다.

## 참고 문서

- [Deployment types for Microsoft Foundry Models](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/deployment-types)
- [Monitor model deployments](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/monitor-models)
- [Foundry Agent Service limits, quotas, and regional support](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/limits-quotas-regions)
- [Bring your own model for your prompts (Copilot Studio)](https://learn.microsoft.com/en-us/microsoft-copilot-studio/bring-your-own-model-prompts)
- [Connect models to Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/connected-models)
