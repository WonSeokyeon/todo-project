# 개발 도구 가이드

이 문서는 `todo-project` 의 코드 품질 도구 구성과 사용법을 설명합니다.

---

## 1. 저장소 구조

이 프로젝트는 **독립된 git 저장소 3개**로 구성되어 있습니다.

```
todo-project/          ← 저장소 ①: 문서 및 프로젝트 메타데이터
├── docs/
├── .editorconfig
├── .gitignore
│
├── todo-frontend/     ← 저장소 ②: Next.js (husky + lint-staged)
│   ├── .husky/
│   ├── eslint.config.mjs
│   ├── .prettierrc.json
│   └── .lintstagedrc.mjs
│
└── todo-backend/      ← 저장소 ③: Spring Boot (셸 훅)
    ├── .githooks/
    └── src/main/resources/
```

> **왜 훅 설정이 두 벌인가**
> git 훅은 저장소 단위로 설정됩니다(`core.hooksPath`).
> 프론트엔드와 백엔드가 서로 다른 저장소이므로, 훅도 각각 설치해야 합니다.
> 루트 저장소는 문서만 담으므로 훅을 두지 않습니다.

> **왜 백엔드는 husky 를 쓰지 않는가**
> husky 는 사실상 `core.hooksPath` 설정과 스크립트 디렉터리 관리가 전부입니다.
> Java 프로젝트에 그것 하나 때문에 `package.json` 과 `node_modules` 를 들이는 대신,
> 같은 메커니즘을 순수 셸 스크립트로 구현했습니다. 추가 의존성이 0개입니다.

---

## 2. 처음 클론했을 때 (필수)

### 프론트엔드

```bash
cd todo-frontend
npm install          # prepare 스크립트가 husky 를 자동 설정합니다
```

### 백엔드

```bash
cd todo-backend

# Windows
.githooks\install.cmd

# macOS / Linux / Git Bash
sh .githooks/install.sh

# 로컬 설정 파일 준비
cp src/main/resources/application-local.properties.example src/main/resources/application-local.properties
# → 복사한 파일에 로컬 DB 비밀번호를 채웁니다
```

**백엔드 훅 설치를 건너뛰면 커밋 검사가 전혀 동작하지 않습니다.**

---

## 3. 도구 구성 한눈에 보기

| 영역 | 도구 | 역할 |
|---|---|---|
| 프론트 · 린트 | ESLint 9 (Flat Config) | 코드 오류 및 프로젝트 규칙 검사 |
| 프론트 · 서식 | Prettier 3 | 서식 통일 (+ Tailwind 클래스 자동 정렬) |
| 프론트 · 타입 | `next typegen` + `tsc --noEmit` | 타입 검사 |
| 프론트 · 훅 | husky + lint-staged | 커밋 전 자동 검사·수정 |
| 프론트 · 커밋 | commitlint | 커밋 메시지 형식 검사 |
| 백엔드 · 훅 | 셸 스크립트 (`.githooks/`) | 비밀 유출 차단, 커밋 메시지 검사, 컴파일 검증 |
| 공통 | EditorConfig | 들여쓰기·인코딩·줄바꿈 통일 |
| 공통 | `.gitattributes` | 줄바꿈 LF 정규화 |

---

## 4. 커밋할 때 무슨 일이 일어나는가

### 프론트엔드에서 `git commit` 하면

```
1. pre-commit
   └─ lint-staged
      ├─ 스테이징 안 된 변경을 임시 보관 (stash)
      ├─ *.ts, *.tsx → eslint --fix → prettier --write → npm run typecheck
      ├─ 성공: 자동 수정된 내용을 다시 스테이징
      └─ 실패: 원래 상태로 완전 복구 후 커밋 중단
2. commit-msg
   └─ commitlint: Conventional Commits 형식 검사
```

### 백엔드에서 `git commit` 하면

```
1. pre-commit  (빠름 — 1초 이내)
   ├─ 비밀 파일이 커밋 대상에 있는가?      (.env, application-local.*, *.pem, *.key …)
   ├─ 추가된 줄에 하드코딩된 비밀이 있는가? (JWT, AWS 키, 리터럴 비밀번호)
   └─ 빌드 산출물이 섞였는가?              (target/, *.class)
2. commit-msg
   └─ Conventional Commits 형식 검사
```

### 백엔드에서 `git push` 하면

```
pre-push
└─ ./mvnw test-compile 로 컴파일 검증
```

컴파일은 수 초가 걸리므로 커밋마다 하지 않고, 상대적으로 드문 push 시점에 한 번만 확인합니다.

---

## 5. 커밋 메시지 규칙

프론트엔드(commitlint)와 백엔드(셸 스크립트)가 **동일한 규칙**을 적용합니다.

```
<타입>(<범위>): <제목>

[본문]

[꼬리말]
```

| 항목 | 규칙 |
|---|---|
| 타입 | `feat` `fix` `docs` `style` `refactor` `perf` `test` `build` `ci` `chore` `revert` |
| 범위 | 선택. 예: `auth`, `todo` |
| 제목 | 한글 가능. 72자 이내. 끝에 마침표 없음 |
| 본문 | 선택. 제목과 빈 줄 하나로 구분 |

```bash
# 통과
git commit -m "feat: 할 일 생성 API 추가"
git commit -m "fix(auth): 리프레시 토큰 회전 시 이전 토큰이 폐기되지 않던 문제 수정"
git commit -m "test(todo): 소유권 검증 테스트 추가"

# 거부
git commit -m "할일 API 추가"      # 타입이 없음
git commit -m "feat: API 추가."     # 제목 끝에 마침표
git commit -m "feature: API 추가"   # 허용되지 않는 타입
```

---

## 6. 자주 쓰는 명령

### 프론트엔드

```bash
cd todo-frontend
npm run validate      # 타입 검사 + 린트(경고 0) + 서식 검사 — 커밋 전 전체 확인
npm run lint:fix      # 린트 자동 수정
npm run format        # 서식 일괄 적용
npm run typecheck     # 타입 검사만
```

### 백엔드

```bash
cd todo-backend
./mvnw test-compile           # 컴파일 검증
./mvnw test                   # 테스트
sh .githooks/pre-commit       # 훅을 직접 실행해 미리 확인
```

---

## 7. 문제 해결

### 훅이 동작하지 않습니다

```bash
git config --get core.hooksPath
# 프론트엔드 → .husky/_ 가 나와야 합니다  (아니면 npm install 재실행)
# 백엔드     → .githooks 가 나와야 합니다  (아니면 install 스크립트 재실행)
```

### 파일을 건드리지 않았는데 전부 수정된 것으로 표시됩니다

줄바꿈 문제입니다. `.gitattributes` 가 적용되기 전에 체크아웃된 파일일 수 있습니다.

```bash
git add --renormalize .
```

### `pre-commit` 이 비밀이 아닌 값을 비밀로 잘못 판단합니다

검사는 보수적으로 설계되어 있어 드물게 오탐이 날 수 있습니다.
값이 실제로 비밀이 아님을 확인한 뒤에만 건너뛰세요.

```bash
git commit --no-verify
```

반복해서 걸린다면 `.githooks/pre-commit` 의 `LITERAL_HIT` 패턴을 조정하는 편이 낫습니다.

### 커밋이 너무 느립니다

프론트엔드의 `pre-commit` 은 타입 검사(프로젝트 전체)를 포함하므로 몇 초가 걸립니다.
타입 검사는 파일 단위로 나눌 수 없어(그렇게 하면 `tsconfig.json` 이 무시됨) 불가피한 비용입니다.
급할 때는 `--no-verify` 로 건너뛰되, push 전에 `npm run validate` 를 반드시 한 번 실행하세요.

---

## 8. 아직 도입하지 않은 것

다음 도구들은 이번 설정에서 제외했습니다. 필요해지면 추가를 검토하세요.

| 도구 | 용도 | 비고 |
|---|---|---|
| Spotless | Java 코드 포매터 | 현재 백엔드는 EditorConfig 로만 서식을 맞춥니다 |
| secretlint | 비밀 탐지 (전용 도구) | 현재는 셸 스크립트로 주요 패턴만 검사합니다 |
| GitHub Actions | CI 자동 검증 | 원격 저장소가 준비되면 도입을 검토합니다 |
| Vitest / JUnit 커버리지 | 테스트 커버리지 | 테스트 코드가 쌓인 뒤 검토합니다 |
