# PRD.md 점검 — 삭제된 문서 복구 + PRD 정비

## Context

원래 요청은 `docs/PRD.md` 점검이었다. 그런데 점검 도중 **`D:\my-workspace\claude\` 의 문서 9개가 17:39에 삭제된 것**을 발견했다.
확인 결과 9개 전부 Windows 휴지통에 남아 있어 **완전 복구가 가능**하다.

따라서 이 계획은 두 부분이다.
1. **복구를 먼저 한다.** (되돌릴 수 있는 상태를 확보)
2. 그 다음 원래 목적인 PRD 불일치 수정을 한다.

---

## ⚠️ 사고 경위 (확인된 사실만)

| 시각 | 사실 |
|---|---|
| 17:28 | `ls`로 `D:\my-workspace\claude\` 에 문서 9개 존재 확인 |
| 17:33 | PRD 점검용 `prd-generator` 서브에이전트 실행 (프롬프트에 "파일을 수정하지 말고(읽기 전용)" 명시) |
| 17:39 | 같은 폴더의 문서 9개가 휴지통으로 이동됨 |
| 17:40 | 삭제 발견, 에이전트 강제 중단 (보고서 미산출) |

**삭제 주체는 확정하지 못했다.** 시간대는 에이전트 실행 구간과 겹치지만, 사용자가 탐색기에서 직접 정리했을 가능성도 배제할 수 없다.
`Remove-Item`은 휴지통을 거치지 않으므로, 삭제는 탐색기 또는 Shell API 경로로 이루어졌다.

### 삭제된 파일 (전부 휴지통에 존재)

| 파일 | 크기 | 다른 사본 유무 |
|---|---|---|
| `PRD.md` | 22,575 | ✅ `todo-project/docs/PRD.md` (바이트 동일 확인함) |
| `CLAUDE.md` | 8,921 | ✅ `todo-project/CLAUDE.md` (바이트 동일 확인함) |
| `ROADMAP.md` | 9,899 | ✅ `todo-project/docs/ROADMAP.md` |
| `API.md` | 8,554 | ⚠️ 사본 없음 (단 이번 세션 컨텍스트에 전문 보유) |
| `SCHEMA.md` | 7,180 | ⚠️ 사본 없음 (단 이번 세션 컨텍스트에 전문 보유) |
| `CHECKLIST.md` | 8,364 | ⚠️ 사본 없음 (단 이번 세션 컨텍스트에 전문 보유) |
| `DESIGN.md` | 6,014 | ❌ **사본 없음. 읽은 적 없음 — 휴지통이 유일한 원본** |
| `PROMPTS.md` | 30,893 | ❌ **사본 없음. 읽은 적 없음 — 휴지통이 유일한 원본** |
| `PRD예시.txt` | 2,912 | ❌ **사본 없음. 읽은 적 없음 — 휴지통이 유일한 원본** |

**`DESIGN.md`, `PROMPTS.md`, `PRD예시.txt` 세 개는 휴지통을 비우면 영구 손실된다.** 복구 전까지 휴지통을 비우지 말 것.

---

## Step 0 — 복구 (최우선, 다른 작업보다 먼저)

휴지통에서 원래 위치(`D:\my-workspace\claude\`)로 되돌린다. Shell API의 복원 동사를 쓰므로 원래 경로가 그대로 복원된다.

```powershell
$sh = New-Object -ComObject Shell.Application
$rb = $sh.NameSpace(0xA)
$targets = @('PRD.md','CLAUDE.md','ROADMAP.md','API.md','SCHEMA.md',
             'CHECKLIST.md','DESIGN.md','PROMPTS.md','PRD예시.txt')
foreach ($item in @($rb.Items())) {
  $orig = $rb.GetDetailsOf($item, 1)
  if ($orig -eq 'D:\my-workspace\claude' -and $targets -contains $item.Name) {
    $item.Verbs() | Where-Object { $_.Name -match '복원|Restore|실행 취소|Undo' } |
      Select-Object -First 1 | ForEach-Object { $_.DoIt() }
  }
}
```

복원 검증:
```bash
ls -la "D:/my-workspace/claude/"   # 9개 파일이 모두 돌아왔는지
# 사본이 있는 3개는 무결성 대조까지 가능
diff "D:/my-workspace/claude/PRD.md"    "D:/my-workspace/claude/todo-project/docs/PRD.md"
diff "D:/my-workspace/claude/CLAUDE.md" "D:/my-workspace/claude/todo-project/CLAUDE.md"
```

복원이 실패하면 → 탐색기에서 휴지통을 열어 수동 복원. 그래도 안 되면 컨텍스트에 보유한
`API.md` / `SCHEMA.md` / `CHECKLIST.md` 전문을 재작성해 3개는 살릴 수 있다 (나머지 3개는 불가).

### Step 0-1 — 재발 방지 (복구 직후 권장)

문서가 버전 관리 밖에 있어서 안전망이 휴지통뿐이었다. 최소한 아래 중 하나는 해두는 게 좋다.
- `D:\my-workspace\claude\` 를 git 저장소로 만들어 1회 커밋, 또는
- 사고 시점 상태를 백업 폴더로 복사

*(범위 밖이면 생략 가능. 다만 지금 구조에서는 같은 사고가 또 나면 복구할 수단이 없다.)*

---

## Step 1 — PRD.md 수정 (사용자 결정 반영)

사용자 결정:
- **수정 범위: `docs/PRD.md` 만.** CLAUDE.md · ROADMAP.md · 에이전트 정의는 이번에 건드리지 않는다.
- **저장소 구조: 문서를 현실에 맞춘다** (독립 저장소 3개).
- **문서 경로 불일치: 이번에는 손대지 않는다** — 파일 이동도, §0 경로 표기 수정도 하지 않고 "알려진 한계"로만 기록.

수정 대상 파일: `D:\my-workspace\claude\todo-project\docs\PRD.md`
(주의: 복원 후 `D:\my-workspace\claude\PRD.md` 사본이 다시 생기므로, **둘 중 어느 쪽을 고칠지 확정한 뒤** 진행한다. 기본은 `docs/PRD.md`.)

### 1-1. §8 제약 조건 — 모노레포 → 독립 저장소 3개 🔴

| 현재 | 수정 후 |
|---|---|
| `저장소 \| 모노레포. todo-project/todo-backend, todo-project/todo-frontend` | `저장소 \| **독립 git 저장소 3개** (todo-project=문서, todo-backend, todo-frontend). git 훅이 저장소 단위(core.hooksPath)라 분리했다. 상세는 docs/DEV_TOOLS.md §1` |

근거: `docs/DEV_TOOLS.md` §1이 "독립된 git 저장소 3개"라고 명시하고 분리 이유까지 설명함. 실제 `.git` 3개 존재를 직접 확인함.

### 1-2. §3 범위 표의 자기모순 🟡

포함에는 "비밀번호 재설정(메일 링크 방식)"이 있는데 제외에는 "알림, 리마인더, **이메일 발송**"이라고 적혀 있다.

| 현재 (제외 표) | 수정 후 |
|---|---|
| `알림, 리마인더, 이메일 발송 \| 후속` | `알림·리마인더 메일 \| 후속. **비밀번호 재설정 메일은 포함**(F-41) — 그 외 용도의 발송만 제외` |

### 1-3. NF-30 rate limit 구현 수단 명시 🟡

PRD NF-30 / API.md `429 TOO_MANY_REQUESTS` / CHECKLIST 5-5가 모두 rate limit을 요구하는데,
CLAUDE.md 기술 스택 표에 해당 라이브러리가 없고 절대 규칙 11이 임의 추가를 금지한다 → Phase 2.5에서 막힌다.

| 현재 | 수정 후 |
|---|---|
| `NF-30 \| 비밀번호 재설정 요청에 rate limit(동일 이메일/IP 10분 3회)을 적용한다` | 뒤에 한 문장 추가: `**추가 라이브러리 없이** password_reset_tokens의 created_at 집계로 판정한다. 별도 rate limit 라이브러리를 도입하려면 CLAUDE.md 절대 규칙 11에 따라 사전 승인이 필요하다.` |

### 1-4. §6 화면 목록 — `/error`, `/not-found` 표기 🟡

Next.js App Router에서 이 둘은 라우트 경로가 아니라 `error.tsx` / `not-found.tsx` **파일 컨벤션**이다.

| 현재 | 수정 후 |
|---|---|
| `/error, /not-found \| 오류 화면 \| 불필요 \| F-32` | `(파일 컨벤션) error.tsx / not-found.tsx \| 오류 화면 \| 불필요 \| F-32` |

### 1-5. §6 화면 목록 — 루트 `/` 행 추가 🟡

현재 `/` 진입 시 거동이 어디에도 정의돼 있지 않다.

추가할 행:
```
| `/` | 진입점 (리다이렉트 전용) | 불필요 | 인증 시 `/todos`, 미인증 시 `/login` (F-09) |
```

### 1-6. `GET /api/auth/me` 대응 요구사항 신설 🟡

API.md에 존재하지만 PRD F-xx 어디에도 매핑되지 않는 고아 엔드포인트다.
동시에 "헤더에 사용자 이름/로그아웃 버튼을 어디에 두는가"가 PRD에 없다.

§4.4에 신규 요구사항 1건 추가 (번호는 기존 최대값 다음인 **F-46** 사용 — 중간 삽입 금지):
```
| **F-46** | 헤더/사용자 표시 | 로그인 상태에서 헤더에 사용자 이름과 로그아웃 버튼을 노출한다. 사용자 정보는 `GET /api/auth/me`로 조회한다 |
```
§6 `/todos` 행의 주요 요구사항에 `F-46` 추가.

### 1-7. §4.1 · §5.1 표에 ID 블록 각주 추가 🟢

인증은 F-01~F-10 **+** F-38~F-45, 보안은 NF-01~NF-09 **+** NF-26~NF-31로 쪼개져 있다.
**재번호는 하지 않는다** (API.md·ROADMAP.md·CHECKLIST.md를 전부 건드려야 해서 위험 대비 이득이 없다).
각 표 위에 한 줄만 넣는다:
```
> 이 절의 ID는 F-01~F-10과 F-38~F-45 두 블록으로 나뉜다. 뒤쪽은 Refresh Token·비밀번호 재설정이 나중에 추가되며 붙은 번호다.
```

### 1-8. §3 포함 범위에 한글 README 추가 🟢

§9 제목이 "알려진 한계 (README에 명시할 것)"인데 README 작성 자체가 범위에 없다.
포함 목록에 `- 한글 README (설치·실행·알려진 한계)` 추가.

### 1-9. §9 알려진 한계에 2건 추가 🟡

사용자 결정에 따라 "고치지 않고 기록만" 하기로 한 항목들이다.
```
8. **PRD가 참조하는 docs/API.md·SCHEMA.md·DESIGN.md·CHECKLIST.md는 실제로는 상위 폴더(D:/my-workspace/claude/)에 있다.**
   §0의 경로 표기와 실제 위치가 다르며, 해당 문서들은 git 저장소 밖에 있어 버전 관리되지 않는다.
9. **PRD.md와 CLAUDE.md가 두 벌씩 존재한다** (상위 폴더 / todo-project). 현재는 내용이 같지만
   한쪽만 수정하면 즉시 갈라진다. 정본 위치 결정은 후속 과제다.
```

---

## 확인했으나 문제 없던 것 (수정 불필요)

| 항목 | 결과 |
|---|---|
| F-01~F-45 번호 연속성 | 빠짐 없음 ✅ |
| NF-01~NF-31 번호 연속성 | 빠짐 없음 ✅ |
| F-13 페이지네이션 ↔ API.md (`size` 1~50, 기본 10) | 일치 ✅ |
| F-16 정렬 ↔ API.md (`createdAt`/`dueDate`/`priority`) | 일치 ✅ |
| F-21 Soft Delete ↔ SCHEMA.md `deleted_at` | 일치 ✅ |
| NF-03 소유권 404 ↔ API.md `TODO_NOT_FOUND` | 일치 ✅ |
| ROADMAP의 CHECKLIST 참조 번호(5-1~5-5, 6-1, 9-1, 11-1~11-4) | 전부 실존 ✅ |
| 인증 정책(Access 30분 / Refresh 14일 / SHA-256 / 회전 / 재사용 감지) | PRD·CLAUDE.md·API.md·SCHEMA.md 4자 일치 ✅ |

---

## 이번 범위에서 제외 (사용자 결정 — 후속 과제로 남김)

| 항목 | 사유 |
|---|---|
| 문서 4개를 `docs/` 로 이동 | 사용자가 "지금은 손대지 않음" 선택. §9에 기록만 |
| CLAUDE.md §1 "모노레포" 수정 | 수정 범위를 PRD.md로 한정. **PRD와 CLAUDE.md가 서로 다른 말을 하게 되는 점은 감수** |
| ROADMAP Phase 0 "모노레포 스캐폴딩" 수정 | 동일 |
| CHECKLIST 2번 `/actuator/health` ↔ actuator 의존성 부재 | 동일. Phase 0에서 실제로 막힐 수 있음 |
| CLAUDE.md `application-local.yml` ↔ 실제 `.properties` 확장자 불일치 | 동일. `.gitignore` 패턴이 빗나가 **DB 비밀번호가 커밋될 수 있는** 지점이라 우선순위 높음 |
| `prd-generator` 에이전트 정의 갱신 (JWT 24h·Kakao·TodoListDB·S3·API_SPEC.md) | 동일. 갱신 전까지 이 에이전트는 계속 낡은 기준으로 PRD를 생성함 |

---

## 검증

```bash
# 1. 복구 확인 — 9개 파일 복귀
ls -la "D:/my-workspace/claude/" | wc -l

# 2. 복원본 무결성 (사본이 있는 3개)
diff "D:/my-workspace/claude/PRD.md"    "D:/my-workspace/claude/todo-project/docs/PRD.md"
diff "D:/my-workspace/claude/CLAUDE.md" "D:/my-workspace/claude/todo-project/CLAUDE.md"

# 3. PRD 수정 반영 확인
cd "D:/my-workspace/claude/todo-project"
grep -n "모노레포" docs/PRD.md          # 결과 없어야 정상
grep -n "독립 git 저장소 3개" docs/PRD.md
grep -n "F-46" docs/PRD.md              # 기능 명세 + 화면 목록 두 곳에 나와야 함
grep -c "^| \*\*F-" docs/PRD.md         # 46 (기존 45 + 신규 1)

# 4. 신규 ID가 다른 문서와 충돌하지 않는지
grep -rn "F-46" docs/ROADMAP.md         # 결과 없음 = 정상(로드맵 미배치는 후속)
```

문서 변경만이라 빌드·테스트 영향은 없다.
