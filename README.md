# tech-notes

프로젝트를 하면서 겪은 **장애 해결**과 **기술 구현** 기록을 모으는 문서 사이트입니다.
[MkDocs](https://www.mkdocs.org/) + Material 테마로 만들고, GitHub Actions가 자동으로 빌드해 GitHub Pages에 게시합니다.

- 사이트: https://bbjcar.github.io/tech-notes/
- 정보 공유 대시보드: https://bbjcar.github.io/tech-dashboard/

## 글 쓰는 방법 (요약)

1. `docs/troubleshooting/`, `docs/implementation/`, `docs/concepts/` 중 알맞은 폴더에 `.md` 파일을 만듭니다. (`docs/templates/`의 템플릿 복사)
2. `mkdocs.yml`의 `nav`에 새 파일을 한 줄 추가합니다.
3. 커밋하면 약 1~2분 뒤 사이트에 반영됩니다. (Actions 탭에서 진행 상황 확인)

자세한 내용은 [작성 가이드](docs/templates/guide.md)를 참고하세요. 작성 가이드와 템플릿은 `docs/templates/`에 있어 공개 사이트에는 게시되지 않습니다.

> ⚠️ 공개 사이트입니다. 고객사명, 서버 주소, 계정·키, 실제 오류 로그 원문, 내부 단가는 올리지 않습니다.

## 내 PC에서 미리보기 (선택)

```bash
pip install -r requirements.txt
mkdocs serve
```

브라우저에서 http://127.0.0.1:8000 을 엽니다.
