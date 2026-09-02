# ROADMAP — Todo List 프로젝트

> **버전** 1.25 · **최종 수정** 2026-09-02
> **v1.25 변경**: Phase 9(인터랙션 다듬기)를 실제 구현·브라우저 검증 후 완료(✅) 처리했다. `feature/interaction-polish` 브랜치에서 진행 후 `develop`에 fast-forward 병합했다. `useToggleTodo`/`useDeleteTodo`에 `onMutate`/`onError`/`onSettled` 낙관적 업데이트를 추가했고, 연타 대응은 `CLAUDE.md` 9장의 두 방법 중 **방법 1(항목별 `scope` 직렬화)**을 채택했다 — 이를 위해 `useToggleTodo`가 목록 화면의 공유 인스턴스가 아니라 `TodoItem` 안에서 `id`를 인자로 받는 항목별 인스턴스로 바뀌었다(`scope.id`가 `useMutation()` 정의 시점에 고정되는 값이라 공유 인스턴스로는 항목별 직렬화가 불가능함을 설계 중 확인). `TodoItem`을 `motion.li`로 전환해 목록 진입 stagger·삭제 시 `AnimatePresence`·체크 시 스프링 펄스(명령형 `animate()`로 Radix 체크박스 리마운트를 피함)를 추가했다. 실측 중 항목별 연타(3회)가 네트워크 로그상 겹치지 않고 순차 처리됨과 최종 DB 값이 화면과 일치함을 확인했고, `window.fetch` monkey-patch로 토글·삭제 각각의 실패 롤백과 삭제 실패 시 "2페이지 마지막 항목" 경계에서 페이지 이동이 일어나지 않음을 재현해 확인했다. `prefers-reduced-motion`은 브라우저 자동화 도구로 라이브 에뮬레이션이 불안정해 코드 리뷰(분기 로직 확인)로 마무리했다.
> **v1.9 변경**: 아래 "스캐폴딩 정합성 점검" 표가 실제 코드 상태와 다르게 "✅ 해소"로 잘못 기록된 항목이 다수 발견되어(Next.js 버전, `src/` 이동, `AGENTS.md`, `pom.xml`의 springdoc·jsoup, `docs/guides/` 등) 재검증 후 정정했다.
> **v1.10 변경**: Phase 6(프론트 스캐폴딩)를 실제로 진행해 v1.9에서 ❌로 표시했던 프론트 관련 항목을 모두 해소하고 DoD 14개 전부를 브라우저로 직접 검증했다. `CLAUDE.md` 6장도 Access+Refresh 2-토큰 설계로 정정했다(4장 `refresh_tokens` 테이블, 5장 `/auth/refresh`·`/auth/logout` API, 11장 `TOKEN_EXPIRED`·`INVALID_REFRESH_TOKEN` 코드 추가 — `docs/PRD.md` 264행이 이미 이 설계를 전제하고 있었음). 백엔드(springdoc·jsoup 등)는 아직 미착수.
> **v1.11 변경**: Phase 0(저장소 초기화)을 실제 git 명령으로 재검증 후 완료 처리했다. 세 저장소(todo-project·todo-backend·todo-frontend) 모두 `main`/`develop` 브랜치 존재, 원격(`origin`, 세 저장소 이름 통일) 연결 및 최초 push 완료, 미커밋 변경사항 커밋 완료를 확인했다. `.env`/`.env.example` gitignore 규칙은 `git check-ignore` exit code만으로 판단하지 않고 실제 `git add` 결과로 재검증했다(음수 패턴 매칭 시 check-ignore가 exit 0을 반환해도 실제로는 무시되지 않을 수 있음에 주의).
> **v1.12 변경**: Phase 1의 작업 목록·DoD에 남아있던 `application.yml` 계열 서술을 `.properties` 유지 결정(`CLAUDE.md` v1.10)에 맞게 정정했다. `application.properties`·`application-local.properties`는 이미 존재하므로 재작업 대상이 아니며, 실제로 남은 작업은 `application-prod.properties` 신규 작성뿐임을 명시했다. `.gitignore`의 `.env*`/`!.env.example` 규칙도 이미 커밋되어 있음을 반영했다.
> **v1.13 변경**: Phase 1(백엔드 스캐폴딩)을 실제 명령 실행으로 재검증 후 완료(✅) 처리했다. springdoc 3.1.0·jsoup 1.21.1 의존성 추가, 계층별 패키지 골격, `application-prod.properties`, `.env.example`·`todo-backend/CLAUDE.md`를 반영했다. 검증 중 `spring-boot-starter-security`가 이미 있어 `SecurityConfig` 없이는 Swagger DoD가 401로 막힘을 발견해 사용자 확인 후 최소 `SecurityConfig`(swagger·api-docs·error·actuator/health만 permitAll)를 선반영했다 — 전체 보안 설정은 여전히 Phase 3에서 완성한다. DoD 9개 전부 실측 통과(`dependency:tree` 성공, local 프로파일 기동, Swagger UI 200, `.env` gitignore 정상 동작).
> **v1.14 변경**: Phase 2 작업 목록에 남아있던 `application-test.yml` 표기를 `.properties`로 정정했다. `createdb todolist_db`가 이미 완료된 상태임을 반영하고, 테스트 DB명(`todolist_test`)이 부모 `CLAUDE.md` 절대규칙 2(`todolist_db_test`)와 다르다는 점을 각주로 남겼다 — 이 프로젝트는 지금까지 모든 문서 충돌에서 프로젝트 SSOT(이 문서·`todo-project/CLAUDE.md`)를 채택해 왔으므로 `todolist_test`로 통일한다.
> **v1.15 변경**: Phase 2(도메인 & DB)를 실제 명령 실행으로 재검증 후 완료(✅) 처리했다. `BaseEntity`·`Priority`·`AuthProvider`·`User`·`Todo`·`UserRepository`·`TodoRepository`, `application-test.properties`, Repository 단위 테스트 4건을 반영했다. 검증 중 `hibernate.jdbc.time_zone=UTC` 설정만으로는 `created_at`이 UTC로 저장되지 않는 실제 버그(9시간 어긋남)를 테스트로 발견해 `DateTimeProvider` 빈으로 근본 수정했고, Spring Boot 4에서 `@DataJpaTest`·`@AutoConfigureTestDatabase`의 패키지가 재편된 사실도 확인해 남겼다. DoD 6개 전부 실측 통과.
> **v1.16 변경**: Phase 3(인증 로컬 + 인증 테스트)을 실제 curl·테스트 실행으로 재검증 후 완료(✅) 처리했다. RefreshToken 엔티티, 공통 응답·예외 처리 인프라, `@MaxByteLength` 검증기, JWT·RefreshTokenService, `JwtAuthenticationFilter`, `SecurityConfig` 확장(CSRF·STATELESS·CORS·EntryPoint), 인증 DTO·`AuthService`·`AuthController`, Swagger `bearerAuth`, 통합 테스트 7건을 반영했다. 검증 중 두 가지 실제 버그를 발견해 수정했다: (1) Spring Boot 4의 Jackson 3(`tools.jackson.*`) 전환으로 `com.fasterxml.jackson.databind.ObjectMapper` 주입이 실패하던 문제, (2) `REQUIRES_NEW`가 self-invocation에서 무시되어 탈취 대응(전체 토큰 폐기)이 트랜잭션 롤백에 함께 취소되던 보안 버그(`TransactionTemplate`으로 수정). DoD 17개 전부 실측 통과.
> **v1.17 변경**: Phase 4(Todo API + Todo 테스트)를 실제 curl·테스트·성능 측정으로 재검증 후 완료(✅) 처리했다. `HtmlSanitizer`(Jsoup Safelist), Todo DTO 4종·`PageResponse`, `TodoRepository.search()`, `TodoService`, `TodoController` 6개 엔드포인트, 시드 스크립트 2종(100건/10,000건), 통합 테스트 4건을 반영했다. 검증 중 두 가지 실제 버그를 발견해 수정했다: (1) `keyword`가 `NULL`일 때 PostgreSQL이 파라미터 타입을 추론 못해 목록 조회가 500이 나던 문제(JPQL `CAST(:keyword AS string)`로 해결), (2) PUT/toggle 응답의 `updatedAt`이 `@PreUpdate`(flush 시점) 이전 값으로 직렬화되던 문제(`flush()` 선호출로 해결). 성능 DoD는 10,000건 기준 중앙값 약 40ms로 500ms 기준을 여유 있게 통과했다. DoD 16개 전부 실측 통과.
> **v1.18 변경**: Phase 5(구글 OAuth2)의 코드·단위 테스트를 완성했다 — `CustomOAuth2User`·`CustomOAuth2UserService`(신규가입/재조회/충돌거부/nickname 결정), `OAuth2SuccessHandler`·`OAuth2FailureHandler`, `SecurityConfig`의 `oauth2Login()` 연결, `CustomOAuth2UserServiceTest` 5건. 다만 실제 Google Cloud Console 자격증명(`GOOGLE_CLIENT_ID`/`SECRET`)이 아직 없어(사용자 확인 완료) 브라우저로 구글 로그인을 끝까지 밟아야 하는 DoD 4개는 **보류**로 명시하고, 진행 현황 표도 ✅가 아니라 🟡로 정정했다(단위 테스트만으로 로직은 검증했으나 실제 왕복은 미검증). `client-id`/`client-secret`에 `changeme` 자리표시자를 둬 자격증명 없이도 기동은 가능하게 했다. 자격증명 발급 후 사용자가 직접 로그인해 4개 DoD를 마저 체크하면 Phase 5가 ✅로 완료된다.
> **v1.19 변경**: 「요구사항 ↔ Phase 추적표」에 v1.9의 2-토큰 설계 정정이 반영되지 않고 남아있던 2개 행을 정정했다. (1) `AUTH-04`의 "JWT 24h"를 "Access 30분/Refresh 14일"로 고쳤다(`CLAUDE.md` 6장 확정값). (2) `AUTH-06`의 구현 Phase를 "7 (프론트 전용, 서버 API 없음)"에서 "3(`/auth/logout` API) · 7(연결)"로, 검증 Phase를 "7 · 10"에서 "3 · 7 · 10"으로 고쳤다 — `/api/v1/auth/logout`은 Phase 3에서 이미 구현되어 `AuthController`에 존재하고 Phase 3 DoD("logout 후 해당 Refresh Token으로 refresh 재시도 시 401")도 실측 통과한 상태였다. **추적표를 그대로 두면 Phase 7에서 로그아웃을 클라이언트 토큰 삭제만으로 구현할 위험이 있었다**(`CLAUDE.md` 5장이 금지하는 동작). 코드 변경은 없고 문서 기록만 사실에 맞게 정정했다.
> **v1.20 변경**: v1.19에서 추적표만 고치고 남겨뒀던 **같은 결함의 나머지 절반**을 정정했다. (1) Phase 7 작업 목록의 `useAuth` 로그아웃이 "토큰 삭제 + 캐시 초기화"로만 적혀 있어 서버 `POST /api/v1/auth/logout` 호출이 빠져 있었다 — `CLAUDE.md` 6장이 금지하는 클라이언트 전용 로그아웃으로 구현될 위험이 그대로 남아 있었다. (2) Phase 7·10 DoD에 **로그아웃 후 `/auth/refresh` 재시도가 401인지** 확인하는 항목을 추가했다(클라이언트만 지운 구현을 걸러내는 항목). (3) Phase 7 DoD 각주의 "`JWT_EXPIRATION`을 낮춰 발급"은 **그런 환경변수가 존재하지 않아** 실행 불가였다 — 실제 값은 `JwtTokenProvider.ACCESS_TOKEN_EXPIRATION`(`Duration.ofMinutes(30)`) 코드 상수이므로 이를 명시하고 되돌리기 주의를 덧붙였다. 코드 변경은 없다.
> **v1.21 변경**: v1.20에서 미결로 남겨둔 **로그아웃 서버 호출 실패 시의 동작**을 확정했다(2026-09-01, 사용자 결정). 서버 호출이 실패해도 **클라이언트 정리와 `/login` 이동은 그대로 진행한다**(가용성 우선). 로그아웃을 중단하는 대안은 서버 장애 시 사용자가 로그아웃조차 못 하게 만들어 더 나쁘다고 판단했다. 남는 Refresh Token은 서버 복구 후 `/auth/refresh` 시점에 정리되거나 최대 14일 후 만료된다. Phase 7 착수 전 결정 사항이므로 코드 변경은 없다.
> **v1.22 변경**: Phase 7(인증 화면)을 실제 구현하고 브라우저·curl로 재검증했다. `(auth)/login`·`(auth)/signup`·`oauth/callback` 3개 화면, `useAuth` 훅, `(main)` 클라이언트 라우트 보호 레이아웃, `/todos` 플레이스홀더, 루트 리다이렉트를 구현했다. DoD 18개 중 16개를 실측 통과시켰다 — 미가입 이메일/비밀번호 오류 문구 동일성, 한글 24자(경계)/25자(초과) 비밀번호 실시간 검증, 중복 이메일 인라인 에러, 네트워크 실패 시 안내(재시도는 별도 버튼 대신 기존 로그인 버튼이 겸함), 로그아웃의 서버 API 실호출과 이후 `/auth/refresh` 401(실제 폐기 확인), 만료 토큰 즉시 차단(보호 화면 미노출), 새로고침 유지, `middleware.ts` 부재, `npm run build` 성공, Tab·Enter만으로 로그인 완주를 모두 확인했다. 구글 로그인 왕복 자체(1건)와 백엔드발 계정 충돌 리다이렉트는 Phase 5의 실제 Google 자격증명이 아직 없어 보류로 남겼다 — 다만 `/oauth/callback`의 토큰 저장·URL 정리 로직 자체는 curl로 발급받은 실제 유효 토큰을 수동으로 넣어 독립 검증했고, `?error=email_conflict` 배너도 쿼리 파라미터로 직접 검증했다. 진행 현황 표를 Phase 5와 동일하게 🟡로 표시했다.
> **v1.23 변경**: Phase 5(구글 OAuth2)를 실제 Google Cloud Console 자격증명 발급 후 브라우저로 끝까지 검증해 완료(✅) 처리했다. Google Cloud 프로젝트 생성 → OAuth 동의 화면(외부, 테스트 사용자 등록) → OAuth 클라이언트 ID 발급(리다이렉트 URI `http://localhost:8080/login/oauth2/code/google` 등록) 순서로 진행했고, `application-local.properties`에 발급받은 `client-id`/`client-secret`을 채웠다(커밋 대상 아님, `.gitignore` 확인됨). 진행 중 Console에 등록한 리다이렉트 URI 오타로 `redirect_uri_mismatch`가 한 차례 발생했으나 정정 후 해소됐다 — **리다이렉트 URI는 Spring Security가 커스텀 설정 없이 자동 생성하는 `{baseUrl}/login/oauth2/code/{registrationId}` 템플릿을 그대로 써야 하며, 트레일링 슬래시·스킴·오탈자 하나까지 정확히 일치해야 한다.** DoD 4개 중 3개(302 리다이렉트, 신규 GOOGLE 계정 저장+실명 nickname, 재로그인 시 중복 미생성)를 실제 로그인·재로그인·로그아웃 반복 후 `users`·`refresh_tokens` 테이블을 직접 조회해 확인했다 — 특히 재로그인 검증은 화면 캡처 대신 **Refresh Token 회전 이력**(로그아웃 시 `revoked_at` 기록 → 재로그인 시 새 토큰 발급, `users` 행 수는 불변)으로 간접 확인하는 방법을 썼다. 나머지 1개(동일 이메일 로컬 계정 존재 시 `email_conflict` 리다이렉트)는 재현에 별도의 구글 테스트 계정이 필요해, 이미 통과한 `CustomOAuth2UserServiceTest`의 단위 테스트 커버리지로 충분하다고 사용자와 확정하고 보류 없이 완료 처리했다.
> **v1.24 변경**: Phase 8(Todo 화면)을 실제 구현하고 브라우저로 재검증해 완료(✅) 처리했다. `feature/todo-screens` 브랜치에서 5개 배치로 나눠 진행했다 — 배치1(목록 화면: 검색·필터·페이지네이션·`useTodos`), 배치2(`TodoForm`+Tiptap 에디터+작성 화면), 배치3(상세/수정 화면), 배치4(`dirty` 판정+3계층 이탈 확인, `TODO-16`), 배치5(경계 상황+360px 반응형+DoD 26개 최종 검증). DoD 26개 전부 실측 통과시켰다. 배치5에서 새로 검증한 3개가 이전 배치에서 미결로 남아 있었다: (1) **2페이지 마지막 항목 삭제 실패 시 롤백**(490행) — `window.fetch`를 monkey-patch해 `DELETE`만 실패시키는 방식으로 재현했다. 코드가 페이지 이동을 `onSuccess`에서만 수행하도록(`app/(main)/todos/page.tsx`) 이미 짜여 있어 실패 시 항목이 그대로 남고 페이지도 이동하지 않음을 확인했다 — 낙관적 업데이트 자체가 없는 Phase 8 구현이라 "롤백"은 "아무 것도 미리 바꾸지 않는다"는 설계로 자연히 충족됐다. (2) **360px 반응형**(504행) — 배치1은 "자동화 환경에서 브라우저 창이 최대화 고정이라 실측 불가"로 남겼던 항목이다. `resize_window` 도구도 실제 `window.innerWidth`를 바꾸지 못함을 재확인했고, 대신 `360×800` `<iframe>`을 페이지에 주입해 그 안에서 목록·작성·상세 3개 화면을 로드하는 방식으로 우회 검증했다(iframe은 자신의 뷰포트 기준으로 미디어 쿼리가 평가되므로 진짜 반응형 검증이 된다) — 세 화면 모두 `scrollWidth <= clientWidth`로 가로 스크롤 없음을 확인했다. (3) **키보드만으로 할 일 생성**(505행) — 배치1은 토글·삭제만 확인했고 생성은 남아 있었다. `Tab`/`Shift+Tab`/`Enter`만으로 목록→작성 화면 이동→제목 입력→Tiptap 툴바 10개를 지나 본문 에디터 포커스→본문 입력→우선순위/마감일을 지나 저장 버튼까지 도달해 `Enter`로 제출, 목록에 정상 반영됨을 확인했다. 이 과정에서 실제 개발 환경 문제 두 가지를 발견해 해결했다(코드 변경 아님) — 이전 세션이 백그라운드에 남긴 `npm run dev` 프로세스가 포트 3000을 점유해 새 서버가 3001로 밀려나면서 두 dev 서버가 같은 `.next` 캐시를 동시에 써 페이지가 간헐적으로 404가 나던 문제, 그리고 dev 서버가 켜진 채로 `npm run build`를 실행해 정적 청크가 503으로 깨지던 문제 — 둘 다 프로세스를 정리하고 `.next`를 지운 뒤 재기동해 해소했다. `npm run build` 최종 재실행으로 프로덕션 빌드 성공을 재확인했다(`/todos` 7.36 kB, `/todos/[id]` 1.39 kB, `/todos/new` 599 B). 코드 변경 없이 검증만 수행했으므로 `todo-frontend`에는 별도 커밋이 없다.
> 이 문서는 "어떤 순서로 만드는가"를 정의하며, **완료 판정의 정본**이다.
> **한 번에 전체를 생성하지 않는다.** Phase 단위로 진행하고, 각 Phase의 DoD를 모두 만족한 뒤 다음으로 넘어간다.
> 기술 규칙은 `CLAUDE.md`, 기능 정의는 `PRD.md` 참조.

---

## 진행 현황

| Phase | 내용                       | 저장소   | 상태 |
| ----- | -------------------------- | -------- | ---- |
| 0     | 저장소 초기화              | 전체     | ✅   |
| 1     | 백엔드 스캐폴딩            | backend  | ✅   |
| 2     | 도메인 & DB                | backend  | ✅   |
| 3     | 인증 (로컬) + 인증 테스트  | backend  | ✅   |
| 4     | Todo API + Todo 테스트     | backend  | ✅   |
| 5     | 구글 OAuth2 + OAuth 테스트 | backend  | ✅   |
| 6     | 프론트 스캐폴딩            | frontend | ✅   |
| 7     | 인증 화면                  | frontend | 🟡   |
| 8     | Todo 화면                  | frontend | ✅   |
| 9     | 인터랙션 다듬기            | frontend | ✅   |
| 10    | 전체 검증                  | 전체     | ⬜   |
| 11    | AWS 배포                   | 전체     | ⬜   |

⬜ 대기 · 🟡 진행중 · ✅ 완료

> ### 스캐폴딩 정합성 점검 (2026-08-28, v1.9 재검증)
>
> 스캐폴딩이 스펙과 어긋난 채 진행되어 있었다. **아래 표는 2026-08-28에 실제 파일을 직접 열어 재검증한 결과다.** 이전 버전(v1.8)이 "✅ 해소"로 기록했던 항목 중 다수가 실제로는 반영되지 않은 상태였다 — 코드는 변경하지 않고 **기록만** 사실대로 정정했다. 이 항목들을 "완료"로 취급하지 말 것.
>
> | 항목                                   | 발견 당시                                                                       | 결과                                                                                                                                                                                         |
> | -------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | `todo-frontend`가 Next.js 16.3.3       | Amplify SSR 지원 범위(12~15) 밖이라 Phase 11 배포 불가 상태였음                 | ✅ **해소 (2026-08-28, v1.10).** `next`·`eslint-config-next` 모두 `15.5.24`로 다운그레이드. `npm run build`·`lint`·`typecheck` 통과 확인                                                    |
> | 프론트에 `src/` 없음                   | `app/`·`components/`·`lib/`가 루트 직하                                         | ✅ **해소.** `src/app/`·`src/components/`·`src/lib/`·`src/hooks/`·`src/providers/`·`src/types/`로 이동. `tsconfig.json`(`@/*`→`./src/*`), `components.json`(css 경로) 동시 수정              |
> | `layout.tsx`가 `LayoutProps<"/">` 사용 | Next 16 전역 타입이라 15에서 컴파일 실패                                        | ✅ **해소.** `{ children: React.ReactNode }`로 교체                                                                                                                                          |
> | `eslint.config.mjs`가 16 방식          | 15의 `eslint-config-next`는 flat config를 직접 내보내지 않음                    | ✅ **해소.** `@eslint/eslintrc`의 `FlatCompat`으로 `next/core-web-vitals`·`next/typescript`를 감싸도록 교체, `npm run lint` 통과 확인                                                       |
> | `AGENTS.md`                            | Next 16의 `next dev`가 자동 생성한 파일. 15에서는 재생성되지 않고 내용도 부정확 | ✅ **해소.** 삭제. `todo-frontend/CLAUDE.md`를 `@../CLAUDE.md` 임포트 + 저장소 전용 보강으로 재작성                                                                                          |
> | 워크스페이스 루트 오인                 | 부모 `todo-project`에도 `package-lock.json`이 있어 Next가 루트를 잘못 추론      | ✅ **해소.** `next.config.ts`에 `outputFileTracingRoot: __dirname` 추가                                                                                                                     |
> | `pom.xml`에 `springdoc` 없음           | Phase 1 DoD "Swagger UI 접속" 불가                                              | ❌ **미해결.** `pom.xml`에 springdoc 의존성 자체가 없음. Phase 1에서 Boot 4.1.1에 대응하는 정확한 버전을 핀할 것(범위 지정 금지, `CLAUDE.md` 3장 참조)                                     |
> | `pom.xml`에 `jsoup` 없음               | Phase 4 XSS 정화 불가                                                           | ❌ **미해결.** `pom.xml`에 jsoup 의존성 자체가 없음. Phase 4에서 추가할 것                                                                                                                  |
> | JWT 라이브러리 미선정                  | `CLAUDE.md` 3장 표에 JWT 라이브러리가 명시되어 있지 않음                        | ✅ **jjwt 0.12.6**(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 `pom.xml`에 핀되어 있고, **`CLAUDE.md` 3장 스택 표와 「버전 관련 확정 사항」에 등재됐다**(v1.8). Phase 3은 이 버전을 그대로 쓴다 |
> | `todo-backend/.gitattributes`          | Git Bash에서 `./mvnw` `bad interpreter` 위험                                    | ✅ 존재함 (`/mvnw text eol=lf`, `*.cmd text eol=crlf`)                                                                                                                                       |
> | 백엔드 버전                            | Spring Boot 4.1.1 + `java.version` 21                                           | ✅ 스펙 일치. 변경 없음                                                                                                                                                                      |
>
> #### 아직 남은 것 (2026-08-28 재확인)
>
> | 항목                     | 현재 상태                                                                                                                                                                                                                                                                                                                         | 처리 Phase |
> | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
> | ~~커밋·원격~~            | ✅ **해소 (2026-08-28, v1.11).** 세 저장소 모두 원격(`origin`, 이름 통일 완료)에 연결되고 `main` 브랜치 최초 push 완료. `git rev-parse main`과 `git ls-remote origin main`을 대조해 로컬-원격 커밋 해시 일치를 확인했다                                                                                                          | —          |
> | ~~브랜치명~~             | ✅ **해소 (v1.11).** 세 저장소 모두 `main`이 존재하고(과거 `master`는 이미 이번 세션 이전에 `main`으로 정리되어 있었다), `develop`을 새로 분기해 원격까지 push했다. `master` 브랜치는 세 저장소 어디에도 남아있지 않다                                                                                                            | —          |
> | 루트 `.gitignore`        | ✅ **해소 (v1.11).** `node_modules/` 안전망 규칙과 세션 스크래치 파일(`scaffold-check.png`, `.playwright-mcp/`, `.claude/plans/`) 제외 규칙을 추가했다                                                                                                                                                                           | —          |
> | ~~백엔드 `.gitignore`~~  | ✅ **해소.** 확인 결과 이번 세션 이전에 이미 `.env`/`.env.*`/`!.env.example` 규칙이 커밋되어 있었다(v1.9 표의 서술이 stale했음). `git add .env.example`로 실제 무시되지 않음을 재검증했다                                                                                                                                        | —          |
> | ~~프론트 `.gitignore`~~  | ✅ **해소 (v1.11로 커밋 반영).** `!.env.example` 예외 줄이 작업 트리에는 이미 있었으나 미커밋 상태였다. Phase 6 마무리 커밋에 포함해 반영했다                                                                                                                                                                                    | —          |
> | ~~문서 경로~~            | ✅ **해소.** `docs/`를 정본으로 확정하고 `CLAUDE.md` 2장 구조도와 상단 참조 경로를 정정했다 (v1.8)                                                                                                                                                                                                                                | —          |
> | ~~루트 npm 파일~~        | ✅ **해소.** 루트 `package.json`·`package-lock.json`·`node_modules/`를 삭제했다. `shadcn`은 `todo-frontend/package.json`에 이미 있어 기능 손실이 없고, `.mcp.json`의 shadcn 서버는 `npx shadcn@latest mcp`라 루트 설치에 의존하지 않는다                                                                                          | —          |
> | `docs/guides/`            | ✅ **해소 (v1.10).** `nextjs-16.md`→`nextjs-15.md`, `forms-react-hook-form.md`→`forms.md`로 개명하고 이 프로젝트 기준으로 재작성(react-hook-form 전제·PRD 1.3·M5·radix-nova·next-themes 등 제거). `README.md` 추가                                                                     | —          |
> | ~~폼 라이브러리 미결정~~ | ✅ **해소.** `CLAUDE.md` 3장에 **"라이브러리를 쓰지 않는다 — `useState` + 수동 검증"**으로 확정. `npx shadcn add form` 금지(=`react-hook-form` 유입 경로)와 Tiptap dirty 판정 주의를 함께 명시                                                                                                                                    | —          |
> | 백엔드 설정 파일         | `application.properties`·`application-local.properties`·`application-local.properties.example`는 이미 존재(형식은 `.properties`가 맞다 — 사용자 전역 CLAUDE.md에 명시된 확정 사항). 다만 `application-prod.properties` 분리는 아직 없음                                                                                        | Phase 1    |
> | 백엔드 문서·예시         | `.env.example`, 저장소용 `CLAUDE.md` 없음                                                                                                                                                                                                                                                                                         | Phase 1    |
> | ~~프론트 마감 작업~~     | ✅ **해소 (v1.10).** 디자인 토큰을 `CLAUDE.md` 8장 팔레트로 교체, 다크 모드를 `@media (prefers-color-scheme: dark)`로 전환(`class` 전략·`.dark` 셀렉터 제거), `components.json` 스타일을 `new-york`으로 변경, `.env.example` 추가, Tiptap(`@tiptap/react`+`@tiptap/starter-kit`)·motion·sonner·date-fns·DOMPurify 설치 완료          | —          |

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
| AUTH-04 로그인·Access 30분/Refresh 14일  | 3 · 7                                                         | 3 · 7 · 10                                       |
| AUTH-05 구글 로그인                      | 5 · 7(`/oauth/callback`)                                      | 5 · 7 · 10                                       |
| AUTH-06 로그아웃                         | 3(`/auth/logout` API) · 7(연결)                               | 3 · 7 · 10                                       |
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

**DoD** (2026-08-28, v1.11 — 전부 실제 git 명령으로 재검증 완료)

- [x] 부모 저장소에서 `git status` 시 하위 폴더가 나타나지 않음
- [x] **부모 저장소에서 `git status` 시 `node_modules/`가 나타나지 않음**
- [x] **세 저장소 각각에서 `.env.example`이 실제로 무시되지 않고, `.env`는 무시됨**
      > ⚠️ `git check-ignore -v .env.example`의 exit code만으로 판단하면 안 된다. 마지막에 매칭된 패턴이 `!.env.example`(음수) 패턴이어도 `check-ignore`가 exit 0을 반환하는 경우가 있었다(todo-project·todo-backend에서 실제로 관찰됨). 실제 판정은 `git add .env.example` 실행 결과("The following paths are ignored" 경고 유무)로 검증했고, 세 저장소 모두 정상적으로 add됨을 확인했다
- [x] 세 저장소 모두 현재 브랜치가 `main`이고 `develop`이 존재함 (`master` 없음) — `git branch -a`로 세 저장소 모두 확인
- [x] 세 저장소 모두 첫 커밋 및 원격 푸시 완료 — `git rev-parse main` == `git ls-remote origin main` 해시 일치 확인
- [x] 첫 커밋의 파일 수가 예상 범위 안임 — todo-project 22개, todo-backend 17개, todo-frontend 19개 (모두 수백 건 미만, 무시 규칙 누락 없음)
- [x] `CLAUDE.md` 2장 구조도의 문서 경로가 실제 파일 위치(`docs/`)와 일치함
- [x] `docs/guides/`가 `README.md` + 재작성된 5개 문서로만 구성됨 (`nextjs-16.md`·`forms-react-hook-form.md` 없음)
- [x] **`todo-frontend/package.json`에 `react-hook-form`·`zod`·`@hookform/resolvers`·`next-themes`가 없음** (스펙 밖 라이브러리 유입 확인, `grep`으로 재확인)

---

## Phase 1 — 백엔드 스캐폴딩

**저장소**: `todo-backend`

**작업**

- Spring Initializr 기준 프로젝트 생성 (Spring Boot 4.x, JDK 21, Maven, `com.example`)
- 의존성: Web, Data JPA, Security, Validation, PostgreSQL Driver, OAuth2 Client, Lombok, **Jsoup**(HTML 정화), **jjwt**(JWT 생성·검증)
- **SpringDoc OpenAPI는 `springdoc-openapi-starter-webmvc-ui` 3.x를 명시한다.** 2.8.x는 Spring Boot 3.x 전용이라 기동에 실패한다
- 패키지 골격 생성: `domain / service / controller / dto / config / exception`
- **설정 파일 형식은 `.properties`로 유지한다** (`CLAUDE.md` v1.10 정정 — 부모 CLAUDE.md 절대규칙 9 및 기존 파일과 정합). `application.properties`·`application-local.properties`는 이미 존재하므로 재작업하지 않고, **`application-prod.properties`를 신규 작성**해 운영 전용 값(`ddl-auto=validate` 등)만 오버라이드한다 (프로파일별 `ddl-auto`는 `CLAUDE.md` 12장 표 참조)
- **`spring.profiles.active` 기본값 지정** — `application.properties`에 이미 `${SPRING_PROFILES_ACTIVE:local}`로 존재함 (없으면 DB 정보 없이 기동을 시도해 실패)
- `.gitignore`(**`.env*` + `!.env.example`**)는 이미 커밋되어 있음 — 재작업 불필요. `.env.example`, 저장소용 `CLAUDE.md` 작성 (상단에 `@../CLAUDE.md` 임포트)
- `.gitattributes`는 이미 있다(`/mvnw text eol=lf`). **삭제하거나 덮어쓰지 않는다** (`CLAUDE.md` 13장)
- **`config/SecurityConfig` 최소 선반영 (2026-08-31 추가)** — `spring-boot-starter-security`가 이미 의존성에 있어 `SecurityConfig` 없이는 모든 경로가 기본 Basic Auth(401)로 잠기고 Swagger DoD를 통과할 수 없음이 실측으로 드러났다. `/swagger-ui/**`·`/v3/api-docs/**`·`/error`·`/actuator/health`(부모 `CLAUDE.md` 절대규칙 9)만 `permitAll`하고 나머지는 `authenticated()`로 두는 `SecurityFilterChain` 빈만 추가했다. **CSRF 비활성화·STATELESS 세션·JWT 필터 등 전체 설정은 그대로 Phase 3에서 이 빈을 확장해 완성한다.**

**DoD** (2026-08-31 실측 검증 완료)

- [x] `pom.xml`의 SpringDoc이 **현재 Boot 버전에 대응하는 정확한 3.x 버전으로 핀**되어 있음 (범위 지정 금지 — `CLAUDE.md` 3장) — `springdoc-openapi-starter-webmvc-ui:3.1.0` 확인
- [x] `pom.xml`에 **`jsoup`이 포함**되어 있음 (Phase 4의 XSS 정화 전제) — `jsoup:1.21.1` 확인
- [x] `pom.xml`에 **jjwt 3종(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 동일 버전으로 핀**되어 있음 (Phase 3의 `JwtTokenProvider` 전제) — 0.12.6 3종 확인
- [x] `./mvnw dependency:tree` 오류 없음 — `BUILD SUCCESS`
- [x] **`src/main/resources`에 `application.properties`·`application-local.properties`·`application-prod.properties` 3종이 있고 `.yml` 파일은 없음**
- [x] `./mvnw spring-boot:run`이 옵션 없이 local 프로파일로 기동 성공
- [x] **기동 로그에 `The following 1 profile is active: "local"`이 찍힘**
- [x] `http://localhost:8080/swagger-ui/index.html`이 200으로 열리고 API 목록 화면이 렌더됨 — `v3/api-docs`도 200으로 OpenAPI JSON 정상 응답. **위 `SecurityConfig` 최소 선반영 이후 통과** (선반영 전에는 401)
- [x] `git check-ignore -v .env`가 규칙에 매칭되고 `.env.example`은 매칭되지 않음 — `git add .env.example`로 실제 스테이징까지 재확인

> SpringDoc 버전은 `CLAUDE.md` 3장에서 3.x로 확정됐다. 다시 조사하거나 2.x로 되돌리지 않는다.

---

## Phase 2 — 도메인 & DB

**저장소**: `todo-backend`

**작업**

- `createdb todolist_db`(2026-08-31 확인 결과 이미 존재) · `createdb todolist_test`(아직 없음, 이번 Phase에서 생성)
- `BaseEntity` (`@MappedSuperclass`, JPA Auditing)
- `User`, `Todo` 엔티티 + `Priority` enum + `AuthProvider` enum
- **`Todo.user`는 `@ManyToOne(fetch = FetchType.LAZY)`** (기본값 EAGER 금지)
- **`hibernate.jdbc.time_zone: UTC` 설정** — 로컬 KST와 RDS UTC의 9시간 어긋남 방지
- 입력값 제약을 스키마와 일치시킨다 (`CLAUDE.md` 4장 제약 표)
- `UserRepository`, `TodoRepository` (`deleted_at IS NULL` 조건 포함 쿼리)
- 인덱스 `idx_todos_user_deleted`
- `src/test/resources/application-test.properties` (`todolist_test`, `ddl-auto=create-drop`)
- **`@EnableJpaAuditing`은 메인 애플리케이션 클래스에 붙인다** (`@Configuration`에 두면 `@DataJpaTest`가 로드하지 않아 `created_at`이 null이 됨)
- Repository 테스트에 **`@AutoConfigureTestDatabase(replace = NONE)` + `@ActiveProfiles("test")`** 필수 (없으면 임베디드 DB로 교체 시도)
  > ⚠️ **Spring Boot 4는 테스트 슬라이스 애노테이션 패키지를 재편했다.** Boot 3의 `org.springframework.boot.test.autoconfigure.orm.jpa.*`는 더 이상 없다. `@DataJpaTest`는 `org.springframework.boot.data.jpa.test.autoconfigure`, `@AutoConfigureTestDatabase`는 `org.springframework.boot.jdbc.test.autoconfigure`로 옮겨졌다(2026-08-31 실제 jar 내용으로 확인). Phase 3~5에서 `@WebMvcTest` 등 다른 슬라이스 애노테이션을 쓸 때도 옛 패키지를 그대로 가정하지 말 것.
  > ⚠️ **테스트 DB명은 `todolist_test`다** (프로젝트 SSOT `todo-project/CLAUDE.md` 4장 기준). 부모 `CLAUDE.md`(`D:\my-workspace\claude\CLAUDE.md`) 절대규칙 2는 `todolist_db_test`를 쓰라고 하지만, 이 프로젝트는 폴리레포 구조·`.properties` 형식·Next.js 15·jjwt 등 지금까지 모든 문서 충돌에서 일관되게 이 프로젝트 SSOT를 채택해 왔으므로 같은 기준을 적용한다. 부모 문서는 수정 권한 밖(상위 폴더)이라 건드리지 않는다 — 혼동하지 않도록 여기 남긴다.
- **`hibernate.jdbc.time_zone=UTC`만으로는 `created_at`/`updated_at`이 UTC로 저장되지 않는다 (2026-08-31 실측 발견)** — `@CreatedDate`/`@LastModifiedDate`가 기본으로 쓰는 `LocalDateTime.now()`는 JVM 기본 타임존(KST)의 벽시계 값을 그대로 반환하고, 이 설정은 그 값을 UTC로 변환해 주지 않는다(Repository 테스트에서 9시간 어긋남을 확인). **Auditing이 값을 만드는 시점 자체를 UTC로 고정해야 한다** — `TodoBackendApplication`에 `@EnableJpaAuditing(dateTimeProviderRef = "utcDateTimeProvider")`와 `LocalDateTime.now(ZoneOffset.UTC)`를 반환하는 `DateTimeProvider` 빈을 함께 둔다(같은 이유로 `@Configuration`이 아닌 메인 클래스). `Todo.softDelete()`처럼 엔티티가 직접 `LocalDateTime.now()`를 호출하는 곳도 전부 `LocalDateTime.now(ZoneOffset.UTC)`로 바꿔야 한다.

**DoD** (2026-08-31 실측 검증 완료)

- [x] 애플리케이션 기동 시 `users`, `todos` 테이블 자동 생성 — `psql \d`로 컬럼·제약·FK·인덱스까지 확인
- [x] `created_at`, `updated_at` 자동 기록 확인
- [x] Repository 단위 테스트(`@DataJpaTest`) 통과 — **통합 테스트 번호 체계(1~8)와 별개**. `UserRepositoryTest` 2건, `TodoRepositoryTest` 2건 총 4건
- [x] 테스트가 `todolist_test`를 바라보고 실행됨 (H2 사용하지 않음) — 로그의 `Database JDBC URL [jdbc:postgresql://localhost:5432/todolist_test]`, `org.postgresql.jdbc.PgConnection`으로 확인
- [x] `@DataJpaTest`에서 `created_at`이 null이 아님 (Auditing 정상 동작)
- [x] 저장된 `created_at`이 UTC 기준임 (KST로 9시간 밀리지 않음) — 위 `DateTimeProvider` 수정 후 `Duration.between(createdAt as UTC, Instant.now())`가 1분 이내임을 테스트로 확인

---

## Phase 3 — 인증 (로컬) + 인증 테스트

**저장소**: `todo-backend` · **관련 요구사항**: AUTH-01~04, 06~08

> **v1.9 변경**: `CLAUDE.md` 6장이 Access(30분)+Refresh(14일, httpOnly 쿠키) 2-토큰 구조로 정정됨에 따라 이 Phase의 작업·DoD도 함께 갱신했다. AUTH-06(로그아웃)은 이제 서버 API(`/auth/logout`)가 있으므로 Phase 3에서 구현하고 Phase 7에서 연결한다.

**작업**

- `SecurityConfig` (SecurityFilterChain, CORS, 경로별 인가)
  - **`permitAll` 경로에 Swagger(`/swagger-ui/**`, `/v3/api-docs/**`)와 `/api/v1/auth/refresh`를 반드시 포함** (`CLAUDE.md` 6장 목록 그대로)
  - CORS 허용 헤더에 `Authorization`, `Content-Type` 명시, **`allowCredentials(true)`**(Refresh 쿠키), 로컬 프로파일은 `SameSite=Lax`
- `JwtTokenProvider` (Access Token 생성/검증, **30분 만료**, **`sub = user.id`**)
- `RefreshTokenService` (불투명 랜덤 토큰 발급, SHA-256 해시로 `refresh_tokens`에 저장, 회전, 재사용 탐지 시 사용자 전체 토큰 폐기)
- `JwtAuthenticationFilter` (`sub` → id 조회, `deleted_at IS NULL` 확인)
- `BCryptPasswordEncoder` 빈
- `AuthController`: signup / login(Access 응답 + Refresh 쿠키 발급) / refresh(쿠키 검증 → 회전) / logout(Refresh 폐기 + 쿠키 만료) / me
- DTO: `SignupRequest`, `LoginRequest`, `TokenResponse`(Access Token만), `UserResponse`
- `GlobalExceptionHandler` + `ErrorCode` + **공통 응답 `ApiResponse<T>`**
- **Swagger `@SecurityScheme`(bearerAuth) 설정** — Authorize 버튼으로 보호된 API 호출 가능하게
- **`AuthenticationEntryPoint` + `AccessDeniedHandler` 구현** — Security 필터 단계의 401/403도 `ApiResponse` 포맷으로 응답 (`GlobalExceptionHandler`는 필터 예외를 잡지 못함)
- **통합 테스트 작성** (회원가입/로그인/미인증 401에 더해 **refresh 회전**과 **재사용 탐지** 케이스 포함)
- **Spring Boot 4는 Jackson 3(`tools.jackson.*`)를 쓴다 (2026-08-31 실측 발견).** `ObjectMapper`를 직접 주입받는 코드(예: `AuthenticationEntryPoint`·`AccessDeniedHandler`에서 수동 JSON 직렬화)는 `com.fasterxml.jackson.databind.ObjectMapper`(Jackson 2 — jjwt-jackson이 런타임 전이 의존성으로 끌어오지만 스프링 빈으로는 등록되지 않는다)가 아니라 **`tools.jackson.databind.ObjectMapper`**를 import해야 한다. 틀리면 `UnsatisfiedDependencyException`으로 기동 자체가 실패한다.
- **`REQUIRES_NEW`는 self-invocation(같은 클래스 내부 호출)에서 무시된다 (2026-08-31 실측 발견).** `RefreshTokenService.rotate()`가 재사용을 감지해 전체 토큰을 폐기한 뒤 `BusinessException`을 던지면, 프록시를 거치지 않는 내부 호출이라 `REQUIRES_NEW`가 적용되지 않아 호출자 트랜잭션 롤백에 전체 폐기까지 함께 취소된다(탈취 대응 무효화). `PlatformTransactionManager` + `TransactionTemplate`으로 프록시 없이 직접 새 트랜잭션을 열어야 한다. `@Lazy` self-주입은 Lombok `@RequiredArgsConstructor`가 필드의 `@Lazy`를 생성자 파라미터에 복사하지 않아 순환참조 오류가 나므로 쓰지 않는다.

**DoD** (2026-08-31 실측 검증 완료)

- [x] **`POST /api/v1/auth/signup`이 403이 아니라 200/409로 응답** (CSRF 비활성화 확인 — 안 하면 여기서 전부 막힌다)
- [x] **응답에 `Set-Cookie: JSESSIONID`가 없음** (`STATELESS` 세션 정책 확인) — curl로 헤더 직접 확인
- [x] 회원가입 → 사용자 생성 + JWT 반환
- [x] **한글 25자 비밀번호로 가입 시 500이 아니라 400 `INVALID_INPUT`** (BCrypt 72바이트 한계 — `CLAUDE.md` 4장)
- [x] 중복 이메일 시 409 `EMAIL_DUPLICATED`
- [x] 로그인 성공 시 유효 Access Token 반환 + `Set-Cookie`로 httpOnly Refresh Token 발급, 실패 시 401
- [x] `/api/v1/auth/refresh`가 유효한 Refresh 쿠키로 새 Access Token과 회전된 Refresh 쿠키를 반환하고, 기존 Refresh Token은 재사용 시 401(탈취 대응으로 해당 사용자 전체 토큰 폐기) — 트랜잭션 롤백 버그를 실측으로 발견해 수정 후 재검증
- [x] `/api/v1/auth/logout` 호출 후 해당 Refresh Token으로 `/auth/refresh` 재시도 시 401
- [x] **미가입 이메일로 로그인한 401과 비밀번호가 틀린 401의 `code`·`message`가 완전히 동일함** (계정 존재 여부를 구분 노출하지 않는다 — `PRD.md` 5.1) — 응답 본문 문자열 완전 일치를 테스트로 확인
- [x] 토큰 없이 `/api/v1/auth/me` 호출 시 401
- [x] `/api/v1/auth/me` 응답에 `nickname`과 `email`이 모두 포함됨 (AUTH-08 — 화면 표시 여부는 Phase 7에서 통제)
- [x] 비밀번호가 DB에 해시로 저장됨 — `psql`로 `$2a$10$...` BCrypt 해시 확인
- [x] **Security 도입 후에도 Swagger UI 접속 가능** (Phase 1 DoD 회귀 방지)
- [x] Swagger Authorize에 토큰을 넣고 `/auth/me` 호출이 200으로 성공 — `v3/api-docs`의 `bearerAuth` 스킴 노출 + curl로 Bearer 헤더 호출 200 확인(Authorize 버튼과 동일한 인증 경로)
- [x] 모든 응답이 `{success, data, error}` 포맷을 따름
- [x] **토큰 없이 호출한 401 응답도** `{success:false, error:{code:"UNAUTHORIZED"}}` 포맷임
- [x] **인증 통합 테스트 3건 통과** — `AuthIntegrationTest` 7건 전부 통과(회원가입 2·로그인 2·me 1·refresh 회전/재사용 1·logout 1)

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
- **PostgreSQL은 파라미터가 `NULL`일 때 정적 타입을 추론하지 못한다 (2026-08-31 실측 발견).** `TodoRepository.search()`의 `keyword`가 `NULL`인 채로 `LOWER(t.title) LIKE LOWER(CONCAT('%', :keyword, '%'))`를 그대로 쓰면 드라이버가 파라미터 타입을 `bytea`로 잘못 추론해 `lower(bytea) 이름의 함수가 없음` 오류로 **목록 조회 자체가 500이 난다.** JPQL에서 `CAST(:keyword AS string)`로 명시적으로 캐스팅해야 한다.
- **`@LastModifiedDate`는 flush 시점에 채워진다 (2026-08-31 실측 발견).** `TodoService.update()`/`toggle()`이 엔티티를 변경한 직후 바로 `TodoResponse.from(todo)`로 응답을 만들면, `@PreUpdate` 콜백이 아직 실행되기 전이라 **응답의 `updatedAt`이 갱신 전 값으로 나간다**(DB에는 커밋 시 정상 저장되지만 그 요청의 응답만 stale). 응답을 만들기 전에 `todoRepository.flush()`를 호출해야 한다.

**DoD** (2026-08-31 실측 검증 완료)

- [x] 목록 API가 `{success, data:{content, page, ...}, error}` 형태로 응답
- [x] `PUT` 저장이 완료 상태를 덮어쓰지 않음
- [x] `toggle`을 같은 값으로 두 번 호출해도 결과가 동일함(멱등)
- [x] `completed` 미지정 시 전체 반환, `true`/`false` 시 필터 적용
- [x] 영문 대소문자를 섞어 검색해도 결과가 나옴 — `MEET`/`meet`/`MeEt` 모두 동일 결과로 curl 확인
- [x] `?sort=foo,desc` 같은 잘못된 정렬 값에도 500이 나지 않음
- [x] 삭제 시 `deleted_at` 기록, 목록에서 제외
- [x] 타 사용자 Todo 접근 시 404
- [x] 제목 미입력·200자 초과 시 400 + 필드 메시지
- [x] 본문 50,000자 초과 시 400
- [x] `<script>` 포함 본문 저장 시 태그 제거, `a` 태그에 `rel` 주입 확인
- [x] **키워드 검색 포함** 목록 조회가 **워밍업 후 3회 측정 중앙값 500ms 이내** (로컬, 시드 **10,000건** 기준) — 워밍업 160ms, 측정 42.1ms/39.9ms/40.3ms(중앙값 약 40ms)로 여유 있게 통과
  > ⚠️ 시드 100건으로는 이 지표가 의미가 없다. 인덱스가 없어도 100행은 1ms 미만이라 **항상 통과한다.** `CLAUDE.md` 4장이 지목한 유일한 성능 위험(`LOWER(title) LIKE '%키워드%'`의 인덱스 미사용)을 검출하려면 데이터가 충분해야 하고, 측정 대상도 검색 경로여야 한다. 또 첫 요청은 JVM 콜드 스타트라 DB가 아니라 워밍업 상태를 재는 셈이 되므로 워밍업 후에 측정한다.
- [x] 날짜가 배열이 아닌 ISO 문자열로 직렬화됨
- [x] 목록 조회 시 user 조회 쿼리가 추가로 발생하지 않음 — SQL 로그로 요청당 users 1회(인증 필터)·todos 1회·count 1회만 확인
- [x] Swagger에서 전체 API 확인 가능 — `/v3/api-docs`에 인증 5개·Todo 6개 엔드포인트 전부 노출 확인
- [x] **Todo 통합 테스트 4건 통과**

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
- **⚠️ 2026-08-31 기준 Google Cloud Console에서 발급받은 실제 `GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET`이 없다.** 사용자와 확인해 코드·단위 테스트까지 먼저 완성하고, 실제 브라우저로 구글 로그인을 끝까지 밟아야 확인되는 DoD는 자격증명 발급 후 사용자가 직접 검증하기로 했다. `client-id`/`client-secret`에 `changeme` 자리표시자 기본값을 둬서, 자격증명이 없어도 애플리케이션 기동 자체는 실패하지 않게 했다(실측 확인 — 값이 비어 있으면 `OAuth2ClientPropertiesRegistrationAdapter`가 기동 시점에 즉시 예외를 던진다).
- **부가 발견**: `oauth2Login()`은 인가 요청~콜백 사이 state/PKCE 값을 저장하기 위해 Spring Security 표준 동작으로 `JSESSIONID` 세션 쿠키를 임시 발급한다. Phase 3의 STATELESS 원칙과 별개로 OAuth2 로그인 왕복 구간에서만 쓰이는 정상 동작이며, 로그인 완료 후에는 우리 앱의 JWT 기반 인증에 영향을 주지 않는다.

**계정 충돌 정책 (확정)**
동일 이메일의 로컬 계정이 있으면 **거부한다.** 자동 연동하지 않고, 별도 계정도 만들지 않는다. 상세는 `CLAUDE.md` 6장.

**DoD** (2026-09-01 기준 — 실제 `GOOGLE_CLIENT_ID`/`SECRET` 발급 후 사용자가 브라우저로 직접 왕복 검증 완료)

- [x] 구글 로그인 후 JWT를 담은 302 리다이렉트 발생 — 실제 자격증명으로 브라우저 왕복 완료. 리다이렉트 성공의 간접 증거로 `refresh_tokens` 테이블에 새 행(회전 포함 2건)이 생성됨을 DB로 확인했다. 진행 중 Console에 등록한 리다이렉트 URI 오타로 `redirect_uri_mismatch`가 한 차례 났으나 정정 후 정상 동작했다.
- [x] 신규 사용자가 `provider=GOOGLE`, nickname이 채워진 상태로 저장됨 — 실제 로그인 후 `users` 테이블에 `provider=GOOGLE`, nickname에 구글 실명이 채워진 행 생성을 DB로 직접 확인했다(이메일 앞부분 fallback이 아니라 구글 `name` 클레임이 정상 사용됨).
- [x] 재로그인 시 중복 계정이 생기지 않음 — 로그아웃 → 같은 구글 계정으로 재로그인을 실제로 반복해, `users` 테이블 행 수가 그대로이고(신규 id 미생성) `refresh_tokens`만 새로 발급(기존 토큰은 로그아웃 시점에 `revoked_at` 기록)됨을 DB로 확인했다.
- [x] 동일 이메일 로컬 계정 존재 시 `error=email_conflict`로 302 리다이렉트 — **단위 테스트 커버리지로 완료 처리하기로 사용자와 확정(2026-09-01).** 실제 브라우저 E2E 재현은 로컬 계정으로 먼저 등록된 이메일을 소유한 별도 구글 테스트 계정이 필요해 이번 범위에서는 준비하지 않았다. `CustomOAuth2UserServiceTest`의 충돌거부 테스트(신규가입/재조회/충돌거부/닉네임 미지정/닉네임 절삭 5건 중 1건)로 로직 자체는 이미 검증되어 있고, `OAuth2FailureHandler`가 예외를 받아 항상 `email_conflict`로 리다이렉트하는 코드도 컴파일·리뷰로 확인됨. 여분의 구글 테스트 계정이 생기면 그때 실제 왕복으로 마저 확인한다.
- [x] **OAuth 서비스 단위 테스트 1건 통과** — `CustomOAuth2UserServiceTest` 5건(신규가입/재조회/충돌거부/닉네임 미지정/닉네임 절삭) 전부 통과, 전체 스위트 21건 회귀 없음

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

**DoD** (2026-08-28, v1.10 — 전부 실제 빌드/브라우저로 검증 완료)

- [x] **`package.json`의 `next`와 `eslint-config-next`가 모두 15.x임 (16 아님)** — `15.5.24` 확인
- [x] **소스가 `src/` 아래에 있음** (`src/app/`, `src/components/`, `src/lib/`, `src/types/`)
- [x] **`package.json`에 `@tiptap/extension-link`가 없음** (v3 StarterKit 내장, `@tiptap/starter-kit@3.30.5` 확인)
- [x] Node 20 이상에서 빌드됨 (Node 24.18.0)
- [x] `npm run build` 성공, 출력 디렉토리가 `.next`
- [x] `package.json`에 `motion`이 있고 `framer-motion`이 없음
- [x] 디자인 토큰이 OS 다크 설정에 따라 전환됨 (`class` 조작 없이 CSS만으로)
- [x] 페이지 `page.tsx`에 `"use client"`가 붙어 있음
- [x] **`components.json`의 `style`이 `new-york`임**
- [x] **`globals.css`에 `.dark` 클래스 셀렉터나 `@custom-variant dark (&:is(.dark *))`가 없고, 다크 토큰이 `@media (prefers-color-scheme: dark)` 안에 정의되어 있음**
- [x] 디자인 토큰 값이 `CLAUDE.md` 8장 팔레트와 일치함 (배경 `#FAFAFA`/`#0A0A0A`, 액센트 `#4F46E5`, 우선순위 3색)
- [x] Pagination 컴포넌트 단독 동작 확인 (더미 데이터, Playwright로 실제 클릭). **페이지 수 1 이하일 때 아무것도 렌더하지 않음** — `totalPages=1`로 토글 시 네비게이션 전체가 사라짐을 확인
- [x] `ErrorState`가 재시도 버튼과 함께 렌더되고, 버튼 클릭이 `onRetry`를 호출함 — Playwright로 클릭 후 카운트 증가 확인
- [x] `apiClient`가 `data` 언래핑과 401 처리를 수행함 (`TOKEN_EXPIRED`는 `/auth/refresh` 1회 자동 시도, 그 외 401은 즉시 `todo-auth:logout` 이벤트) — 코드 리뷰로 검증. 실제 백엔드가 없어(Phase 3 미착수) 살아있는 서버 대상 통합 테스트는 Phase 7에서 수행
- [x] **`apiClient`가 던진 에러를 `lib/errorMessages.ts`에 넣으면 `PRD.md` 5.1 표의 문구가 그대로 나옴** (네트워크 실패 케이스 포함) — 코드 리뷰로 검증. 실서버 기준 종단 검증은 Phase 7에서 수행

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
- `useAuth` 훅 (로그인 / **로그아웃: 서버 `POST /api/v1/auth/logout` 호출 + 토큰 삭제 + 캐시 초기화** / 현재 사용자)
  - **클라이언트 토큰 삭제만으로 끝내지 않는다.** 서버 API가 Refresh Token을 DB에서 폐기하고 httpOnly 쿠키를 만료시켜야 로그아웃이 실제로 완료된다 (`CLAUDE.md` 6장). 이 API는 **Phase 3에서 이미 구현되어 있으므로 여기서는 연결만 한다**
  - **서버 호출이 실패해도(네트워크 오류·서버 다운) 클라이언트 정리와 `/login` 이동은 그대로 진행한다.** (2026-09-01 확정) 로그아웃을 중단하면 서버가 죽어 있는 동안 사용자가 로그아웃조차 못 하고 갇힌다. 남은 Refresh Token은 httpOnly 쿠키라 JS로 지울 수 없지만, 서버가 복구된 뒤 `/auth/refresh` 시점에 정리되고 최대 14일 후 만료된다 — **갇히는 쪽이 더 나쁜 경험이라고 판단해 가용성을 택했다.** 실패해도 사용자에게 에러를 띄우지 않는다
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

- [x] 이메일 가입·로그인 정상 동작 — 실제 signup/login e2e로 확인
- [x] **가입 화면에서 한글 25자 비밀번호를 입력하면 제출 전에 바이트 초과 안내가 인라인으로 뜸** (서버 400에만 의존하지 않음 — `AUTH-02`) — 24자(72바이트, 경계값)는 통과, 25자(75바이트)에서 실시간 에러 확인
- [x] **중복 이메일 가입 시 "이미 사용 중인 이메일입니다."가 이메일 입력 아래 인라인으로 뜸** (`AUTH-03`, `PRD.md` 5.1)
- [x] **미가입 이메일과 비밀번호 오류의 화면 문구가 동일함** ("이메일 또는 비밀번호가 올바르지 않습니다.") — 계정 존재 여부가 드러나지 않음
- [x] **백엔드를 내린 채 로그인을 시도하면 "연결에 실패했습니다." + 재시도 버튼이 나옴** (네트워크 실패 매핑 — `UX-04`) — 문구는 정상 표시. 별도 "재시도" 버튼은 추가하지 않고 **기존 "로그인" 버튼이 그 역할을 겸하도록 설계**했다(입력값이 보존된 채로 다시 클릭하면 재시도된다). 백엔드 재기동 후 같은 버튼으로 재클릭해 성공 확인
- [ ] 구글 로그인 → 콜백 → `/todos` 이동, URL에서 토큰 제거됨 — ⏸ **보류.** 실제 Google 자격증명이 없어(Phase 5) 왕복 전체는 검증 불가. 다만 `/oauth/callback` 자체의 로직(토큰 저장 → `/todos`로 `replace` → URL에서 토큰 제거)은 curl로 발급받은 실제 유효 토큰을 콜백 URL에 수동으로 넣어 확인했다 — `href`가 `/todos`로 바뀌고 `localStorage`에 토큰이 저장됨을 확인
- [x] **`/oauth/callback`에 `?token=` 없이 직접 접근하면 `/login`으로 이동함** (`PRD.md` 5.4)
- [x] **`/oauth/callback` 화면에 공통 헤더와 조작 가능한 요소가 없음** — `Header` 컴포넌트를 아예 import하지 않고 스켈레톤만 렌더
- [x] **헤더에 닉네임이 보이고 이메일은 어디에도 렌더되지 않음** (DevTools에서 DOM 검색 — `AUTH-08`) — 접근성 트리 전체를 덤프해 이메일 문자열이 어디에도 없음을 확인
- [x] 계정 충돌 시 안내 문구 노출 — `/login?error=email_conflict`로 직접 접근해 배너 렌더 확인(실제 백엔드발 리다이렉트는 Phase 5 자격증명 필요, 프론트 표시 로직은 독립적으로 검증됨)
- [x] **로그아웃 시 서버 `POST /api/v1/auth/logout`이 실제로 호출되고**(DevTools Network 확인), 토큰·캐시가 모두 제거되며 `/login`으로 이동 — 네트워크 로그에서 `POST .../auth/logout 200` 확인, `localStorage` null 확인
- [x] **로그아웃 직후 `/api/v1/auth/refresh`를 호출하면 401** — 서버의 Refresh Token 폐기가 실제로 이뤄졌는지 확인한다. 클라이언트 토큰만 지우고 끝낸 구현은 이 항목에서 걸러진다 (`AUTH-06`, `CLAUDE.md` 6장) — 브라우저에서 직접 fetch로 재현, `401 INVALID_REFRESH_TOKEN` 확인
- [x] 새로고침해도 로그인 상태 유지
- [x] 미인증 상태로 `/todos` 접근 시 로그인으로 이동
- [x] **만료된 토큰을 localStorage에 직접 넣고 `/todos`에 접근했을 때, 보호 화면이 한 프레임도 노출되지 않고 곧바로 `/login`으로 이동** — `exp`가 과거인 위조 JWT로 재현, `/login`으로 즉시 이동하고 "할 일 목록" 텍스트가 DOM에 나타나지 않음을 확인
  > `useAuth`가 토큰 존재 여부만 보면 만료 토큰이 판정을 통과해, 401 왕복 동안 보호 화면이 노출된다. `exp`를 디코드해야 한다 (`CLAUDE.md` 9장). 검증용 만료 토큰은 `JwtTokenProvider`의 `ACCESS_TOKEN_EXPIRATION`(현재 `Duration.ofMinutes(30)`)을 일시적으로 낮춰 발급받는다 — **환경변수가 아니라 코드 상수이므로** 검증 후 값을 되돌리는 것을 잊지 말 것
- [x] `middleware.ts` 파일이 존재하지 않음
- [x] `npm run build` 성공 (`useSearchParams` Suspense 경계 확인)
- [x] **모든 입력에 label 연결, Tab·Enter만으로 가입·로그인 완주 가능** — 로그인 화면에서 Tab→입력→Tab→입력→Enter만으로 `/todos` 진입 확인

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

- [x] **목록이 페이지당 정확히 10건씩 끊기고, 항목 순서가 생성일 내림차순임** (시드 데이터의 `created_at`과 대조 — `TODO-05`, `TODO-06`) — 배치1: DB `created_at DESC` 상위 10건과 화면이 완전히 일치
- [x] 20건 이상에서 페이지네이션 정상 — seed-dev 계정(10,101건, 1,010페이지)으로 실측
- [x] **정렬 기준을 고르는 UI가 화면에 없음** (생성일 내림차순 고정 — `PRD.md` 1장 비목표) — 화면에 정렬 UI 없음 확인
- [x] 검색·필터 상태가 URL에 반영되고 새로고침·뒤로가기에서 유지됨 — 배치1 실측
- [x] **완료 처리한 항목의 제목에 취소선과 흐린 색상이 적용됨** (`PRD.md` 5.5) — 배치1 + 배치5 재확인(화면 캡처로 취소선 확인)
- [x] `npm run build` 성공 — 배치5에서 최종 재확인(`/todos` 7.36 kB, `/todos/[id]` 1.39 kB, `/todos/new` 599 B, 전체 라우트 정상 생성)
- [x] Tiptap 내용 저장 후 재조회 시 서식 유지 (툴바 항목 전부) — 배치2: H2·굵게·불릿목록 적용 후 DB `content` 컬럼 직접 조회로 확인
- [x] 본문에 `# `, `~~취소선~~`, `---`를 입력해도 서식이 생성되지 않음 (저장 후 소실되는 입력이 없음) — 배치2 실측
- [x] **본문에서 `Ctrl+U`를 눌러도 밑줄(`<u>`)이 생성되지 않음** (v3 StarterKit의 Underline이 꺼져 있는지 확인 — 켜져 있으면 저장 시 서식이 조용히 사라진다) — 배치2 실측
- [x] 2페이지 이상에서 마지막 항목 삭제 시 이전 페이지로 이동 — 배치1(11건 전용 계정) + 배치5 재확인
- [x] **2페이지 마지막 항목 삭제가 실패했을 때, 사용자가 보고 있는 화면에서 롤백이 눈으로 확인됨** (페이지가 먼저 넘어가 버려 롤백이 안 보이면 실패) — 배치5: `window.fetch`를 monkey-patch해 `DELETE`만 실패시켜 재현. 항목이 그대로 남고 페이지도 이동하지 않음을 확인(코드가 `onSuccess`에서만 페이지를 이동하도록 이미 작성돼 있어, 낙관적 업데이트가 없는 이 Phase에서는 "실패 시 아무 것도 먼저 바꾸지 않는다"는 설계로 롤백이 자연히 충족됨)
- [x] 타인 소유 id로 접근 시 "찾을 수 없습니다" 화면 표시 + **"목록으로 가기" 버튼이 있고, 자동 리다이렉트가 일어나지 않음** (`TODO-14`, `PRD.md` 5.6) — 배치3 실측
- [x] `/todos/[id]` 로딩 중 **폼 형태 스켈레톤**이 보임 (`UX-01`) — 배치3: `FormSkeleton` 추가·확인
- [x] **백엔드를 내린 채 저장을 누르면 입력한 제목·본문이 그대로 남아 있고 에러가 표시됨** (`TODO-13`, `UX-04`) — 배치3 실측(탭 전환으로 인한 백그라운드 refetch 상황까지 포함해 폼이 사라지지 않음을 확인)
- [x] **`/todos/[id]`에서 삭제하면 `/todos`로 이동함** (`TODO-12`) — 배치3 실측(Soft Delete 확인 포함)
- [x] **`setContent()` 직전에 `lib/sanitize.ts`가 호출됨** (본문에 `<script>`가 섞인 데이터를 DB에 직접 넣고 상세 화면 진입 시 실행되지 않음) — 배치3: DB에 `<script>`·`onerror` 핸들러를 직접 삽입해 재현, 실행되지 않음을 확인
- [x] `/todos/new`와 `/todos/[id]`가 `TodoForm`을 재사용 — 배치2·3 구현 및 코드 확인
- [x] 수정 화면에서 저장해도 완료 상태가 바뀌지 않음 — 배치3 실측
- [x] 변경 후 이탈 시 확인 대화상자 노출 (**새로고침 / 페이지 내 취소 버튼 / 브라우저 뒤로가기 3경로 모두**) — 배치4: `window.confirm` 몽키패치로 호출 여부·인자·반환값을 추적해 3경로 모두 실측(popstate는 Next.js 라우터와의 리스너 등록 순서 문제를 근본 수정한 뒤 확인)
- [x] **저장 직후에는 확인 대화상자가 뜨지 않음** (`dirty` 해제 확인) — 배치4: `key={updatedAt}` 리마운트로 해결, 실측 확인
- [x] **본문이 있는 할 일을 열어 아무것도 고치지 않고 나갈 때 확인 대화상자가 뜨지 않음** (Tiptap 정규화로 dirty가 오판되지 않는지 — 서식이 섞인 본문으로 시험한다) — 배치4: `onReady` 스냅샷으로 정규화 값끼리 비교하도록 수정 후 실측 확인
- [x] 4가지 화면 상태 모두 눈으로 확인 — 배치1(로딩/에러/빈상태/검색결과없음) + 배치5: `fetch`에 인위적 지연을 걸어 로딩 스켈레톤(`role="status" aria-label="불러오는 중"`, 항목형 3개)이 실제로 렌더링됨을 시각적으로 재확인
- [x] **에러 상태의 재시도 버튼을 눌렀을 때 실제로 재요청이 나가고, 서버를 다시 올리면 목록이 정상 렌더됨** (`UX-04` — 버튼이 보이기만 하고 동작하지 않는 경우를 걸러낸다) — 배치1 실측
- [x] **빈 상태 문구("아직 할 일이 없어요")와 검색 결과 없음 문구가 서로 다름** (`UX-03`) — 배치1 실측
- [x] 360px 화면에서 레이아웃 정상 (가로 스크롤 없음) — 배치1은 "자동화 환경의 브라우저 창이 최대화 고정이라 실측 불가"로 남겼던 항목. 배치5: `resize_window` 도구도 실제 뷰포트를 바꾸지 못함을 재확인한 뒤, `360×800` `<iframe>`을 페이지에 주입해(iframe은 자신의 뷰포트 기준으로 미디어 쿼리가 평가됨) 목록·작성·상세 3개 화면을 모두 확인 — 세 화면 전부 `scrollWidth <= clientWidth`로 가로 스크롤 없음을 확인
- [x] **키보드만으로 할 일 생성·완료 토글·삭제 수행 가능** — 배치1(토글·삭제: `Tab`/`Space`/`Enter`) + 배치5(생성: `Tab`/`Shift+Tab`/`Enter`만으로 목록→작성 이동→제목 입력→Tiptap 툴바 10개를 지나 본문 포커스→본문 입력→저장까지 완주, 목록에 정상 반영 확인)

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

**DoD** (2026-09-02 실측 검증 완료)

- [x] 토글·삭제 시 대기 시간 없이 즉시 반영 — `onMutate`로 클릭 즉시 체크박스·취소선 스타일이 바뀜을 브라우저로 확인
- [x] **체크박스를 빠르게 연타해도 새로고침 후 상태가 UI와 일치** — 항목을 3회 연타 후 네트워크 로그로 PATCH 3건이 겹치지 않고 순차 처리됨을 확인, DB(`SELECT completed`)와 화면 값이 일치함을 확인
- [x] **연타를 멈춘 뒤 최종 상태가 "마지막에 클릭한 값"과 일치하며, 잠시 후 반대 값으로 되돌아가지 않음** — 항목별 `scope: { id: 'todo-toggle-{id}' }` 직렬화로 요청이 클릭 순서대로 서버에 도달함을 네트워크 로그로 확인(방법 2였다면 재조회만 한 번으로 줄일 뿐 도착 순서는 보장하지 못했을 것)
  > 앞 항목만 보면 결함이 통과한다. `invalidateQueries`가 어떤 값으로든 수렴시키므로 "UI와 서버가 일치"는 항상 참이 된다. 문제는 **수렴한 값이 사용자 의도와 다를 수 있다는 것**이다
- [x] 서버를 내린 상태에서 실패 → UI 롤백 + 알림 확인 — `window.fetch`를 monkey-patch해 `/toggle`·`DELETE`만 각각 실패시켜 재현(Phase 8 배치5와 동일한 기법). 토글은 즉시 원래 값으로 롤백 + "연결에 실패했습니다." 토스트 노출, 삭제는 항목이 사라졌다가 되돌아오고 **페이지 이동이 일어나지 않음**(2페이지 마지막 항목 케이스, 전용 테스트 계정으로 재현)을 확인
- [x] 애니메이션이 200ms 이내, 과하지 않음 — 진입 `duration:0.2`+`stagger 30ms`, 삭제 `duration:0.15`, 체크 펄스 `duration:0.2`로 코드에 고정. 기능 테스트 중 렌더링 오류·콘솔 에러 없이 자연스럽게 표시됨을 확인
- [x] 애니메이션 관련 import가 모두 `motion/react`에서 이루어짐 — `TodoItem.tsx`·`TodoList.tsx` 전량 확인, `framer-motion` 참조 없음
  > `prefers-reduced-motion` 라이브 에뮬레이션은 이번 세션의 브라우저 자동화 도구로 안정적으로 재현하지 못해, `useReducedMotion()` 분기(진입 `initial=false`, exit 단순화, `layout` 비활성화, 체크 펄스 스킵)가 코드에 전부 걸려 있음을 코드 리뷰로 확인하는 선에서 마무리했다. 실제 OS 설정으로 재현해보고 싶다면 Chrome DevTools의 Rendering 탭에서 `prefers-reduced-motion: reduce`를 켜고 `/todos`를 새로고침해 확인한다.

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
- [ ] 로그아웃 시 **서버 Refresh Token 폐기**(`/auth/refresh` 재시도 401) + 클라이언트 토큰·캐시 제거
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
