# CMR(Managed Apps)에 조건부 액세스 적용 — 엔터프라이즈 앱 등록

| 항목 | 내용 |
|---|---|
| 제품 | Copilot Managed Runtime (CMR, 구 Managed Apps) · Microsoft Entra 조건부 액세스 |
| 날짜 | 2026-11 |
| 기준 | 실제 테넌트 적용 경험 + Microsoft Learn 공개 문서 |

## 한 줄 요약

CMR 앱에 **"이 앱에만"** 조건부 액세스(CA)를 걸려면, CMR이 쓰는 Microsoft 리소스 2개를 먼저 테넌트의 **엔터프라이즈 앱(서비스 주체)으로 등록**해야 CA의 대상 리소스 목록에서 검색됩니다. 등록은 Graph Explorer로 하며, **클라우드 애플리케이션 관리자** 또는 **애플리케이션 관리자** 역할이 필요합니다.

## 왜 필요한가

- Microsoft Learn은 CMR 앱이 Entra ID로 로그인하고 **조직의 조건부 액세스 정책을 따른다**고 설명합니다. 정책 집행은 CMR 앱 호스트가 맡습니다.
- 하지만 CA 정책의 **대상 리소스 목록에는 테넌트에 서비스 주체가 있는 앱만** 나옵니다.
- 일부 Microsoft 앱은 Microsoft 쪽 사전 승인으로 동작하므로, 테넌트에 서비스 주체가 **없어도 정상적으로 쓸 수 있습니다.** 그래서 처음에는 CA 대상 목록에서 찾을 수 없습니다. 프리뷰 서비스에서 자주 생기는 상황입니다.

| 정책 방식 | 등록 필요 여부 |
|---|---|
| 대상 리소스 = **모든 리소스** | 보통 필요 없음 (CMR 로그인도 정책 범위에 포함) |
| 대상 리소스 = **CMR만 골라서** 지정 (예: CMR에만 MFA·규정 준수 디바이스 요구) | **필요** — 서비스 주체를 먼저 만들어야 목록에 나옴 |

!!! note "등록 대상은 개별 앱이 아닙니다"
    등록하는 것은 개별 CMR 앱이 아니라 **서비스 입구 2곳**입니다. 앱이 몇 개로 늘어나든 테넌트당 한 번만 등록하면 됩니다.

## 먼저 짚고 갈 개념: 앱 등록 vs 엔터프라이즈 앱

| 구분 | 앱 등록 (Application object) | 엔터프라이즈 앱 (Service principal) |
|---|---|---|
| 비유 | 여권 원본 (설계도) | 우리 건물 출입증 (운영) |
| 어디에 있나 | 앱을 만든 회사의 테넌트에 **1개** | 앱을 **쓰는 테넌트마다 1개** |
| 관리 대상 | 시크릿·인증서, 요청 권한, 리디렉션 주소 | 부여된 권한(동의), 사용자·그룹 할당, 로그인 로그, **조건부 액세스 대상** |
| Microsoft 앱의 경우 | Microsoft 테넌트에 있음 → 우리 앱 등록 메뉴에는 **안 보임(정상)** | 우리 테넌트에 만들면 엔터프라이즈 앱 메뉴에 보임 |

- **애플리케이션(클라이언트) ID(appId)** 는 원본과 모든 테넌트의 사본이 같은 값을 씁니다. 그래서 어느 테넌트에서든 이 값으로 앱을 찾을 수 있습니다.
- **개체 ID** 는 테넌트마다 다릅니다.
- 앱 ID는 공개 식별자이지 비밀이 아닙니다. 보안은 시크릿·인증서, 동의된 권한, 조건부 액세스가 맡습니다.

## 등록 대상 (Microsoft 리소스 2개)

| 애플리케이션 ID | 표시 이름 | 역할 (적용 당시 관찰 기준) |
|---|---|---|
| `423e1d3b-eada-4c4b-9afd-ad3bbc1932d3` | PowerPlatform-Vibe-Prod | Managed Apps 포털 (만들기·관리) |
| `3e62f81e-590b-425b-9531-cad6683656cf` | PowerApps - apps.powerapps.com | 앱 실행 주소 (모든 Managed App 실행) |

!!! warning "공식 목록이 아닙니다"
    이 앱 ID 2개는 Microsoft 안내를 받아 적용한 값입니다. **Microsoft Learn에 공개 목록으로 게시된 값은 아닙니다**(2026-11 기준). GA 이후 로그인 로그에서도 `PowerPlatform-Vibe-Prod`가 그대로 보였지만, 적용 전에 아래 **'GA 이후 앱 ID 확인 방법'** 절차로 직접 확인하는 것을 권장합니다.

## 필요한 역할

| 작업 | 필요한 Entra 역할 |
|---|---|
| 엔터프라이즈 앱 등록 | **클라우드 애플리케이션 관리자** 또는 **애플리케이션 관리자** |
| CA 정책에서 리소스 선택·정책 생성 | **조건부 액세스 관리자** 또는 **보안 관리자** |
| 등록 결과 조회만 | 위 역할 또는 전역 읽기 권한자 |
| 로그인 로그로 차단 확인 | 보고서 읽기 권한자 또는 보안 읽기 권한자 |

- ❌ **AI 관리자로는 등록할 수 없습니다.** 위임 권한 동의는 가능하지만 서비스 주체를 만들 권한이 없습니다.
- 권장: 관리자 역할은 PIM으로 **'적격'** 할당하고 작업할 때만 활성화합니다.

## 등록 절차 (Graph Explorer · 권장)

준비물: 위 역할이 있는 회사 계정, 브라우저(별도 설치 없음), 등록할 앱 ID 2개.

### 1단계. 이미 등록돼 있는지 확인

1. Entra 관리 센터 → **Entra ID → 엔터프라이즈 앱 → 모든 애플리케이션**
2. 애플리케이션 유형 필터를 **'Microsoft 애플리케이션'** 으로 바꾼 뒤 앱 ID로 검색합니다.
3. 이미 있으면 → 5단계로 갑니다.

!!! tip "기본 필터에서는 안 보입니다"
    엔터프라이즈 앱의 '애플리케이션 유형'은 **원본을 누가 만들었는지**로 정해집니다. 우리가 직접 등록해도 Microsoft 앱은 'Microsoft 애플리케이션'으로 분류되고, 기본 필터(엔터프라이즈 애플리케이션)에서는 보이지 않습니다.

Graph로 확인하려면:

```http
GET https://graph.microsoft.com/v1.0/servicePrincipals(appId='<앱 ID>')
```

`200`이면 이미 있음, `404`면 없음입니다.

### 2단계. Graph Explorer 로그인

- [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer) 접속 → 오른쪽 위에서 회사 계정으로 로그인합니다.
- PIM을 쓰는 경우, 로그인 **전에** 역할부터 활성화합니다.

### 3단계. 권한 동의

- 오른쪽 위 **프로필 아이콘 → Consent to permissions**
- `Application.ReadWrite.All` 검색 → **Consent** → 권한 요청 창에서 수락
- (다른 방법) 쿼리를 입력한 뒤 **Modify permissions** 탭에서 동의

!!! danger "'조직을 대신하여 동의'는 체크하지 않습니다"
    체크하면 다른 계정도 이 권한으로 Graph Explorer를 쓸 수 있게 됩니다.

### 4단계. 등록 요청 실행 (앱 ID마다 1번씩, 총 2번)

```http
POST https://graph.microsoft.com/v1.0/servicePrincipals
Content-Type: application/json

{
  "appId": "423e1d3b-eada-4c4b-9afd-ad3bbc1932d3"
}
```

```http
POST https://graph.microsoft.com/v1.0/servicePrincipals
Content-Type: application/json

{
  "appId": "3e62f81e-590b-425b-9531-cad6683656cf"
}
```

**Run query** → 응답이 `201 Created`이고, 응답 본문의 `appDisplayName`이 위 표의 표시 이름과 같으면 성공입니다.

### 5단계. 결과 확인

- 엔터프라이즈 앱 목록(필터: Microsoft 애플리케이션)에 두 이름이 보이는지 확인합니다.
- **조건부 액세스 → 정책 → 새 정책 → 대상 리소스 → 리소스 선택** 에서 앱 ID로 검색해 두 리소스가 나오면 완료입니다.

### 6단계. 권한 정리

- Graph Explorer에 준 `Application.ReadWrite.All` 동의를 철회합니다. (Graph Explorer 권한 목록, 또는 Entra → 엔터프라이즈 앱 → Graph Explorer → 권한)
- PIM 역할을 비활성화합니다.

## 작업 체크리스트

- [ ] 역할 확인 (클라우드 애플리케이션 관리자 / 애플리케이션 관리자)
- [ ] 'Microsoft 애플리케이션' 필터로 기존 등록 여부 확인
- [ ] Graph Explorer 로그인 · `Application.ReadWrite.All` 동의 (조직 대신 동의 체크 안 함)
- [ ] `423e1d3b-…` 등록 → 201 Created
- [ ] `3e62f81e-…` 등록 → 201 Created
- [ ] CA 대상 리소스에서 두 앱 ID 검색 확인
- [ ] Graph Explorer 권한 철회 · PIM 비활성화

## 다른 등록 방법 (명령줄)

필요한 역할은 같고, 도구를 따로 설치해야 합니다.

**Microsoft Graph PowerShell** (`Microsoft.Graph.Applications` 모듈 필요)

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"
New-MgServicePrincipal -AppId <앱 ID>
```

**Azure CLI** (Azure CLI 설치 필요)

```bash
az login --allow-no-subscriptions
az ad sp create --id <앱 ID>
```

## 문제 해결

| 증상 | 원인 | 조치 |
|---|---|---|
| 등록했는데 목록에 안 보임 | 필터가 '엔터프라이즈 애플리케이션' | 'Microsoft 애플리케이션' 또는 '모든 애플리케이션'으로 변경 |
| 동의 단계에서 '관리자 승인 필요' 화면 | 계정에 관리자 역할이 없거나 PIM 미활성 | 역할 활성화 후 다시 로그인 |
| 4단계에서 `403 Forbidden` | 동의는 됐지만 역할이 부족함 (예: AI 관리자) | 클라우드 애플리케이션 관리자 / 애플리케이션 관리자 계정으로 실행 |
| 4단계에서 '이미 있음' 오류 | 이미 등록돼 있음 | 5단계로 진행 |
| 관리자 동의 URL로 시도하면 `AADSTS65002` | Microsoft 자사 앱끼리의 권한은 Microsoft 사전 승인이 필요해 테넌트 관리자 동의로 처리되지 않음 | 관리자 동의 URL 방식은 쓰지 않고 Graph로 서비스 주체를 직접 생성 |
| 정책 적용 후 접속 차단 (오류 코드 `53003`) | CA 정책에 의해 차단됨 (등록과는 별개 작업) | 로그인 로그에서 적용된 정책 확인 |

## GA 이후 앱 ID 확인 방법

프리뷰 때 등록한 앱 ID가 GA 이후에도 쓰이는지는 **로그인 로그**로 확인합니다.

1. Entra 관리 센터 → **모니터링 및 상태 → 로그인 로그**
2. CMR 앱을 한 번 열어본 뒤, **사용자 로그인(대화형)** 탭에서 시간·사용자로 필터
3. 해당 로그의 **애플리케이션 / 애플리케이션 ID** 를 위 표의 앱 ID와 비교

| 로그에 보이는 값 | 해석 |
|---|---|
| 애플리케이션 = `PowerPlatform-Vibe-Prod`, 리소스 = Microsoft Graph | 정상. 등록한 앱으로 로그인 중이며, 기본 프로필 조회 때문에 리소스가 Graph로 기록됨 |
| 리소스 = Azure Resource Manager (`797f4846-ba00-4fd7-ba43-dac1f8f63013`) | CMR과 무관. Entra 관리 센터 등 Azure 관리 화면에 접속한 기록 |
| 위 표에 없는 새 애플리케이션 ID | GA에서 새 앱이 쓰일 가능성 → 그 ID로 1~5단계를 반복해 등록 |

## 등록 후 운영 시 주의할 점

- CA 정책은 **보고 전용(Report-only)** 으로 시험 그룹에 먼저 적용합니다. 로그인 로그로 영향을 확인한 뒤 켭니다.
- `PowerApps - apps.powerapps.com`에 정책을 걸면 **Power Apps의 다른 앱 유형도 같은 조건을 받을 수 있습니다.** 사전에 영향 범위를 확인합니다.
- 등록·동의 권한이 있는 역할은 PIM '적격'으로 두고, Entra 감사 로그의 **'서비스 주체 추가'** 이벤트를 정기 점검합니다.
- CA와 별개로, CMR 자체 통제(환경 보안 그룹, 공유 제한, CSP, 고급 커넥터 정책)도 함께 검토합니다.

## 참고

- [Microsoft Copilot Managed Runtime overview for admins (Microsoft Learn)](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/apps/?view=o365-worldwide)
- [Manage security and compliance for Copilot Managed Runtime (Microsoft Learn)](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/apps/security-compliance?view=o365-worldwide)
- [Create an enterprise application from a multitenant application (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/create-service-principal-cross-tenant)
- [Create servicePrincipal (Microsoft Graph)](https://learn.microsoft.com/en-us/graph/api/serviceprincipal-post-serviceprincipals?view=graph-rest-1.0)
- [Work with Graph Explorer (Microsoft Learn)](https://learn.microsoft.com/en-us/graph/graph-explorer/graph-explorer-features)
- [Microsoft Entra built-in roles (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference)
