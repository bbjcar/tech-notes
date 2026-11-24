# Microsoft IQ와 Foundry IQ — 무엇이 다르고, 무엇이 필요한가

> 분류: 개념 정리 · 작성 기준: 2026-11-24 (Microsoft Learn 확인) · 관련 제품: Microsoft IQ, Foundry IQ(Azure AI Search), Microsoft 365 Copilot Retrieval API

!!! abstract "한 줄 요약"
    **Microsoft IQ**는 에이전트에 조직 맥락을 공급하는 **4개 계층(Work IQ · Fabric IQ · Foundry IQ · Web IQ)의 묶음 이름**이고,
    **Foundry IQ**는 그중 하나로 **사내 문서·지식을 지식 베이스로 묶어 에이전트에 검색해 주는 서비스**입니다.
    Foundry IQ는 **Azure AI Search의 새 이름**이라서, 쓰려면 Azure에 **검색 서비스 리소스를 따로 만들어야** 합니다.

## 1. 관계 한눈에 보기

```text
Microsoft IQ  (우산 브랜드 · 엔터프라이즈 인텔리전스 계층)
├─ Work IQ     사람·협업·업무 흐름 (M365 메일, 회의, 문서, 활동)
├─ Fabric IQ   비즈니스 데이터의 의미 (엔터티, 관계, 지표, 규칙 · 온톨로지)
├─ Foundry IQ  큐레이션된 사내 지식 (정책, 공식 문서, 재사용 지식 베이스)
└─ Web IQ      최신 웹 정보 (웹, 뉴스, 이미지, 동영상)
```

| 구분 | Microsoft IQ | Foundry IQ |
|---|---|---|
| 성격 | 4개 IQ를 묶은 **개념·브랜드** | 그중 하나인 **실제 서비스** |
| 실체 | 별도 리소스 없음 | **Foundry IQ (Azure AI Search) 서비스** |
| 다루는 것 | 사람·업무 / 비즈니스 데이터 / 지식 / 웹 | 문서·지식 검색(RAG) |
| 주 사용처 | M365 Copilot, Copilot Studio, Foundry 에이전트 전반 | Foundry 에이전트, Agent Framework, 자체 앱 (Copilot Studio 연결도 가능) |

!!! note "세 IQ는 독립적이지만 함께 쓸 수 있습니다"
    공식 FAQ 기준으로 Fabric IQ(비즈니스 분석), Work IQ(M365 협업), Foundry IQ(사내 지식)는 각각 독립 워크로드이고,
    필요하면 조합해서 하나의 에이전트 답변에 쓰는 구조입니다.

## 2. Foundry IQ의 구성 요소

| 구성 요소 | 역할 |
|---|---|
| **지식 베이스 (Knowledge base)** | 최상위 객체. 어떤 지식 원본을 조회할지, 검색 추론 강도(minimal / low / medium), 출력 방식을 정의. 여러 에이전트가 하나를 공유 가능 |
| **지식 원본 (Knowledge source)** | 실제 데이터 연결. **인덱스형** 또는 **원격형** |
| **에이전트형 검색 (Agentic retrieval)** | 질문을 하위 질의로 쪼개 병렬 검색 → 재순위 → 결과 통합하는 엔진. 질의 계획에 LLM을 선택적으로 사용 |

```text
에이전트 / 앱
   │  retrieve 요청
   ▼
지식 베이스 ── 에이전트형 검색 (질의 계획 → 하위 질의 병렬 실행 → 재순위 → 통합)
   ├─ 인덱스형 지식 원본  → Foundry IQ 서비스 안의 검색 인덱스
   └─ 원격형 지식 원본    → 외부 시스템 API를 질의 시점에 직접 호출
```

## 3. Q. 인덱스를 미리 만들어야만 쓸 수 있나요?

**지식 원본 종류에 따라 다릅니다.** 예전 Azure AI Search처럼 "무조건 인덱스부터"는 아닙니다.

| 유형 | 지식 원본 예 | 인덱스 | 특징 |
|---|---|---|---|
| **인덱스형** | 기존 검색 인덱스, 파일 직접 업로드, Azure Blob, Azure SQL, OneLake, SharePoint(인덱스형) | 필요 — 단, **대부분 자동 생성** | 청킹·임베딩·메타데이터 추출을 서비스가 처리. 인덱서 주기 실행으로 증분 갱신. 출처 URL 반환 가능 |
| **원격형** | SharePoint(원격, preview), Fabric 데이터 에이전트(preview), Fabric 온톨로지(preview), MCP 서버(preview), Work IQ(preview), 웹(Bing) | **불필요** | 데이터를 복사하지 않고 질의 시점에 원본 API 호출. 항상 최신. 출처 URL은 반환하지 않음 |

- **파일 업로드형**은 외부 스토리지나 인덱서 없이 파일만 올리면 서비스가 인덱스를 만들어 줍니다.
- **Blob · OneLake · 인덱스형 SharePoint**는 데이터 원본 → 스킬셋 → 인덱서 → 인덱스 파이프라인을 자동으로 만들어 줍니다.

## 4. Q. 결국 Azure에 리소스를 따로 만들어야 하나요?

**네.** 지식 베이스는 **Foundry IQ (Azure AI Search) 서비스** 안에 만드는 객체라서 검색 서비스 리소스가 반드시 필요합니다.

| 항목 | 필요 여부 | 비고 |
|---|---|---|
| Foundry IQ (Azure AI Search) 서비스 | **필수** | 지역별 제공 여부 확인. 모델 접근에 관리 ID를 쓰려면 **Basic 이상** |
| Foundry Agent Service | 선택 | Agent Framework나 자체 앱에서 지식 베이스 API를 직접 호출해도 됨 |
| LLM 배포 (Azure OpenAI in Foundry Models) | 선택 (권장) | `minimal`은 LLM 없이 동작. `low` / `medium`의 질의 계획·답변 합성에는 필요 |
| Microsoft 365 Copilot 라이선스 | 원격 SharePoint를 쓸 때 | 원격 SharePoint 지식 원본은 **최종 사용자에게 M365 Copilot 라이선스**가 있어야 함 |

**비용 구조** (공식 FAQ 기준)

- 검색 서비스 요금(무료 티어 있음)
- 에이전트형 검색 토큰: 무료 할당량을 넘으면 토큰 사용량 기준 과금
- 질의 계획·답변 합성에 쓰는 LLM 토큰: **Azure OpenAI 요금으로 별도** 과금
- Foundry Agent Service는 에이전트 인스턴스 자체에 요금을 매기지 않음

**검색 추론 강도별 한도**

| 강도 | 지식 원본 수 | 하위 질의 | LLM | 답변 합성 |
|---|---|---|---|---|
| minimal | 최대 10 | — | 사용 안 함 | 없음 |
| low | 최대 3 | 최대 3 | 사용 | 5,000 토큰 예산 |
| medium | 최대 5 | 최대 5 | 사용 (반복 검색 포함) | 10,000 토큰 예산 |

## 5. Q. Copilot Retrieval API는 뭔가요?

**Microsoft 365 Copilot이 쓰는 검색 인덱스에서 관련 텍스트 조각(chunk)만 꺼내 주는 Graph API**입니다.
데이터를 별도 인덱스로 복사·청킹하지 않고 **제자리에서(in place)** 권한을 지킨 채 RAG용 근거를 가져옵니다.

| 항목 | 내용 |
|---|---|
| 엔드포인트 | `POST https://graph.microsoft.com/v1.0/copilot/retrieval` |
| 대상 데이터 | SharePoint, OneDrive, Copilot 커넥터(외부 항목) |
| 권한 | **위임 권한만** 지원 (`Files.Read.All` + `Sites.Read.All`, 커넥터는 `ExternalItem.Read.All`). 앱 전용 권한 불가 |
| 보안 | 호출한 사용자 권한으로 결과를 걸러냄(security trimming) |
| 주요 파라미터 | `queryString`(한 문장, 최대 1,500자), `dataSource`, `filterExpression`(KQL), `maximumNumberOfResults`(1~25) |

**Foundry IQ와의 연결 지점**

```text
에이전트 질문
   ↓
Foundry IQ 지식 베이스 ── 질의 계획 (LLM이 질문을 하위 질의로 분해)
   ↓
원격 SharePoint 지식 원본
   ↓
Copilot Retrieval API  (사용자 위임 토큰 · M365 권한·민감도 레이블 적용)
   ↓
텍스트 조각 반환 → 다른 원본 결과와 합쳐 재순위 → 에이전트에 전달
```

- 원격 SharePoint 지식 원본은 내부적으로 Copilot Retrieval API를 호출하므로 **ACL과 Purview 민감도 레이블이 기본 적용**됩니다.
- 자연어를 바로 검색에 던지는 것이 아니라, 지식 베이스의 **질의 계획 단계에서 하위 질의로 다듬은 뒤** 원본별로 호출한다고 이해하면 정확합니다.

## 6. 헷갈리기 쉬운 점

!!! warning "Copilot의 지식 원본과 Foundry IQ의 지식 원본은 서로 호환되지 않습니다"
    개념(에이전트를 사내 데이터에 연결)은 같지만 플랫폼별로 지원 원본이 다르고, Copilot 쪽 지식 원본을 Foundry IQ에서, Foundry IQ 지식 원본을 Copilot에서 그대로 쓸 수 없습니다.
    (Copilot Studio 에이전트에 **Foundry IQ 지식 베이스를 연결**하는 기능은 별도로 제공됩니다.)

!!! warning "사용자별 권한은 자동이 아닙니다"
    인덱스형 원본에서 사용자 단위 접근 제어는 **해당 원본이 문서 수준 권한을 지원하고, 동기화 설정을 명시적으로 켰을 때만** 적용됩니다.

!!! info "네트워크 경계"
    에이전트 → Foundry IQ 구간을 프라이빗으로 막아도, Foundry IQ → 원본·임베딩 모델·질의 계획 모델 구간이 자동으로 보호되지는 않습니다. 구간별로 따로 설계해야 합니다.

## 참고 문서

- [Microsoft IQ documentation](https://learn.microsoft.com/en-us/microsoft-iq/)
- [What is Foundry IQ?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq)
- [Foundry IQ FAQ](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/foundry-iq-faq)
- [What is a knowledge source?](https://learn.microsoft.com/en-us/azure/search/agentic-knowledge-source-overview)
- [Create a knowledge base](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-create-knowledge-base)
- [Microsoft 365 Copilot Retrieval API overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/retrieval/overview)
- [Retrieve grounding data (API reference)](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/retrieval/copilotroot-retrieval)
