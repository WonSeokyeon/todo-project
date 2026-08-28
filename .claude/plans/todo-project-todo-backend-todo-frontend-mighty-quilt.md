# 3개 저장소 초기/현재 상태 커밋

## Context

`todo-project`는 폴리레포 구조로, `todo-project`(문서) · `todo-backend`(Spring Boot) · `todo-frontend`(Next.js)가 각각 독립된 git 저장소다. 세 저장소 모두 지금까지 작업한 파일이 커밋되지 않은 상태(untracked / unstaged)이므로, 저장소별로 현재 상태를 커밋해 작업 이력을 남긴다. `.gitignore` 확인 결과 세 저장소 모두 비밀값/빌드 산출물이 적절히 제외되고 있어 그대로 커밋해도 안전하다.

## 저장소별 커밋 계획

### 1) todo-project (문서 저장소) — 최초 커밋
대상: `.claude/`(agents, commands, plans, settings.json — `settings.local.json`은 gitignore됨), `.editorconfig`, `.gitignore`, `.mcp.json`, `CLAUDE.md`, `docs/`(PRD.md, ROADMAP.md, DEV_TOOLS.md, guides/)

```
git add .claude .editorconfig .gitignore .mcp.json CLAUDE.md docs
git commit -m "docs: 프로젝트 문서 및 개발 환경 설정 초기 커밋"
```

### 2) todo-backend — 최초 커밋
대상: `src/`(TodoBackendApplication.java 등 스캐폴딩), `pom.xml`, `mvnw`/`mvnw.cmd`, `.mvn/`, `README.md`, `.editorconfig`, `.gitattributes`, `.githooks/`, `.gitignore`
(※ `application-local.properties`는 `.gitignore`에 의해 자동 제외됨 — 확인 완료)

```
git add .
git commit -m "chore: Spring Boot 프로젝트 초기 스캐폴딩"
```

### 3) todo-frontend — 개발 도구 및 UI 초기 설정 커밋
이미 "Initial commit from Create Next App" 커밋이 존재하는 상태에서, 이후 추가된 변경분을 하나로 커밋한다.
대상: 수정된 `README.md`/`app/*`/`eslint.config.mjs`/`next.config.ts`/`package.json`/`package-lock.json` + 신규 `.editorconfig`, `.gitattributes`, `.husky/`, `.lintstagedrc.mjs`, `.nvmrc`, `.prettierignore`, `.prettierrc.json`, `.vscode/`, `commitlint.config.mjs`, `components.json`, `components/`(shadcn `ui/button.tsx`), `lib/utils.ts`

```
git add .
git commit -m "chore: 코드 품질 도구 및 shadcn/ui 초기 설정 추가"
```

## 확인 사항 (커밋과 별개로 사용자에게 보고)

- **`todo-frontend/package.json`에 `"next": "16.3.3"`으로 설치되어 있음.** `todo-project/CLAUDE.md`의 확정 사항은 "Next.js는 15를 쓴다, 16을 쓰지 않는다(Amplify SSR 미지원)"이므로 스펙과 실제 설치 버전이 어긋난다. 커밋 자체는 현재 상태를 그대로 남기는 것이므로 진행하되, 다운그레이드 여부는 사용자 확인이 필요하다.
- `docs/guides/`에 `nextjs-16.md`, `forms-react-hook-form.md`가 있는데, `CLAUDE.md`는 Next.js 15 및 폼 라이브러리 미사용을 확정했다. `guides/`는 참고자료로 스펙에 우선하지 않는다고 문서에 명시되어 있어 커밋에는 영향 없음(참고로만 보고).
- `todo-backend`의 실제 패키지는 `com.example`이며, 상위(`claude/CLAUDE.md`)는 `com.example.todoapp`을 요구하지만 `todo-project/CLAUDE.md`(SSOT 선언)는 `com.example`로 되어 있다. 커밋에는 영향 없으나 향후 패키지 구조 작업 시 참고.

## 검증

각 커밋 후 `git status`, `git log --oneline -3`으로 커밋이 반영되고 워킹트리가 깨끗한지 확인한다.
