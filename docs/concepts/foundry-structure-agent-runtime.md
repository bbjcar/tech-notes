# Foundry 구조와 에이전트 실행 방식 — 리소스·프로젝트·관측, 프롬프트형 vs 호스트형

> 분류: 개념 정리 · 작성 기준: 2026-11-24 (Microsoft Learn 확인) · 관련 제품: Microsoft Foundry, Foundry Agent Service, Azure Monitor

!!! abstract "한 줄 요약"
    **Foundry 리소스**가 Azure의 관리 경계(네트워크·보안·모델 배포)이고 **프로젝트**는 그 안의 개발 경계입니다.
    **프롬프트형 에이전트**는 정의(모델+지시문+도구)만 저장하고 MS가 관리하는 런타임에서 실행되며,
    **호스트형 에이전트**는 내 컨테이너를 **세션마다 VM 격리 샌드박스**로 띄웠다가 유휴 시간이 지나면 내립니다. 에이전트별로 24시간 켜져 있는 전용 VM은 없습니다.

## 1. 리소스 계층

```text
Azure 구독
└─ 리소스 그룹
   └─ Microsoft Foundry 리소스   (Microsoft.CognitiveServices/accounts · kind: AIServices)
      ├─ 모델 배포, 네트워크, 보안, 연결(Connections)
      ├─ 프로젝트 A                (accounts/projects · 하위 리소스)
      │   ├─ 에이전트, 평가, 파일
      └─ 프로젝트 B

(별도 리소스 · 별도 거버넌스) Storage, Key Vault, Foundry IQ(Azure AI Search) …
```

| 계층 | 역할 | 주로 다루는 곳 |
|---|---|---|
| Foundry 리소스 | 네트워크, 보안, 모델 배포, RBAC 등 **거버넌스 경계** | Azure 포털 (Foundry 포털·CLI·IaC로도 생성 가능) |
| 프로젝트 | 팀·유스케이스별 **개발 경계**. 상위 리소스의 모델 배포·연결 재사용 | Foundry 포털, SDK, CLI |
| 연결된 리소스 | Storage, Key Vault, 검색 등 | 각 리소스에서 **따로** 네트워크·권한 관리 |

!!! tip "처음 시작할 때"
    1인 실험은 Foundry 리소스 1개 + 프로젝트 1개가 기본 권장 구성입니다. 에이전트·평가 없이 모델 호출만 필요하면 단독 Azure OpenAI 리소스로도 충분할 수 있습니다.

## 2. Q. 프로젝트 관측은 Azure에서 하는 게 맞나요?

**둘 다 맞습니다. 보는 대상이 다릅니다.**

| 관측 위치 | 무엇을 보나 | 준비할 것 |
|---|---|---|
| **Foundry 포털 → 프로젝트 → Build → Monitor** (preview) | 에이전트 활동량, 에이전트·모델 지연 시간, 토큰 사용량, 오류, 프로젝트 전체 트레이스, 품질 평가, 추정 비용 (최대 3개월 조회) | 기본 지표는 바로 보임. **오류·트레이스·품질**은 프로젝트에 **Application Insights 연결** 필요 |
| **Azure 포털 → Azure Monitor** | 모델 배포의 지표(요청 수, 토큰, 지연 등), 진단 로그, 알림 | 지표는 자동 수집. 로그는 **진단 설정** 구성 필요 |
| **Azure 포털 → Cost Management** | 실제 청구 비용 (입·출력 토큰 비용) | 비용 반영까지 약 5시간 지연 |

- 권한: 지표 조회는 **Monitoring Reader**, 진단 설정은 **Monitoring Contributor**, 트레이스 조회는 연결된 Application Insights / Log Analytics에 **Log Analytics Reader**가 필요합니다.
- 정리하면 **"에이전트가 어떻게 동작했나"는 Foundry 포털**, **"리소스·배포가 건강한가, 얼마 썼나"는 Azure 포털**에서 보는 구조입니다.

## 3. 프롬프트형 vs 호스트형 에이전트

| 구분 | 프롬프트형 (Prompt agent) | 호스트형 (Hosted agent) |
|---|---|---|
| 만드는 방식 | 포털·SDK에서 모델, 지시문, 도구 등 **정의만** 구성 | 내 코드(Agent Framework, LangGraph, Semantic Kernel 등)를 **컨테이너 이미지**로 빌드 → ACR → 배포 |
| 오케스트레이션 | MS 관리 런타임이 수행 | **내 코드**가 수행 |
| 실행 환경 | MS 관리 런타임 (CPU·메모리·VM 개념이 고객에게 노출되지 않음) | **세션별 VM 격리 샌드박스** |
| 컴퓨팅 설정 | 불가 | CPU·메모리 지정 가능 |
| 상태 유지 | 대화(conversation) 단위로 플랫폼이 관리 | 세션별 `$HOME`, `/files` 영속 파일시스템 |
| 신원 | 프로젝트 관리 ID 등 | 배포 시 **전용 Entra 에이전트 ID** 부여 |
| 적합한 경우 | 빠른 구성, 표준 도구 조합 | 자체 프레임워크, 웹훅 등 사용자 지정 프로토콜, 장시간·상태 유지 작업 |

## 4. Q. 에이전트마다 VM을 받아서 24시간 대기하나요?

**아니요.** 질문에서 제시한 두 번째 모델, 즉 **"평소엔 정의만 있고, 요청·세션이 생길 때 실행 환경을 할당"**하는 쪽에 가깝습니다.

**호스트형 에이전트의 세션 수명주기**

```text
요청 도착 ─▶ 세션용 샌드박스 할당 (VM 격리, 내 컨테이너 실행)
              │
              ├─ 대화 진행 ($HOME · /files 유지)
              │
              ├─ 유휴 시간 초과 (기본 15분, 2~240분 설정 가능)
              │     └─▶ 컴퓨팅 해제 (scale-to-zero), 파일시스템은 보존
              │
              ├─ 같은 세션으로 다시 호출 ─▶ 컴퓨팅 재할당 + 상태 복원
              │
              └─ 30일간 비활성 ─▶ 세션 영구 삭제
```

- 세션끼리는 서로 격리되고, `$HOME` 저장 공간은 새 세션 기준 최대 100 GiB(이미지·CPU 등급에 따라 다름)입니다.
- 세션 ID 없이 호출하면 매번 새 샌드박스가 뜹니다. 파일이나 `$HOME` 상태를 이어 쓰려면 `agent_session_id`를 계속 보내거나 conversation 객체를 써야 합니다.

**프롬프트형 에이전트**

- 고객이 VM을 할당받는 구조가 아닙니다. 요청이 오면 MS 관리 런타임이 모델 추론과 도구 호출을 조율합니다.
- 내부적으로 요청마다 실행 환경을 즉시 만들고 지우는지는 **공개되지 않은 구현 영역**입니다. 문서에서 확인되는 것은 "고객이 컴퓨팅을 설정·관리하지 않는다"까지입니다.
- 프롬프트형에 **Copilot harness**를 켜면 대화마다 별도 작업 공간이 생기고, 샌드박스의 할당·유휴·재개·교체를 서비스가 관리합니다. CPU·메모리·수명주기 설정은 고객이 바꿀 수 없습니다.

## 5. Q. 둘 다 MS가 관리하는 런타임인데 뭐가 다른가요?

| 관점 | 관리형 에이전트 런타임 (프롬프트형) | VM 샌드박스 (호스트형) |
|---|---|---|
| 그 안에서 도는 코드 | MS의 오케스트레이터 | **내 컨테이너** |
| 고객이 책임지는 범위 | 정의(지시문·도구·모델 선택) | 코드, 의존성, 이미지, 자체 안전 장치 |
| 격리 단위 | 서비스 내부 (비공개) | **세션 단위 VM 격리** (문서에 명시) |
| 리소스 조정 | 불가 | CPU·메모리, 유휴 시간 |

"둘 다 결국 MS 데이터센터의 서버 위에서 돈다"는 점은 같습니다. 차이는 **누구의 코드가 실행되느냐, 그리고 격리·자원을 고객이 얼마나 제어하느냐**입니다.

## 6. Q. Power Platform처럼 쿼터·한도가 있나요?

**있지만 걸리는 위치가 다릅니다.** Foundry Agent Service는 API 호출 자체에 별도 속도 제한을 두지 않고, **에이전트가 호출하는 모델 배포의 쿼터(TPM 등)**에서 제한이 걸립니다. 초과하면 `429 rate_limit_exceeded`가 반환됩니다.

에이전트 서비스 자체의 고정 한도 (증설 불가)

| 항목 | 한도 |
|---|---|
| 에이전트당 등록 도구 수 | 128 |
| 에이전트·스레드당 파일 수 | 10,000 |
| 파일 1개 크기 | 512 MB |
| 업로드 파일 총량 | 300 GB |
| 벡터 저장소에 붙이는 파일 크기 | 2,000,000 토큰 |
| 스레드당 메시지 수 | 100,000 |
| 메시지당 텍스트 크기 | 1,500,000자 |

!!! note "Copilot Studio와 비교할 때"
    Copilot Studio 에이전트도 MS 관리 런타임에서 실행되고, 사용량은 Copilot 크레딧 기반으로 관리됩니다.
    Foundry는 **모델 배포 쿼터 = 처리 속도 상한**, **실제 토큰 사용량 = 과금**으로 나뉘어 있다는 점이 다릅니다. 자세한 내용은 [모델 배포·쿼터·배포 유형](foundry-model-deployment-quota.md)에 정리했습니다.

## 참고 문서

- [Microsoft Foundry architecture](https://learn.microsoft.com/en-us/azure/foundry/concepts/architecture)
- [Project-level monitoring (preview)](https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/monitor-project)
- [Monitor model deployments](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/monitor-models)
- [What are hosted agents?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agents)
- [Manage hosted agent sessions](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/manage-hosted-sessions)
- [Foundry Agent Service limits, quotas, and regional support](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/limits-quotas-regions)
