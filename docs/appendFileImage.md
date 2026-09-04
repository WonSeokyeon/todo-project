# Tiptap 이미지 첨부 기능 — 원본 프롬프트 분석 및 재설계

> **작성일** 2026-09-04
> **대상 문서** `docs/tiptap-s3-image-upload-prompt.md`
> **목적** 원본 프롬프트를 실제 코드베이스와 대조해 검증하고, 수정·추가·삭제 항목을 확정한 뒤 재설계안을 제시한다.
>
> 이 문서는 **분석 결과와 설계 결정**을 담는다. 실제 작업 지시서는 이 결정을 반영해 원본 프롬프트를 재작성하는 형태로 만든다.

---

## 1. 검증 방법

원본 프롬프트는 구조는 좋으나 **실제 코드를 보지 않고 일반적인 Spring + Next 프로젝트를 가정하고** 작성되었다. 따라서 문서의 모든 전제를 다음 근거로 하나씩 대조했다.

| 대조 대상 | 근거 |
|---|---|
| 백엔드 구조·엔티티·설정 | `todo-backend/src` 전수 확인 (패키지 트리, `Todo.java`, `SecurityConfig`, `HtmlSanitizer`, `pom.xml`, `application*.properties`) |
| 프론트엔드 구조·에디터·정화 | `todo-frontend/src` 전수 확인 (`apiClient.ts`, `sanitize.ts`, `tiptapExtensions.ts`, `TodoForm.tsx`, `package.json`) |
| 규칙 정본 | 루트 `CLAUDE.md` v1.12, `docs/PRD.md` v1.10, `docs/ROADMAP.md` |

### 확정된 방침 (사용자 승인 완료)

| 항목 | 결정 |
|---|---|
| S3 전환 시점 | **이번 범위에서 제외.** 로컬 스토리지까지 완성하고 S3는 Phase 11(AWS 배포) 이후로 미룬다 |
| 저장 HTML의 `img` | **`data-attachment-id`만 저장하고 `src`는 정화 단계에서 제거** |
| `@tiptap/extension-image` | **추가 승인** (CLAUDE.md 절대규칙 11에 따른 사전 승인 절차 완료) |
| 문서 갱신 순서 | **구현 전에 문서 정본(PRD·CLAUDE.md·ROADMAP)을 먼저 수정하고 커밋** |

---

## 2. 결론 요약

원본을 그대로 실행하면 다음 세 가지가 발생한다.

### 2-1. 이미지가 저장 직후 조용히 사라진다 (치명적)

원본은 **XSS 정화를 단 한 줄도 언급하지 않는다.** 그러나 이 프로젝트는 저장 시 Jsoup, 렌더 시 DOMPurify로 이중 정화하며, 실측 결과 **양쪽 화이트리스트에 `img`가 없다.**

```java
// todo-backend  service/HtmlSanitizer.java (현재)
Safelist.none().addTags("p","br","strong","em","h2","h3","ul","ol","li","a","code","pre","blockquote")
// img 없음
```

```ts
// todo-frontend  src/lib/sanitize.ts (현재)
ALLOWED_TAGS = ["p","br","strong","em","h2","h3","ul","ol","li","a","code","pre","blockquote"];
// img 없음
```

업로드는 성공하고 에디터에도 이미지가 들어가지만, `TodoService.create()/update()`가 호출하는 `htmlSanitizer.sanitize()`가 `<img>`를 통째로 제거한다. **저장 버튼을 누르는 순간 이미지만 사라지고 에러는 나지 않는다.** 원인 추적이 가장 어려운 종류의 결함이다.

CLAUDE.md 6장은 **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify 설정 네 곳이 항상 같은 태그 집합을 가리켜야 한다**고 불변식으로 규정하고 있는데, 원본 문서는 이 규칙의 존재 자체를 모른다.

### 2-2. "프론트 코드가 로컬/S3에서 동일하다"는 목표가 달성되지 않는다

원본 3장은 이것을 핵심 설계 목표로 내세우지만, 인증 방식 때문에 성립하지 않는다.

- 로컬 업로드 엔드포인트(`PUT .../upload`)는 JWT 헤더가 필요하다.
- S3 presigned PUT은 `Authorization` 헤더를 붙이면 **서명이 깨져 403이 난다.**

즉 원본 설계대로면 `uploadFile` 안에 분기가 생기고, 문서가 스스로 금지한 "로컬인지 S3인지 판단하는 로직"이 들어간다.

### 2-3. 프로젝트 불변 규칙 위반

- **Flyway 도입 지시** — CLAUDE.md 4장 "MVP에서는 도입하지 않는다"와 정면 충돌. 현재는 `ddl-auto=update`가 엔티티로 테이블을 만든다.
- **`.yml` → `.properties` 전환 단계** — 이미 `.properties` 3종이 존재한다. 부모 CLAUDE.md 절대규칙 9로 확정된 사항이다.

---

## 3. 판정표

### 3-1. 삭제 (6건)

| 위치 | 내용 | 근거 |
|---|---|---|
| 0장 4번 | "`application.yml`이 있다면 `.properties`로 전환" | 이미 `.properties`. 절대규칙 9로 확정 |
| 0장 5번 | "프로파일 분리 여부 — 없다면 새로 만든다" | `application-local/prod.properties` 이미 분리됨 |
| 1장 끝 | "별도 방식이 없다면 Flyway를 도입" | CLAUDE.md 4장 위반 |
| 5장 | `spring.profiles.active=local`을 공통 파일에 명시 | 이미 `${SPRING_PROFILES_ACTIVE:local}`. 덮어쓰면 운영 프로파일 주입이 막힌다 |
| 10장 2단계 | "`.yml` → `.properties` 전환" 단계 | 단계 자체가 불필요 |
| 9장 전체 | S3 전환 작업 | 이번 범위 밖(Phase 13). 인터페이스 계약만 남긴다 |

### 3-2. 수정 (7건)

| 위치 | 원본 서술 | 실제 | 
|---|---|---|
| 0장 3번, 6장 | 본문 컬럼 `description` | **`content`** (`Todo.java`, `@Column(columnDefinition="TEXT")`) |
| 1장 | 테이블 `attachment`, FK → `todo(id)` | 복수형 관례. **`attachments`**, FK → **`todos(id)`** |
| 3·6장 | `/api/attachments/**` | base path는 **`/api/v1`** |
| 6장 검증규칙 | 소유자 불일치 → **403** | 프로젝트 전체가 **404**로 통일 (존재 여부 노출 방지, `TodoService.getOwned()`) |
| 0장 1번 | "`User`(또는 Member)" | `User` 확정. **엔티티가 그대로 `@AuthenticationPrincipal`로 주입됨** |
| 4장 | "두 저장소의 `.gitignore`에 `upload/` 추가" | `upload/`는 **루트 문서 저장소** 바로 아래에 생긴다. 루트 `.gitignore`에 추가해야 한다 |
| 7장 | `lib/api/attachments.ts` | `lib/api/` 디렉토리 없음. `src/lib/`·`src/hooks/`·`src/types/` 관례를 따른다 |

### 3-3. 추가 (9건)

| # | 누락 항목 | 영향 |
|---|---|---|
| **1** | **정화 화이트리스트 4곳 동시 수정** | **미조치 시 기능 자체가 동작하지 않음** (2-1 참조) |
| **2** | **업로드 URL도 서명 토큰으로 인증** | 미조치 시 "프론트 코드 동일" 목표 실패 (2-2 참조) |
| 3 | `ApiResponse<T>` 래핑 | 모든 REST 응답에 예외 없이 적용 (CLAUDE.md 5장) |
| 4 | `ErrorCode` enum 신규 값 + 프론트 union·메시지 맵 동기화 | 백엔드만 고치면 프론트 타입이 어긋난다 |
| 5 | `@EnableScheduling` | 현재 `@Scheduled` 사용처가 0건이라 켜져 있지 않다 |
| 6 | 편집 화면 dirty 판정 오염 | 저장 안 했는데 이탈 확인창이 뜬다 (6-1 참조) |
| 7 | base64 붙여넣기 차단 | `content` 50,000자 제한을 즉시 초과 |
| 8 | 조회 URL 만료(30분)와 React Query 캐시 상호작용 | 탭을 오래 열어두면 이미지가 깨진다 |
| 9 | Soft Delete와 물리 파일 삭제의 관계 | 원본은 즉시 파일 삭제 → 향후 휴지통(PRD 9장 #3) 복원 불가 |

---

## 4. 핵심 설계 결정

원본에서 가장 크게 바뀌는 세 지점이다.

### 4-1. 정화 화이트리스트 — `src` 없는 `img`

네 곳을 동시에 바꾸되, **`src`는 어디서도 허용하지 않는다.**

```java
// todo-backend  service/HtmlSanitizer.java — SAFELIST에 추가
.addTags("img")
.addAttributes("img", "data-attachment-id", "alt")
// src는 추가하지 않는다 → Jsoup이 제거한다
```

```ts
// todo-frontend  src/lib/sanitize.ts
ALLOWED_TAGS: [...기존 13개, "img"]
ALLOWED_ATTR: ["href", "target", "rel", "data-attachment-id", "alt"]
ALLOW_DATA_ATTR: false   // 명시한 data-* 하나만 통과시킨다
```

이렇게 하면 얻는 것이 둘이다.

1. **만료되는 URL이 DB에 박히지 않는다.** 조회 URL은 30분 만료이므로 본문에 저장하면 하루 뒤 전부 깨진다.
2. **임의의 외부 URL을 본문에 심는 경로가 원천 차단된다.** `src`를 허용하면 사용자가 외부 추적 픽셀이나 임의 호스트 이미지를 넣을 수 있다.

> ⚠️ `ALLOWED_ATTR`에 명시한 `data-attachment-id`는 `ALLOW_DATA_ATTR: false`에서도 통과한다(명시 목록이 우선). 구현 시 DOMPurify 공식 문서로 재확인할 것 — CLAUDE.md 3장 "라이브러리 API를 기억에 의존하지 않는다".

#### 렌더링 파이프라인 (순서가 중요)

`src`가 없으므로 화면에 그리기 직전에 주입한다.

```
서버 HTML(src 없음)
  → sanitizeHtml()            ← 정화를 먼저
  → DOM 파싱
  → img[data-attachment-id]에 src 주입   ← 주입을 나중에
  → 렌더
```

**반대로 하면 방금 넣은 `src`를 DOMPurify가 지운다.** 주입은 문자열 이어붙이기가 아니라 `setAttribute`로 하고, 값이 `http(s)`로 시작하는지 확인한다.

적용 지점은 두 곳이다.
- `/todos/[id]` 확인 화면 — `dangerouslySetInnerHTML` 직전
- `/todos/[id]/edit` 수정 화면 — Tiptap `content` 주입 직전

### 4-2. 업로드·조회 URL 모두 서명 토큰으로 인증

원본은 조회 URL에만 서명 토큰을 제안했다. 그러면 업로드는 JWT 헤더가 필요해 S3와 호출 방식이 갈린다. **양쪽 다 서명 토큰으로 통일해야** 원본이 내세운 목표가 실제로 성립한다.

| | 로컬 | S3 (향후) | 프론트 코드 |
|---|---|---|---|
| uploadUrl | `.../attachments/{id}/upload?token=…` | presigned PUT | **동일** |
| viewUrl | `.../attachments/{id}/raw?token=…` | presigned GET | **동일** |

프론트는 두 경우 모두 `Authorization` 없이, `credentials: 'omit'`으로 순수 `fetch` PUT을 보낸다. S3는 헤더가 붙으면 서명이 깨지고 쿠키는 불필요하므로 **이것이 유일하게 양쪽이 성립하는 형태다.**

구현 규칙:

- `JwtTokenProvider`에 `generateAttachmentToken(attachmentId, purpose, ttl)` / 검증 메서드를 추가해 기존 키 관리를 재사용한다.
- 클레임에 `typ: "att-upload" | "att-view"`를 넣고 **검증 시 반드시 `typ`을 확인**한다. 일반 Access Token이 조회 토큰으로 통용되면 안 된다.
- `SecurityConfig` permitAll에 `/api/v1/attachments/*/upload`, `/api/v1/attachments/*/raw` 두 경로를 추가하고 컨트롤러가 토큰을 직접 검증한다 — `/auth/refresh`와 같은 패턴이다.
- **이 두 경로만 `JwtAuthenticationFilter` 대상이 아니다.** 나머지 첨부 API(`presign`·`complete`·`url`·`DELETE`)는 기존대로 Bearer 인증이다.

> `<img>` 태그는 `Authorization` 헤더를 실을 수 없으므로 조회 URL에 서명 토큰이 필요하다는 원본의 판단은 옳다. 같은 논리가 업로드에도 적용된다는 점만 빠져 있었다.

### 4-3. `storage_type`은 기록용이며 런타임 분기를 하지 않는다

원본 1장은 이 컬럼을 "로컬 데이터와 S3 데이터가 섞여도 각각 올바르게 조회하기 위함"이라 설명한다. 그러나 2장이 지시하는 `@ConditionalOnProperty`는 **구현체를 하나만 등록**하므로, 다른 타입 행을 처리할 빈이 애초에 존재하지 않는다. **설명과 구조가 어긋난다.**

| 선택지 | 판단 |
|---|---|
| 두 빈을 모두 등록하고 `Map<StorageType, StorageService>`로 분기 | 로컬 개발에도 AWS SDK가 상시 필요해진다. 과설계 |
| **컬럼은 유지하되 감사·마이그레이션 기록 용도로 한정** | **채택** |

- 로컬 DB(`todolist_db`)와 운영 RDS는 애초에 별개이므로 혼재 자체가 정상 경로가 아니다.
- `storage_type`이 현재 활성 타입과 다른 행에 접근하면 **조용히 실패하지 말고 명시적 예외**를 던진다.

---

## 5. 산출물

### 5-1. 문서 정본 수정 (구현 전, 먼저 커밋)

| 파일 | 변경 |
|---|---|
| `docs/PRD.md` | 25절 비목표에서 "파일 첨부, 이미지 업로드" 삭제 · 9장 향후 후보 #10 삭제 · 3장에 신규 요구사항 항목 추가 · 버전 v1.11 |
| `CLAUDE.md` | 3장 "S3는 MVP 제외" 정정 · 4장 `attachments` 테이블 · 5장 첨부 API · 6장 XSS 절에 `img` 반영 · 8장 툴바에 이미지 버튼 · 11장 신규 `ErrorCode` · 버전 v1.13 |
| `docs/ROADMAP.md` | **Phase 12(이미지 첨부, 로컬)**, **Phase 13(S3 전환)** 신설. 실행 순서는 `12 → 11 → 13`이며 13은 11 완료가 선행 조건임을 명시 |
| `.gitignore` (루트) | `upload/` 추가 |

### 5-2. 작업 지시서 재작성

원본을 대체할 문서의 목차. 파일명은 **`tiptap-image-upload-prompt.md`**로 바꾼다(S3가 이번 범위가 아니므로).

```
0. 전제 (실측 완료 — 재조사 불필요)   ← 원본 0장을 "확인할 것"에서 "확인된 사실"로 전환
1. 범위와 제외
2. DB 스키마 (attachments, Flyway 없음)
3. 정화 화이트리스트 4곳 동시 수정      ← 신설, 최우선
4. 스토리지 추상화 + 서명 토큰 인증
5. 업로드 흐름
6. 로컬 스토리지 구현 (경로 탈출 방어 포함)
7. 백엔드 설정·구현 목록
8. Todo 저장 시 연결 처리
9. 정리 배치
10. 프론트엔드
11. 로컬 테스트 시나리오
12. S3 전환 시 계약 (Phase 13에서 수행)
```

원본 0장이 "코드 수정 전에 파악하고 멈춰라"였던 것을 **"이미 확인된 사실"**로 바꾸는 것이 핵심이다. 이번 분석으로 조사가 끝났으므로 같은 조사를 반복시킬 이유가 없다.

### 5-3. 구현 범위

**백엔드** (`todo-backend` — 계층별 패키지 구조를 따른다. 기능별 아님)

| 패키지 | 추가/수정 |
|---|---|
| `domain/` | `Attachment`, `AttachmentStatus`, `StorageType`, `AttachmentRepository` |
| `service/` | `StorageService` 인터페이스, `LocalStorageService`, `AttachmentService`, `AttachmentCleanupScheduler` |
| `service/HtmlSanitizer` | `img` 태그 추가 + **`data-attachment-id` 수집 메서드** 추가 |
| `service/TodoService` | `create`/`update`에서 첨부 ID 수집 → 소유권 확인 → `LINKED` 전환 → 사라진 첨부 soft delete |
| `controller/` | `AttachmentController` (6개 엔드포인트) |
| `exception/ErrorCode` | `ATTACHMENT_NOT_FOUND`(404), `ATTACHMENT_INVALID_STATE`(409) 추가 |
| `config/` | `JwtTokenProvider` 첨부 토큰 메서드, `SecurityConfig` permitAll 2경로, 메인 클래스에 `@EnableScheduling` |

엔티티 작성 규칙(기존 코드 패턴 준수):
- `BaseEntity` 상속, `softDelete()`는 `LocalDateTime.now(ZoneOffset.UTC)` 명시 (`Todo.java`와 동일)
- `@Setter` 금지 — `link(todo)`, `complete(size)`, `softDelete()` 같은 의미 있는 메서드로만 상태 변경
- 경로 탈출 방어: `base.resolve(key).normalize().startsWith(base)` 확인 후 예외

**프론트엔드** (`todo-frontend`)

| 파일 | 내용 |
|---|---|
| `src/types/attachment.ts` | 백엔드 DTO와 이름 일치 |
| `src/types/api.ts`, `src/lib/errorMessages.ts` | 신규 `ErrorCode` 2개 동기화 |
| `src/lib/attachmentHtml.ts` | `collectAttachmentIds(html)`, `injectAttachmentSrc(sanitizedHtml, urlMap)` |
| `src/lib/sanitize.ts` | 화이트리스트 확장 |
| `src/lib/tiptapExtensions.ts` | `@tiptap/extension-image`를 extend해 `attachmentId` 속성 추가 |
| `src/hooks/useAttachments.ts` | presign/upload/complete/url/delete |
| `src/components/todo/TodoEditor.tsx` | 툴바 이미지 버튼, paste·drop 핸들러 |
| `src/app/(main)/todos/[id]/page.tsx` | 읽기 전용 화면의 URL 주입 |

- **`uploadFile`만 `apiClient`를 우회한 순수 `fetch`를 쓴다.** "API 호출은 `apiClient`로 통일"(CLAUDE.md 10장)의 유일한 예외이며, 반드시 주석으로 근거(S3 presigned URL에 헤더를 붙이면 서명이 깨짐)를 남긴다.
- 쿼리 키에 `queryKeys.attachmentUrls(sortedIds)` 추가. `staleTime`은 토큰 만료(30분)보다 짧게(20분) 두고, `<img onError>`에서 1회만 재요청한다.

### 5-4. 정리 배치 정책 (원본 수정)

원본은 "DELETE → Soft Delete + 실제 파일 삭제"였다. 그러면 PRD 9장 향후 후보 #3(휴지통, Soft Delete 복원)이 불가능해진다. 다음으로 바꾼다.

| 대상 | 조치 |
|---|---|
| `status=TEMP`이고 생성 후 24시간 경과 | 파일 + 행 삭제 (업로드하다 만 고아 파일) |
| `deleted_at`이 30일 경과 | **파일만 삭제, 행은 유지** (감사 기록 보존) |
| 본문에서 사라진 첨부 | `deleted_at`만 채운다. 파일은 즉시 지우지 않는다 |

---

## 6. 특히 조심할 함정

셋 다 늦게 발견되는 종류다.

### 6-1. 편집 화면 dirty 판정 오염

`TodoForm`은 `TodoEditor.onReady`가 넘겨준 **정규화된 HTML**을 dirty 판정 baseline(`contentBaseline`)으로 잡는다. 첨부 URL 주입이 baseline 확정 **이후**에 일어나면, 사용자가 아무것도 고치지 않아도 HTML이 달라져 dirty가 된다.

**증상**: 수정 화면에 들어갔다가 그냥 나가려는데 "변경 사항이 있습니다" 확인창이 뜬다.

**대응**: URL이 모두 해석된 뒤에 `TodoForm`을 마운트한다. 해석 전에는 스켈레톤을 보여준다.

### 6-2. base64 붙여넣기

클립보드 이미지를 그대로 붙이면 Tiptap이 `data:` URI를 본문에 넣는다.

- `content` 50,000자 제한(CLAUDE.md 4장)을 **한 장으로 즉시 초과**한다.
- 정화가 `src`를 지우므로 이미지가 조용히 사라진다.

**대응**: paste 핸들러가 이미지 파일을 가로채 업로드 경로로 보낸다. 드래그앤드롭도 동일.

### 6-3. 저장 왕복 시 HTML 불일치

에디터가 보내는 HTML에는 `src`가 있고, 서버가 저장하는 HTML에는 없다. 저장 후 서버 응답으로 캐시를 덮어쓰므로(`useUpdateTodo`) 동작은 정상이지만, 이를 모르면 **"보낸 것과 응답이 다르다"를 버그로 오인**하기 쉽다. 재작성 문서에 의도된 동작임을 명시한다.

---

## 7. 검증

### 7-1. 문서 단계 (커밋 전)

- `docs/PRD.md`에서 "파일 첨부"가 비목표·향후후보 양쪽에서 사라졌는지 `grep`으로 확인
- `CLAUDE.md` 6장의 태그 목록 네 곳(Jsoup·DOMPurify·Tiptap·툴바)이 전부 `img`를 포함하는지 육안 대조

### 7-2. 구현 단계

원본 8장 시나리오를 이 프로젝트에 맞게 교정한 것. **5·11번이 원본에 없던 항목이다.**

| # | 확인 항목 |
|---|---|
| 1 | 기동 시 `todo-project/upload` 디렉토리 자동 생성 |
| 2 | 첨부 시 `upload/todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}` 실제 생성 |
| 3 | `attachments` 행이 `status=TEMP`로 생성 |
| 4 | 할 일 저장 후 `status=LINKED` + `todo_id` 채워짐 |
| **5** | **저장된 HTML에 `src`가 없고 `data-attachment-id`만 있는지 DB에서 직접 확인** |
| 6 | 확인 화면(`/todos/[id]`)과 수정 화면 양쪽에서 이미지가 보이는지 |
| 7 | 본문에서 이미지 삭제 후 저장 → `deleted_at` 채워지고 파일은 남아 있는지 |
| 8 | 5MB 초과 파일, `.exe` 파일 업로드 거부 |
| 9 | 타인 `attachmentId` 조회 시 **404** (403 아님) |
| 10 | `storageKey`에 `../`를 넣은 요청 차단 |
| **11** | **저장만 하고 나갈 때 이탈 확인창이 뜨지 않는지** (6-1 회귀 테스트) |
| 12 | `git status`에 `upload/`가 잡히지 않는지 |

### 7-3. 실행 명령

```bash
# 백엔드 (test는 local 프로파일을 상속하지 않으므로 환경변수 필요)
cd todo-backend && ./mvnw test
cd todo-backend && ./mvnw spring-boot:run

# 프론트엔드
cd todo-frontend && npm run validate   # typecheck + lint + format:check
cd todo-frontend && npm run dev
```

---

## 8. 별건 — 문서 물리적 손상

이번 작업과 무관하지만 조사 중 발견했다. **`CLAUDE.md`와 `docs/ROADMAP.md`가 파일 자체로 잘려 있다.**

| 파일 | 끊긴 지점 |
|---|---|
| `CLAUDE.md` | 11장 마지막 문장 — `...에러 코드를... (10KB 남음)` |
| `docs/ROADMAP.md` | Phase 11-3 — `...상시 비용... (12KB 남음)` |

`(NKB 남음)`은 컨텍스트 truncation 마커가 파일에 그대로 저장된 것이다. **최초 커밋(`f00422b`)부터 이 상태**라 git에서 복구할 온전한 버전은 없다 — 유실이 아니라 처음부터 잘린 채 작성됐다.

두 문서 모두 **규칙 정본**이므로, 잘린 뒷부분에 어떤 규칙이 있었는지 확인할 수 없는 상태다. 이번 계획에는 포함하지 않았으며 별도 작업으로 복원할지 판단이 필요하다.
