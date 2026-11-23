# Standard → Microsoft Copilot harness 업그레이드

| 항목 | 내용 |
|---|---|
| 제품 | Microsoft Copilot Studio |
| 상태 | 업그레이드 기능 프리뷰 (2026-11 기준) |
| 날짜 | 2026-11 |

## 사용 방법

1. Copilot Studio **홈 화면 입력창**에 자연어로 요청 (예: `upgrade my Coffee Coach agent`)
2. 채팅에서 진행 상황 확인, 대상 에이전트·기능 처리 방식 질문에 답변
3. 완료되면 새 에이전트(이름 뒤 **(Upgraded)**)와 워크플로가 카드로 표시됨

원본을 바꾸지 않고 **새 하네스 위에 복사본을 만듭니다.** 원본과 나란히 비교할 수 있습니다.

## 옮겨지는 것 / 안 옮겨지는 것

| 옮겨짐 | 직접 다시 설정 |
|---|---|
| 지시문, 도구, 추천 프롬프트 | 지식 원본 |
| 사용자 토픽 → 스킬·워크플로로 재구성 | Azure AI Search (미지원 → Foundry IQ 또는 커넥터 도구로 대체) |
| 마이그레이션 보고서 제공 | Power Fx 식, 자식 에이전트 |

## 과금 차이 (가장 중요)

| 구분 | Standard | Microsoft Copilot harness |
|---|---|---|
| 과금 단위 | 이벤트 단가 (생성형 답변, 작업, 그라운딩 등) | 소비량 (LLM 토큰 + 도구·지식·MCP + 하네스) |
| 과금 시작 | 게시 이후 | 만들기 시작부터 (테스트·평가 포함) |
| M365 Copilot 라이선스 사용자의 M365 채널 사용 | 포함 | 포함 안 됨 |

!!! warning "비용 절감 수치의 의미"
    MS가 밝힌 "입력 비용 15~21% 절감"은 같은 과제를 처리할 때의 **토큰 처리 효율**입니다. 청구 크레딧이 줄어든다는 뜻이 아니며, 사내 라이선스 사용자 대상 에이전트는 오히려 비용이 새로 생길 수 있습니다.

## 언제 옮기나

| 상황 | 권장 |
|---|---|
| 정해진 Q&A, 단순 라우팅, 라이선스 사용자 사내용 | Standard 유지 |
| 장시간·다단계 작업, 대용량 파일, 실패 복구가 중요 | Copilot harness |
| 외부 고객용 | 과제당 크레딧 실측 후 결정 |

## 비교 테스트 체크리스트

- [ ] 테스트 환경에 크레딧 할당·한도 먼저 설정
- [ ] 같은 테스트 과제 10~20개 준비
- [ ] 완료 여부, 재질문 횟수, 응답 시간 기록
- [ ] Copilot harness: 에이전트 Monitor 페이지에서 크레딧 확인
- [ ] Standard: 이벤트 단가로 환산

## 참고

- [Upgrade agents to the Microsoft Copilot harness (Microsoft Learn)](https://learn.microsoft.com/en-us/microsoft-copilot-studio/upgrade-to-microsoft-copilot-from-standard-harness)
- [Overview of usage-based billing (Microsoft Learn)](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/billing-credit-overview)
- [Billing rates and management (Microsoft Learn)](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management)
