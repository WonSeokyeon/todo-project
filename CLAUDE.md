# Todo List 프로젝트 개발 가이드

> **버전** 1.8 · **최종 수정** 2026-08-28
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
│       │   ├── application.yml
│       │   ├── application-local.yml
│       │   └── application-prod.yml
│       └── test/
│           ├── java/com/example/
│           └── resources/application-test.yml
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
        │       └── [id]/page.tsx     # 상세(편집)
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
| 에디터     | Tiptap                                                               |
| 토스트     | **shadcn/ui `sonner`**                                               |
| 날짜       | **date-fns** (shadcn Calendar 의존)                                  |
| HTML 정화  | **DOMPurify**                                                        |

### 인프라

AWS Amplify(프론트), EC2(백엔드), RDS(PostgreSQL) · Git/GitHub

> **S3는 MVP 범위에서 제외한다.** 파일 첨부가 비목표이므로 사용처가 없다. 프론트 정적 자산은 Amplify가 처리한다.

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

- `application.yml`에 `spring.jpa.properties.hibernate.jdbc.time_zone: UTC`를 설정한다.
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

| Method | Endpoint                       | 설명                                    | 인증 |
| ------ | ------------------------------ | --------------------------------------- | ---- |
| POST   | `/api/v1/auth/signup`          | 회원가입 (email, password, nickname)    | X    |
| POST   | `/api/v1/auth/login`           | 로그인 → JWT 반환                       | X    |
| GET    | `/api/v1/auth/me`              | 내 정보 조회                            | O    |
| GET    | `/oauth2/authorization/google` | 구글 로그인 시작 (Spring Security 제공) | X    |

> **로그아웃은 서버 API를 만들지 않는다.** Refresh Token과 토큰 블랙리스트가 없으므로, 프론트에서 localStorage의 토큰을 제거하고 React Query 캐시를 비운 뒤 `/login`으로 이동하는 것으로 처리한다.

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

---

## 6. 인증 설계

### 토큰 정책

- Access Token만 사용 (**24시간 만료**), Refresh Token 없음
- 만료 시 401 → 프론트에서 로그인 페이지로 리다이렉트
- 저장 위치: **localStorage** (MVP 기준)
- 요청 시 `Authorization: Bearer {token}`

### JWT 클레임 구성

```
sub    : user.id (숫자 문자열)
email  : user.email
iat    : 발급 시각
exp    : 발급 + 24시간
```

- **`sub`는 이메일이 아니라 id를 담는다.** 이메일 변경 기능이 없어도 id 기반이 조회에 유리하고, 인증 필터에서 PK 조회로 끝난다.
- `JwtAuthenticationFilter`는 `sub`를 파싱해 사용자 id를 얻고, `deleted_at IS NULL` 조건으로 조회한다.

### SecurityConfig 인가 경로 (필수)

```
permitAll:
  /api/v1/auth/signup
  /api/v1/auth/login
  /oauth2/**
  /login/oauth2/**
  /swagger-ui/**
  /v3/api-docs/**
  /error

그 외: authenticated
```

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
- 쿠키를 쓰지 않으므로 `allowCredentials`는 false

### 구글 OAuth2 흐름

1. 프론트가 `/oauth2/authorization/google`로 이동
2. 구글 인증 후 백엔드 콜백 처리
3. 분기
   - 신규 이메일 → `provider=GOOGLE`로 가입
   - 기존 GOOGLE 계정 → 조회
   - **동일 이메일의 LOCAL 계정 존재 → 거부**
4. 성공 시 JWT 생성 후 `{FRONTEND_URL}/oauth/callback?token=xxx`로 **302 리다이렉트**
5. 프론트 콜백 페이지가 토큰 저장 → URL에서 제거 → `/todos`로 이동

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
p, br, strong, em, h2, h3, ul, ol, li, a, code, pre, blockquote
```

- `a`는 `href`만 허용하고, **`rel="noopener noreferrer"`와 `target="_blank"`를 정화 단계에서 강제 주입**한다 (tabnabbing 방지).
- `href`는 `http`, `https`, `mailto` 스킴만 허용한다. `javascript:` 차단.
- `script`, `iframe`, `style` 태그와 모든 `on*` 속성, `style` 속성은 제거한다.
- 직접 정규식으로 구현하지 않는다. Jsoup Safelist를 사용한다.
- **`Jsoup.clean(html, "", safelist, outputSettings)` 오버로드를 쓰고 `new Document.OutputSettings().prettyPrint(false)`를 넘긴다.** 기본 pretty-print가 블록 요소를 재포맷해 **코드 블록(`pre`)의 공백과 줄바꿈이 망가진다.**

**렌더 시 (프론트)** — `lib/sanitize.ts`에서 DOMPurify로 한 번 더 정화한다. 서버를 신뢰하더라도 이중 방어를 유지한다.

#### ⚠️ 정화의 적용 지점은 `dangerouslySetInnerHTML`이 아니다 (중요)

상세 화면을 **"진입 즉시 편집 가능, 보기 모드 없음"**(7장)으로 확정했기 때문에, **이 앱에는 `dangerouslySetInnerHTML` 호출 지점이 하나도 없다.** 본문 HTML은 오직 Tiptap 에디터로만 들어간다. "`dangerouslySetInnerHTML` 사용 전에 정화한다"고만 적으면 **적용 대상이 없어 이중 방어가 실질적으로 사라진다.**

따라서 적용 지점을 다음으로 고정한다.

```ts
// 서버에서 받은 본문을 에디터에 주입하기 직전에 정화한다
editor.commands.setContent(sanitizeHtml(todo.content));
```

- **`editor.commands.setContent()` 호출 직전**에 `lib/sanitize.ts`를 반드시 거친다.
- 향후 목록 미리보기 등 `dangerouslySetInnerHTML`을 쓰는 화면이 생기면 그곳도 동일하게 거친다.

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
  ],
  ALLOWED_ATTR: ["href", "target", "rel"], // target·rel을 빼면 서버가 주입한 값이 렌더에서 지워진다
});
```

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
| `/todos/[id]`     | 상세 (항상 편집 가능)         | O    |

- 미인증 상태로 보호된 경로 접근 시 `/login`으로 리다이렉트.
- 인증 화면의 공통 헤더에는 **닉네임과 로그아웃 버튼**을 둔다.

> **상세 화면은 보기/편집 모드를 나누지 않는다.** 진입 즉시 모든 필드가 편집 가능한 상태이며, 자동 저장 없이 "저장" 버튼을 눌러야 반영된다. `/todos/new`와 **`TodoForm` 컴포넌트를 재사용**하고, 초기값 유무와 삭제 버튼 노출로만 구분한다. 변경 사항이 있는 상태에서 이탈하려 하면 확인 대화상자를 띄운다.

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

**툴바**: 굵게(`strong`) · 기울임(`em`) · 제목 H2 · 제목 H3 · 불릿 목록 · 번호 목록 · 링크 · 인라인 코드 · 코드 블록 · 인용

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

- `/login`, `/signup`, `/oauth/callback`, `/todos`, `/todos/new`, `/todos/[id]`의 `page.tsx`에 **`"use client"`를 붙인다.**
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

`JWT_EXPIRATION`이 24시간이라 개발 중에는 만료를 만나기도 어려워, 이 결함은 배포 후에 발견되기 쉽다.

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
- **`/todos/[id]`에서 404**: 없는 id이거나 타인 소유이면 서버가 404를 준다. 프론트는 목록으로 리다이렉트하지 않고 **"할 일을 찾을 수 없습니다" 화면과 목록으로 가기 버튼**을 보여준다. Next.js `notFound()`는 쓰지 않는다(클라이언트 데이터 페칭이므로).
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

| 예외                        | 상태 | 코드               |
| --------------------------- | ---- | ------------------ |
| 유효성 검증 실패            | 400  | `INVALID_INPUT`    |
| 인증 실패 / 토큰 만료       | 401  | `UNAUTHORIZED`     |
| 권한 없음                   | 403  | `FORBIDDEN`        |
| 리소스 없음 / 소유자 불일치 | 404  | `TODO_NOT_FOUND`   |
| 이메일 중복 (회원가입)      | 409  | `EMAIL_DUPLICATED` |
| 서버 오류                   | 500  | `INTERNAL_ERROR`   |

- **OAuth 계정 충돌은 이 표에 없다.** REST 응답이 아니라 `?error=email_conflict` 쿼리를 붙인 302 리다이렉트로 처리하므로 에러 코드를... (10KB 남음)
