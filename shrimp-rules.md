# Development Guidelines (AI Agent 전용 운영 규칙)

> 이 문서는 AI Agent가 이 저장소에서 작업할 때 즉시 참조하는 운영 규칙이다. 일반 개발 지식은 담지 않는다.
> 기술 규칙의 정본은 `CLAUDE.md`(todo-project 루트, v1.8), 완료 판정 절차의 정본은 `docs/ROADMAP.md`다. 단 아래 "문서 신뢰도 경고"를 반드시 먼저 읽을 것.

---

## 0. 문서 신뢰도 원칙 (반드시 최우선으로 확인)

**2026-08-28에 `docs/ROADMAP.md`의 "스캐폴딩 정합성 점검" 표가 실제 코드와 다르게 "✅ 해소"로 잘못 기록된 항목(Next.js 버전, `src/` 이동, `AGENTS.md`, `pom.xml`의 springdoc·jsoup, `docs/guides/` 파일명 등 6건)이 발견되어 v1.9에서 정정했다.** 지금은 ROADMAP.md의 표시가 실제 상태와 일치한다(❌ 미해결 / 🟡 부분 미해결로 정확히 표시됨). 이 문서 자체를 다시 의심할 필요는 없다.

다만 이 사건이 보여주는 행동 규칙은 계속 유효하다:

- **완료 표시(✅)만 보고 다음 작업을 건너뛰지 마라.** 특히 버전 다운그레이드·디렉터리 이동·의존성 추가처럼 "완료했다고 적었지만 실제로는 하지 않기 쉬운" 종류의 작업은, 착수 전에 관련 파일(`package.json`, `pom.xml`, 실제 디렉터리 구조 등)을 직접 Read해서 확인하라.
- 실제로 어떤 항목을 처리했다면, 그 즉시 `docs/ROADMAP.md`의 해당 행을 갱신하라. 기록이 코드보다 뒤처지는 상태를 다시 만들지 마라.

### 상위 폴더 CLAUDE.md와의 충돌

`D:\my-workspace\claude\CLAUDE.md`(todo-project의 부모 폴더, Claude Code가 자동 로드함)는 이 저장소의 `todo-project/CLAUDE.md`(v1.8, SSOT)와 다음 항목에서 정면충돌한다. **충돌 시 `todo-project/CLAUDE.md`(및 `docs/`)를 따르고, 상위 폴더 문서의 해당 서술은 무시하라.**

| 항목 | 상위 `D:\my-workspace\claude\CLAUDE.md` | `todo-project/CLAUDE.md`(v1.8, 우선) |
|---|---|---|
| 저장소 구조 | 모노레포 1개 | 폴리레포 3개(독립 git) |
| 백엔드 설정 파일 형식 | `application.yml` 등 `.yml` | `.properties`(실제 파일도 `.properties`) |
| 인증 토큰 | Access(30분)+Refresh(14일, httpOnly 쿠키), 회전·재사용 탐지 | Access Only(24시간), localStorage, Refresh 없음 |
| 테스트 DB명 | `todolist_db_test` | `todolist_test` |

인증 토큰 정책 충돌은 DB 스키마(`refresh_tokens` 테이블 유무)·API 목록(`/api/v1/auth/refresh`, `/api/v1/auth/logout` 유무)에 직접 영향을 준다. `todo-project/CLAUDE.md` 6장은 Access-Only·Refresh 없음으로 명시하고 있으므로, 관련 코드는 그 설계로 작성하되, 이 문서만으로 판단이 서지 않으면 사용자에게 먼저 확인하라(추측 금지).

---

## 1. 저장소 구조

- 3개의 독립 git 저장소: `todo-project`(문서), `todo-backend`(Spring Boot), `todo-frontend`(Next.js). 하나의 커밋/브랜치 작업이 다른 저장소에 영향을 주지 않는다.
- `todo-project/.gitignore`는 `todo-backend/`, `todo-frontend/`를 반드시 포함해야 한다(gitlink 오커밋 방지). 새 최상위 폴더를 추가할 때도 동일하게 점검하라.
- 세 저장소 모두 현재 브랜치가 `main`이 아니라 `master`이고 원격이 연결되지 않은 상태다(Phase 0 미완료). `main` ← `develop` ← `feature/{작업명}` 전략은 아직 실제로 적용되지 않았으니, 브랜치를 새로 만들 때 이 전략을 기준으로 하라.

## 2. 백엔드 규칙 (`todo-backend/`)

- 패키지 루트는 `com.example.todoapp`로 고정되어 있다(`pom.xml` groupId, `TodoBackendApplication.java` 패키지 일치 확인됨). 다른 이름으로 바꾸지 마라.
- 현재 기능 코드가 전혀 없다(스캐폴딩만 존재: `TodoBackendApplication.java` 하나). 새 코드를 추가할 때는 `com.example.todoapp` 아래에 `domain`, `service`, `controller`, `dto`, `config`, `exception` 패키지로 나눠 만들되, `CLAUDE.md` 3장이 언급하는 기능별 패키지(`auth`, `user`, `todo`, `global`)와 계층별 패키지 중 어느 조합을 쓸지 착수 전 `docs/ROADMAP.md`의 해당 Phase 작업 항목을 확인하라.
- 설정 파일은 `.properties` 형식만 사용한다(`application.properties`, `application-local.properties`, `application-local.properties.example`가 이미 이 형식으로 존재). `.yml` 파일을 새로 만들지 마라.
- `pom.xml`은 Spring Boot 4.1.1 고정. 의존성 아티팩트명이 Boot 4 신규 명명(`spring-boot-starter-webmvc`, `spring-boot-starter-security-oauth2-client`, `spring-boot-starter-*-test`)을 쓰고 있다 — Boot 3 시절 이름(`spring-boot-starter-web` 등)으로 되돌리지 마라.
- `jjwt-api`/`jjwt-impl`/`jjwt-jackson` 0.12.6이 이미 pom에 있다. 버전을 바꾸지 말고, `jjwt-impl`·`jjwt-jackson`의 `runtime` scope를 유지하라. API는 0.12 문법(`Jwts.parser().verifyWith(key).build()`)만 사용하라.
- `pom.xml`에 springdoc-openapi, jsoup 의존성이 없다. 추가할 때 버전을 임의로 정하지 말고 `todo-project/CLAUDE.md` 3장 "버전 관련 확정 사항"의 규칙(springdoc은 Boot 마이너와 1:1 매칭, 범위 지정 금지)을 따르라.
- Lombok이 `optional=true`로 존재한다. 엔티티에 `@Setter`를 열지 마라. 상태 변경은 의미 있는 메서드(`complete()`, `softDelete()` 등)로 구현하라.
- 컨트롤러에서 엔티티를 직접 반환하지 마라. 항상 DTO(record)로 변환한다.
- 삭제는 전부 Soft Delete(`deleted_at` 채움). `DELETE FROM` 물리 삭제 금지.
- Todo 조회/수정/삭제 시 `user_id` 소유권 검증 필수. 불일치 시 404(존재 여부 비노출).

## 3. 프론트엔드 규칙 (`todo-frontend/`)

- **작업 전 Next.js 버전을 먼저 확인하라.** 현재 16.3.3이 설치되어 있으나 SSOT는 15.x 고정을 요구한다(Amplify Hosting SSR 지원 범위가 12~15이기 때문). 16 전용 API(`LayoutProps<...>` 전역 타입 등)를 사용하는 새 코드를 작성하지 마라. 다운그레이드가 아직 안 된 상태에서 기능 코드를 쌓지 마라 — 먼저 버전 문제를 해결해야 하는지 사용자와 확인하라.
- `src/` 디렉터리가 없다. `todo-project/CLAUDE.md` 2장 구조도는 `src/app`, `src/components`, `src/lib`, `src/types`, `src/hooks`를 전제로 한다. 새 파일을 루트 `app/`에 계속 추가하지 말고, `src/` 마이그레이션이 선행되어야 하는지 확인 후 진행하라(`tsconfig.json`의 `@/*` 경로, `components.json`의 css 경로도 함께 갱신 대상이다).
- 폼 라이브러리(`react-hook-form`, `zod`, `@hookform/resolvers`)를 설치하지 마라. 이 프로젝트의 폼은 `useState` + 수동 검증으로만 구현한다. `npx shadcn add form`을 실행하지 마라(react-hook-form이 의존성으로 딸려 들어온다).
- `docs/guides/forms-react-hook-form.md`는 다른 프로젝트에서 넘어온 미정리 참고자료이며 위 규칙과 충돌한다("이미 설치되어 있습니다"라고 적힌 react-hook-form/zod는 실제 `package.json`에 없다). 이 문서의 코드 예시를 그대로 채용하지 마라. "폼 상태 관리 패턴"이 필요하면 `useState` 기반으로 새로 작성하라.
- 애니메이션은 `motion` 패키지만 사용하고 import는 반드시 `motion/react`에서 한다(`framer-motion` 금지).
- Tiptap 관련 작업 시 툴바 버튼·Tiptap extension 설정·Jsoup allowlist(백엔드)·DOMPurify allowlist 네 곳의 허용 태그 집합을 반드시 동일하게 유지하라. 한 곳을 바꾸면 나머지 세 곳도 함께 바꿔야 한다.
- 다크 모드는 `prefers-color-scheme` 미디어쿼리 전략만 사용한다. `class` 전략이나 토글 UI를 추가하지 마라(MVP 범위 밖으로 확정됨).
- 라우트 보호에 `middleware.ts`를 만들지 마라(토큰이 localStorage에 있어 서버 미들웨어가 접근 불가). `(main)` 클라이언트 레이아웃 + `useAuth`(exp 클레임 검사)로 구현한다.
- `useSearchParams`를 쓰는 페이지(`/oauth/callback`, `/todos`)는 반드시 `<Suspense>`로 감싸라. 개발 서버는 통과하지만 `npm run build`가 실패한다.

## 4. API/데이터 규칙

- 모든 REST 응답은 `{ "success": boolean, "data": ..., "error": {code, message} | null }` 포맷을 예외 없이 따른다(OAuth2 콜백 302 리다이렉트만 예외).
- `TodoUpdateRequest`(PUT)에 `completed` 필드를 넣지 마라. 완료 상태는 오직 `PATCH /todos/{id}/toggle`로만 변경하고, 바디로 목표 상태(boolean)를 명시적으로 받아 멱등하게 구현한다(서버가 값을 반전시키는 방식 금지).
- 목록 조회 `sort` 파라미터는 `createdAt`, `dueDate`만 허용한다. 그 외 값이 들어오면 기본값(`createdAt,desc`)으로 대체하고, Pageable에 임의 문자열을 그대로 넘기지 마라(500 방지).
- 비밀번호 검증은 최소 길이(문자 수 6자 이상)와 최대 길이(UTF-8 바이트 수 72바이트 이하)를 분리해서 커스텀 validator로 검증하라. `@Size(max=64)`만으로는 한글 비밀번호에서 BCrypt가 500을 던진다.

## 5. Git/커밋 규칙

- 커밋 메시지는 한글, Conventional Commits(`feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`) 형식을 따른다.
- API 계약이 바뀌면 `todo-project`(문서 저장소)를 먼저 수정한 뒤 `todo-backend` → `todo-frontend` 순으로 반영하라.
- 승인 없이 대규모 파일 생성을 시작하지 마라. 한 Phase가 끝나면 해당 Phase의 완료 조건(DoD) 명령을 실제로 실행해 통과를 확인한 뒤 커밋하라.

## 6. 금지 사항

- 임의로 라이브러리를 추가하지 마라. 필요하면 이유와 함께 먼저 제안하고 승인을 받아라.
- Docker 관련 파일(Dockerfile, docker-compose.yml, Testcontainers)을 만들지 마라.
- Amazon S3 관련 코드·설정을 추가하지 마라(파일 업로드 기능은 MVP 범위 밖).
- `docs/ROADMAP.md`의 "✅ 해소" 표시만 보고 검증 없이 후속 작업을 건너뛰지 마라(0장 "문서 신뢰도 경고" 참조).
- 비밀/키를 코드에 하드코딩하지 마라. 환경변수 또는 `application-local.properties`(백엔드, gitignore 대상)로 분리하라.
