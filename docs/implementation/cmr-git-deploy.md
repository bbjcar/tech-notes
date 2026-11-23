# CMR(Managed Apps) 배포 흐름과 Git의 역할

| 항목 | 내용 |
|---|---|
| 제품 | Copilot Managed Runtime (CMR, 구 Managed Apps) |
| 날짜 | 2026-11 |
| 기준 | Microsoft Learn 공개 문서 · 학습 정리 |

## 한 줄 요약

**Git push가 곧 배포 경로**입니다. 소스를 플랫폼 저장소에 올리면 플랫폼이 직접 빌드하고, 그 빌드 결과물(정적 파일 묶음)을 URL로 제공합니다.

## 전체 흐름

```text
로컬 개발 (JS/TS, Vite 등)
   │  git push
   ▼
앱별 저장소 (플랫폼 관리형 기본 / GitHub Enterprise Cloud 연결 가능)
   │  미리보기를 열 때 커밋 단위로 빌드
   ▼
빌드 결과물 = index.html + 해시가 붙은 JS·CSS·에셋
   ├─ Preview URL : main의 최신 성공 빌드 (저장소 쓰기 권한자만)
   └─ Live URL    : 배포 명령을 실행할 때만 바뀌는 고정 스냅샷
```

## Git이 맡는 역할

| 역할 | 설명 |
|---|---|
| 전달 경로 | 소스가 플랫폼에 도달하는 유일한 길. 플랫폼이 직접 빌드해 무엇이 배포되는지 통제 |
| 버전 | 커밋이 곧 버전. 특정 커밋 빌드 상태 확인·롤백 배포 가능 |
| 협업 | 동료가 저장소를 clone해서 같은 앱을 이어서 개발 |

## 저장소 모드 (앱 생성 시 결정, 이후 변경 불가)

| 모드 | 내용 |
|---|---|
| 플랫폼 관리형 (기본) | 플랫폼이 저장소를 만들어 관리 |
| GitHub Enterprise Cloud 연결 | 조직 저장소에서 브랜치·리뷰·릴리스 관행 그대로 사용 |
| 저장소 없음 + 외부 빌드 산출물 | GitHub Actions 등으로 빌드한 결과물 배포. **기본적으로 허용 안 됨** |

## "빌드 결과물"이란

- HTML 한 파일이 아니라 **정적 파일 묶음 전체**입니다.
- SPA라서 `index.html`은 거의 빈 껍데기이고, 화면은 브라우저에서 JS가 그립니다.
- 서버는 파일을 내려줄 뿐이고, 데이터는 각 사용자의 권한으로 브라우저에서 가져옵니다.
- 빌드가 하는 일: ① TS/JSX 변환과 import 해석(필수) ② 번들·압축(효율) ③ 검사·검증(플랫폼 통제)

## 주의할 점

- `ms.config.json`은 직접 수정하지 않습니다.
- 기본 보안 정책(CSP)이 외부 CDN·외부 호출을 막습니다. 라이브러리는 npm으로 설치해 번들에 포함합니다.
- 기본 허용 커넥터는 1st-party 일부만이며, 관리자가 범위를 넓힐 수 있습니다.
- CLI 패키지 이름이 문서와 npm에서 다르게 보이는 경우가 있으니 최신 Learn 문서로 확인합니다.

## 참고

- [Copilot Managed Runtime 개발 내부 루프 (Microsoft Learn)](https://learn.microsoft.com/en-us/microsoft-365/copilot-managed-runtime/developer/apps/dev-inner-loop)
- [Copilot Managed Runtime GA 발표](https://techcommunity.microsoft.com/blog/copilot-managed-runtime-blog/copilot-managed-runtime-is-now-generally-available-with-additional-new-preview-f/4571846)
