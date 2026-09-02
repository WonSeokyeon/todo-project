# todo-project

개인용 Todo List 풀스택 서비스의 **문서 저장소**입니다. 로그인한 사용자가 할 일을 리치 텍스트로 작성하고 완료/삭제를 즉각적인 반응으로 관리하는 서비스를 만듭니다.

이 저장소 자체는 코드를 담지 않습니다. 기술 규칙과 요구사항, 개발 순서를 정의하고, 실제 구현은 아래 두 독립 저장소에 있습니다.

## 구성

**폴리레포**입니다 — 모노레포가 아니라 3개의 독립된 Git 저장소로 관리합니다.

| 저장소 | 역할 | 스택 |
| --- | --- | --- |
| `todo-project` (이 저장소) | 문서: `CLAUDE.md`(기술 규칙), `docs/PRD.md`(제품 요구사항), `docs/ROADMAP.md`(개발 순서 · 완료 판정 정본) | — |
| [`todo-backend`](./todo-backend) | REST API 서버 | Spring Boot 4.x · JDK 21 · PostgreSQL |
| [`todo-frontend`](./todo-frontend) | 웹 클라이언트 | Next.js 15 (App Router) · React 19 · TypeScript |

각 하위 저장소는 이 폴더 안에 있지만(`todo-project/todo-backend`, `todo-project/todo-frontend`) 별도의 `.git`을 가진 독립 저장소이며, 이 저장소의 `.gitignore`가 두 폴더를 추적하지 않도록 제외합니다. 저장소를 새로 구성한다면 세 저장소를 각각 clone합니다.

## 문서 읽는 순서

1. **[`CLAUDE.md`](./CLAUDE.md)** — 기술 규칙의 단일 기준(Single Source of Truth). 스택 버전, 데이터 모델, API 명세, 인증 설계, 코딩 컨벤션 등 코드를 생성하기 전에 반드시 확인해야 하는 내용입니다. 개별 작업 지시와 충돌하면 이 문서가 우선합니다.
2. **[`docs/PRD.md`](./docs/PRD.md)** — 무엇을 만드는가(제품 요구사항, 화면별 상세 동작, 에러 문구 매핑).
3. **[`docs/ROADMAP.md`](./docs/ROADMAP.md)** — 어떤 순서로 만드는가, 그리고 **각 Phase의 완료 판정 정본**(DoD 체크리스트와 실측 검증 이력).
4. `docs/guides/` — 참고 자료. 스펙이 아니며, `CLAUDE.md`와 충돌하면 `CLAUDE.md`가 우선합니다.

## 로컬에서 전체 서비스 실행하기

1. PostgreSQL을 로컬에 설치하고 `todolist_db`, `todolist_test` 두 데이터베이스를 만듭니다. (Docker는 사용하지 않습니다.)
2. `todo-backend/`에서 `application-local.properties.example`을 복사해 `application-local.properties`를 만들고 DB 비밀번호 등을 채운 뒤 `./mvnw spring-boot:run`. 자세한 내용은 [`todo-backend/README.md`](./todo-backend/README.md) 참고.
3. `todo-frontend/`에서 `npm install && npm run dev`. 자세한 내용은 [`todo-frontend/README.md`](./todo-frontend/README.md) 참고.
4. `http://localhost:3000`으로 접속합니다.

## 저장소 규칙

- 브랜치: `main`(배포 가능 상태) ← `develop` ← `feature/{작업명}`
- 커밋 메시지: `feat:` `fix:` `refactor:` `test:` `docs:` `chore:` (Conventional Commits, 본문은 한글 가능)
- **API 계약이 바뀌면 이 문서 저장소를 먼저 수정**한 뒤 백엔드 → 프론트엔드 순으로 반영합니다.
- Phase 완료 시 해당 저장소에 `v0.{Phase번호}.0` 태그, Phase 10(전체 검증) 통과 시 세 저장소 모두 `v1.0.0` 태그를 답니다.

자세한 내용은 [`CLAUDE.md`](./CLAUDE.md) 2장 "저장소 구조"를 참고하세요.
