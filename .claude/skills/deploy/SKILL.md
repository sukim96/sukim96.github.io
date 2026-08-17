---
name: deploy
description: sukim96.github.io 배포 방식 — main/dev 브랜치 구조, GitHub Actions 기반 Pages 배포, 배포 확인 방법. 사이트 수정·배포·브랜치 작업 전에 참조.
---

# sukim96.github.io 배포 방식

## 구조

- 저장소: `sukim96/sukim96.github.io` (GitHub Pages user site)
- **main 브랜치** → https://sukim96.github.io (실제 사이트, 루트)
- **dev 브랜치** → https://sukim96.github.io/dev/ (개발용 미리보기)
- 개발은 dev 브랜치에서 하고, 확정되면 main에 머지한다.

## 배포 원리

GitHub Pages는 저장소당 배포 소스가 하나라서 브랜치를 경로에 직접 연결할 수 없다.
그래서 Pages 빌드 방식을 **GitHub Actions(workflow)** 로 전환했고(레거시 branch 빌드 아님),
`.github/workflows/deploy.yml`이 배포를 담당한다:

1. main 또는 dev에 푸시되면 트리거 (workflow_dispatch 수동 실행도 가능)
2. main을 `site/`에, dev를 `site/dev/`에 체크아웃
3. `.git`, `.github`, `.claude` 등 내부 파일 제거 후 합쳐서 Pages에 배포

즉 **어느 브랜치에 푸시하든 항상 두 브랜치 최신 내용이 함께 배포**된다.

## 일상 작업 흐름

```bash
# 개발: dev에서 수정 → /dev 에 반영
git checkout dev
git commit -am "..." && git push

# 릴리스: main에 머지 → 실제 사이트에 반영
git checkout main && git merge dev && git push
```

## 배포 확인

```bash
gh run watch $(gh run list --workflow=deploy.yml --limit 1 --json databaseId --jq '.[0].databaseId')
curl -s -o /dev/null -w "%{http_code}" https://sukim96.github.io/dev/
```

배포 후 반영까지 수십 초 걸릴 수 있고, CDN 캐시로 이전 내용이 잠시 보일 수 있다.

## 주의사항

- Pages build_type을 다시 legacy로 바꾸면 안 됨 — /dev 라우트가 사라진다.
- 워크플로 파일은 main과 dev 양쪽에 존재해야 dev 푸시에서도 트리거된다.
  워크플로를 수정하면 main에 커밋 후 dev에도 머지할 것.
- dev 페이지는 `/dev/` 경로 기준으로 서빙되므로, 페이지 간 링크는 절대 경로(`/...`) 대신
  상대 경로를 쓸 것 (main과 dev에서 동일하게 동작하게).
- gh CLI가 sukim96 계정으로 로그인되어 있고 git은 SSH 프로토콜을 쓴다.
