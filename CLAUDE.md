# Todo List 프로젝트 개발 가이드

> **버전** 1.13 · **최종 수정** 2026-09-04
> **v1.13 변경**: **본문 이미지 첨부를 MVP 범위로 편입**했다(`PRD.md` v1.11의 `TODO-17`·`TODO-18`). 3장의 "S3는 MVP 범위에서 제외한다"를 정정하고, 4장에 `attachments` 테이블, 5장에 첨부 API 6종, 6장 XSS 절에 `img` 허용 규칙, 8장 툴바에 이미지 버튼, 11장에 `ATTACHMENT_*` 에러 코드를 추가했다. **저장소는 이번엔 로컬 디렉토리이고 S3 전환은 Phase 13**(AWS 배포 이후)이다 — 인터페이스만 지금 확정한다. 이번 건은 원칙대로 **문서를 먼저 고치고 구현에 들어간다**(6장). 상세 지시서는 `docs/tiptap-image-upload-prompt.md`, 원본 프롬프트 분석 근거는 `docs/appendFileImage.md`.
> **v1.12 변경**: 7장 화면 목록과 6장 XSS 방어 절의 전제였던 "상세 화면은 보기/편집 모드를 나누지 않는다"(`/todos/[id]` 진입 즉시 편집)를 뒤집었다. 목록에서 항목을 선택하면 이제 읽기 전용 확인 화면(`/todos/[id]`)으로 가고, 수정은 별도 `/todos/[id]/edit`로 분리했으며 목록에도 전용 "수정" 버튼을 뒀다. **이번 건은 예외적으로 코드가 먼저 바뀌고 문서가 뒤따라간 경우다**(사용자가 실사용 중 "목록에서 누르면 곧장 수정 화면이 열린다"를 버그로 보고했고, `todo-frontend`에 이미 반영·커밋됨) — 원칙(문서 먼저, CLAUDE.md 6장)의 예외이지 새 원칙은 아니다. 읽기 전용 화면에 `dangerouslySetInnerHTML` 호출 지점이 처음 생겨 6장의 "호출 지점이 없다" 서술도 함께 정정했다. `docs/PRD.md`의 `TODO-09`·5.6절도 같은 이유로 함께 고쳤다. `docs/ROADMAP.md`의 Phase 8 완료 기록(과거 실측 로그)은 당시 사실이었으므로 소급 수정하지 않는다.
> **v1.11 변경**: 9장 「`useAuth`는 `exp`를 봐야 한다」 절에 v1.9 이전 서술이 남아 있었다 — "`JWT_EXPIRATION`이 24시간이라 개발 중에는 만료를 만나기도 어렵다"는 근거가 두 군데 틀렸다. (1) v1.9에서 Access Token 만료를 **30분**으로 확정했으므로 24시간이 아니고, (2) `JWT_EXPIRATION`이라는 환경변수는 존재하지 않는다 — 실제 값은 `JwtTokenProvider.ACCESS_TOKEN_EXPIRATION`(`Duration.ofMinutes(30)`) 코드 상수다(`todo-backend` 실제 코드로 확인). 결론(`exp`를 디코드해야 한다)은 그대로 유효하며, 근거 문장만 사실에 맞게 고쳤다. 코드 변경은 없다.
> **v1.10 변경**: 2장 저장소 구조도와 4장 UTC 타임존 설정 지시문에 남아있던 `application.yml` 계열 언급을 `application.properties` 계열로 정정했다. 부모 CLAUDE.md 절대규칙 9와 실제 `todo-backend`에 이미 존재하는 파일 형식이 `.properties`였음을 근거로 `.properties` 유지가 확정됐다(설정 파일 형식은 애초에 바뀐 적이 없고, 이 문서의 서술이 stale했던 것).
> **v1.9 변경**: 6장 인증 설계를 Access(30분)+Refresh(14일, httpOnly 쿠키) 2-토큰 구조로 정정(기존 서술은 `docs/PRD.md`가 이미 전제하던 설계와 반대였음). 4장에 `refresh_tokens` 테이블, 5장에 `/auth/refresh`·`/auth/logout` API를 추가했다. 코드는 아직 없으므로 문서만 수정.
> 이 문서는 **기술 규칙의 단일 기준(Single Source of Truth)**이다.
> 코드 생성 전 반드시 이 문서를 확인하고, 문서와 충돌하는 구현을 하지 않는다.
> 문서에 없는 결정이 필요하면 임의로 진행하지 말고 먼저 질문한다.
>
> 관련 문서: `docs/PRD.md`(무엇을 만드는가) · `docs/ROADMAP.md`(어떤 순서로 만드는가 + **완료 판정의 정본**)
> **이 문서만 루트에 두고, 나머지 문서는 `docs/` 아래에 둔다.** Claude Code가 상위 디렉토리를 거슬러 올라가며 자동 로드하는 대상이 `CLAUDE.md`이기 때문이다.

---

## 1. 프로젝트 개요

- **주제**: 개인용 Todo List 풀스택 웹 서비스
- **핵심 가치**: 로그인한 사용자가 자신의 할 일을 리치 텍스트로 작성하고, 완료/삭제를 즉각적인 반응으로 관리한다.
- **범위**: 로컬 개발 완료 후 AWS 배포. **Docker는 사용하지 않는다.**

---

## 2. 저장소 구조 (폴리레포)

**3개의 독립된 Git 저장소**로 관리한다. 모노레포가 아니다.

```
todo-project/                    # [저장소 1] 문서 저장소
├── .git/
├── .gitignore                   # todo-backend/, todo-frontend/ 제외
├── CLAUDE.md                    # 기술 규칙 (이 문서). 루트에 둔다
├── docs/                        # 나머지 문서는 전부 이 아래
│   ├── PRD.md                   # 제품 요구사항
│   ├── ROADMAP.md               # 개발 로드맵 · 완료 판정
│   └── guides/                  # 참고 자료. 스펙 아님 — 충돌 시 이 문서가 우선
│
├── todo-backend/                # [저장소 2] 독립 저장소
│   ├── .git/
│   ├── CLAUDE.md                # 백엔드 전용 규칙
│   ├── pom.xml
│   ├── mvnw
│   └── src/
│       ├── main/java/com/example/
│       │   ├── domain/          # 엔티티, Repository
│       │   ├── service/         # 비즈니스 로직, HtmlSanitizer
│       │   ├── controller/      # REST API
│       │   ├── dto/             # 요청/응답 DTO
│       │   ├── config/          # Security, JWT, Swagger, CORS 설정
│       │   └── exception/       # 예외 처리
│       ├── main/resources/
│       │   ├── application.properties
│       │   ├── application-local.properties
│       │   └── application-prod.properties
│       └── test/
│           ├── java/com/example/
│           └── resources/application-test.properties
│
└── todo-frontend/               # [저장소 3] 독립 저장소
    ├── .git/
    ├── CLAUDE.md                # 프론트엔드 전용 규칙
    ├── package.json
    ├── public/                  # 정적 파일. public/static 은 만들지 않을 것
    └── src/
        ├── app/
        │   ├── (auth)/
        │   │   ├── login/
        │   │   └── signup/
        │   ├── oauth/callback/
        │   └── (main)/todos/
        │       ├── page.tsx          # 목록
        │       ├── new/page.tsx      # 생성
        │       └── [id]/
        │           ├── page.tsx      # 확인(읽기 전용)
        │           └── edit/page.tsx # 수정
        ├── components/
        │   ├── ui/              # shadcn/ui
        │   ├── common/          # Pagination, EmptyState, ErrorState, Skeleton
        │   └── todo/            # TodoList, TodoItem, TodoForm, TodoEditor
        ├── hooks/               # useTodos, useAuth
        ├── lib/                 # apiClient, queryClient, sanitize, utils
        └── types/
```

### ⚠️ 폴리레포 필수 설정

부모 폴더가 Git 저장소이면서 하위 폴더도 Git 저장소이므로, **부모 저장소가 하위 폴더를 추적하지 않도록 반드시 제외해야 한다.** 이 설정을 빠뜨리면 Git이 하위 폴더를 gitlink로 커밋해 버리고, 클론했을 때 빈 폴더만 남는다.

`todo-project/.gitignore`:

```gitignore
todo-backend/
todo-frontend/

.DS_Store
*.log
```

### Git 전략

| 저장소        | 원격 이름(예시) | 담는 것                       |
| ------------- | --------------- | ----------------------------- |
| todo-project  | `todo-docs`     | CLAUDE.md, PRD.md, ROADMAP.md |
| todo-backend  | `todo-backend`  | Spring Boot 애플리케이션      |
| todo-frontend | `todo-frontend` | Next.js 애플리케이션          |

- 브랜치: `main`(배포 가능 상태) ← `develop` ← `feature/{작업명}`
- 커밋 메시지: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`
- **API 계약이 바뀌면 문서 저장소를 먼저 수정**한 뒤 백엔드 → 프론트엔드 순으로 반영한다.
- **태그 규칙**: Phase 완료 시 해당 저장소에 `v0.{Phase번호}.0`. Phase 10 전체 검증 통과 시 세 저장소 모두 `v1.0.0`.

### 각 저장소의 CLAUDE.md

Claude Code는 현재 디렉토리에서 **상위 디렉토리로 거슬러 올라가며 CLAUDE.md를 찾아 로드**한다. 따라서 `todo-backend/`에서 실행해도 부모의 이 문서가 함께 읽힌다.

다만 저장소를 단독으로 클론하면 부모 문서가 없으므로, 각 하위 저장소에도 자체 `CLAUDE.md`를 두고 **해당 저장소에만 해당하는 규칙**(빌드 명령, 계층 규칙, 컨벤션)을 적는다. 전체 스펙은 이 문서를 정본으로 삼는다.

---

## 3. 기술 스택

### Backend

| 항목           | 선택                                                                              |
| -------------- | --------------------------------------------------------------------------------- |
| 프레임워크     | Spring Boot 4.x                                                                   |
| JDK            | 21                                                                                |
| 빌드           | Maven (mvnw 래퍼)                                                                 |
| ORM            | Spring Data JPA / Hibernate                                                       |
| 보안           | Spring Security + JWT                                                             |
| JWT 라이브러리 | **jjwt 0.12.6** — `jjwt-api` + `jjwt-impl`(runtime) + `jjwt-jackson`(runtime) 3종 |
| HTML 정화      | **Jsoup**                                                                         |
| API 문서       | SpringDoc OpenAPI (Swagger UI)                                                    |
| DB             | PostgreSQL                                                                        |
| 패키지명       | `com.example.todoapp`                                                             |

### Frontend

| 항목       | 선택                                                                 |
| ---------- | -------------------------------------------------------------------- |
| 프레임워크 | **Next.js 15** (App Router)                                          |
| Node.js    | **20 이상** (권장 22)                                                |
| 라이브러리 | React 19, TypeScript                                                 |
| 스타일     | Tailwind CSS 4                                                       |
| UI         | shadcn/ui, lucide-react                                              |
| 서버 상태  | React Query (TanStack Query v5)                                      |
| 폼         | **라이브러리를 쓰지 않는다** — `useState` + 수동 검증 (아래 ⚠️ 참조) |
| 애니메이션 | **Motion** (`motion` 패키지, 구 framer-motion)                       |
| 에디터     | Tiptap (`@tiptap/react` + `@tiptap/starter-kit` + **`@tiptap/extension-image`**) |
| 토스트     | **shadcn/ui `sonner`**                                               |
| 날짜       | **date-fns** (shadcn Calendar 의존)                                  |
| HTML 정화  | **DOMPurify**                                                        |

### 인프라

AWS Amplify(프론트), EC2(백엔드), RDS(PostgreSQL) · Git/GitHub

> **S3는 Phase 13까지 사용하지 않는다.** (v1.13 정정 — 이전에는 "MVP 범위에서 제외"였다)
> 본문 이미지 첨부가 범위에 들어오면서 파일 저장소가 필요해졌으나, **이번 구현의 저장소는 로컬 디렉토리(`todo-project/upload/`)**다. Phase 11(AWS 배포)이 끝난 뒤 Phase 13에서 S3로 전환한다.
> 백엔드는 `StorageService` 인터페이스에만 의존하고 구현체는 `app.storage.type`으로 갈아끼운다. **프론트엔드 코드는 로컬·S3에서 동일해야 한다.** 프론트 정적 자산은 그대로 Amplify가 처리한다.

### ⚠️ 버전 관련 확정 사항

아래는 조사로 확인이 끝난 사항이다. **다시 확인하거나 다른 버전으로 바꾸지 않는다.**

- **Next.js는 15를 쓴다. 16을 쓰지 않는다.** AWS Amplify Hosting compute의 SSR 지원 범위가 Next.js 12~15이기 때문이다. 16으로 올리면 배포가 불가능해진다. App Router·React 19·Tailwind 4는 모두 15에서 정상 동작하므로 이 프로젝트가 잃는 기능은 없다.
- **Node.js는 20 이상을 쓴다.** Amplify Hosting은 Node 14·16·18 런타임 지원을 종료했고 20·22·24만 지원한다. 로컬 개발 Node 버전도 여기에 맞춘다.
- **SpringDoc OpenAPI는 3.x를 쓴다.** `org.springdoc:springdoc-openapi-starter-webmvc-ui` 버전 3.x가 Spring Boot 4.x 대응이다. **2.8.x는 Spring Boot 3.x 전용이므로 쓰면 기동에 실패한다.**
  > ⚠️ **"3.x"로 두지 말고 정확한 버전을 `pom.xml`에 핀한다.** SpringDoc 3.x 안에서도 Boot 마이너 버전과 1:1로 대응한다(3.0.0→Boot 4.0.0, 3.0.3→4.0.5, 3.1.0→4.1.0). 범위로 두면 Boot 4.0.x에 3.1.0이 딸려 들어와 관리 버전이 어긋난다. **현재 `pom.xml`의 Boot 버전을 확인하고 대응하는 SpringDoc 버전을 명시적으로 적는다.**
- **JWT 라이브러리는 jjwt 0.12.6으로 고정한다.** `pom.xml`에 세 아티팩트가 모두 필요하며, **`jjwt-impl`과 `jjwt-jackson`은 `<scope>runtime</scope>`이 의도된 설정이다.** 컴파일 시점에는 `jjwt-api`만 참조하고 구현체는 실행 시점에 주입되는 구조이므로, "왜 3개나 있지" 하고 정리하면 기동 시 `ClassNotFoundException`이 난다.
  > ⚠️ **0.11.x → 0.12.x에서 API가 바뀌었다.** 인터넷 예제 다수가 구버전 문법(`Jwts.parser().setSigningKey(...)`, `SignatureAlgorithm.HS256`)이라 그대로 옮기면 컴파일에 실패한다. 0.12에서는 `Jwts.parser().verifyWith(key).build()` · `Jwts.builder().signWith(key)` 형태다. 버전을 올리거나 내리지 않는다.
- **애니메이션 패키지는 `motion`이다.** `framer-motion`은 이름이 바뀌기 전의 deprecated 별칭이다. `npm install motion`으로 설치하고 **import는 반드시 `motion/react`에서 한다.** API는 동일하다.
- **shadcn/ui는 React 19 + Tailwind 4를 정식 지원한다.** 단 npm으로 설치할 때 peer dependency 충돌이 나므로 **`--legacy-peer-deps` 플래그를 쓴다.** (pnpm·yarn·bun은 플래그 불필요) 또한 toast 컴포넌트는 deprecated이므로 **sonner**를 쓰고, 신규 프로젝트 스타일은 **new-york**을 쓴다.
- **폼 라이브러리를 도입하지 않는다.** `react-hook-form`·`zod`·`@hookform/resolvers`를 설치하지 않는다. 이 앱의 폼은 셋뿐이고(`/login` 2필드, `/signup` 3필드, `TodoForm` 4필드) 검증 규칙도 4장 제약 표로 고정되어 있어, `useState` + 수동 검증으로 충분하다. 스택을 늘리지 않는 편이 MVP에 맞다.
  > ⚠️ **shadcn/ui의 `form` 컴포넌트를 추가하지 않는다.** 이 컴포넌트만 `react-hook-form` 위에 만들어져 있어, `npx shadcn add form`을 실행하면 `react-hook-form`과 `@hookform/resolvers`가 **의존성으로 함께 설치된다.** 다른 shadcn 컴포넌트(`input`, `label`, `button`, `select`, `checkbox`, `calendar` 등)는 영향이 없으므로 그대로 쓴다. 폼 마크업은 `label` + `input`을 직접 조합한다.
  > ⚠️ **대신 `dirty` 판정을 직접 구현해야 한다.** `TODO-16`(이탈 확인)이 이를 요구하는데, 라이브러리의 `formState.isDirty`가 없으므로 초기값과 현재값을 직접 비교한다. **Tiptap 본문이 특히 까다롭다** — 에디터가 HTML을 정규화하므로 사용자가 아무것도 고치지 않아도 서버가 준 HTML 문자열과 `editor.getHTML()` 결과가 달라질 수 있다. 그대로 비교하면 저장하고 나가는데도 확인창이 뜬다. **초기 스냅샷은 `setContent()` 직후의 `editor.getHTML()`로 잡는다**(정규화를 거친 값끼리 비교). 9장 참조.
- **Tailwind CSS 4는 CSS-first 설정**이다. `tailwind.config.js` 대신 `globals.css`에서 `@import "tailwindcss";` + `@theme { ... }`로 토큰을 정의한다. v3 방식으로 작성하지 않는다.
- Spring Security는 `SecurityFilterChain` 빈(람다 DSL)으로만 설정한다. `WebSecurityConfigurerAdapter`는 사용하지 않는다.

### ⚠️ Amplify 배포 제약 (프론트 개발 시 반드시 지킬 것)

- **`public/static` 경로를 만들지 않는다.** Amplify가 배포용으로 예약한 경로다. 정적 파일은 `public/` 바로 아래나 `public/assets/`에 둔다.
- **한 앱에서 SSR 브랜치와 SSG 브랜치를 섞어 배포할 수 없다.** `main`과 `develop` 모두 동일한 렌더링 방식이어야 한다. 이 프로젝트는 전부 SSR/CSR 혼합(`next build`)으로 통일한다.
- 빌드 출력 디렉토리는 `.next`여야 한다. `next.config.js`에 `distDir`을 설정하지 않는다.
- `next/image` 사용 시 이미지 응답 크기 제한이 있다. 이 프로젝트는 이미지 업로드가 비목표이므로 해당 없음.

---

## 4. 데이터 모델

DB 스키마명: **`todolist_db`** (소문자) · 테스트: **`todolist_test`**

### users

| 컬럼        | 타입         | 제약                         |
| ----------- | ------------ | ---------------------------- |
| id          | BIGSERIAL    | PK                           |
| email       | VARCHAR(255) | UNIQUE, NOT NULL (로그인 ID) |
| password    | VARCHAR(255) | NULL 허용 (소셜 전용 계정)   |
| nickname    | VARCHAR(50)  | NOT NULL                     |
| provider    | VARCHAR(20)  | LOCAL / GOOGLE               |
| provider_id | VARCHAR(255) | 소셜 고유 ID                 |
| created_at  | TIMESTAMP    | NOT NULL                     |
| updated_at  | TIMESTAMP    | NOT NULL                     |
| deleted_at  | TIMESTAMP    | NULL                         |

> **`users.deleted_at`은 이번 범위에서 항상 NULL이다.** 회원 탈퇴가 비목표이므로 값을 채우는 경로가 없다. 다만 스키마와 조회 조건은 유지해, 향후 탈퇴 기능 추가 시 구조를 바꾸지 않는다. 로그인·인증 시 `deleted_at IS NULL` 검사는 걸어둔다.

### todos

| 컬럼       | 타입         | 제약                                          |
| ---------- | ------------ | --------------------------------------------- |
| id         | BIGSERIAL    | PK                                            |
| user_id    | BIGINT       | FK → users.id, NOT NULL                       |
| title      | VARCHAR(200) | NOT NULL                                      |
| content    | TEXT         | Tiptap HTML, 정화 후 저장                     |
| completed  | BOOLEAN      | NOT NULL, DEFAULT false                       |
| priority   | VARCHAR(10)  | NOT NULL, HIGH / MEDIUM / LOW, DEFAULT MEDIUM |
| due_date   | DATE         | NULL 허용                                     |
| created_at | TIMESTAMP    | NOT NULL                                      |
| updated_at | TIMESTAMP    | NOT NULL                                      |
| deleted_at | TIMESTAMP    | NULL (Soft Delete)                            |

**인덱스**: `idx_todos_user_deleted` on `(user_id, deleted_at)`

### refresh_tokens

> **v1.9 추가.** 6장 인증 설계를 Access(30분)+Refresh(14일) 2-토큰 구조로 확정하면서 신설했다. 자세한 배경은 6장 참조.

| 컬럼       | 타입         | 제약                                          |
| ---------- | ------------ | --------------------------------------------- |
| id         | BIGSERIAL    | PK                                            |
| user_id    | BIGINT       | FK → users.id, NOT NULL                       |
| token_hash | VARCHAR(255) | NOT NULL, UNIQUE (SHA-256 해시, 평문 저장 금지) |
| expires_at | TIMESTAMP    | NOT NULL (발급 + 14일)                        |
| revoked_at | TIMESTAMP    | NULL (회전·로그아웃·탈취 대응 시 채움)        |
| created_at | TIMESTAMP    | NOT NULL                                      |

**인덱스**: `idx_refresh_tokens_user` on `(user_id)`, `idx_refresh_tokens_token_hash` on `(token_hash)`

- 평문 Refresh Token은 DB에 저장하지 않는다. 발급 시 SHA-256 해시만 저장하고, 검증 시 들어온 토큰을 해시해 비교한다.
- 사용(회전) 시 기존 행을 `revoked_at` 채워 폐기하고 새 행을 만든다. 물리 삭제하지 않는다(탈취 감사 로그로 남긴다).

### attachments

> **v1.13 추가.** 본문 이미지 첨부(`PRD.md` `TODO-17`)를 위해 신설했다.

| 컬럼                | 타입         | 제약                                        |
| ------------------- | ------------ | ------------------------------------------- |
| id                  | BIGSERIAL    | PK                                          |
| user_id             | BIGINT       | FK → users.id, NOT NULL (업로더)            |
| todo_id             | BIGINT       | FK → todos.id, **NULL 허용**                |
| storage_type        | VARCHAR(20)  | NOT NULL, LOCAL / S3                        |
| storage_key         | VARCHAR(512) | NOT NULL, UNIQUE                            |
| original_filename   | VARCHAR(255) | NOT NULL                                    |
| content_type        | VARCHAR(100) | NOT NULL                                    |
| file_size           | BIGINT       | NOT NULL (확정 시 실측값으로 갱신)          |
| status              | VARCHAR(20)  | NOT NULL, TEMP / LINKED                     |
| created_at          | TIMESTAMP    | NOT NULL                                    |
| updated_at          | TIMESTAMP    | NOT NULL                                    |
| deleted_at          | TIMESTAMP    | NULL (Soft Delete)                          |

**인덱스**: `idx_attachments_todo` on `(todo_id)`, `idx_attachments_status_created` on `(status, created_at)`, `idx_attachments_user` on `(user_id)`

**저장 키 규칙**: `todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}` (로컬·S3 동일)

- **`todo_id`가 NULL 허용인 이유**: 에디터에서는 할 일을 저장하기 *전에* 이미지가 먼저 업로드된다. 업로드 시점에는 `status=TEMP`, `todo_id=NULL`이고, 할 일 저장 시 본문에 실제로 남아 있는 첨부만 `LINKED`로 전환하며 `todo_id`를 채운다.
- **`storage_type`은 기록용이며 런타임 분기를 하지 않는다.** `@ConditionalOnProperty`가 구현체를 하나만 등록하므로 다른 타입 행을 처리할 빈이 애초에 없다. 감사·향후 마이그레이션 기록 용도이며, 현재 활성 타입과 다른 행에 접근하면 조용히 실패하지 말고 **명시적 예외**를 던진다.
- **본문에서 사라진 첨부는 `deleted_at`만 채우고 파일은 즉시 지우지 않는다.** 즉시 지우면 향후 휴지통(`PRD.md` 9장 #3) 복원이 구조적으로 불가능해진다. 물리 파일 삭제는 배치가 유예 기간 뒤에 수행한다.
  - `status=TEMP` + 생성 후 24시간 경과 → 파일·행 삭제 (업로드하다 만 고아 파일)
  - `deleted_at` 30일 경과 → **파일만 삭제, 행은 유지**

### 입력값 제약 (DTO 검증과 스키마를 일치시킬 것)

| 필드       | 제약                                               | 이유                                              |
| ---------- | -------------------------------------------------- | ------------------------------------------------- |
| `email`    | 형식 검증, 최대 255자                              |                                                   |
| `password` | **6자 이상, 그리고 UTF-8 인코딩 시 72바이트 이하** | 아래 ⚠️ 참조                                      |
| `nickname` | 1~50자                                             |                                                   |
| `title`    | 1~200자, 필수 (`@NotBlank @Size(max=200)`)         | 스키마 VARCHAR(200)과 일치                        |
| `content`  | **최대 50,000자**                                  | TEXT는 무제한이라 대용량 붙여넣기로 요청이 터진다 |
| `priority` | enum 값만 허용                                     |                                                   |
| `dueDate`  | `yyyy-MM-dd`, 선택                                 |                                                   |

#### ⚠️ 비밀번호 상한은 "문자 수"가 아니라 "바이트"다 (중요)

**`@Size(max=64)`만 걸면 한글 비밀번호에서 500이 난다.**

BCrypt의 한계는 **72바이트**다. UTF-8에서 한글 1자는 3바이트이므로 **한글 25자 = 75바이트**로 이미 한계를 넘는다. 그런데 `@Size(max=64)`는 문자 수를 세므로 이 입력을 **통과시킨다.**

그리고 최신 Spring Security의 `BCryptPasswordEncoder`는 72바이트 초과분을 **조용히 버리지 않는다.** `IllegalArgumentException("password cannot be more than 72 bytes")`를 **던진다**(CVE-2025-22228 대응). 즉 검증을 통과한 요청이 인코딩 단계에서 터지고, `GlobalExceptionHandler`에 매핑이 없으면 **500 `INTERNAL_ERROR`**로 나간다. 한국어 사용자 대상 서비스에서 한글 비밀번호는 충분히 현실적인 입력이다.

따라서 **커스텀 validator로 바이트 길이를 검증**한다.

```java
// 최소 길이는 문자 수, 최대 길이는 바이트로 검증한다
@Size(min = 6, message = "비밀번호는 6자 이상이어야 합니다.")
@MaxByteLength(value = 72, message = "비밀번호가 너무 깁니다. (한글은 1자가 3바이트로 계산됩니다)")
private String password;
```

- 위반 시 **400 `INVALID_INPUT`** + 필드 메시지로 응답한다. 500이 나가면 안 된다.
- 안전장치로 `GlobalExceptionHandler`에 `IllegalArgumentException` → 400 매핑도 함께 걸어둔다.

### 공통 규칙

- 모든 엔티티는 `BaseEntity`를 상속해 `created_at`, `updated_at`을 자동 관리한다 (`@EnableJpaAuditing`).
- **Soft Delete**: 물리 삭제 금지. `deleted_at`에 현재 시각을 기록하고, 모든 조회에 `deleted_at IS NULL` 조건을 포함한다.
- 로컬 개발은 `ddl-auto: update`. **운영 전환 절차는 아래 참조.**

### ⚠️ 타임존은 UTC로 고정한다 (중요)

로컬 개발 환경은 KST(+09:00)이고 RDS 기본 타임존은 UTC다. 아무 조치 없이 `LocalDateTime`을 쓰면 **로컬에서 만든 데이터와 운영 데이터의 시각이 9시간 어긋나고**, 배포 후에야 드러난다.

- **모든 타임스탬프는 애플리케이션 코드에서 직접 UTC로 계산한다** — JPA Auditing(`created_at`/`updated_at`)은 `DateTimeProvider` 빈이 `LocalDateTime.now(ZoneOffset.UTC)`를 반환하게 하고(`@EnableJpaAuditing`과 함께 메인 애플리케이션 클래스에 둔다), 엔티티가 직접 시각을 만드는 곳(`softDelete()`, Refresh Token 발급/폐기 등)도 전부 `LocalDateTime.now(ZoneOffset.UTC)`를 명시한다.
- **`application.properties`에 `spring.jpa.properties.hibernate.jdbc.time_zone=UTC`를 설정하지 않는다.** ⚠️ 예전에는 이 설정을 두라고 했으나 **틀렸다** — JVM 기본 타임존이 KST인 로컬 환경에서 이 값을 주면 Hibernate가 `LocalDateTime` 바인딩에도 Calendar 기반 구식 경로를 타면서, 위에서 이미 UTC로 계산해 둔 값을 "JVM 기본 타임존(KST)의 로컬 시각"으로 다시 해석해 UTC로 한 번 더 변환해버린다 — 결과적으로 실제 저장값이 **9시간 뒤로 밀린다**(2026-09-02 실측 발견 및 수정). Repository 테스트가 `saveAndFlush` 직후의 **메모리 상 엔티티 값**만 검증하고 DB에 실제로 쓰인 값을 재조회하지 않으면 이 왜곡을 못 잡으므로, UTC 저장을 검증하는 테스트는 반드시 JDBC로 컬럼 원본 값을 직접 재조회해서 대조한다(`todo-backend`의 `UserRepositoryTest` 참조).
- 저장·비교는 전부 UTC로 한다. 표시할 때만 브라우저 로컬 시각으로 변환한다(프론트 `date-fns`).
- `due_date`는 `LocalDate`(시각 없음)라 타임존 영향을 받지 않는다. `created_at`·`updated_at`·`deleted_at`만 해당된다.

### ⚠️ 연관관계는 반드시 LAZY로 지정한다

`@ManyToOne`의 기본값은 **EAGER**다. `Todo.user`를 그대로 두면 목록 조회 시 사용자 정보를 매번 함께 가져와 불필요한 쿼리가 늘어난다.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id", nullable = false)
private User user;
```

- 소유권 검증은 `todo.getUser().getId()`로 한다. 프록시 상태에서 id만 읽으면 추가 쿼리가 발생하지 않는다.
- **`TodoResponse`에 사용자 정보를 넣지 않는다.** 본인 데이터만 조회하므로 불필요하고, 넣으면 목록에서 N+1이 난다.

### 검색 쿼리 주의

`LOWER(title) LIKE '%키워드%'`는 앞쪽 와일드카드 때문에 인덱스를 타지 못한다. MVP 데이터 규모(수백 건)에서는 문제없지만, **`title`에 인덱스를 추가해도 검색은 빨라지지 않는다는 점을 알고 있어야 한다.** 규모가 커지면 전문 검색(`pg_trgm`)을 검토한다.

### 운영 스키마 생성 절차 (Flyway 미사용)

`validate`로 바꾸면 테이블을 만들 주체가 사라진다. Phase 11에서 다음 순서로 처리한다.

1. 로컬에서 `spring.jpa.properties.jakarta.persistence.schema-generation.scripts`로 DDL 스크립트를 추출한다.
2. 추출된 DDL을 검토 후 RDS에 **1회 수동 적용**한다.
3. 운영 프로파일을 `ddl-auto: validate`로 고정한다.
4. DDL 스크립트를 `todo-backend/src/main/resources/db/schema.sql`에 커밋해 이력을 남긴다.

> 스키마 변경이 잦아지면 그때 Flyway를 도입한다. MVP에서는 도입하지 않는다.

---

## 5. API 명세

Base path: `/api/v1`

### 공통 응답 포맷 (예외 없이 모든 REST 응답에 적용)

```json
// 성공
{ "success": true, "data": { ... }, "error": null }

// 실패
{ "success": false, "data": null,
  "error": { "code": "TODO_NOT_FOUND", "message": "할 일을 찾을 수 없습니다." } }
```

> **목록 API도 이 포맷을 따른다.** `PageResponse`는 최상위가 아니라 **`data` 안에** 들어간다. 프론트는 `res.data.data.content`로 접근한다. 아래 예시 참조.

**단, OAuth2 콜백은 REST 응답이 아니라 302 리다이렉트이므로 이 포맷이 적용되지 않는다.**

### 인증

| Method | Endpoint                       | 설명                                              | 인증               |
| ------ | ------------------------------- | ------------------------------------------------- | ------------------ |
| POST   | `/api/v1/auth/signup`          | 회원가입 (email, password, nickname)              | X                  |
| POST   | `/api/v1/auth/login`           | 로그인 → Access Token 반환 + Refresh Token 쿠키 발급 | X                  |
| POST   | `/api/v1/auth/refresh`         | Refresh Token 쿠키로 Access Token 재발급(회전)     | X (쿠키로 인증)    |
| POST   | `/api/v1/auth/logout`          | Refresh Token 폐기 + 쿠키 만료                     | O                  |
| GET    | `/api/v1/auth/me`              | 내 정보 조회                                      | O                  |
| GET    | `/oauth2/authorization/google` | 구글 로그인 시작 (Spring Security 제공)           | X                  |

> **v1.9**: 6장을 Access+Refresh 2-토큰 설계로 확정하면서 `/auth/refresh`·`/auth/logout`을 추가했다. 로그아웃은 클라이언트 토큰 삭제만으로 끝내지 않고 반드시 이 API를 호출해 서버의 Refresh Token을 폐기한다(6장 참조).

### Todo

| Method | Endpoint                    | 설명                 | 요청 바디               |
| ------ | --------------------------- | -------------------- | ----------------------- |
| GET    | `/api/v1/todos`             | 목록 (페이지네이션)  | —                       |
| POST   | `/api/v1/todos`             | 생성                 | `TodoCreateRequest`     |
| GET    | `/api/v1/todos/{id}`        | 단건 조회            | —                       |
| PUT    | `/api/v1/todos/{id}`        | 수정 (**전체 교체**) | `TodoUpdateRequest`     |
| PATCH  | `/api/v1/todos/{id}/toggle` | 완료 상태 변경       | `{ "completed": true }` |
| DELETE | `/api/v1/todos/{id}`        | Soft Delete          | —                       |

#### ⚠️ PUT과 toggle의 역할 분리 (중요)

- **`TodoUpdateRequest`에 `completed`를 포함하지 않는다.** 필드는 `title`, `content`, `priority`, `dueDate` 넷뿐이다.
- 완료 상태는 **오직 toggle 엔드포인트로만** 변경한다.
- 이유: PUT에 `completed`가 있으면 상세 화면에서 저장할 때마다 완료 상태를 덮어써, 목록에서 체크한 결과가 되돌아가는 버그가 난다.
- PUT은 부분 수정이 아니라 **전체 교체**다. 네 필드를 모두 보낸다. 다만 `title`은 필수이므로 누락 시 null이 아니라 **400**이고, `content`·`dueDate`는 누락하면 null로 저장된다(값 삭제로 취급).

#### ⚠️ toggle은 바디로 명시적 값을 받는다 (중요)

서버가 현재 값을 뒤집는 방식(`completed = !completed`)을 **쓰지 않는다.** 낙관적 업데이트와 함께 쓰면 사용자가 빠르게 연타할 때 요청 순서가 뒤바뀌어 서버 상태와 UI가 어긋나고, 롤백으로도 복구되지 않는다.

바디로 목표 상태를 받으면 멱등해져 순서가 뒤바뀌어도 최종 상태가 일치한다.

#### 목록 쿼리 파라미터

| 파라미터    | 기본값           | 동작                                                                                                                                                                                                                    |
| ----------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `page`      | 0                | 0부터 시작                                                                                                                                                                                                              |
| `size`      | 10               |                                                                                                                                                                                                                         |
| `sort`      | `createdAt,desc` | **MVP에서는 고정.** API는 받되 정렬 선택 UI는 만들지 않는다. **허용 필드는 `createdAt`, `dueDate`뿐이며, 그 외 값이 들어오면 기본값으로 대체한다.** Pageable에 임의 문자열을 그대로 넘기면 없는 프로퍼티에서 500이 난다 |
| `completed` | 미지정           | **미지정 시 전체 반환.** `true`/`false`일 때만 필터 적용                                                                                                                                                                |
| `keyword`   | 미지정           | 제목 부분 일치, **대소문자 무시** (`LOWER(title) LIKE LOWER(:keyword)`)                                                                                                                                                 |

#### 목록 응답 예시 (공통 포맷 적용)

```json
{
  "success": true,
  "data": {
    "content": [ { "id": 1, "title": "...", "completed": false, ... } ],
    "page": 0,
    "size": 10,
    "totalElements": 42,
    "totalPages": 5,
    "first": true,
    "last": false
  },
  "error": null
}
```

Spring의 `Page` 객체를 그대로 반환하지 않고 `PageResponse<T>` DTO로 변환한 뒤 `ApiResponse.data`에 담는다.

#### 날짜 직렬화 포맷

- `createdAt`, `updatedAt` → **ISO-8601 UTC 문자열** (`2026-08-28T04:30:00Z`)
- `dueDate` → **`yyyy-MM-dd`** (시각 없음)
- 배열 형태(`[2026,8,28,...]`)로 직렬화되지 않도록 `WRITE_DATES_AS_TIMESTAMPS`를 비활성 상태로 유지한다.

> ⚠️ **Spring Boot 4는 Jackson 3를 쓴다. 별도 설정을 넣지 않는다.**
> Jackson 3의 기본값이 이미 ISO-8601 문자열이므로 위 요구는 **무설정으로 충족된다.**
> Boot 3 시절 튜토리얼을 보고 `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS`를 코드에서 참조하면 **컴파일에 실패한다.** 이 상수는 Jackson 3에서 `DateTimeFeature`로 이동했고, 프로퍼티 경로도 `spring.jackson.serialization.*` → `spring.jackson.datatype.datetime.*`으로 바뀌었다. 굳이 명시하지 말고 기본값에 맡긴다.

#### 인증 예외 케이스

토큰은 유효한데 해당 사용자가 조회되지 않는 경우(토큰 발급 후 계정이 사라진 상황)는 **404가 아니라 401 `UNAUTHORIZED`**로 응답한다. 프론트는 401을 자동 로그아웃으로 처리하므로 일관된다.

### 첨부 (Attachment)

> **v1.13 추가.** 본문 이미지 첨부(`TODO-17`·`TODO-18`).

| Method | Endpoint                                | 인증           | 설명                        |
| ------ | --------------------------------------- | -------------- | --------------------------- |
| POST   | `/api/v1/attachments/presign`           | Bearer         | 업로드 URL 발급 + TEMP 행 생성 |
| PUT    | `/api/v1/attachments/{id}/upload?token=` | **서명 토큰**  | 파일 수신 (로컬 전용)        |
| POST   | `/api/v1/attachments/{id}/complete`     | Bearer         | 업로드 확정 (존재·크기·매직바이트 검증) |
| GET    | `/api/v1/attachments/urls?ids=1,2,3`    | Bearer         | 조회 URL **일괄** 발급       |
| GET    | `/api/v1/attachments/{id}/raw?token=`   | **서명 토큰**  | 파일 스트림 (로컬 전용)      |
| DELETE | `/api/v1/attachments/{id}`              | Bearer         | Soft Delete                 |

#### ⚠️ 업로드·조회 URL은 헤더가 아니라 서명 토큰으로 인증한다 (중요)

"프론트 코드가 로컬·S3에서 동일하다"는 목표는 **인증 방식을 통일해야만 성립한다.**

로컬 업로드 엔드포인트에 `Authorization` 헤더를 요구하면, Phase 13에서 S3 presigned PUT으로 바꾸는 순간 **헤더 때문에 서명이 깨져 403**이 난다. 그래서 로컬도 헤더가 아니라 **URL 쿼리의 단기 서명 토큰**으로 인증한다. `<img>` 태그가 헤더를 실을 수 없어 조회 URL에 토큰이 필요한 것과 같은 논리다.

- 프론트는 두 경우 모두 **`Authorization` 없이 `credentials: "omit"`으로** 순수 `fetch` PUT을 보낸다.
- 서명 토큰은 `JwtTokenProvider`에 메서드를 추가해 기존 키를 재사용한다. 클레임은 `sub`=attachmentId, `typ`=`att-upload`/`att-view`, `exp`.
- **검증 시 반드시 `typ`을 확인한다.** 일반 Access Token이 조회 토큰으로 통용되면 안 된다.
- 만료: 업로드 토큰 **10분**, 조회 토큰 **30분**.
- 나머지 첨부 API(`presign`·`complete`·`urls`·`DELETE`)는 **기존대로 Bearer 인증**이다.

#### ⚠️ `raw`는 `ApiResponse` 래핑에서 제외된다

바이너리 스트림이라 JSON으로 감쌀 수 없다. **공통 응답 포맷의 두 번째 예외**다(첫 번째는 OAuth2 302 리다이렉트).

#### 조회는 일괄(batch)이다

본문에 이미지가 N개면 요청도 N번이 되는 것을 막기 위해 `GET /urls?ids=1,2,3` 형태로 한 번에 받는다. 단건 조회 엔드포인트를 만들지 않는다.

#### 소유권 위반은 404다

첨부도 Todo와 동일하다. 타인 소유·없는 id·삭제됨을 **모두 404 `ATTACHMENT_NOT_FOUND`**로 응답한다(존재 여부 노출 방지). **403을 쓰지 않는다.**

#### 파일 제약

| 항목        | 값                                                  |
| ----------- | --------------------------------------------------- |
| 최대 크기   | **5MB**                                             |
| 허용 타입   | `image/jpeg`, `image/png`, `image/gif`, `image/webp` |
| 타입 검증   | 화이트리스트 + **매직바이트 확인** (클라이언트가 보낸 `contentType`을 그대로 믿지 않는다) |
| 확장자      | `contentType`에서 역산한다. 원본 파일명의 확장자를 쓰지 않는다 |

---

## 6. 인증 설계

> **v1.9 변경**: 이 장은 원래 Access Token 단일 사용(24시간, Refresh 없음)으로 작성되어 있었으나, `docs/PRD.md`(264행)가 이미 Access+Refresh 2-토큰 설계(`todo_access_token`, httpOnly Refresh 쿠키)를 전제로 작성되어 있었고 사용자가 이 설계로 확정했다. 4장의 `refresh_tokens` 테이블, 5장의 `/auth/refresh`·`/auth/logout` API와 함께 이 장을 정정했다.

### 토큰 (2종 구조)

|                | Access Token                             | Refresh Token                              |
| -------------- | ----------------------------------------- | ------------------------------------------- |
| 형식           | JWT                                       | 불투명 랜덤 문자열 (JWT 아님)               |
| 만료           | **30분**                                  | **14일**                                    |
| 저장 위치      | 프론트엔드 **localStorage** (키: `todo_access_token`) | **httpOnly + Secure + SameSite 쿠키** (`refresh_token`) |
| 전송           | `Authorization: Bearer <token>` 헤더      | 브라우저가 자동 전송                        |
| 서버 저장      | 저장하지 않음(stateless)                  | `refresh_tokens` 테이블에 **SHA-256 해시로** 저장(4장) |

- **Refresh Token을 localStorage나 JS에서 접근 가능한 곳에 절대 두지 않는다.** 서버 로그아웃과 XSS 방어의 근거다.
- Refresh Token은 **회전(rotation)**한다. `/auth/refresh` 호출마다 새로 발급하고 기존 행은 `revoked_at`을 채워 폐기한다.
- 이미 폐기된 Refresh Token이 다시 들어오면 **탈취로 간주해 해당 사용자의 모든 Refresh Token을 폐기**하고 재로그인시킨다.
- 프론트는 401(`TOKEN_EXPIRED`)을 받으면 자동으로 `/api/v1/auth/refresh`를 1회 시도하고, 성공하면 원래 요청을 재시도한다. 실패하면 토큰을 지우고 `/login`으로 보낸다. **재시도 루프에 빠지지 않도록 refresh 요청 자체는 재시도 대상에서 제외한다.**
- 로그아웃은 서버 `POST /api/v1/auth/logout`을 호출해 **Refresh Token을 DB에서 폐기하고 쿠키를 만료**시킨다. 클라이언트 토큰 삭제만으로 끝내지 않는다.

### JWT 클레임 구성 (Access Token)

```
sub    : user.id (숫자 문자열)
email  : user.email
iat    : 발급 시각
exp    : 발급 + 30분
```

- **`sub`는 이메일이 아니라 id를 담는다.** 이메일 변경 기능이 없어도 id 기반이 조회에 유리하고, 인증 필터에서 PK 조회로 끝난다.
- `JwtAuthenticationFilter`는 `sub`를 파싱해 사용자 id를 얻고, `deleted_at IS NULL` 조건으로 조회한다.
- Refresh Token은 JWT가 아니므로 클레임이 없다. 불투명 랜덤 문자열(예: `SecureRandom` 기반 256비트)을 그대로 발급하고, 서버는 해시만 대조한다.

### SecurityConfig 인가 경로 (필수)

```
permitAll:
  /api/v1/auth/signup
  /api/v1/auth/login
  /api/v1/auth/refresh
  /oauth2/**
  /login/oauth2/**
  /swagger-ui/**
  /v3/api-docs/**
  /error

그 외: authenticated
```

> `/api/v1/auth/refresh`는 `Authorization` 헤더가 아니라 httpOnly 쿠키로 인증하므로 `JwtAuthenticationFilter`(Bearer 토큰 검사) 대상에서 제외하고 permitAll에 둔다. 컨트롤러 내부에서 쿠키의 Refresh Token을 직접 검증한다.

> ⚠️ **Swagger 경로를 빼먹으면 Phase 3에서 Swagger UI가 막힌다.** Phase 1의 DoD("Swagger 접속 확인")가 조용히 회귀하므로 SecurityConfig 작성 시 반드시 함께 넣는다.

#### ⚠️ CSRF 비활성화와 STATELESS 세션은 필수다 (중요)

**이 두 줄이 없으면 `POST /api/v1/auth/signup`부터 403으로 막힌다.**

`SecurityFilterChain`을 직접 정의하면 **CSRF 보호가 기본으로 켜진다.** 이 프로젝트는 쿠키가 아니라 `Authorization: Bearer` 헤더로 인증하는 stateless API이므로 CSRF 토큰을 발급하는 경로 자체가 없다. 즉 모든 상태 변경 요청(POST/PUT/PATCH/DELETE)이 CSRF 토큰 누락으로 거부된다. Spring Boot 4가 번들하는 Spring Security 7에서 특히 흔한 실패다.

세션도 마찬가지다. 명시하지 않으면 Security가 `JSESSIONID`를 발급해, 토큰 기반 설계와 세션 기반 상태가 뒤섞인다.

```java
http
    .csrf(AbstractHttpConfigurer::disable)                    // JWT stateless — CSRF 토큰 경로 없음
    .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))  // 세션 미사용
    .authorizeHttpRequests(auth -> auth
        .requestMatchers(...).permitAll()
        .anyRequest().authenticated()
    )
    .exceptionHandling(...)   // 아래 11장의 EntryPoint / AccessDeniedHandler
```

> **`authorizeRequests()`는 Spring Security 7에서 제거되었다.** 반드시 `authorizeHttpRequests()`를 쓴다. Boot 3 시절 예제를 그대로 옮기면 컴파일에 실패한다.

### Swagger JWT 인증 설정 (필수)

경로를 열어두는 것만으로는 부족하다. 보호된 API를 Swagger에서 실제로 호출하려면 Authorize 버튼이 있어야 한다. 없으면 API 목록만 보이고 모든 호출이 401이라, "Swagger에서 전체 API 확인"이 형식적으로만 통과한다.

`config/`에 다음을 설정한다.

```java
@SecurityScheme(
    name = "bearerAuth",
    type = SecuritySchemeType.HTTP,
    scheme = "bearer",
    bearerFormat = "JWT"
)
```

- `OpenAPI` 빈에 `addSecurityItem`으로 전역 적용하거나, 보호된 컨트롤러에 `@SecurityRequirement(name = "bearerAuth")`를 붙인다.

### CORS

#### ⚠️ `FRONTEND_URL`과 `CORS_ALLOWED_ORIGINS`는 반드시 분리한다 (중요)

두 용도는 **양립할 수 없는 형식**을 요구한다. 하나의 변수로 겸용하면 **운영에서만 구글 로그인이 깨진다.**

| 변수                   | 형식               | 용도                                                                   |
| ---------------------- | ------------------ | ---------------------------------------------------------------------- |
| `FRONTEND_URL`         | **단일 URL**       | OAuth2 리다이렉트의 기준 주소 (`{FRONTEND_URL}/oauth/callback?token=`) |
| `CORS_ALLOWED_ORIGINS` | **쉼표 구분 목록** | CORS 허용 오리진                                                       |

CORS 요구대로 `FRONTEND_URL`에 쉼표 목록을 넣으면, `OAuth2SuccessHandler`가 만드는 리다이렉트 URL이
`https://todo.example.com,https://main.d123.amplifyapp.com/oauth/callback?token=...`
이 되어 **깨진 주소로 302를 보낸다.** 로컬은 단일값이라 정상 동작하므로 Phase 5 DoD를 통과하고, **Phase 11에서야 발현한다.**

- 허용 오리진: 환경변수 `CORS_ALLOWED_ORIGINS`. 쉼표로 구분된 목록을 받는다. 운영에서 Amplify 브랜치 도메인과 커스텀 도메인을 동시에 허용해야 하는 경우가 생긴다 (로컬은 `http://localhost:3000` 하나)
- 허용 메서드: `GET, POST, PUT, PATCH, DELETE, OPTIONS`
- **허용 헤더에 `Authorization`, `Content-Type`을 명시한다.** 기본값에 의존하면 프리플라이트에서 막히는 경우가 잦다.
- **Refresh Token을 httpOnly 쿠키로 주고받으므로 `allowCredentials(true)`가 필수다.** 와일드카드(`*`) 오리진은 `allowCredentials(true)`와 함께 쓸 수 없으므로 명시적 오리진 목록만 허용한다.
  - 로컬(`localhost:3000` ↔ `localhost:8080`)은 same-site이므로 `SameSite=Lax`로 동작한다.
  - 운영(Amplify 도메인 ↔ API 도메인)은 cross-site이므로 쿠키에 `SameSite=None; Secure`가 필요하다. 이 값은 프로파일별로 분리한다.
- 프론트의 모든 인증 요청은 `credentials: 'include'`로 보낸다(`apiClient` 공통 설정).

### 구글 OAuth2 흐름

1. 프론트가 `/oauth2/authorization/google`로 이동
2. 구글 인증 후 백엔드 콜백 처리
3. 분기
   - 신규 이메일 → `provider=GOOGLE`로 가입
   - 기존 GOOGLE 계정 → 조회
   - **동일 이메일의 LOCAL 계정 존재 → 거부**
4. 성공 시 Access Token 생성 + Refresh Token 발급(httpOnly 쿠키로 심음, `/auth/login`과 동일한 방식) 후 `{FRONTEND_URL}/oauth/callback?token=xxx`(Access Token만)로 **302 리다이렉트**
5. 프론트 콜백 페이지가 Access Token 저장 → URL에서 제거 → `/todos`로 이동

### 구글 가입 시 nickname 결정

`nickname`이 NOT NULL이므로 반드시 채워야 한다.

1. 구글이 반환한 `name`을 사용한다.
2. 없거나 비어 있으면 이메일의 `@` 앞부분을 사용한다.
3. 50자를 넘으면 절삭한다.

### 계정 충돌 정책 (확정)

같은 이메일로 **로컬 계정이 이미 존재하는 상태에서 구글 로그인을 시도하면 거부한다.**

- 자동 연동하지 않는다. 구글이 반환한 이메일만 믿고 기존 계정 접근을 허용하면 계정 탈취 경로가 된다.
- 별도 계정도 만들지 않는다. `email` UNIQUE 제약을 유지한다.
- **REST 에러 응답이 아니라 302 리다이렉트로 처리한다.** `OAuth2FailureHandler`가 `{FRONTEND_URL}/login?error=email_conflict`로 보내고, 프론트가 "이미 이메일로 가입된 계정입니다. 이메일로 로그인해 주세요."를 표시한다.
- 반대 방향(구글 계정 존재 + 로컬 가입 시도)은 일반 회원가입 경로이므로 `EMAIL_DUPLICATED`(409) JSON 응답으로 처리한다.

### 보안 규칙

- 비밀번호: **BCrypt** 해싱, 6자 이상 + **UTF-8 72바이트 이하** (4장 ⚠️ 참조)
- 이메일: 형식 검증 + 중복 검사
- 모든 요청 DTO에 `@Valid` + Bean Validation
- **소유권 검증**: Todo 조회/수정/삭제 시 `todo.user_id == 인증 사용자 id` 확인. 불일치 시 **404** 반환(존재 여부 노출 방지)
- JWT Secret 하드코딩 금지

### XSS 방어 (필수)

Tiptap이 생성한 HTML을 저장하고 렌더링하는 구조이므로 **양쪽에서 모두 정화한다.** 토큰을 localStorage에 두기로 했기 때문에, 저장된 스크립트가 실행되면 곧바로 토큰 탈취로 이어진다.

**저장 시 (백엔드)** — `service/HtmlSanitizer`에서 Jsoup Safelist로 정화한 뒤 저장한다.

허용 태그 (Tiptap 툴바와 1:1로 맞춘다):

```
p, br, strong, em, h2, h3, ul, ol, li, a, code, pre, blockquote, img
```

- `a`는 `href`만 허용하고, **`rel="noopener noreferrer"`와 `target="_blank"`를 정화 단계에서 강제 주입**한다 (tabnabbing 방지).
- **`img`는 `data-attachment-id`와 `alt`만 허용하고 `src`는 허용하지 않는다** (v1.13 추가, 아래 ⚠️ 참조).
- `href`는 `http`, `https`, `mailto` 스킴만 허용한다. `javascript:` 차단.
- `script`, `iframe`, `style` 태그와 모든 `on*` 속성, `style` 속성은 제거한다.
- 직접 정규식으로 구현하지 않는다. Jsoup Safelist를 사용한다.
- **`Jsoup.clean(html, "", safelist, outputSettings)` 오버로드를 쓰고 `new Document.OutputSettings().prettyPrint(false)`를 넘긴다.** 기본 pretty-print가 블록 요소를 재포맷해 **코드 블록(`pre`)의 공백과 줄바꿈이 망가진다.**

**렌더 시 (프론트)** — `lib/sanitize.ts`에서 DOMPurify로 한 번 더 정화한다. 서버를 신뢰하더라도 이중 방어를 유지한다.

#### ⚠️ `img`에 `src`를 저장하지 않는다 (중요, v1.13)

저장되는 본문은 다음 형태다. **`src`가 없다.**

```html
<p>회의 자료</p>
<img data-attachment-id="42" alt="스크린샷">
```

이유가 둘이다.

1. **만료되는 URL이 DB에 박히지 않는다.** 조회 URL은 30분 만료라 본문에 저장하면 하루 뒤 전부 깨진다.
2. **임의의 외부 URL을 본문에 심는 경로가 원천 차단된다.** `src`를 허용하면 외부 추적 픽셀이나 임의 호스트 이미지를 넣을 수 있다.

렌더링 시 URL을 주입하는데, **순서를 지켜야 한다.**

```
서버 HTML(src 없음)
  → sanitizeHtml()                      ← ① 정화를 먼저
  → DOM 파싱
  → img[data-attachment-id]에 src 주입   ← ② 주입을 나중에
  → 렌더
```

> ⚠️ **반대로 하면 방금 넣은 `src`를 DOMPurify가 지운다.** 주입은 문자열 이어붙이기가 아니라 `setAttribute`로 하고, 값이 `http(s)`로 시작하는지 확인한다. 헬퍼는 `src/lib/attachmentHtml.ts`에 둔다.

> ⚠️ **에디터가 서버에 보내는 HTML에는 `src`가 있고, 서버가 저장하는 HTML에는 없다.** 저장 후 서버 응답으로 캐시를 덮어쓰므로 동작은 정상이다. "보낸 것과 응답이 다르다"를 버그로 오인하지 말 것.

#### ⚠️ 정화의 적용 지점은 두 곳이다 — 하나만 하면 이중 방어가 아니다 (중요)

> **v1.12 변경**: 이전 버전은 "상세 화면이 진입 즉시 편집 가능이라 `dangerouslySetInnerHTML` 호출 지점이 아예 없다"고 단정했다. 7장이 바뀌어 `/todos/[id]`가 읽기 전용 확인 화면이 되면서 그 전제가 깨졌다 — 본문을 읽기 전용으로 그리려면 `dangerouslySetInnerHTML`이 필요하고, 여기도 정화를 거치지 않으면 이중 방어의 두 번째 축이 통째로 빠진다.

적용 지점을 다음 두 곳으로 고정한다.

```ts
// 1. 에디터에 주입하기 직전 (편집 화면, TodoEditor.tsx)
editor.commands.setContent(sanitizeHtml(todo.content));

// 2. 읽기 전용으로 그리기 직전 (확인 화면, /todos/[id]/page.tsx)
<div dangerouslySetInnerHTML={{ __html: sanitizeHtml(todo.content ?? "") }} />
```

- **`editor.commands.setContent()` 호출 직전**과 **읽기 전용 확인 화면의 `dangerouslySetInnerHTML` 직전** 둘 다 `lib/sanitize.ts`를 반드시 거친다.
- 앞으로 목록 미리보기 등 본문을 그리는 화면이 더 생기면 그곳도 동일하게 거친다.

#### DOMPurify 허용 목록 (Jsoup과 동일하게 유지할 것)

네 번째 방어선의 설정을 명시하지 않으면 "네 곳이 같은 태그 집합"이라는 규칙 자체를 검증할 수 없다. DOMPurify 기본값은 Jsoup 목록보다 훨씬 넓으므로(`img`, `table`, `u`, `h1` 등) **반드시 명시적으로 좁힌다.**

```ts
DOMPurify.sanitize(html, {
  ALLOWED_TAGS: [
    "p",
    "br",
    "strong",
    "em",
    "h2",
    "h3",
    "ul",
    "ol",
    "li",
    "a",
    "code",
    "pre",
    "blockquote",
    "img", // v1.13 — 본문 이미지 첨부
  ],
  ALLOWED_ATTR: [
    "href",
    "target",
    "rel", // target·rel을 빼면 서버가 주입한 값이 렌더에서 지워진다
    "data-attachment-id", // v1.13 — src는 넣지 않는다
    "alt",
  ],
  ALLOW_DATA_ATTR: false, // 명시한 data-* 하나만 통과시킨다
});
```

> ⚠️ `ALLOWED_ATTR`에 명시한 `data-attachment-id`는 `ALLOW_DATA_ATTR: false`에서도 통과한다(명시 목록이 우선). 구현 전에 **DOMPurify 공식 문서로 재확인**할 것 — 3장 "라이브러리 API를 기억에 의존하지 않는다".

> ⚠️ `ALLOWED_ATTR`에서 `target`·`rel`을 빠뜨리면, 백엔드가 강제 주입한 `rel="noopener noreferrer"`가 **렌더 단계에서 제거되어 tabnabbing 방어가 무효화된다.**

정화 로직은 통합 테스트로 검증한다.

---

## 7. 화면 목록

| 경로              | 화면                          | 인증 |
| ----------------- | ----------------------------- | ---- |
| `/login`          | 로그인 (이메일 + 구글)        | X    |
| `/signup`         | 회원가입                      | X    |
| `/oauth/callback` | 소셜 토큰 처리                | X    |
| `/todos`          | 목록 (필터/검색/페이지네이션) | O    |
| `/todos/new`      | 작성 (Tiptap)                 | O    |
| `/todos/[id]`     | 확인 (읽기 전용)              | O    |
| `/todos/[id]/edit`| 수정 (Tiptap)                 | O    |

- 미인증 상태로 보호된 경로 접근 시 `/login`으로 리다이렉트.
- 인증 화면의 공통 헤더에는 **닉네임과 로그아웃 버튼**을 둔다.

> **v1.12 변경**: 상세 화면을 확인(읽기 전용)과 수정으로 분리했다. 이전에는 목록에서 항목을 누르면 곧장 편집 폼이 열렸는데, 실사용 중 "선택만 했는데 수정 화면이 뜬다"는 혼란을 일으켜 뒤집었다.
>
> - **`/todos/[id]`(확인)**: 제목·우선순위·마감일·본문을 읽기 전용으로 보여준다. 본문은 Tiptap을 띄우지 않고 `dangerouslySetInnerHTML` + `sanitizeHtml`로 그린다(6장 참조). "수정"·"목록으로" 버튼만 있다.
> - **`/todos/[id]/edit`(수정)**: 기존 "진입 즉시 편집 가능" 폼 그대로다. `/todos/new`와 **`TodoForm` 컴포넌트를 재사용**하고, 초기값 유무와 삭제 버튼 노출로만 구분한다. 변경 사항이 있는 상태에서 이탈하려 하면 확인 대화상자를 띄운다(`useLeaveGuard`). 저장 성공 시 확인 화면(`/todos/[id]`)으로, 삭제 성공 시 목록(`/todos`)으로 이동한다. "취소"도 확인 화면으로 돌아간다.
> - **목록(`/todos`)**: 제목 링크는 확인 화면으로 가고, 항목마다 별도 연필 아이콘("수정") 버튼이 `/todos/[id]/edit`로 바로 연결된다.

---

## 8. UI 디자인 가이드

**방향**: 심플·모던. 장식보다 여백과 타이포그래피로 위계를 만든다.

### 컬러 (Tailwind 4 `@theme` 토큰)

- 배경 `#FAFAFA` / 다크 `#0A0A0A`
- 카드 `#FFFFFF` / 다크 `#171717`
- 텍스트 `#171717` / 다크 `#FAFAFA`, 보조 `#737373`
- 액센트: **단일 컬러 1개만** (`#4F46E5`)
- 우선순위: HIGH `#EF4444`, MEDIUM `#F59E0B`, LOW `#10B981`

### 스타일 원칙

- 그림자 대신 **1px border**(`#E5E5E5`)로 면 구분. 그림자는 모달/드롭다운에만.
- 라운드: 카드 `rounded-xl`, 버튼/인풋 `rounded-lg`
- 폰트: **Pretendard**. ⚠️ **Google Fonts에 없으므로 `next/font/google`로 불러올 수 없다.** 폰트 파일(`.woff2`)을 `src/app/fonts/`에 넣고 **`next/font/local`**로 로드한다. 가변 폰트(`PretendardVariable.woff2`) 하나면 충분하다.
- 본문 15px / 항목 제목 16px semibold / 캡션 13px
- 컨테이너 `max-w-3xl`, 패딩 모바일 16px · 데스크톱 24px
- **다크 모드**: 토큰을 라이트/다크 양쪽으로 정의하고 **`@media (prefers-color-scheme: dark)`로 전환**한다. 사용자가 전환하는 **토글 UI는 MVP 범위 밖**이다.
  > ⚠️ **`class` 전략을 쓰지 않는다.** 토글이 없는데 `class` 전략을 쓰면 서버 렌더 시점에 클래스가 없어 라이트로 그려졌다가 클라이언트에서 다크로 바뀌는 깜빡임(FOUC)이 생기고, 이를 막으려면 `<head>`에 인라인 스크립트를 넣어야 한다. 미디어쿼리는 CSS만으로 처리되어 hydration 문제가 아예 없다. 나중에 토글을 추가할 때 `class` 전략으로 바꾼다.
  > ⚠️ **`@theme`을 `@media` 안에 중첩하지 않는다.** Tailwind v4에서 `@theme`은 **최상위에만 올 수 있다**(공식 문서: "Theme variables must be defined at the top level and cannot be nested under other selectors or media queries"). `@theme`은 유틸리티 클래스를 생성하는 지시어이지 단순 변수 선언이 아니기 때문이다. 라이트 값을 `@theme`에 한 번 선언해 유틸리티를 만들고, **다크에서는 생성된 커스텀 프로퍼티를 `:root`에서 덮어쓴다.**
  >
  > ```css
  > @theme {
  >   --color-bg: #fafafa;
  > } /* 유틸리티 생성 */
  > @media (prefers-color-scheme: dark) {
  >   :root {
  >     --color-bg: #0a0a0a;
  >   } /* 값만 교체 */
  > }
  > ```

### Tiptap 설정 (정화 화이트리스트와 일치시킬 것)

**툴바**: 굵게(`strong`) · 기울임(`em`) · 제목 H2 · 제목 H3 · 불릿 목록 · 번호 목록 · 링크 · 인라인 코드 · 코드 블록 · 인용 · **이미지**(v1.13)

#### 이미지 확장 (v1.13)

**`@tiptap/extension-image`를 설치한다.** StarterKit v3에는 포함되어 있지 않다. 이것이 이 프로젝트에서 유일하게 추가 승인된 라이브러리다(절대규칙 11).

```ts
import Image from "@tiptap/extension-image";

// 첨부 ID를 data-attachment-id로 왕복시키는 커스텀 노드.
// src는 저장되지 않으므로(6장) 렌더 시점에 주입한다.
const AttachmentImage = Image.extend({
  addAttributes() {
    return {
      ...this.parent?.(),
      attachmentId: {
        default: null,
        parseHTML: (el) => el.getAttribute("data-attachment-id"),
        renderHTML: (attrs) =>
          attrs.attachmentId ? { "data-attachment-id": attrs.attachmentId } : {},
      },
    };
  },
});

AttachmentImage.configure({ inline: false, allowBase64: false });
```

> ⚠️ **`allowBase64: false`가 중요하다.** 클립보드 이미지를 그대로 붙이면 `data:` URI가 본문에 들어가는데, `content` 50,000자 제한(4장)을 **한 장으로 즉시 초과**한다. 게다가 정화가 `src`를 지우므로 이미지가 조용히 사라진다. 붙여넣기·드래그앤드롭은 `editorProps`의 `handlePaste`·`handleDrop`으로 가로채 업로드 경로로 보낸다.

#### ⚠️ StarterKit을 기본값으로 쓰지 않는다 (중요)

StarterKit은 툴바에 버튼이 없어도 **마크다운 입력 규칙을 함께 켠다.** 사용자가 본문에 `# `를 치면 `<h1>`, `~~취소선~~`은 `<s>`, `---`는 `<hr>`이 생성된다. 이 태그들은 화이트리스트에 없어 서버 정화 단계에서 제거되므로, **사용자가 입력한 서식이 저장 후 조용히 사라진다.** 원인을 찾기 어려운 종류의 버그다.

**Tiptap v3의 StarterKit은 `Link`와 `Underline`을 이미 포함한다.** v2에서는 둘 다 별도 패키지였으나 v3에서 StarterKit에 흡수되었다. 이 차이를 모르고 v2 방식으로 쓰면 두 가지 문제가 생긴다.

1. **`Underline`이 켜진 채로 남는다.** 툴바에 버튼이 없어도 **Ctrl+U가 동작해 `<u>`가 생성되고**, 화이트리스트에 없어 저장 시 제거된다 → 위에서 말한 "서식 무음 소실" 버그가 그대로 발생한다.
2. **`@tiptap/extension-link`를 따로 설치해 등록하면 Link가 중복 등록된다.**

따라서 사용하지 않는 확장을 명시적으로 끄고, **Link는 StarterKit 내장으로 설정한다.**

```ts
import StarterKit from "@tiptap/starter-kit";
// @tiptap/extension-link 는 설치하지 않는다. v3 StarterKit에 포함되어 있다.

const extensions = [
  StarterKit.configure({
    heading: { levels: [2, 3] }, // h1, h4~h6 차단
    strike: false, // <s> 차단
    horizontalRule: false, // <hr> 차단
    underline: false, // <u> 차단 — Ctrl+U까지 함께 꺼진다
    link: {
      // v3 내장 Link를 그대로 설정
      openOnClick: false,
      HTMLAttributes: { rel: "noopener noreferrer", target: "_blank" },
      protocols: ["http", "https", "mailto"],
    },
  }),
];
```

> 밑줄(`u`), 취소선(`s`)은 넣지 않는다. **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify 설정 네 곳이 항상 같은 태그 집합을 가리켜야 한다.** 한 곳을 바꾸면 나머지 세 곳도 함께 바꾼다.
>
> 확인 방법: 에디터 본문에서 **`# `, `~~취소선~~`, `---`, Ctrl+U** 네 가지를 모두 시도해 아무 서식도 생성되지 않아야 한다.

### 인터랙션 (Motion)

import는 항상 `motion/react`에서 한다.

```ts
import { motion, AnimatePresence, useReducedMotion } from "motion/react";
```

- 목록 등장: `opacity 0→1` + `y 8→0`, stagger 30ms
- 삭제: `opacity→0` + `height→0`, `AnimatePresence`
- 토글: scale 스프링
- **200ms 이내, 과한 모션 금지.** `useReducedMotion`으로 `prefers-reduced-motion` 존중

---

## 9. 상태 처리 규칙

### ⚠️ App Router 렌더링 경계 (중요)

인증 토큰이 localStorage에 있고 데이터를 React Query로 가져오므로, **이 프로젝트의 페이지는 사실상 전부 클라이언트 컴포넌트다.**

- `/login`, `/signup`, `/oauth/callback`, `/todos`, `/todos/new`, `/todos/[id]`, `/todos/[id]/edit`의 `page.tsx`에 **`"use client"`를 붙인다.**
- 서버 컴포넌트에서 데이터를 미리 가져오려 시도하지 않는다. 서버에는 토큰이 없다.
- `app/layout.tsx`(루트)만 서버 컴포넌트로 두고, Provider들은 별도 클라이언트 컴포넌트로 분리해 감싼다.

### ⚠️ `useSearchParams`는 Suspense 경계가 필요하다 (중요)

Next.js 15에서 `useSearchParams`를 쓰는 컴포넌트는 **`<Suspense>`로 감싸지 않으면 `npm run build`가 프리렌더 단계에서 실패한다.** 개발 서버에서는 통과하다가 빌드에서 터지므로 늦게 발견된다.

해당되는 곳이 둘이다.

- `/oauth/callback` — `?token=` 을 읽는다
- `/todos` — 검색어·필터·페이지를 URL 쿼리로 관리한다

두 페이지 모두 실제 로직을 내부 컴포넌트로 빼고 `<Suspense fallback={<Skeleton />}>`로 감싼다.

### 목록 상태는 URL 쿼리로 관리한다

검색어·완료 필터·페이지 번호는 `useState`가 아니라 **URL 쿼리 파라미터**에 둔다.

```
/todos?page=2&completed=false&keyword=회의
```

- 새로고침해도 상태가 유지되고, 뒤로가기가 자연스럽게 동작하며, 링크 공유가 된다.
- 쿼리 키(`['todos', {...}]`)가 URL 상태와 1:1로 대응해 캐시가 명확해진다.
- 이 선택 때문에 위의 Suspense 경계가 **선택이 아니라 필수**가 된다.

### ⚠️ 라우트 보호는 middleware로 하지 않는다 (중요)

토큰을 **localStorage**에 두기로 했으므로, Next.js middleware로 라우트를 보호할 수 없다. middleware는 서버에서 실행되어 쿠키만 읽을 수 있고 localStorage에는 접근하지 못한다. 관행대로 middleware를 만들면 토큰을 읽지 못해 무한 리다이렉트가 나거나 보호가 전혀 걸리지 않는다.

- **`middleware.ts`를 만들지 않는다.**
- `(main)` 그룹에 클라이언트 레이아웃을 두고 `useAuth`로 인증 여부를 판정한다.
- 판정이 끝나기 전에는 스켈레톤을 보여준다. 서버 렌더 결과와 어긋나지 않도록 인증 상태는 마운트 이후에만 읽는다.
- 미인증이면 `router.replace("/login")`.

#### ⚠️ `useAuth`는 토큰 존재 여부가 아니라 `exp`를 봐야 한다 (중요)

**"localStorage에 토큰 문자열이 있는가"만 검사하면 만료된 토큰이 판정을 통과한다.**

그러면 이런 순서가 된다 — 보호 레이아웃이 인증으로 판정 → 화면 렌더 → API 호출 → **401** → `apiClient`가 자동 로그아웃 → `/login`. 그 왕복 동안 **보호된 화면이 사용자에게 노출된다.** `AUTH-07`("토큰이 없거나 **만료된** 상태로 보호된 화면에 접근하면 로그인 화면으로 이동")의 위반이다.

Access Token 만료는 **30분**이다(`JwtTokenProvider.ACCESS_TOKEN_EXPIRATION` 상수 — 환경변수가 아니다). 만료 자체는 개발 중에도 자주 만나지만, 이 결함은 **"보호 화면이 잠깐 보였다가 로그인으로 튕긴다"는 산발적 증상**으로만 드러나기 때문에 정상적인 세션 만료 동작으로 오해하기 쉽다. 재현하려면 만료된 토큰을 localStorage에 직접 넣고 진입해야 한다.

```ts
// 서명 검증은 서버가 한다. 프론트는 만료 시각만 읽으면 된다.
function isExpired(token: string): boolean {
  try {
    const { exp } = JSON.parse(atob(token.split(".")[1]));
    return typeof exp !== "number" || exp * 1000 <= Date.now();
  } catch {
    return true; // 형식이 깨진 토큰도 만료로 취급
  }
}
```

- 만료로 판정되면 **즉시 토큰을 폐기**하고 미인증으로 처리한다.
- 라이브러리를 추가하지 않는다. `atob` + `JSON.parse`로 충분하다.

### React Query 쿼리 키 규약 (필수)

낙관적 업데이트의 캐시 수정·롤백 대상이 명확해야 하므로 키를 고정한다.

```ts
["todos", { page, size, completed, keyword }][("todos", id)][("auth", "me")]; // 목록 // 단건 // 내 정보
```

- 목록 캐시를 조작할 때는 **현재 화면의 필터·페이지가 포함된 키**를 대상으로 한다.
- `onSettled`의 `invalidateQueries`는 `['todos']` 접두사로 걸어 관련 캐시를 함께 갱신한다.

### 낙관적 업데이트 (React Query)

완료 토글·삭제는 서버 응답을 기다리지 않고 즉시 UI에 반영한다.

- `onMutate`: `cancelQueries` → 스냅샷 저장 → 캐시 직접 수정
- `onError`: 롤백 + 토스트 알림(`sonner`)
- `onSettled`: `invalidateQueries`
- 토글은 목표 상태를 그대로 서버에 보낸다(§5 참조). 클라이언트가 계산한 값과 서버 값이 일치한다.

#### ⚠️ 멱등성만으로는 연타를 막지 못한다 (중요)

§5에서 toggle을 "바디로 목표 상태를 받는" 멱등 설계로 정한 것은 맞다. 그러나 **멱등성(같은 요청을 두 번 보내도 결과가 같다)과 순서 무관성(요청 순서가 뒤바뀌어도 최종 상태가 같다)은 다른 성질이다.** 목표 상태 전송은 앞의 것만 보장한다.

**React Query v5의 mutation은 기본적으로 병렬 실행된다.** 사용자가 3연타하면 요청 3건이 동시에 in-flight 상태가 되고, 네트워크 재정렬로 서버 도착 순서가 뒤바뀌면 서버 최종값이 사용자가 마지막에 의도한 값과 달라진다. `invalidateQueries`가 결국 서버 값으로 수렴시키므로 "UI와 서버가 어긋난 채 남지는" 않지만, **화면이 사용자 의도와 반대로 되돌아가는 깜빡임**이 생긴다.

둘 중 하나로 처리한다.

```ts
// 방법 1 — 항목별로 mutation을 직렬화한다. 서로 다른 항목은 병렬 유지.
useMutation({ scope: { id: `todo-toggle-${todoId}` }, ... })

// 방법 2 — 마지막 mutation이 끝났을 때만 재조회한다.
onSettled: () => {
  if (queryClient.isMutating({ mutationKey: ["todo-toggle"] }) === 1) {
    queryClient.invalidateQueries({ queryKey: ["todos"] });
  }
}
```

> `scope`를 쓰면 `onMutate`가 큐에서 꺼내질 때 실행되어 "클릭 즉시 반영"이 지연될 수 있다. Phase 9에서 실제로 눌러보고 지연이 체감되면 방법 2로 바꾼다.

### 로딩 / 빈 상태 / 에러

- **로딩**: 스피너 대신 스켈레톤. 목록은 항목형 스켈레톤 3개
- **빈 상태**: 아이콘 + "아직 할 일이 없어요" + CTA 버튼
- **검색 결과 없음**: 빈 상태와 문구를 구분
- **에러**: 인라인 에러 카드 + 재시도 버튼. 401은 로그인으로 이동

### 경계 상황 처리

- **마지막 항목 삭제로 현재 페이지가 비는 경우**: `page > 0`이고 삭제 후 항목이 0개면 **이전 페이지로 이동**한다. 빈 상태 문구를 띄우지 않는다.
  > ⚠️ **페이지 이동은 `onMutate`가 아니라 `onSuccess`에서 한다.** 낙관적 제거로 화면은 즉시 비지만 URL은 그대로 둔다. `onMutate`에서 이동해 버리면 쿼리 키가 바뀌어, 삭제가 실패했을 때 `onError`의 롤백이 **사용자가 보고 있지 않은 캐시에 적용된다.** 사용자는 다른 페이지에서 토스트만 보고 무엇이 되돌아갔는지 알 수 없다 — `TODO-13`("UI를 이전 상태로 되돌린다") 위반이다.
- **`/todos/[id]`, `/todos/[id]/edit`에서 404**: 없는 id이거나 타인 소유이면 서버가 404를 준다. 두 라우트 모두 목록으로 리다이렉트하지 않고 **"할 일을 찾을 수 없습니다" 화면과 목록으로 가기 버튼**을 보여준다. Next.js `notFound()`는 쓰지 않는다(클라이언트 데이터 페칭이므로).
- **검색 중 항목 삭제**: 검색 결과 캐시에서만 제거하고, `invalidateQueries`로 다른 캐시를 갱신한다.

### ⚠️ 이탈 확인 대화상자는 두 계층으로 나눠 구현한다 (중요)

`TODO-16`("변경 사항이 있는 상태로 이탈 시 확인")은 **한 줄짜리 작업이 아니다.** App Router에는 Pages Router의 `router.events`가 없고, **공식 내비게이션 차단 API도 없다.** `beforeunload` 하나로 끝난다고 가정하면 앱 내부 이동이 전혀 막히지 않는다.

| 이탈 경로                             | 방어 수단                                                  |
| ------------------------------------- | ---------------------------------------------------------- |
| 새로고침 · 탭 닫기 · 주소창 직접 이동 | `beforeunload` 이벤트                                      |
| 페이지 내 "취소"·"목록으로" 버튼      | **버튼 자체 핸들러**에서 확인 후 `router.push`             |
| 브라우저 뒤로가기                     | `popstate` 리스너 + 취소 시 `history.pushState`로 되돌리기 |

- **`next-navigation-guard` 같은 서드파티를 도입하지 않는다.** 3장 스택에 없다.
- 폼이 `dirty`일 때만 가드를 켠다. 저장 직후에는 반드시 해제해, 저장하고 나가는데도 확인창이 뜨는 일이 없게 한다.
- 이 항목은 Phase 8에서 **별도 공수(4~8시간)**로 잡는다.

### 페이지네이션 공용 컴포넌트

`src/components/common/Pagination.tsx`

- Props: `currentPage`, `totalPages`, `onPageChange`
- 현재 페이지 주변 5개 + 처음/이전/다음/마지막
- 페이지 수 1 이하면 렌더링하지 않음
- 모바일은 "3 / 12" 형태로 축약

---

## 10. 코딩 컨벤션

### Java

- 클래스 `PascalCase`, 메서드/변수 `camelCase`, 상수 `UPPER_SNAKE_CASE`
- DTO 네이밍: `TodoCreateRequest`, `TodoResponse`, `PageResponse<T>`, `ApiResponse<T>` — **record** 사용
- **엔티티를 컨트롤러에서 직접 반환하지 않는다.** 항상 DTO로 변환
- 엔티티에 `@Setter` 금지. 변경은 의미 있는 메서드로 (`updateCompleted(boolean)`, `softDelete()`)
- 생성자 주입 + `@RequiredArgsConstructor`
- Service에 `@Transactional`, 조회는 `readOnly = true`

### TypeScript

- 컴포넌트 `PascalCase`, 훅 `useCamelCase`
- 타입은 `src/types/`에 모아 백엔드 DTO와 이름을 맞춘다
- `any` 금지. 불가피하면 `unknown` + 타입 가드
- 서버 상태는 React Query, UI 상태만 `useState`
- API 호출은 `src/lib/apiClient.ts`로 통일 (토큰 주입, **`ApiResponse` 언래핑**, 401 처리)

### 공통

- **모든 주석은 한글로 작성한다.**

---

## 11. 에러 처리

`exception/` 패키지에 `BusinessException`(+ `ErrorCode` enum)과 `GlobalExceptionHandler`(`@RestControllerAdvice`)를 구현한다.

| 예외                          | 상태 | 코드                     |
| ----------------------------- | ---- | ------------------------ |
| 유효성 검증 실패              | 400  | `INVALID_INPUT`          |
| 인증 실패(자격 증명 불일치 등) | 401  | `UNAUTHORIZED`           |
| **Access Token 만료**         | 401  | **`TOKEN_EXPIRED`**      |
| **Refresh Token 무효·재사용** | 401  | **`INVALID_REFRESH_TOKEN`** |
| 권한 없음                     | 403  | `FORBIDDEN`              |
| 리소스 없음 / 소유자 불일치   | 404  | `TODO_NOT_FOUND`         |
| **첨부 없음 / 소유자 불일치** | 404  | **`ATTACHMENT_NOT_FOUND`** |
| 이메일 중복 (회원가입)        | 409  | `EMAIL_DUPLICATED`       |
| **첨부 상태 충돌 (재업로드 등)** | 409 | **`ATTACHMENT_INVALID_STATE`** |
| 서버 오류                     | 500  | `INTERNAL_ERROR`         |

> **v1.13 추가**: 첨부도 Todo와 동일하게 **소유권 위반을 404로** 응답한다(존재 여부 노출 방지). 403을 쓰지 않는다. 파일 크기·타입 위반은 기존 `INVALID_INPUT`(400), 서명 토큰 무효는 기존 `UNAUTHORIZED`(401)를 재사용한다.
>
> ⚠️ **`ErrorCode`를 추가하면 프론트도 함께 고쳐야 한다.** `src/types/api.ts`의 `ErrorCode` union과 `src/lib/errorMessages.ts`의 `MESSAGES` 맵 두 곳이다. 백엔드만 바꾸면 타입이 어긋난다.

> **v1.9 추가**: `TOKEN_EXPIRED`는 프론트가 `/auth/refresh`를 자동 시도하는 신호로 쓰고(6장), 그 외 `UNAUTHORIZED`는 즉시 로그아웃 처리한다. 이 둘을 같은 코드로 묶으면 프론트가 매번 불필요한 refresh 시도를 하게 된다. `INVALID_REFRESH_TOKEN`은 `/auth/refresh` 자체가 거부될 때만 쓴다(`PRD.md` 5.1).

- **OAuth 계정 충돌은 이 표에 없다.** REST 응답이 아니라 `?error=email_conflict` 쿼리를 붙인 302 리다이렉트로 처리하므로 에러 코드를... (10KB 남음)
