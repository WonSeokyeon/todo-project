# ROADMAP — Todo List 프로젝트

> **버전** 1.8 · **최종 수정** 2026-08-28
> 이 문서는 "어떤 순서로 만드는가"를 정의하며, **완료 판정의 정본**이다.
> **한 번에 전체를 생성하지 않는다.** Phase 단위로 진행하고, 각 Phase의 DoD를 모두 만족한 뒤 다음으로 넘어간다.
> 기술 규칙은 `CLAUDE.md`, 기능 정의는 `PRD.md` 참조.

---

## 진행 현황

| Phase | 내용                       | 저장소   | 상태 |
| ----- | -------------------------- | -------- | ---- |
| 0     | 저장소 초기화              | 전체     | 🟡   |
| 1     | 백엔드 스캐폴딩            | backend  | 🟡   |
| 2     | 도메인 & DB                | backend  | ⬜   |
| 3     | 인증 (로컬) + 인증 테스트  | backend  | ⬜   |
| 4     | Todo API + Todo 테스트     | backend  | ⬜   |
| 5     | 구글 OAuth2 + OAuth 테스트 | backend  | ⬜   |
| 6     | 프론트 스캐폴딩            | frontend | 🟡   |
| 7     | 인증 화면                  | frontend | ⬜   |
| 8     | Todo 화면                  | frontend | ⬜   |
| 9     | 인터랙션 다듬기            | frontend | ⬜   |
| 10    | 전체 검증                  | 전체     | ⬜   |
| 11    | AWS 배포                   | 전체     | ⬜   |

⬜ 대기 · 🟡 진행중 · ✅ 완료

> ### 스캐폴딩 정합성 점검 (2026-08-28)
>
> 스캐폴딩이 스펙과 어긋난 채 진행되어 아래를 바로잡았다. **해결됨** 항목은 재발 방지 DoD가 각 Phase에 들어가 있다.
>
> | 항목                                   | 발견 당시                                                                       | 결과                                                                                                                                                                                         |
> | -------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | `todo-frontend`가 Next.js 16.3.3       | Amplify SSR 지원 범위(12~15) 밖이라 Phase 11 배포 불가 상태였음                 | ✅ **15.5.24로 마이그레이션.** `next`·`eslint-config-next` 동시 적용, `npm run build`·`lint` 통과                                                                                            |
> | 프론트에 `src/` 없음                   | `app/`·`components/`·`lib/`가 루트 직하                                         | ✅ **`src/` 아래로 이동.** `tsconfig` paths(`@/*` → `./src/*`), `components.json`의 css 경로 동시 수정                                                                                       |
> | `layout.tsx`가 `LayoutProps<"/">` 사용 | Next 16 전역 타입이라 15에서 컴파일 실패                                        | ✅ 명시적 `{ children: React.ReactNode }`로 교체                                                                                                                                             |
> | `eslint.config.mjs`가 16 방식          | 15의 `eslint-config-next`는 flat config를 직접 내보내지 않음                    | ✅ `FlatCompat` 방식으로 교체                                                                                                                                                                |
> | `AGENTS.md`                            | Next 16의 `next dev`가 자동 생성한 파일. 15에서는 재생성되지 않고 내용도 부정확 | ✅ 삭제. 저장소용 `CLAUDE.md`를 `@../CLAUDE.md` 임포트 형태로 재작성                                                                                                                         |
> | 워크스페이스 루트 오인                 | 부모 `todo-project`에도 `package-lock.json`이 있어 Next가 루트를 잘못 추론      | ✅ `next.config.ts`에 `outputFileTracingRoot` 지정. **원인이던 루트 npm 파일도 이후 삭제**했으나, 재발 시 조용히 어긋나므로 설정은 유지한다                                                  |
> | `pom.xml`에 `springdoc` 없음           | Phase 1 DoD "Swagger UI 접속" 불가                                              | ✅ **3.1.0 핀.** 이 버전의 부모가 `spring-boot-starter-parent` 4.1.0이라 Boot 4.1.x에 대응                                                                                                   |
> | `pom.xml`에 `jsoup` 없음               | Phase 4 XSS 정화 불가                                                           | ✅ **1.23.2 핀**                                                                                                                                                                             |
> | JWT 라이브러리 미선정                  | `CLAUDE.md` 3장 표에 JWT 라이브러리가 명시되어 있지 않음                        | ✅ **jjwt 0.12.6**(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 `pom.xml`에 핀되어 있고, **`CLAUDE.md` 3장 스택 표와 「버전 관련 확정 사항」에 등재됐다**(v1.8). Phase 3은 이 버전을 그대로 쓴다 |
> | `todo-backend/.gitattributes`          | Git Bash에서 `./mvnw` `bad interpreter` 위험                                    | ✅ 존재함 (`/mvnw text eol=lf`, `*.cmd text eol=crlf`)                                                                                                                                       |
> | 백엔드 버전                            | Spring Boot 4.1.1 + `java.version` 21                                           | ✅ 스펙 일치. 변경 없음                                                                                                                                                                      |
>
> #### 아직 남은 것 (2026-08-28 재확인)
>
> | 항목                     | 현재 상태                                                                                                                                                                                                                                                                                                                         | 처리 Phase |
> | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
> | 커밋·원격                | `todo-project` 커밋 0건, `todo-backend` 커밋 0건, `todo-frontend` 커밋 1건(`Initial commit from Create Next App`). 세 저장소 모두 원격 미연결                                                                                                                                                                                     | Phase 0    |
> | 브랜치명                 | 세 저장소 모두 `master` (루트·백엔드는 커밋이 없어 unborn `master`). `CLAUDE.md` 2장 Git 전략은 `main` ← `develop` ← `feature/{작업명}`                                                                                                                                                                                           | Phase 0    |
> | 루트 `.gitignore`        | `todo-backend/`, `todo-frontend/`, `.DS_Store`, `*.log`뿐. `node_modules/` 규칙이 없다. **유입원(루트 npm 파일)은 제거됐으므로 재발 방지용 한 줄만 남았다**                                                                                                                                                                       | Phase 0    |
> | 백엔드 `.gitignore`      | Spring Initializr 기본값 그대로라 **`.env*` 규칙 자체가 없다.** `.env`를 만들면 그대로 커밋된다                                                                                                                                                                                                                                   | Phase 0    |
> | 프론트 `.gitignore`      | `.env*`는 있으나 **`!.env.example` 예외 줄이 없다.** 예시 파일까지 무시된다                                                                                                                                                                                                                                                       | Phase 0    |
> | ~~문서 경로~~            | ✅ **해소.** `docs/`를 정본으로 확정하고 `CLAUDE.md` 2장 구조도와 상단 참조 경로를 정정했다 (v1.8)                                                                                                                                                                                                                                | —          |
> | ~~루트 npm 파일~~        | ✅ **해소.** 루트 `package.json`·`package-lock.json`·`node_modules/`를 삭제했다. `shadcn`은 `todo-frontend/package.json`에 이미 있어 기능 손실이 없고, `.mcp.json`의 shadcn 서버는 `npx shadcn@latest mcp`라 루트 설치에 의존하지 않는다                                                                                          | —          |
> | ~~`docs/guides/`~~       | ✅ **해소.** 5개 전부 다른 프로젝트에서 넘어온 문서였고(존재하지 않는 "PRD 1.3 기술 스택" 참조, 모노레포·Next 16·없는 npm 스크립트 전제), **이 프로젝트 기준으로 전부 재작성**했다. `nextjs-16.md`→`nextjs-15.md`, `forms-react-hook-form.md`→`forms.md`로 개명. `README.md`를 추가해 "참고 자료이며 `CLAUDE.md`가 우선"임을 명시 | —          |
> | ~~폼 라이브러리 미결정~~ | ✅ **해소.** `CLAUDE.md` 3장에 **"라이브러리를 쓰지 않는다 — `useState` + 수동 검증"**으로 확정. `npx shadcn add form` 금지(=`react-hook-form` 유입 경로)와 Tiptap dirty 판정 주의를 함께 명시                                                                                                                                    | —          |
> | 백엔드 설정 파일         | `src/main/resources/application.properties` 하나뿐. `application.yml` + `-local` + `-prod` 분리 미완                                                                                                                                                                                                                              | Phase 1    |
> | 백엔드 문서·예시         | `.env.example`, 저장소용 `CLAUDE.md` 없음                                                                                                                                                                                                                                                                                         | Phase 1    |
> | 프론트 마감 작업         | 디자인 토큰이 shadcn 기본 neutral, 다크 모드가 `@custom-variant dark (&:is(.dark *))`(= `class` 전략), `components.json` 스타일이 `radix-nova`, `.env.example` 없음, Tiptap·motion·sonner·date-fns·DOMPurify 미설치                                                                                                               | Phase 6    |

> **테스트는 마지막에 몰아 쓰지 않는다.** 기능을 만든 Phase에서 함께 작성해 그 Phase의 DoD로 삼는다. Phase 10은 새 테스트를 쓰는 단계가 아니라 전체를 확인하는 단계다.

---

## 요구사항 ↔ Phase 추적표

> `PRD.md` 3장의 P0 요구사항이 **어느 Phase에서 구현되고 어느 Phase에서 검증되는지**의 정본이다.
> **여기에 행이 없는 P0는 구현되지 않는다.** `PRD.md` 3장에 요구사항을 추가하면 이 표에 먼저 행을 넣고, 해당 Phase의 작업·DoD에 실제로 기술한다.

| ID                                       | 구현 Phase                                                    | 검증 Phase (DoD)                                 |
| ---------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------ |
| AUTH-01 회원가입                         | 3(API) · 7(화면)                                              | 3 · 7 · 10                                       |
| AUTH-02 비밀번호 6자 이상 + 72바이트     | 3(바이트 validator) · 7(실시간 검증)                          | 3(한글 25자 → 400) · 7(제출 전 인라인 안내) · 10 |
| AUTH-03 이메일 중복                      | 3(409) · 7(인라인 문구)                                       | 3 · 7                                            |
| AUTH-04 로그인·JWT 24h                   | 3 · 7                                                         | 3 · 7 · 10                                       |
| AUTH-05 구글 로그인                      | 5 · 7(`/oauth/callback`)                                      | 5 · 7 · 10                                       |
| AUTH-06 로그아웃                         | 7 (프론트 전용, 서버 API 없음)                                | 7 · 10                                           |
| AUTH-07 라우트 보호                      | 7 (`(main)` 클라이언트 레이아웃)                              | 7(만료 토큰 케이스) · 10                         |
| AUTH-08 헤더 닉네임 / 이메일 미표시      | 3(`/auth/me` 응답) · 7(헤더)                                  | 3(응답 필드) · 7(DOM에 이메일 없음) · 10         |
| AUTH-09 계정 충돌 거부                   | 5 · 7(안내 문구)                                              | 5 · 7 · 10                                       |
| TODO-01 제목 필수·200자                  | 4 · 8                                                         | 4 · 10                                           |
| TODO-02 Tiptap 본문                      | 4(정화·50,000자) · 8(에디터)                                  | 4 · 8 · 10                                       |
| TODO-03 우선순위                         | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-04 마감일                           | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-05 생성일 내림차순                  | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-06 10건 페이지네이션                | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-07 완료 필터                        | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-08 제목 검색                        | 4 · 8                                                         | 4 · 8 · 10                                       |
| TODO-09 상세(항상 편집)                  | 4 · 8                                                         | 8 · 10                                           |
| TODO-10 저장(완료 상태 불변)             | 4(PUT에 `completed` 없음) · 8                                 | 4 · 8 · 10                                       |
| TODO-11 완료 토글 즉시 반영              | 4(멱등 toggle) · 9(낙관적 업데이트)                           | 4 · 9(연타 수렴값) · 10                          |
| TODO-12 삭제 즉시 반영 / Soft Delete     | 4 · 8(상세→목록 이동) · 9(낙관적 제거)                        | 4 · 8 · 9 · 10                                   |
| TODO-13 실패 시 롤백·알림                | **8(저장 실패: 폼 유지 + 에러)** · 9(토글·삭제 롤백 + 토스트) | 8 · 9 · 10                                       |
| TODO-14 타인 리소스 차단                 | 4(404) · 8(전용 화면)                                         | 4 · 8 · 10                                       |
| TODO-15 XSS 이중 방어                    | 4(Jsoup) · 6(`sanitize.ts`) · 8(`setContent` 직전)            | 4 · 8 · 10                                       |
| TODO-16 이탈 확인                        | 8 (3계층)                                                     | 8                                                |
| UX-01 스켈레톤                           | 6(컴포넌트) · 7(인증 판정·콜백) · 8(목록·상세)                | 7 · 8 · 10                                       |
| UX-02 빈 상태                            | 6 · 8                                                         | 8 · 10                                           |
| UX-03 검색 결과 없음                     | 8                                                             | 8 · 10                                           |
| UX-04 에러 + 재시도 버튼                 | 6(`ErrorState.onRetry` 필수) · 7 · 8                          | 6 · 7 · 8(재요청 확인) · 10                      |
| UX-05 반응형 360~1920                    | 6(토큰·컨테이너) · 8                                          | 8(360px) · 10(1920px)                            |
| UX-06 label·키보드                       | 7 · 8                                                         | 7 · 8 · 10                                       |
| UX-07 다크 토큰 (`prefers-color-scheme`) | 6                                                             | 6 · 10                                           |

> `PRD.md` 5.1의 **에러 문구 매핑 표**는 Phase 6에서 `lib/errorMessages.ts`로 단일화하고, Phase 7·8에서 화면별로 적용, Phase 10에서 6종 전부를 대조한다.
> `PRD.md` 7장 비기능 요구사항의 검증 위치(응답 속도 4 · 체감 반응 9 · 브라우저 10 · 반응형 8·10 · 인증 보안 3·4 · 시크릿 관리 10·11 · XSS 4·8 · 문서화 4 · 접근성 7·8)는 위 표 및 각 Phase DoD와 일치한다.

---

## Phase 0 — 저장소 초기화

**목표**: 폴리레포 3개 저장소를 만들고 문서를 자리잡게 한다.

**작업**

- `todo-project/` 생성 후 `git init`
- `CLAUDE.md`, `PRD.md`, `ROADMAP.md` 배치
- `.gitignore`에 `todo-backend/`, `todo-frontend/` 추가 (**필수**)
- **루트 `.gitignore`에 `node_modules/` 추가** — 재발 방지용이다
  > 루트의 `package.json`·`package-lock.json`·`node_modules/`는 **이미 삭제했다.** `shadcn`은 `todo-frontend`에 있고 `.mcp.json`은 `npx`로 받아 쓰므로 문서 저장소에 npm 파일을 두지 않는다. 다만 루트에서 `npx shadcn`을 실수로 실행하면 `node_modules/`가 다시 생기므로 무시 규칙은 남긴다.
- `todo-backend/`, `todo-frontend/` 폴더 생성 후 각각 `git init`
- 각 저장소 `.gitignore`에 `.env*` + **`!.env.example`** (예외 줄이 없으면 예시 파일까지 무시됨)
  > 현재 백엔드는 `.env*` 규칙 자체가 없고, 프론트는 `.env*`만 있고 `!.env.example` 예외가 없다. **양쪽 모두 손봐야 한다.**
- **브랜치 정리** — 세 저장소 모두 `master`다. 첫 커밋 시 `git branch -m main`(또는 `git init -b main`)으로 `main`을 만들고, `main`에서 `develop`을 분기한다. 이후 작업은 `feature/{작업명}` → `develop` → `main` (`CLAUDE.md` 2장)
- ~~문서 경로 확정~~ — ✅ **완료(v1.8).** `docs/`를 정본으로 확정했다. `CLAUDE.md`만 루트에 두는데, Claude Code가 상위 디렉토리를 거슬러 올라가며 자동 로드하는 대상이 `CLAUDE.md`이기 때문이다. 2장 구조도와 상단 참조 경로를 정정했다
- ~~`nextjs-16.md` 처리~~ — ✅ **완료.** 삭제하고 **`docs/guides/nextjs-15.md`로 재작성**했다. 단순 버전 치환이 아니라 이 프로젝트 기준으로 다시 썼다: "Server Components 우선"을 **"클라이언트 컴포넌트 우선"**으로 뒤집고(토큰이 localStorage에 있어 서버 페칭이 불가능), Next 16 전용 내용(`proxy.ts`, `cacheComponents`, 최상위 `typedRoutes`)을 걷어내고, 쓰지 않는 기능(Server Actions·Route Handlers·Parallel/Intercepting Routes·ISR·`notFound()`)을 **금지 표로 명시**했다
- ~~나머지 가이드 4개 처리~~ — ✅ **완료.** 넷 다 이 프로젝트 기준으로 재작성했다. `component-patterns`(서버 우선 → **클라이언트 우선**), `styling-guide`(`next-themes` 토글 → **미디어쿼리 다크 모드**), `project-structure`(모노레포 → **폴리레포**), `forms-react-hook-form.md` → **`forms.md`**(라이브러리 없는 폼). `guides/README.md`를 추가해 지위(참고 자료, `CLAUDE.md` 우선)와 교체 이력을 남겼다
  > ⚠️ 옛 `forms-react-hook-form.md`는 `react-hook-form`·`zod`가 **"이미 설치되어 있다"고 서술**했고, 비밀번호 규칙을 **"6자 이상이 전부"**로 적어 UTF-8 72바이트 한계를 누락하고 있었다. 그대로 뒀다면 Phase 7에서 `AUTH-02`가 조용히 깨졌을 문서다.
- GitHub에 원격 저장소 3개 생성 및 연결

**DoD**

- [ ] 부모 저장소에서 `git status` 시 하위 폴더가 나타나지 않음
- [ ] **부모 저장소에서 `git status` 시 `node_modules/`가 나타나지 않음**
- [ ] **세 저장소 각각에서 `git check-ignore -v .env.example`이 아무것도 반환하지 않고, `git check-ignore -v .env`는 규칙에 매칭됨** (백엔드·프론트 양쪽에서 확인)
- [ ] 세 저장소 모두 현재 브랜치가 `main`이고 `develop`이 존재함 (`master` 없음)
- [ ] 세 저장소 모두 첫 커밋 및 원격 푸시 완료
- [ ] 첫 커밋의 파일 수가 예상 범위 안임 (`git show --stat HEAD | tail -1`로 확인 — 문서 저장소가 수천 건이면 무시 규칙이 빠진 것)
- [ ] `CLAUDE.md` 2장 구조도의 문서 경로가 실제 파일 위치(`docs/`)와 일치함
- [ ] `docs/guides/`가 `README.md` + 재작성된 5개 문서로만 구성됨 (`nextjs-16.md`·`forms-react-hook-form.md` 없음)
- [ ] **`todo-frontend/package.json`에 `react-hook-form`·`zod`·`@hookform/resolvers`·`next-themes`가 없음** (스펙 밖 라이브러리 유입 확인)

---

## Phase 1 — 백엔드 스캐폴딩

**저장소**: `todo-backend`

**작업**

- Spring Initializr 기준 프로젝트 생성 (Spring Boot 4.x, JDK 21, Maven, `com.example`)
- 의존성: Web, Data JPA, Security, Validation, PostgreSQL Driver, OAuth2 Client, Lombok, **Jsoup**(HTML 정화), **jjwt**(JWT 생성·검증)
- **SpringDoc OpenAPI는 `springdoc-openapi-starter-webmvc-ui` 3.x를 명시한다.** 2.8.x는 Spring Boot 3.x 전용이라 기동에 실패한다
- 패키지 골격 생성: `domain / service / controller / dto / config / exception`
- **`application.properties`를 삭제하고 `application.yml`로 교체한다** (현재 저장소에는 `.properties`만 있다). 이어 `application-local.yml` + **`application-prod.yml`** 분리, 환경변수 바인딩 (프로파일별 `ddl-auto`는 `CLAUDE.md` 12장 표 참조)
  > ⚠️ 둘을 동시에 두면 `.properties`가 `.yml`보다 우선 적용되어, `.yml`에 적은 설정이 조용히 무시된다.
- **`application.yml`에 `spring.profiles.active: local` 기본값 지정** (없으면 DB 정보 없이 기동을 시도해 실패)
- `.gitignore`(**`.env*` + `!.env.example` 추가** — 현재 Initializr 기본값이라 `.env` 규칙이 없다), `.env.example`, 저장소용 `CLAUDE.md` 작성 (상단에 `@../CLAUDE.md` 임포트)
- `.gitattributes`는 이미 있다(`/mvnw text eol=lf`). **삭제하거나 덮어쓰지 않는다** (`CLAUDE.md` 13장)

**DoD**

- [ ] `pom.xml`의 SpringDoc이 **현재 Boot 버전에 대응하는 정확한 3.x 버전으로 핀**되어 있음 (범위 지정 금지 — `CLAUDE.md` 3장)
- [ ] `pom.xml`에 **`jsoup`이 포함**되어 있음 (Phase 4의 XSS 정화 전제)
- [ ] `pom.xml`에 **jjwt 3종(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 동일 버전으로 핀**되어 있음 (Phase 3의 `JwtTokenProvider` 전제)
- [ ] `./mvnw dependency:tree` 오류 없음
- [ ] **`src/main/resources`에 `application.properties`가 없고 `application.yml`·`application-local.yml`·`application-prod.yml`이 있음**
- [ ] `./mvnw spring-boot:run`이 옵션 없이 local 프로파일로 기동 성공
- [ ] **기동 로그에 `The following 1 profile is active: "local"`이 찍힘**
- [ ] `http://localhost:8080/swagger-ui/index.html`이 200으로 열리고 API 목록 화면이 렌더됨
- [ ] `git check-ignore -v .env`가 규칙에 매칭되고 `.env.example`은 매칭되지 않음

> SpringDoc 버전은 `CLAUDE.md` 3장에서 3.x로 확정됐다. 다시 조사하거나 2.x로 되돌리지 않는다.

---

## Phase 2 — 도메인 & DB

**저장소**: `todo-backend`

**작업**

- `createdb todolist_db` · `createdb todolist_test`
- `BaseEntity` (`@MappedSuperclass`, JPA Auditing)
- `User`, `Todo` 엔티티 + `Priority` enum + `AuthProvider` enum
- **`Todo.user`는 `@ManyToOne(fetch = FetchType.LAZY)`** (기본값 EAGER 금지)
- **`hibernate.jdbc.time_zone: UTC` 설정** — 로컬 KST와 RDS UTC의 9시간 어긋남 방지
- 입력값 제약을 스키마와 일치시킨다 (`CLAUDE.md` 4장 제약 표)
- `UserRepository`, `TodoRepository` (`deleted_at IS NULL` 조건 포함 쿼리)
- 인덱스 `idx_todos_user_deleted`
- `src/test/resources/application-test.yml` (todolist_test, `ddl-auto: create-drop`)
- **`@EnableJpaAuditing`은 메인 애플리케이션 클래스에 붙인다** (`@Configuration`에 두면 `@DataJpaTest`가 로드하지 않아 `created_at`이 null이 됨)
- Repository 테스트에 **`@AutoConfigureTestDatabase(replace = NONE)` + `@ActiveProfiles("test")`** 필수 (없으면 임베디드 DB로 교체 시도)

**DoD**

- [ ] 애플리케이션 기동 시 `users`, `todos` 테이블 자동 생성
- [ ] `created_at`, `updated_at` 자동 기록 확인
- [ ] Repository 단위 테스트(`@DataJpaTest`) 통과 — **통합 테스트 번호 체계(1~8)와 별개**
- [ ] 테스트가 `todolist_test`를 바라보고 실행됨 (H2 사용하지 않음)
- [ ] `@DataJpaTest`에서 `created_at`이 null이 아님 (Auditing 정상 동작)
- [ ] 저장된 `created_at`이 UTC 기준임 (KST로 9시간 밀리지 않음)

---

## Phase 3 — 인증 (로컬) + 인증 테스트

**저장소**: `todo-backend` · **관련 요구사항**: AUTH-01~04, 07~08 (AUTH-06 로그아웃은 프론트 전용이라 Phase 7)

**작업**

- `SecurityConfig` (SecurityFilterChain, CORS, 경로별 인가)
  - **`permitAll` 경로에 Swagger(`/swagger-ui/**`, `/v3/api-docs/**`)를 반드시 포함** (`CLAUDE.md` 6장 목록 그대로)
  - CORS 허용 헤더에 `Authorization`, `Content-Type` 명시
- `JwtTokenProvider` (생성/검증, 24시간 만료, **`sub = user.id`**)
- `JwtAuthenticationFilter` (`sub` → id 조회, `deleted_at IS NULL` 확인)
- `BCryptPasswordEncoder` 빈
- `AuthController`: signup / login / me (**로그아웃 API는 만들지 않는다**)
- DTO: `SignupRequest`, `LoginRequest`, `TokenResponse`, `UserResponse`
- `GlobalExceptionHandler` + `ErrorCode` + **공통 응답 `ApiResponse<T>`**
- **Swagger `@SecurityScheme`(bearerAuth) 설정** — Authorize 버튼으로 보호된 API 호출 가능하게
- **`AuthenticationEntryPoint` + `AccessDeniedHandler` 구현** — Security 필터 단계의 401/403도 `ApiResponse` 포맷으로 응답 (`GlobalExceptionHandler`는 필터 예외를 잡지 못함)
- **통합 테스트 1~3번 작성**

**DoD**

- [ ] **`POST /api/v1/auth/signup`이 403이 아니라 200/409로 응답** (CSRF 비활성화 확인 — 안 하면 여기서 전부 막힌다)
- [ ] **응답에 `Set-Cookie: JSESSIONID`가 없음** (`STATELESS` 세션 정책 확인)
- [ ] 회원가입 → 사용자 생성 + JWT 반환
- [ ] **한글 25자 비밀번호로 가입 시 500이 아니라 400 `INVALID_INPUT`** (BCrypt 72바이트 한계 — `CLAUDE.md` 4장)
- [ ] 중복 이메일 시 409 `EMAIL_DUPLICATED`
- [ ] 로그인 성공 시 유효 JWT, 실패 시 401
- [ ] **미가입 이메일로 로그인한 401과 비밀번호가 틀린 401의 `code`·`message`가 완전히 동일함** (계정 존재 여부를 구분 노출하지 않는다 — `PRD.md` 5.1)
- [ ] 토큰 없이 `/api/v1/auth/me` 호출 시 401
- [ ] `/api/v1/auth/me` 응답에 `nickname`과 `email`이 모두 포함됨 (AUTH-08 — 화면 표시 여부는 Phase 7에서 통제)
- [ ] 비밀번호가 DB에 해시로 저장됨
- [ ] **Security 도입 후에도 Swagger UI 접속 가능** (Phase 1 DoD 회귀 방지)
- [ ] Swagger Authorize에 토큰을 넣고 `/auth/me` 호출이 200으로 성공
- [ ] 모든 응답이 `{success, data, error}` 포맷을 따름
- [ ] **토큰 없이 호출한 401 응답도** `{success:false, error:{code:"UNAUTHORIZED"}}` 포맷임
- [ ] **인증 통합 테스트 3건 통과**

---

## Phase 4 — Todo API + Todo 테스트

**저장소**: `todo-backend` · **관련 요구사항**: TODO-01~15

**작업**

- `TodoController` 6개 엔드포인트 (목록/생성/단건/수정/토글/삭제)
- **`PUT`은 전체 교체이며 `TodoUpdateRequest`에 `completed`를 넣지 않는다**
- **`PATCH /toggle`은 바디로 `{"completed": true}`를 받는다** (서버가 뒤집지 않음)
- `TodoService`: 소유권 검증, Soft Delete, **HTML 정화**
- **HtmlSanitizer**: Jsoup Safelist, 허용 태그·`rel` 주입·스킴 제한 (`CLAUDE.md` 6장)
- 페이지네이션 + `completed`(미지정 시 전체) + `keyword`(대소문자 무시) 필터
- 정렬은 `createdAt,desc` 고정. **허용 필드 화이트리스트(`createdAt`, `dueDate`) 밖의 값은 기본값으로 대체** (없는 프로퍼티로 500 방지)
- `PageResponse<T>` DTO → **`ApiResponse.data` 안에 담아 반환**
- **`TodoResponse`에 사용자 정보를 넣지 않는다** (본인 데이터만 조회하므로 불필요, 넣으면 N+1)
- 날짜 직렬화: `createdAt`/`updatedAt`은 ISO-8601 UTC, `dueDate`는 `yyyy-MM-dd`
- DTO: `TodoCreateRequest`, `TodoUpdateRequest`, `TodoResponse`
- **개발용 시드 스크립트** — 용도가 둘이므로 파일을 나눈다
  - `db/seed-dev.sql` — 테스트 계정 1개 + Todo **100건** (우선순위·완료·마감일 혼합). 페이지네이션·필터·정렬을 **눈으로** 확인하는 용도
  - `db/seed-perf.sql` — 같은 계정에 Todo **10,000건**. 성능 DoD 측정 전용. 제목에 검색 대상 키워드가 골고루 섞이도록 생성한다
- **통합 테스트 4~7번 작성**

**DoD**

- [ ] 목록 API가 `{success, data:{content, page, ...}, error}` 형태로 응답
- [ ] `PUT` 저장이 완료 상태를 덮어쓰지 않음
- [ ] `toggle`을 같은 값으로 두 번 호출해도 결과가 동일함(멱등)
- [ ] `completed` 미지정 시 전체 반환, `true`/`false` 시 필터 적용
- [ ] 영문 대소문자를 섞어 검색해도 결과가 나옴
- [ ] `?sort=foo,desc` 같은 잘못된 정렬 값에도 500이 나지 않음
- [ ] 삭제 시 `deleted_at` 기록, 목록에서 제외
- [ ] 타 사용자 Todo 접근 시 404
- [ ] 제목 미입력·200자 초과 시 400 + 필드 메시지
- [ ] 본문 50,000자 초과 시 400
- [ ] `<script>` 포함 본문 저장 시 태그 제거, `a` 태그에 `rel` 주입 확인
- [ ] **키워드 검색 포함** 목록 조회가 **워밍업 후 3회 측정 중앙값 500ms 이내** (로컬, 시드 **10,000건** 기준)
  > ⚠️ 시드 100건으로는 이 지표가 의미가 없다. 인덱스가 없어도 100행은 1ms 미만이라 **항상 통과한다.** `CLAUDE.md` 4장이 지목한 유일한 성능 위험(`LOWER(title) LIKE '%키워드%'`의 인덱스 미사용)을 검출하려면 데이터가 충분해야 하고, 측정 대상도 검색 경로여야 한다. 또 첫 요청은 JVM 콜드 스타트라 DB가 아니라 워밍업 상태를 재는 셈이 되므로 워밍업 후에 측정한다.
- [ ] 날짜가 배열이 아닌 ISO 문자열로 직렬화됨
- [ ] 목록 조회 시 user 조회 쿼리가 추가로 발생하지 않음
- [ ] Swagger에서 전체 API 확인 가능
- [ ] **Todo 통합 테스트 4건 통과**

---

## Phase 5 — 구글 OAuth2 + OAuth 테스트

**저장소**: `todo-backend` · **관련 요구사항**: AUTH-05, AUTH-09

**작업**

- Google Cloud Console에서 OAuth 클라이언트 생성, 리다이렉트 URI 등록 (로컬 + 운영 둘 다)
- `spring.security.oauth2.client` 설정
- `CustomOAuth2UserService`: 신규 가입 / 기존 조회 / **충돌 거부** 분기
- **nickname 결정**: 구글 `name` → 없으면 이메일 `@` 앞부분 → 50자 초과 시 절삭
- `OAuth2SuccessHandler`: JWT 발급 후 `{FRONTEND_URL}/oauth/callback?token=` **302 리다이렉트**
- `OAuth2FailureHandler`: 충돌 시 `{FRONTEND_URL}/login?error=email_conflict` **302 리다이렉트** (JSON 에러 응답을 반환하지 않는다)
- **테스트 8번 작성 — 통합 테스트가 아니라 `CustomOAuth2UserService` 단위 테스트로 작성한다.** OAuth2 흐름은 실제 구글 서버와 통신하므로 MockMvc로 끝까지 검증할 수 없다 (`CLAUDE.md` 14장)

**계정 충돌 정책 (확정)**
동일 이메일의 로컬 계정이 있으면 **거부한다.** 자동 연동하지 않고, 별도 계정도 만들지 않는다. 상세는 `CLAUDE.md` 6장.

**DoD**

- [ ] 구글 로그인 후 JWT를 담은 302 리다이렉트 발생
- [ ] 신규 사용자가 `provider=GOOGLE`, nickname이 채워진 상태로 저장됨
- [ ] 재로그인 시 중복 계정이 생기지 않음
- [ ] 동일 이메일 로컬 계정 존재 시 `error=email_conflict`로 302 리다이렉트
- [ ] **OAuth 서비스 단위 테스트 1건 통과**

---

## Phase 6 — 프론트 스캐폴딩

**저장소**: `todo-frontend` · **관련 요구사항**: UX-01·02·04·05·07 (공용 컴포넌트·토큰 수준) · **선행 조건**: Phase 0만 필요하다. **백엔드 Phase 1~5와 병렬로 진행할 수 있다** (이 Phase는 실서버를 호출하지 않는다)

**작업**

- **Node 20 이상** 확인 후 `create-next-app` (**Next.js 15**, App Router, TypeScript)
- Tailwind CSS 4 설정 — `globals.css`의 `@theme`에 디자인 토큰 정의 (**v3 방식 금지**)
- **디자인 토큰은 라이트/다크 양쪽 정의 + `@media (prefers-color-scheme: dark)`** (`class` 전략 금지 — 토글이 없어 FOUC만 생김)
- shadcn/ui 초기화 (**npm 사용 시 `--legacy-peer-deps`**, 스타일 `new-york`), lucide-react 설치
- **`npm install motion`** (`framer-motion` 아님), sonner, date-fns, DOMPurify 설치
- **Tiptap 설치**: `@tiptap/react`, `@tiptap/starter-kit` **두 개만.** `@tiptap/extension-link`는 **설치하지 않는다** — v3 StarterKit에 `Link`가 포함되어 있어 중복 등록이 된다 (`CLAUDE.md` 8장)
- **Pretendard 폰트** — Google Fonts에 없으므로 `.woff2` 파일을 `src/app/fonts/`에 넣고 **`next/font/local`**로 로드 (`next/font/google` 사용 불가)
- React Query Provider, **쿼리 키 규약 상수화** (`CLAUDE.md` 9장), `apiClient` (토큰 주입 + **`ApiResponse` 언래핑** + 에러 정규화 + 401 처리)
- **`lib/errorMessages.ts` — `PRD.md` 5.1 「에러 문구 매핑」 표를 코드로 옮긴다.** `error.code`(`INVALID_INPUT` / `UNAUTHORIZED` / `EMAIL_DUPLICATED` / `TODO_NOT_FOUND` / `INTERNAL_ERROR`)와 **네트워크 실패**를 화면 문구로 변환하는 단일 함수를 둔다
  > ⚠️ 화면마다 문구를 직접 쓰면 Phase 7·8에서 서로 다른 문구가 생겨 매핑 표가 사문화된다. `apiClient`가 던지는 에러를 이 함수 하나로만 문구화한다. 네트워크 실패는 `error.code`가 없으므로 **정규화 단계에서 별도 구분자를 남겨야** 한다.
- **`lib/validation.ts` — `CLAUDE.md` 4장 「입력값 제약」 표를 코드로 옮긴다.** 폼 라이브러리를 쓰지 않기로 확정했으므로(`CLAUDE.md` 3장) 검증이 화면에 흩어지기 쉽다. 이메일 형식·닉네임 1~50자·제목 200자·본문 50,000자와 **비밀번호 6자 이상 + UTF-8 72바이트 이하**를 한곳에 둔다
  > ⚠️ **비밀번호 상한은 문자 수가 아니라 바이트다.** `maxLength={64}` 같은 문자 수 제한만 걸면 한글 25자(=75바이트)가 통과해 서버 BCrypt 단계에서 터진다. `new TextEncoder().encode(v).length`로 센다.
- `lib/sanitize.ts` (DOMPurify 래퍼) — **`ALLOWED_TAGS`·`ALLOWED_ATTR`을 명시한다.** 기본값은 Jsoup 화이트리스트보다 넓고, `ALLOWED_ATTR`에서 `rel`·`target`을 빠뜨리면 서버가 주입한 tabnabbing 방어가 렌더 단계에서 지워진다 (`CLAUDE.md` 6장)
- 공용 컴포넌트: `Pagination`, `EmptyState`, `ErrorState`, `Skeleton`
  - **`ErrorState`는 `onRetry`를 필수 prop으로 받고 재시도 버튼을 항상 렌더한다** (`UX-04`). 선택 prop으로 두면 호출부에서 빠뜨려도 타입 검사가 통과한다
- 루트 레이아웃 + Provider 분리 (루트는 서버 컴포넌트, Provider는 클라이언트 컴포넌트)
- 공통 헤더 **껍데기만** 만든다 (닉네임·로그아웃 자리는 비워둠 — `useAuth`가 없는 시점이므로 Phase 7에서 연결)
- 타입 정의 (`src/types/`) — 백엔드 DTO와 이름 일치, `ApiResponse<T>` / `PageResponse<T>` 포함
- `.env.example`, 저장소용 `CLAUDE.md` (상단에 `@../CLAUDE.md` 임포트)
- **`public/static` 경로를 만들지 않는다** (Amplify 예약 경로)

**DoD**

- [ ] **`package.json`의 `next`와 `eslint-config-next`가 모두 15.x임 (16 아님)** — 둘 중 하나만 확인하면 놓친다
- [ ] **소스가 `src/` 아래에 있음** (`src/app/`, `src/components/`, `src/lib/`, `src/types/`)
- [ ] **`package.json`에 `@tiptap/extension-link`가 없음** (v3 StarterKit 내장)
- [ ] Node 20 이상에서 빌드됨
- [ ] `npm run build` 성공, 출력 디렉토리가 `.next`
- [ ] `package.json`에 `motion`이 있고 `framer-motion`이 없음
- [ ] 디자인 토큰이 OS 다크 설정에 따라 전환됨 (`class` 조작 없이 CSS만으로)
- [ ] 페이지 `page.tsx`에 `"use client"`가 붙어 있음
- [ ] **`components.json`의 `style`이 `new-york`임** (현재 `radix-nova`)
- [ ] **`globals.css`에 `.dark` 클래스 셀렉터나 `@custom-variant dark (&:is(.dark *))`가 없고, 다크 토큰이 `@media (prefers-color-scheme: dark)` 안에 정의되어 있음** (현재 shadcn 기본값이 `class` 전략이라 반드시 걷어내야 한다)
- [ ] 디자인 토큰 값이 `CLAUDE.md` 8장 팔레트와 일치함 (배경 `#FAFAFA`/`#0A0A0A`, 액센트 `#4F46E5`, 우선순위 3색) — shadcn 기본 neutral이 남아 있지 않음
- [ ] Pagination 컴포넌트 단독 동작 확인 (더미 데이터). **페이지 수 1 이하일 때 아무것도 렌더하지 않음**
- [ ] `ErrorState`가 재시도 버튼과 함께 렌더되고, 버튼 클릭이 `onRetry`를 호출함
- [ ] `apiClient`가 `data` 언래핑과 401 처리를 수행함
- [ ] **`apiClient`가 던진 에러를 `lib/errorMessages.ts`에 넣으면 `PRD.md` 5.1 표의 문구가 그대로 나옴** (네트워크 실패 케이스 포함 — 서버를 내리고 확인)

> 버전·설치 방법은 `CLAUDE.md` 3장에서 모두 확정됐다. 이 Phase에서 재조사하지 않는다.

---

## Phase 7 — 인증 화면

**저장소**: `todo-frontend` · **관련 요구사항**: AUTH-01~09, UX-01, UX-06 · **선행 조건**: 백엔드 **Phase 3 완료**(가입·로그인·`/auth/me`), 구글 로그인 DoD는 **Phase 5 완료** 필요. 백엔드를 로컬에서 띄운 상태로 진행한다

**작업**

- `/login`, `/signup`, `/oauth/callback`
- **`/oauth/callback`은 `useSearchParams`를 쓰므로 `<Suspense>`로 감싼다** (없으면 `npm run build` 실패)
- **`/oauth/callback` 세부** (`PRD.md` 5.4)
  - 토큰 저장 → **URL에서 토큰 제거**(히스토리·공유 링크에 남지 않도록) → `/todos` 이동
  - **`token` 파라미터가 없거나 빈 문자열이면 `/login`으로 보낸다**
  - 처리 중에는 스켈레톤만 보여주고 **사용자가 조작할 요소를 두지 않는다**
  - **공통 헤더를 두지 않는다** (인증 처리 중 화면이라 닉네임을 알 수 없다)
- **`/todos` 플레이스홀더 페이지 생성** — 라우트 보호를 검증하려면 대상 페이지가 존재해야 한다 (내용은 Phase 8)
- Phase 6에서 비워둔 **헤더의 닉네임·로그아웃을 `useAuth`에 연결**
  - **이메일은 화면에 표시하지 않는다.** `/auth/me` 응답에는 들어오지만 헤더에는 닉네임만 노출한다 (`AUTH-08`)
- `useAuth` 훅 (로그인 / **로그아웃: 토큰 삭제 + 캐시 초기화** / 현재 사용자)
- **라우트 보호는 `(main)` 클라이언트 레이아웃에서 처리한다. `middleware.ts`를 만들지 않는다** (localStorage는 middleware에서 읽을 수 없음 — `CLAUDE.md` 9장)
  - **인증 판정이 끝나기 전에는 스켈레톤을 보여준다** (`UX-01`, `PRD.md` 5.1)
- 401 응답 시 자동 로그아웃 처리
- `?error=email_conflict` 안내 문구 표시
- **`/signup` 실시간 검증** (`PRD.md` 5.3) — 이메일 형식, **비밀번호 6자 이상 + UTF-8 72바이트 이하**, 닉네임 1~50자. 안내 문구에 **한글 1자 = 3바이트**임을 밝힌다
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다** (`PRD.md` 5.1 매핑 표)
  - `INVALID_INPUT` → 서버가 준 필드별 메시지를 해당 입력 아래 인라인
  - `UNAUTHORIZED`(로그인 시) → "이메일 또는 비밀번호가 올바르지 않습니다."를 폼 상단 인라인
  - `UNAUTHORIZED`(그 외) → 문구 없이 `/login` 이동
  - `EMAIL_DUPLICATED` → "이미 사용 중인 이메일입니다."를 이메일 입력 아래 인라인
  - 네트워크 실패 → "연결에 실패했습니다." + 재시도 버튼

**DoD**

- [ ] 이메일 가입·로그인 정상 동작
- [ ] **가입 화면에서 한글 25자 비밀번호를 입력하면 제출 전에 바이트 초과 안내가 인라인으로 뜸** (서버 400에만 의존하지 않음 — `AUTH-02`)
- [ ] **중복 이메일 가입 시 "이미 사용 중인 이메일입니다."가 이메일 입력 아래 인라인으로 뜸** (`AUTH-03`, `PRD.md` 5.1)
- [ ] **미가입 이메일과 비밀번호 오류의 화면 문구가 동일함** ("이메일 또는 비밀번호가 올바르지 않습니다.") — 계정 존재 여부가 드러나지 않음
- [ ] **백엔드를 내린 채 로그인을 시도하면 "연결에 실패했습니다." + 재시도 버튼이 나옴** (네트워크 실패 매핑 — `UX-04`)
- [ ] 구글 로그인 → 콜백 → `/todos` 이동, URL에서 토큰 제거됨
- [ ] **`/oauth/callback`에 `?token=` 없이 직접 접근하면 `/login`으로 이동함** (`PRD.md` 5.4)
- [ ] **`/oauth/callback` 화면에 공통 헤더와 조작 가능한 요소가 없음**
- [ ] **헤더에 닉네임이 보이고 이메일은 어디에도 렌더되지 않음** (DevTools에서 DOM 검색 — `AUTH-08`)
- [ ] 계정 충돌 시 안내 문구 노출
- [ ] 로그아웃 시 토큰·캐시 모두 제거되고 `/login`으로 이동
- [ ] 새로고침해도 로그인 상태 유지
- [ ] 미인증 상태로 `/todos` 접근 시 로그인으로 이동
- [ ] **만료된 토큰을 localStorage에 직접 넣고 `/todos`에 접근했을 때, 보호 화면이 한 프레임도 노출되지 않고 곧바로 `/login`으로 이동**
  > `useAuth`가 토큰 존재 여부만 보면 만료 토큰이 판정을 통과해, 401 왕복 동안 보호 화면이 노출된다. `exp`를 디코드해야 한다 (`CLAUDE.md` 9장). 검증용 만료 토큰은 `JWT_EXPIRATION`을 일시적으로 낮춰 발급받으면 된다
- [ ] `middleware.ts` 파일이 존재하지 않음
- [ ] `npm run build` 성공 (`useSearchParams` Suspense 경계 확인)
- [ ] **모든 입력에 label 연결, Tab·Enter만으로 가입·로그인 완주 가능**

---

## Phase 8 — Todo 화면

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-01~10, 12~16, UX-01~06 · **선행 조건**: 백엔드 **Phase 4 완료**(Todo API 6종 + 시드). `db/seed-dev.sql`을 로컬에 적용한 상태로 진행해야 페이지네이션·필터 DoD를 눈으로 확인할 수 있다

**작업**

- `/todos`: 목록, 검색, 완료 필터, 페이지네이션
- **검색어·필터·페이지는 URL 쿼리로 관리** (`?page=2&completed=false&keyword=...`)하고, 페이지 전체를 `<Suspense>`로 감싼다
- `useTodos` 훅 (React Query)
- **`TodoForm` 공용 컴포넌트** → `/todos/new`와 `/todos/[id]`가 재사용 (진입 즉시 편집 가능, 명시적 저장)
- **`TodoForm`에 완료 체크박스를 두지 않는다.** 완료는 목록에서만 변경
- **Tiptap 통합 — StarterKit을 기본값으로 쓰지 않는다.** `heading.levels [2,3]`, `strike: false`, `horizontalRule: false`, **`underline: false`**로 설정하고 **`link`는 StarterKit 내장 옵션으로 설정**한다 (v3에서 Link·Underline이 StarterKit에 포함됨 — `CLAUDE.md` 8장)
- **`editor.commands.setContent()` 호출 직전에 `lib/sanitize.ts`로 DOMPurify 정화** — 이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점이다 (`CLAUDE.md` 6장)
- 우선순위 뱃지, 마감일 표시 (date-fns 포맷)
- **완료 항목은 제목에 취소선 + 흐린 색상**을 적용한다 (`PRD.md` 5.5)
- **`/todos/[id]` 저장 실패 처리** — 폼 내용을 유지한 채 에러를 표시한다. 입력을 날리거나 목록으로 튕기지 않는다 (`TODO-13`, `UX-04`, `PRD.md` 5.6)
  > ⚠️ `TODO-13`은 Phase 9의 토글·삭제 롤백만으로 충족되지 않는다. **저장(PUT) 실패 경로는 낙관적 업데이트를 쓰지 않으므로 Phase 9가 손대지 않는다.** 이 Phase에서 별도로 처리한다.
- **`/todos/[id]` 삭제 성공 시 `/todos`로 이동**한다 (`TODO-12`, `PRD.md` 5.6)
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다.** `TODO_NOT_FOUND`는 전체 화면 상태, `INTERNAL_ERROR`는 토스트 또는 에러 카드, 네트워크 실패는 재시도 버튼이 있는 에러 카드 (`PRD.md` 5.1)
- **이탈 확인 대화상자 — 3계층으로 구현** (`beforeunload` + 버튼 핸들러 + `popstate` 가드). App Router에 공식 차단 API가 없어 한 줄로 끝나지 않는다. **별도 공수 4~8시간을 잡는다** (`CLAUDE.md` 9장)
- **`dirty` 판정을 직접 구현한다.** 폼 라이브러리를 쓰지 않으므로 `formState.isDirty`가 없다. 제목·우선순위·마감일은 단순 비교로 끝나지만 **본문은 그렇지 않다** — Tiptap이 HTML을 자기 스키마로 정규화하므로 서버 원본과 `editor.getHTML()`을 직접 비교하면 사용자가 아무것도 고치지 않아도 dirty로 판정된다. **초기 스냅샷은 `setContent()` 직후의 `editor.getHTML()`로 잡는다**(정규화를 거친 값끼리 비교)
- **삭제 실패 시 페이지 이동까지 되돌린다** — 페이지 이동은 `onMutate`가 아니라 `onSuccess`에서 수행 (`CLAUDE.md` 9장)
- **경계 상황**: 마지막 항목 삭제로 페이지가 비면 이전 페이지로 이동, `/todos/[id]` 404 시 전용 화면 (`CLAUDE.md` 9장)
- 로딩 스켈레톤 / 빈 상태 / 검색 결과 없음 / 에러 상태

**DoD**

- [ ] **목록이 페이지당 정확히 10건씩 끊기고, 항목 순서가 생성일 내림차순임** (시드 데이터의 `created_at`과 대조 — `TODO-05`, `TODO-06`)
- [ ] 20건 이상에서 페이지네이션 정상
- [ ] **정렬 기준을 고르는 UI가 화면에 없음** (생성일 내림차순 고정 — `PRD.md` 1장 비목표)
- [ ] 검색·필터 상태가 URL에 반영되고 새로고침·뒤로가기에서 유지됨
- [ ] **완료 처리한 항목의 제목에 취소선과 흐린 색상이 적용됨** (`PRD.md` 5.5)
- [ ] `npm run build` 성공
- [ ] Tiptap 내용 저장 후 재조회 시 서식 유지 (툴바 항목 전부)
- [ ] 본문에 `# `, `~~취소선~~`, `---`를 입력해도 서식이 생성되지 않음 (저장 후 소실되는 입력이 없음)
- [ ] **본문에서 `Ctrl+U`를 눌러도 밑줄(`<u>`)이 생성되지 않음** (v3 StarterKit의 Underline이 꺼져 있는지 확인 — 켜져 있으면 저장 시 서식이 조용히 사라진다)
- [ ] 2페이지 이상에서 마지막 항목 삭제 시 이전 페이지로 이동
- [ ] **2페이지 마지막 항목 삭제가 실패했을 때, 사용자가 보고 있는 화면에서 롤백이 눈으로 확인됨** (페이지가 먼저 넘어가 버려 롤백이 안 보이면 실패)
- [ ] 타인 소유 id로 접근 시 "찾을 수 없습니다" 화면 표시 + **"목록으로 가기" 버튼이 있고, 자동 리다이렉트가 일어나지 않음** (`TODO-14`, `PRD.md` 5.6)
- [ ] `/todos/[id]` 로딩 중 **폼 형태 스켈레톤**이 보임 (`UX-01`)
- [ ] **백엔드를 내린 채 저장을 누르면 입력한 제목·본문이 그대로 남아 있고 에러가 표시됨** (`TODO-13`, `UX-04`)
- [ ] **`/todos/[id]`에서 삭제하면 `/todos`로 이동함** (`TODO-12`)
- [ ] **`setContent()` 직전에 `lib/sanitize.ts`가 호출됨** (본문에 `<script>`가 섞인 데이터를 DB에 직접 넣고 상세 화면 진입 시 실행되지 않음)
- [ ] `/todos/new`와 `/todos/[id]`가 `TodoForm`을 재사용
- [ ] 수정 화면에서 저장해도 완료 상태가 바뀌지 않음
- [ ] 변경 후 이탈 시 확인 대화상자 노출 (**새로고침 / 페이지 내 취소 버튼 / 브라우저 뒤로가기 3경로 모두**)
- [ ] **저장 직후에는 확인 대화상자가 뜨지 않음** (`dirty` 해제 확인)
- [ ] **본문이 있는 할 일을 열어 아무것도 고치지 않고 나갈 때 확인 대화상자가 뜨지 않음** (Tiptap 정규화로 dirty가 오판되지 않는지 — 서식이 섞인 본문으로 시험한다)
- [ ] 4가지 화면 상태 모두 눈으로 확인
- [ ] **에러 상태의 재시도 버튼을 눌렀을 때 실제로 재요청이 나가고, 서버를 다시 올리면 목록이 정상 렌더됨** (`UX-04` — 버튼이 보이기만 하고 동작하지 않는 경우를 걸러낸다)
- [ ] **빈 상태 문구("아직 할 일이 없어요")와 검색 결과 없음 문구가 서로 다름** (`UX-03`)
- [ ] 360px 화면에서 레이아웃 정상 (가로 스크롤 없음)
- [ ] **키보드만으로 할 일 생성·완료 토글·삭제 수행 가능**

---

## Phase 9 — 인터랙션 다듬기

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-11~13 · **선행 조건**: Phase 8 완료(목록·상세 화면이 실제 API로 동작하는 상태). 백엔드 Phase 4의 `toggle` 멱등 동작이 전제다

> `TODO-13` 중 **저장(PUT) 실패 처리는 Phase 8에서 끝낸다.** 이 Phase가 다루는 것은 낙관적 업데이트를 쓰는 **토글·삭제**의 롤백뿐이다.

**작업**

- 완료 토글·삭제 낙관적 업데이트 (`onMutate` / `onError` 롤백 / `onSettled`)
- 토글은 **목표 상태를 그대로 서버에 전송** (서버 계산에 의존하지 않음)
- **연타 대비 — mutation 직렬화 또는 마지막 1회 invalidate.** React Query v5 mutation은 기본 병렬이라 목표 상태 전송(멱등)만으로는 요청 재정렬을 막지 못한다. `scope: { id: \`todo-toggle-${todoId}\` }`또는`onSettled`의 `isMutating() === 1` 가드를 적용한다 (`CLAUDE.md` 9장)
- Motion: 목록 등장(stagger), 삭제(`AnimatePresence`), 토글 스프링 — **import는 `motion/react`**
- 실패 시 토스트 알림(`sonner`)
- `prefers-reduced-motion` 대응

**DoD**

- [ ] 토글·삭제 시 대기 시간 없이 즉시 반영
- [ ] **체크박스를 빠르게 연타해도 새로고침 후 상태가 UI와 일치**
- [ ] **연타를 멈춘 뒤 최종 상태가 "마지막에 클릭한 값"과 일치하며, 잠시 후 반대 값으로 되돌아가지 않음**
  > 앞 항목만 보면 결함이 통과한다. `invalidateQueries`가 어떤 값으로든 수렴시키므로 "UI와 서버가 일치"는 항상 참이 된다. 문제는 **수렴한 값이 사용자 의도와 다를 수 있다는 것**이다
- [ ] 서버를 내린 상태에서 실패 → UI 롤백 + 알림 확인
- [ ] 애니메이션이 200ms 이내, 과하지 않음
- [ ] 애니메이션 관련 import가 모두 `motion/react`에서 이루어짐

---

## Phase 10 — 전체 검증

**저장소**: 전체 · **새 테스트를 작성하지 않는다.** 전체 통과와 아래 체크리스트만 확인한다.

**작업**

- 각 저장소 README 작성
- 전체 검증 체크리스트 수행

### 최종 검증 체크리스트 (완료 판정 정본)

**환경**

- [ ] `todolist_db`, `todolist_test`와 함께 PostgreSQL 실행
- [ ] `./mvnw spring-boot:run` 오류 없이 기동
- [ ] `./mvnw test` 전체 통과 (통합 테스트 8건 + Repository 단위 테스트)
- [ ] `npm run build` 성공
- [ ] Swagger UI에서 전체 API 확인
- [ ] 세 저장소의 브랜치가 `main`/`develop` 체계이고 `master`가 남아 있지 않음
- [ ] `CLAUDE.md`·`PRD.md`·`ROADMAP.md`가 서로를 참조하는 경로가 실제 파일 위치와 일치함

**인증**

- [ ] 회원가입 시 사용자 생성 및 JWT 반환
- [ ] 로그인 시 유효한 JWT 반환 (`sub`에 user id)
- [ ] 보호된 엔드포인트에 유효 토큰 필요
- [ ] 구글 소셜 로그인 정상 동작, nickname 채워짐
- [ ] 동일 이메일 로컬 계정 존재 시 구글 로그인 거부 및 안내
- [ ] 로그아웃 시 토큰·캐시 제거
- [ ] 헤더에 닉네임만 표시되고 이메일은 화면 어디에도 노출되지 않음 (`AUTH-08`)
- [ ] 만료 토큰으로 보호 화면 접근 시 화면 노출 없이 `/login`으로 이동 (`AUTH-07`)

**기능**

- [ ] Todo CRUD가 페이지네이션과 함께 작동
- [ ] 모든 응답이 `{success, data, error}` 포맷 (목록 포함)
- [ ] 완료 필터(미지정 시 전체)·제목 검색(대소문자 무시) 동작
- [ ] 수정 저장이 완료 상태를 덮어쓰지 않음
- [ ] 토글 연타 후에도 서버 상태와 UI 일치
- [ ] Soft Delete 시 `deleted_at` 갱신 및 목록 제외
- [ ] 타 사용자 리소스 접근 시 404
- [ ] Tiptap 저장/렌더링 정상, 우선순위·마감일 반영
- [ ] 낙관적 업데이트 및 실패 롤백 동작

**보안**

- [ ] `<script>` 포함 본문이 저장 시 정화됨
- [ ] 링크에 `rel="noopener noreferrer"` 주입됨 — **저장 시(Jsoup)뿐 아니라 렌더 후 DOM에서도 남아 있는지 확인.** DOMPurify의 `ALLOWED_ATTR`에 `rel`·`target`이 없으면 렌더 단계에서 지워진다
- [ ] **`setContent()` 직전에 DOMPurify가 적용됨** (이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점 — `CLAUDE.md` 6장)
- [ ] **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify `ALLOWED_TAGS` 네 곳의 태그 집합이 일치**
- [ ] 입력값 상한(비밀번호 **UTF-8 72바이트**, 제목 200자, 본문 50,000자) 검증 동작 — **한글 비밀번호로도 시험한다**
- [ ] 시크릿이 저장소에 커밋되지 않음

**UX**

- [ ] 로딩/빈 상태/검색 결과 없음/에러 상태 모두 확인 (**에러 상태의 재시도 버튼이 실제로 재요청을 보냄** — `UX-04`)
- [ ] `PRD.md` 5.1 에러 문구 매핑 6종이 화면에서 표 그대로 나옴 (`INVALID_INPUT` / `UNAUTHORIZED` 2경우 / `EMAIL_DUPLICATED` / `TODO_NOT_FOUND` / `INTERNAL_ERROR` / 네트워크 실패)
- [ ] 로그인 실패 문구가 계정 존재 여부를 구분하지 않음
- [ ] 360px ~ 1920px 반응형 정상
- [ ] **Chromium 계열 1종 + 사용 가능한 다른 엔진 1종에서 확인** (Mac은 Chrome+Safari, Windows는 Chrome+Edge/Firefox)
- [ ] OS 다크 설정에 따라 테마 전환
- [ ] 폼 label 연결 및 키보드 조작 가능

→ 전 항목 통과 시 세 저장소에 `v1.0.0` 태그

---

## Phase 11 — AWS 배포

**저장소**: 전체 · **Docker 사용하지 않음**

### 11-0. 사전 준비

- **도메인 확보** (미보유 시 구입). API용 서브도메인이 필요하다 — 예: 프론트 `todo.example.com`(Amplify), API `api.example.com`(EC2)
- Route 53 호스팅 영역 생성 또는 기존 DNS 제공자에서 레코드 관리 준비

### 11-1. 네트워크 & DB

- VPC 기본 구성, 퍼블릭/프라이빗 서브넷 확인
- **RDS는 프라이빗 서브넷에 배치**하고 퍼블릭 액세스를 비활성화한다
- RDS 보안그룹: 인바운드 5432를 **EC2 보안그룹에서만** 허용 (0.0.0.0/0 금지)
- RDS PostgreSQL 생성 후 `todolist_db` 데이터베이스 생성
- **스키마 적용** (`CLAUDE.md` 4장 절차)
  1. 로컬에서 DDL 스크립트 추출
  2. 검토 후 RDS에 1회 수동 적용
  3. 운영 프로파일을 `ddl-auto: validate`로 고정
  4. `db/schema.sql`을 저장소에 커밋
     > ⚠️ **RDS가 프라이빗 서브넷에 있으므로 로컬 PC에서 직접 접속할 수 없다.** 2번을 실행할 경로가 필요하다:
     > EC2에 `postgresql-client` 설치 → `scp`로 DDL 파일을 EC2에 전송 → **EC2에서 `psql -h <rds-endpoint> -U <user> -d todolist_db -f schema.sql`** 실행.
     > 이 경로를 준비하지 않으면 11-1 중반에 막힌다. EC2를 먼저 띄운 뒤 RDS 스키마를 적용하는 순서가 된다.

### 11-2. 백엔드 (EC2)

- EC2에 JDK 21 설치
- `./mvnw package`로 jar 생성 후 전송
- **systemd 서비스로 등록** (자동 재시작, 부팅 시 기동)
- 환경변수는 systemd `EnvironmentFile`로 주입 (`.env` 커밋 금지)
- EC2 보안그룹 인바운드: **80과 443을 공개**, 22는 본인 IP로 제한, 8080은 외부에 열지 않는다
  > ⚠️ **80을 닫으면 안 된다.** 11-3의 `80 → 443` 리다이렉트가 도달 불가능해지고, **certbot의 HTTP-01 챌린지도 실패해 인증서 발급 자체가 안 된다.** 80은 열되 nginx가 443으로 리다이렉트만 하도록 구성한다 (평문으로 서비스하지 않는다)

### 11-3. HTTPS (방식 확정: nginx + certbot)

개인 프로젝트 규모이므로 **EC2 한 대에 nginx 리버스 프록시 + Let's Encrypt**로 간다. ALB + ACM은 관리가 편하지만 상시 비용... (12KB 남음)
