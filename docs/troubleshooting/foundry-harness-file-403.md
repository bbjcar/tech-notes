# Foundry 하네스 에이전트 파일 반환 시 403

| 항목 | 내용 |
|---|---|
| 제품 | Microsoft Foundry · 프롬프트 에이전트 (Microsoft Copilot harness, 프리뷰) |
| 분류 | 권한 |
| 날짜 | 2026-11 |
| 상태 | 문서 기반 예시 (실제 프로젝트 기록 아님) |

!!! info "이 글에 대해"
    장애 해결 글의 형식을 보여주기 위해 Microsoft Learn 문서에 나온 사례로 작성한 예시입니다.

## 증상

- 에이전트가 모델 호출, 셸 명령, 작업공간 파일 생성까지는 정상으로 끝난다.
- 생성한 파일을 다운로드 가능한 결과물로 반환하는 단계에서만 **HTTP 403**이 난다.

## 원인

하네스는 생성 파일을 **에이전트 자신의 ID**로 Foundry Files에 업로드합니다. 이 에이전트 ID(`instance_identity.principal_id`)에 프로젝트 범위 **Foundry User** 역할이 없거나, 부여 직후 아직 반영되지 않은 상태입니다.

- 내 계정(만든 사람)의 역할과는 별개입니다.
- 에이전트를 만든 뒤에야 부여할 수 있습니다.

## 해결

1. 에이전트 ID 조회 후 역할 부여 (역할 이름 대신 GUID 사용 권장)

    ```bash
    AGENT_IDENTITY_ID=$(az rest --method GET \
      --url "$PROJECT_ENDPOINT/agents/$AGENT_NAME?api-version=v1" \
      --resource "https://ai.azure.com" \
      --query "instance_identity.principal_id" --output tsv)

    az role assignment create \
      --assignee-object-id "$AGENT_IDENTITY_ID" \
      --assignee-principal-type ServicePrincipal \
      --role "53ca6127-db72-4b80-b1b0-d745d6d5456d" \
      --scope "$PROJECT_ID"
    ```

2. 최대 5분 정도 반영을 기다린 뒤 **새 대화**에서 다시 테스트

!!! warning "주의"
    프로젝트 관리 ID(project managed identity)에 부여하면 해결되지 않습니다. 반드시 에이전트 ID에 부여합니다.

## 재발 방지

- [ ] 에이전트 배포 스크립트에 역할 부여 단계를 포함
- [ ] Toolbox를 붙이는 에이전트도 같은 역할이 필요함을 체크리스트에 추가

## 참고

- [Use the Microsoft Copilot harness with prompt agents (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/use-tools-skills-copilot-harness)
