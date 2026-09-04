# Tiptap 에디터 이미지 첨부 기능 추가 (로컬 스토리지)

> **작성일** 2026-09-04 · **대체 대상** `docs/tiptap-s3-image-upload-prompt.md`
> **근거** `docs/appendFileImage.md` (원본 분석 및 재설계 — 삭제 6건 / 수정 7건 / 추가 9건 판정)
>
> 이 문서는 클로드 코드에 그대로 붙여넣는 **작업 지시서**다.
> 원본과 달리 **0장은 "확인할 것"이 아니라 "확인이 끝난 사실"**이다. 같은 조사를 반복하지 말 것.

---

## 0. 전제 (실측 완료 — 재조사 불필요)

2026-09-04에 두 저장소를 전수 확인한 결과다. 이 내용을 다시 조사하느라 시간을 쓰지 말고, **틀린 부분을 발견하면 진행을 멈추고 보고**하라.

### 저장소

`todo-backend`와 `todo-frontend`는 **독립된 git 저장소**다(모노레포 아님). 백엔드를 먼저 완성하고 프론트로 넘어간다.

### 백엔드 (`todo-backend`)

| 항목 | 확인된 사실 |
|---|---|
| 패키지 | `com.example.todoapp` — **계층별** 구조(`domain` / `service` / `controller` / `dto` / `config` / `exception`). 기능별 아님 |
| 스택 | Spring Boot **4.1.1**, JDK 21, jjwt 0.12.6, jsoup 1.21.1, springdoc 3.1.0 |
| 본문 컬럼 | **`Todo.content`** (`@Column(columnDefinition = "TEXT")`) — `description` 아님 |
| API base path | **`/api/v1`** |
| 응답 포맷 | 모든 REST 응답이 `ApiResponse<T>`(record: `success` / `data` / `error`) 래핑 |
| 인증 주체 | `User` **엔티티가 그대로** `@AuthenticationPrincipal User user`로 주입됨. 커스텀 UserDetails 없음 |
| 소유권 위반 | **404** (`TODO_NOT_FOUND`) — 존재 여부 노출 방지. **403이 아니다** |
| Soft Delete | `deleted_at`에 `LocalDateTime.now(ZoneOffset.UTC)` 명시. 물리 삭제 금지 |
| 스키마 관리 | **Flyway·Liquibase 없음.** 로컬은 `ddl-auto=update`가 엔티티로 테이블 생성. 운영 DDL 추출은 Phase 11 |
| 설정 파일 | **이미 `.properties`** — `application.properties` / `-local` / `-local.example` / `-prod` 4종 존재. `spring.profiles.active=${SPRING_PROFILES_ACTIVE:local}` |
| 스케줄링 | `@Scheduled`·`@EnableScheduling` **사용처 0건** — 이번에 켜야 한다 |
| AWS SDK | **없음** |
| 테스트 | 실제 PostgreSQL(`todolist_test`) 사용, `ddl-auto=create-drop`. `./mvnw test`는 `local` 프로파일을 상속하지 않아 `DB_PASSWORD`·`JWT_SECRET` 환경변수를 셸에서 직접 설정해야 한다 |

### 프론트엔드 (`todo-frontend`)

| 항목 | 확인된 사실 |
|---|---|
| 스택 | Next.js **15.5.24**(App Router), React 19, TS strict, Tailwind 4 |
| Tiptap | `@tiptap/react` ^3.30.5 + `@tiptap/starter-kit` ^3.30.5 **둘뿐**. 별도 extension 패키지 없음 |
| 정화 | **`dompurify` ^3.4.14** (`isomorphic-dompurify` 아님) |
| API 호출 | `src/lib/apiClient.ts`로 통일 — `get/post/put/patch/delete<T>()`가 `ApiResponse`를 **언랩해서 `data`만 반환**. `credentials: "include"` 고정, 401·`TOKEN_EXPIRED` 시 자동 refresh 1회 |
| Base URL | `` `${process.env.NEXT_PUBLIC_API_BASE_URL ?? ""}/api/v1` `` |
| 쿼리 키 | `src/lib/queryClient.ts`의 `queryKeys` — `todos(filters)` / `todo(id)` / `me()` |
| 디렉토리 | `src/{app,components,hooks,lib,types}`. **`lib/api/` 디렉토리는 없다** |
| 에디터 | `src/lib/tiptapExtensions.ts`의 `buildTiptapExtensions()` → `src/components/todo/TodoEditor.tsx`가 사용 |
| 파일 업로드 | 관련 코드 **전무** (`FormData`·`input[type=file]`·`next/image` 매치 0건) |
| 폼 라이브러리 | 없음(`useState` + 수동 검증). `react-hook-form`·`zod` 미설치이며 **설치 금지** |

### 이번 작업으로 승인된 예외

| 항목 | 승인 내용 |
|---|---|
| 라이브러리 추가 | **`@tiptap/extension-image`** (CLAUDE.md 절대규칙 11에 따른 사전 승인 완료) |
| `apiClient` 우회 | **`uploadFile` 하나만** 순수 `fetch` 사용 (4-2 참조). 그 외 전부 `apiClient` |

---

## 1. 범위와 제외

### 목표

할 일 작성/수정 시 Tiptap 본문에 **이미지를 첨부**할 수 있게 한다. 파일 실체는 로컬 디렉토리에 저장하고 메타데이터는 PostgreSQL에서 관리한다.

### 이번 범위

- `attachments` 테이블 및 첨부 API
- `StorageService` 인터페이스 + **`LocalStorageService`**
- 정화 화이트리스트 4곳 확장
- Tiptap 이미지 노드 · 업로드 UX · 조회 URL 주입
- 고아 파일 정리 배치

### 제외 (Phase 13에서 수행)

- **`S3StorageService` 구현** — AWS SDK 의존성도 이번에 추가하지 않는다
- AWS 버킷·IAM·CORS 설정

> **왜 미루는가**: Phase 11(AWS 배포)이 아직 시작 전이라 버킷과 IAM이 존재하지 않는다. 실행 순서는 **Phase 12(이 작업) → Phase 11(배포) → Phase 13(S3 전환)**이다.
>
> 다만 **인터페이스 계약은 지금 확정**한다(12장). S3 전환 시 프론트 코드가 한 줄도 바뀌지 않아야 한다.

### 선행 조건

**문서 정본을 먼저 수정하고 커밋한 뒤** 구현에 들어간다(CLAUDE.md 6장 "문서 먼저"). 이 기능은 현재 `docs/PRD.md` 25절에 **비목표로 명시**돼 있다.

| 파일 | 변경 |
|---|---|
| `docs/PRD.md` | 25절 비목표에서 "파일 첨부, 이미지 업로드" 삭제 · 9장 향후후보 #10 삭제 · 3장에 요구사항 항목 신설 |
| `CLAUDE.md` | 3장 "S3는 MVP 제외" 정정 · 4장 `attachments` 테이블 · 5장 첨부 API · 6장 XSS 절 `img` 반영 · 8장 툴바 · 11장 `ErrorCode` |
| `docs/ROADMAP.md` | Phase 12 · Phase 13 신설 (실행 순서 `12 → 11 → 13` 명시) |
| 루트 `.gitignore` | `upload/` 추가 |

---

## 2. DB 스키마

### `attachments` 테이블

> **복수형이다.** 기존 테이블이 `users` · `todos` · `refresh_tokens`로 전부 복수형이다.

| 컬럼 | 타입 | 제약 |
|---|---|---|
| `id` | BIGSERIAL | PK |
| `user_id` | BIGINT | FK → `users(id)`, NOT NULL — 업로더(권한 검증용) |
| `todo_id` | BIGINT | FK → `todos(id)`, **NULL 허용** |
| `storage_type` | VARCHAR(20) | NOT NULL — `LOCAL` / `S3` |
| `storage_key` | VARCHAR(512) | NOT NULL, UNIQUE |
| `original_filename` | VARCHAR(255) | NOT NULL |
| `content_type` | VARCHAR(100) | NOT NULL |
| `file_size` | BIGINT | NOT NULL — presign 시 신고값, complete 시 **실측값으로 갱신** |
| `status` | VARCHAR(20) | NOT NULL — `TEMP` / `LINKED` |
| `created_at` | TIMESTAMP | NOT NULL (`BaseEntity`) |
| `updated_at` | TIMESTAMP | NOT NULL (`BaseEntity`) |
| `deleted_at` | TIMESTAMP | NULL (Soft Delete) |

**인덱스**: `idx_attachments_todo` on `(todo_id)` · `idx_attachments_status_created` on `(status, created_at)` · `idx_attachments_user` on `(user_id)`

**저장 키 규칙**: `todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}` (로컬·S3 동일)

### `todo_id`를 NULL 허용으로 두는 이유

에디터에서는 할 일을 저장하기 **전에** 이미지가 먼저 업로드된다. 업로드 시점에는 `status=TEMP`, `todo_id=NULL`로 만들고, 할 일 저장/수정 시 본문에 실제로 남아 있는 첨부만 `LINKED`로 전환하며 `todo_id`를 채운다.

### ⚠️ `storage_type`은 기록용이며 런타임 분기를 하지 않는다

`@ConditionalOnProperty`는 **구현체를 하나만 등록**하므로, 다른 타입 행을 처리할 빈이 애초에 존재하지 않는다. 이 컬럼의 용도를 **감사·향후 마이그레이션 기록**으로 한정한다.

- 로컬 DB(`todolist_db`)와 운영 RDS는 별개이므로 타입 혼재는 정상 경로가 아니다.
- `storage_type`이 현재 활성 타입과 다른 행에 접근하면 **조용히 실패하지 말고 명시적 예외**를 던진다.

### ⚠️ 마이그레이션 파일을 만들지 않는다

**Flyway를 도입하지 않는다.** CLAUDE.md 4장이 "MVP에서는 도입하지 않는다"로 확정했다. 로컬은 `ddl-auto=update`가 엔티티로부터 테이블을 만든다. 운영 DDL 추출·적용은 Phase 11의 절차를 따른다.

### 엔티티 작성 규칙

기존 코드 패턴을 그대로 따른다.

- `BaseEntity` 상속 (`created_at` / `updated_at` 자동)
- **`@Setter` 금지.** 상태 변경은 의미 있는 메서드로 — `complete(long actualSize)`, `link(Todo todo)`, `softDelete()`
- `softDelete()`는 `LocalDateTime.now(ZoneOffset.UTC)`를 **명시**한다 (`Todo.java`와 동일)
- `@ManyToOne(fetch = FetchType.LAZY)` — `user`, `todo` 둘 다
- 정적 팩토리 `Attachment.createTemp(...)` + `@NoArgsConstructor(access = PROTECTED)`

---

## 3. 정화 화이트리스트 4곳 동시 수정 ⚠️ 최우선

**이 장을 건너뛰면 기능이 아예 동작하지 않는다.** 업로드는 성공하고 에디터에도 이미지가 들어가지만, 저장 버튼을 누르는 순간 `HtmlSanitizer.sanitize()`가 `<img>`를 통째로 제거한다. **에러 없이 이미지만 사라진다.**

CLAUDE.md 6장은 **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify 설정 네 곳이 항상 같은 태그 집합**을 가리켜야 한다고 규정한다. 네 곳을 한 커밋에서 함께 바꾼다.

### 3-1. `src`를 어디서도 허용하지 않는다

저장되는 HTML은 다음 형태다.

```html
<p>회의 자료</p>
<img data-attachment-id="42" alt="스크린샷">
```

`src`가 없다. 이유가 둘이다.

1. **만료되는 URL이 DB에 박히지 않는다.** 조회 URL은 30분 만료라 본문에 저장하면 하루 뒤 전부 깨진다.
2. **임의의 외부 URL을 본문에 심는 경로가 원천 차단된다.** 외부 추적 픽셀이나 임의 호스트 이미지를 넣을 수 없다.

### 3-2. 백엔드 — Jsoup

```java
// service/HtmlSanitizer.java
private static final Safelist SAFELIST = Safelist.none()
        .addTags("p", "br", "strong", "em", "h2", "h3", "ul", "ol", "li",
                 "a", "code", "pre", "blockquote",
                 "img")                                        // 추가
        .addAttributes("a", "href")
        .addAttributes("img", "data-attachment-id", "alt")      // 추가 — src는 넣지 않는다
        .addProtocols("a", "href", "http", "https", "mailto")
        .addEnforcedAttribute("a", "rel", "noopener noreferrer")
        .addEnforcedAttribute("a", "target", "_blank");
```

`Jsoup.clean(html, "", SAFELIST, OUTPUT_SETTINGS)` 형태와 `prettyPrint(false)`는 **그대로 유지**한다(코드 블록 공백 보존).

### 3-3. 프론트엔드 — DOMPurify

```ts
// src/lib/sanitize.ts
const ALLOWED_TAGS = [
  "p", "br", "strong", "em", "h2", "h3", "ul", "ol", "li",
  "a", "code", "pre", "blockquote",
  "img",                                                   // 추가
];
const ALLOWED_ATTR = [
  "href", "target", "rel",
  "data-attachment-id", "alt",                             // 추가 — src는 넣지 않는다
];

export function sanitizeHtml(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS,
    ALLOWED_ATTR,
    ALLOW_DATA_ATTR: false, // 명시한 data-* 하나만 통과시킨다
  });
}
```

> ⚠️ `ALLOWED_ATTR`에 명시한 `data-attachment-id`는 `ALLOW_DATA_ATTR: false`에서도 통과한다(명시 목록이 우선). **구현 전에 DOMPurify 공식 문서로 재확인하라** — CLAUDE.md 3장 "라이브러리 API를 기억에 의존하지 않는다".

### 3-4. Tiptap 확장 · 툴바

10장 참조. `Image.configure({ inline: false, allowBase64: false })`로 등록하고 툴바에 이미지 버튼을 추가한다.

### 3-5. ⚠️ 렌더링 파이프라인 — 정화가 먼저, 주입이 나중

`src`가 없으므로 화면에 그리기 직전에 주입한다. **순서를 뒤집으면 방금 넣은 `src`를 DOMPurify가 지운다.**

```
서버 HTML(src 없음)
  → sanitizeHtml()                      ← ① 정화
  → DOM 파싱
  → img[data-attachment-id]에 src 주입   ← ② 주입
  → 렌더
```

```ts
// src/lib/attachmentHtml.ts (신규)

/** 본문에서 첨부 ID를 모두 수집한다. 조회 URL 일괄 요청에 쓴다. */
export function collectAttachmentIds(html: string): number[] { ... }

/** 정화가 끝난 HTML에 조회 URL을 주입한다. 반드시 sanitizeHtml() 이후에 호출한다. */
export function injectAttachmentSrc(sanitizedHtml: string, urlMap: Map<number, string>): string {
  const template = document.createElement("template");
  template.innerHTML = sanitizedHtml;
  template.content.querySelectorAll("img[data-attachment-id]").forEach((el) => {
    const id = Number(el.getAttribute("data-attachment-id"));
    const url = urlMap.get(id);
    // 문자열 이어붙이기가 아니라 setAttribute로 넣고, 스킴을 확인한다
    if (url && /^https?:\/\//.test(url)) el.setAttribute("src", url);
  });
  return template.innerHTML;
}
```

적용 지점은 두 곳이다.

- `/todos/[id]` 확인 화면 — `dangerouslySetInnerHTML` 직전
- `/todos/[id]/edit` 수정 화면 — Tiptap `content` 주입 직전

> `document`를 쓰므로 클라이언트 전용이다. 두 페이지 모두 `"use client"`이므로 문제없다.

---

## 4. 스토리지 추상화 + 서명 토큰 인증

### 4-1. `StorageService` 인터페이스

로컬과 S3를 같은 인터페이스로 다룬다. **`AttachmentService`는 이 인터페이스만 의존하고 어떤 구현체인지 알지 못한다.**

```java
public interface StorageService {

    StorageType getType();

    /** 업로드용 URL 발급 (로컬: 서명 토큰이 붙은 백엔드 엔드포인트 / S3: presigned PUT) */
    String createUploadUrl(Long attachmentId, String storageKey, String contentType);

    /** 조회용 URL 발급 (로컬: 서명 토큰이 붙은 백엔드 엔드포인트 / S3: presigned GET) */
    String createViewUrl(Long attachmentId, String storageKey);

    /** 업로드 완료 후 실제 파일 존재 및 크기 확인. 없으면 예외 */
    long verifyUploaded(String storageKey);

    void delete(String storageKey);
}
```

구현체는 `app.storage.type` 값에 따라 `@ConditionalOnProperty`로 **하나만** 등록한다.

- `LocalStorageService` — `havingValue = "local"`
- `S3StorageService` — Phase 13에서 추가

### 4-2. ⚠️ 업로드·조회 URL 모두 서명 토큰으로 인증 (핵심)

**"프론트 코드가 로컬/S3에서 동일하다"는 목표는 인증 방식을 통일해야만 성립한다.**

- 로컬 업로드 엔드포인트에 JWT 헤더를 요구하면 → S3 presigned PUT은 `Authorization` 헤더가 붙는 순간 **서명이 깨져 403**이 난다.
- 따라서 로컬도 헤더가 아니라 **URL 쿼리의 단기 서명 토큰**으로 인증한다.

| | 로컬 | S3 (Phase 13) | 프론트 코드 |
|---|---|---|---|
| uploadUrl | `.../attachments/{id}/upload?token=…` | presigned PUT | **동일** |
| viewUrl | `.../attachments/{id}/raw?token=…` | presigned GET | **동일** |

프론트는 두 경우 모두 **`Authorization` 없이, `credentials: "omit"`으로** 순수 `fetch` PUT을 보낸다. S3는 헤더가 붙으면 서명이 깨지고 쿠키는 불필요하므로, **이것이 유일하게 양쪽이 성립하는 형태다.**

> `<img>` 태그는 `Authorization` 헤더를 실을 수 없으므로 조회 URL에 서명 토큰이 필요하다 — 이 판단은 원본 문서에도 있었다. **같은 논리가 업로드에도 적용된다는 점만 빠져 있었다.**

### 4-3. 서명 토큰 구현

기존 `config/JwtTokenProvider`에 메서드를 추가해 키 관리를 재사용한다. 새 시크릿을 만들지 않는다.

```java
// config/JwtTokenProvider.java 에 추가
private static final Duration UPLOAD_TOKEN_EXPIRATION = Duration.ofMinutes(10);
private static final Duration VIEW_TOKEN_EXPIRATION   = Duration.ofMinutes(30);

/** 첨부 전용 단기 토큰 발급. sub=attachmentId, typ=purpose */
public String generateAttachmentToken(Long attachmentId, AttachmentTokenPurpose purpose);

/** 검증 후 attachmentId 반환. typ이 기대값과 다르면 예외 */
public Long parseAttachmentToken(String token, AttachmentTokenPurpose expected);
```

- 클레임: `sub` = attachmentId, `typ` = `"att-upload"` / `"att-view"`, `exp`
- **검증 시 반드시 `typ`을 확인한다.** 일반 Access Token(`typ` 없음)이 조회 토큰으로 통용되면 안 되고, 조회 토큰이 업로드에 쓰여도 안 된다.
- jjwt 0.12 문법(`Jwts.parser().verifyWith(key).build()`)을 그대로 쓴다. 0.11 예제를 옮기지 말 것.

### 4-4. SecurityConfig

permitAll에 **두 경로만** 추가한다.

```
/api/v1/attachments/*/upload
/api/v1/attachments/*/raw
```

- 이 둘은 `JwtAuthenticationFilter`(Bearer 검사) 대상이 아니며, **컨트롤러가 쿼리의 서명 토큰을 직접 검증**한다. `/api/v1/auth/refresh`와 같은 패턴이다.
- 나머지 첨부 API(`presign` · `complete` · `urls` · `DELETE`)는 **기존대로 Bearer 인증**이다.

---

## 5. 업로드 흐름

```
1. 프론트 → 백엔드    POST /api/v1/attachments/presign   { filename, contentType, fileSize }
2. 백엔드             검증 후 attachments 행 생성(status=TEMP) + 업로드 URL 발급
                      ← { attachmentId, uploadUrl, storageKey }
3. 프론트 → uploadUrl PUT (파일 본문, Content-Type 헤더)
                      ※ Authorization 없음 · credentials: "omit"
4. 프론트 → 백엔드    POST /api/v1/attachments/{id}/complete
5. 백엔드             실제 파일 존재·크기·매직바이트 확인 후 확정
                      ← { attachmentId, viewUrl }
6. 프론트             에디터 노드의 attachmentId를 채우고 src를 viewUrl로 교체
```

프론트는 **`uploadUrl`이 어디를 가리키는지 신경 쓰지 않는다.** 그래서 Phase 13에서 S3로 바꿔도 프론트 코드가 그대로다.

### API 목록

| 메서드 | 경로 | 인증 | 응답 |
|---|---|---|---|
| POST | `/api/v1/attachments/presign` | Bearer | `ApiResponse<AttachmentPresignResponse>` |
| PUT | `/api/v1/attachments/{id}/upload?token=` | **서명 토큰** | `ApiResponse<Void>` |
| POST | `/api/v1/attachments/{id}/complete` | Bearer | `ApiResponse<AttachmentResponse>` |
| GET | `/api/v1/attachments/urls?ids=1,2,3` | Bearer | `ApiResponse<List<AttachmentUrlResponse>>` |
| GET | `/api/v1/attachments/{id}/raw?token=` | **서명 토큰** | **파일 스트림** (아래 참조) |
| DELETE | `/api/v1/attachments/{id}` | Bearer | `ApiResponse<Void>` |

> **`raw`만 `ApiResponse` 래핑에서 제외된다.** 바이너리 스트림이라 JSON으로 감쌀 수 없다. CLAUDE.md 5장의 "예외 없이 모든 REST 응답"에 대한 두 번째 예외이므로(첫 번째는 OAuth2 302 리다이렉트), 문서에 명시한다.
>
> **조회는 일괄(batch)이다.** 본문에 이미지가 N개면 요청도 N번이 되는 것을 막는다. 원본의 `GET /{id}/url` 단건은 쓰지 않는다.

### DTO (전부 record)

```java
AttachmentPresignRequest(String filename, String contentType, long fileSize)
AttachmentPresignResponse(Long attachmentId, String uploadUrl, String storageKey)
AttachmentResponse(Long id, String originalFilename, String contentType, long fileSize, String viewUrl)
AttachmentUrlResponse(Long id, String viewUrl)
```

---

## 6. 로컬 스토리지 구현

### 6-1. 저장 위치

`todo-project/upload` 디렉토리를 쓴다. `todo-backend` · `todo-frontend`와 형제 레벨이다.

```
todo-project/
├── todo-backend/
├── todo-frontend/
└── upload/          ← 여기
```

- 경로는 설정값으로 주입한다(**하드코딩 금지**)
- 애플리케이션 시작 시 디렉토리가 없으면 생성한다
- ⚠️ **`upload/`는 루트 문서 저장소 바로 아래에 생긴다.** 따라서 **루트 `todo-project/.gitignore`**에 `upload/`를 추가한다. (원본은 "두 하위 저장소의 `.gitignore`"라고 했으나 잘못이다 — 하위 저장소는 이 경로를 추적하지 않는다)

### 6-2. 경로 조작 방어 (필수)

`storageKey`로 상위 디렉토리 탈출(`../`)이 가능하면 서버 파일이 노출된다.

```java
Path base = Path.of(baseDir).toAbsolutePath().normalize();
Path target = base.resolve(storageKey).normalize();
if (!target.startsWith(base)) {
    throw new BusinessException(ErrorCode.INVALID_INPUT, "잘못된 저장 경로입니다.");
}
```

저장·조회·삭제 **모든 경로**에서 확인한다.

### 6-3. 업로드 엔드포인트 `PUT /api/v1/attachments/{id}/upload?token=`

- 서명 토큰을 검증해 `attachmentId`를 얻고, **경로의 `{id}`와 일치하는지 확인**한다
- `status`가 `TEMP`인지 확인. 이미 파일이 있으면 **409 `ATTACHMENT_INVALID_STATE`**
- 요청 본문을 **스트림으로** 파일에 기록한다. **메모리 전체 적재 금지**
- 크기 제한을 **두 번** 건다
  1. `Content-Length`를 먼저 확인해 상한 초과 시 즉시 거부(불필요한 전송 차단)
  2. 스트림 복사 중 누적 바이트를 세어 초과하면 중단하고 **쓰다 만 파일을 삭제**

> ⚠️ **`server.tomcat.max-swallow-size`에 의존하지 말 것.** 이 값은 "읽지 않은 요청 본문을 얼마나 버릴지"를 정하는 값이지 업로드 크기 상한이 아니다. 원본 문서의 설명은 틀렸다.

### 6-4. 조회 엔드포인트 `GET /api/v1/attachments/{id}/raw?token=`

- 서명 토큰(`typ=att-view`) 검증 → `attachmentId` 일치 확인
- `deleted_at IS NULL` 확인
- `Content-Type`과 함께 파일 스트림 반환, `Content-Disposition: inline`
- **`Cache-Control: private, max-age=…`**로 토큰 만료 이내 캐시 허용

### 6-5. 파일 검증

`contentType`은 클라이언트가 보낸 값이라 그대로 믿을 수 없다.

- **화이트리스트**: `image/jpeg` · `image/png` · `image/gif` · `image/webp`
- **매직바이트 확인**: `complete` 단계에서 파일 앞부분을 읽어 선언된 타입과 일치하는지 검사한다
  - JPEG `FF D8 FF` · PNG `89 50 4E 47` · GIF `47 49 46 38` · WebP `RIFF….WEBP`
  - 불일치 시 파일을 삭제하고 **400 `INVALID_INPUT`**
- 확장자는 `contentType`에서 **역산**한다. 원본 파일명의 확장자를 그대로 쓰지 않는다

---

## 7. 백엔드 설정 · 구현 목록

### 7-1. 설정

**`application.properties` (공통)** — 기존 내용은 건드리지 않고 아래만 추가한다.

```properties
# 첨부 파일 (이미지 업로드)
app.storage.type=${STORAGE_TYPE:local}
app.upload.max-file-size=5242880
app.upload.allowed-content-types=image/jpeg,image/png,image/gif,image/webp
app.storage.upload-url-expiry-minutes=10
app.storage.view-url-expiry-minutes=30
```

> ⚠️ **`spring.profiles.active=local`을 새로 적지 말 것.** 이미 `${SPRING_PROFILES_ACTIVE:local}`로 되어 있고, 고정값으로 덮으면 운영 프로파일 주입이 막힌다.

**`application-local.properties`**

```properties
app.storage.type=local
app.storage.local.base-dir=${LOCAL_UPLOAD_DIR:../upload}
app.storage.local.base-url=${LOCAL_STORAGE_BASE_URL:http://localhost:8080}
```

`application-local.properties.example`에도 같은 키를 플레이스홀더로 추가한다.

**`application-prod.properties`** — Phase 13에서 `app.storage.type=s3`와 버킷 설정을 넣는다. 이번에는 건드리지 않는다.

### 7-2. 클래스 목록

| 패키지 | 추가/수정 |
|---|---|
| `domain/` | `Attachment`, `AttachmentStatus`, `StorageType`, `AttachmentRepository` |
| `service/` | `StorageService`(인터페이스), `LocalStorageService`, `AttachmentService`, `AttachmentCleanupScheduler` |
| `service/HtmlSanitizer` | **수정** — `img` 허용 + `data-attachment-id` 수집 메서드 추가 |
| `service/TodoService` | **수정** — 8장 연결 처리 |
| `controller/` | `AttachmentController` |
| `dto/` | 5장의 record 4종 |
| `config/JwtTokenProvider` | **수정** — 4-3의 첨부 토큰 메서드 |
| `config/SecurityConfig` | **수정** — 4-4의 permitAll 2경로 |
| `TodoBackendApplication` | **수정** — `@EnableScheduling` 추가 |
| `exception/ErrorCode` | **수정** — 아래 2개 추가 |

### 7-3. `ErrorCode` 추가

| 코드 | 상태 | 용도 |
|---|---|---|
| `ATTACHMENT_NOT_FOUND` | **404** | 없는 첨부 · **타인 소유** · 삭제됨 |
| `ATTACHMENT_INVALID_STATE` | 409 | 이미 업로드된 첨부에 재업로드 등 |

- **소유권 위반은 403이 아니라 404다.** `TodoService.getOwned()`와 동일한 정책(존재 여부 노출 방지).
- 크기·타입 위반은 기존 `INVALID_INPUT`(400), 서명 토큰 무효는 기존 `UNAUTHORIZED`(401)를 쓴다.
- ⚠️ **프론트 `src/types/api.ts`의 `ErrorCode` union과 `src/lib/errorMessages.ts`의 `MESSAGES`에 같이 추가**한다. 백엔드만 고치면 타입이 어긋난다.

### 7-4. Swagger

기존 컨트롤러와 동일하게 `@SecurityRequirement(name = "bearerAuth")`를 붙인다. 단 `upload` · `raw`는 Bearer가 아니므로 붙이지 않고, 토큰 쿼리 파라미터를 `@Parameter`로 문서화한다.

---

## 8. Todo 저장 시 연결 처리

`TodoService`의 `create()` · `update()`에 추가한다. **정화 직후, 저장 직전**에 수행한다.

1. 정화된 `content` HTML을 파싱해 `data-attachment-id` 값을 모두 수집한다 (`HtmlSanitizer`에 메서드 추가 — Jsoup을 이미 쓰므로 별도 파서를 들이지 않는다)
2. 수집한 첨부를 조회해 **모두 요청자 소유인지 확인**한다. 타인 것이 섞여 있으면 **404 `ATTACHMENT_NOT_FOUND`**
3. 해당 첨부를 `LINKED`로 전환하고 `todo_id`를 채운다
4. 기존에 이 Todo에 연결돼 있었으나 **본문에서 사라진** 첨부는 `deleted_at`을 채운다 (파일은 즉시 지우지 않는다 — 9장)

> `create()`는 Todo가 저장된 뒤에야 `todo_id`를 알 수 있다. **save 이후에 연결 처리**를 하되 같은 트랜잭션 안에서 수행한다.

---

## 9. 정리 배치

`@Scheduled`를 처음 쓰므로 `TodoBackendApplication`에 **`@EnableScheduling`을 추가**해야 한다.

`service/AttachmentCleanupScheduler` — 하루 1회.

| 대상 | 조치 |
|---|---|
| `status=TEMP`이고 `created_at`이 24시간 경과 | **파일 + 행 삭제** (업로드하다 만 고아 파일) |
| `deleted_at`이 30일 경과 | **파일만 삭제, 행은 유지** (감사 기록 보존) |

### ⚠️ 본문에서 사라진 첨부의 파일을 즉시 지우지 않는 이유

원본은 "DELETE → Soft Delete + 실제 파일 삭제"였다. 그러면 `PRD.md` 9장 향후 후보 #3(**휴지통 — Soft Delete 복원**)이 구조적으로 불가능해진다. 행만 살리고 바이트를 버리면 복원해도 이미지가 깨진다.

유예 기간(30일)을 두면 Soft Delete 철학과 저장 비용이 양립한다.

---

## 10. 프론트엔드

### 10-1. Tiptap 이미지 노드

```ts
// src/lib/tiptapExtensions.ts
import Image from "@tiptap/extension-image";

// 첨부 ID를 data-attachment-id로 왕복시키는 커스텀 이미지 노드.
// src는 저장되지 않으므로(3장) 렌더 시점에 주입한다.
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

// buildTiptapExtensions()에 추가
AttachmentImage.configure({ inline: false, allowBase64: false })
```

> ⚠️ **`allowBase64: false`가 중요하다.** 클립보드 이미지를 그대로 붙이면 `data:` URI가 본문에 들어가는데, `content` 50,000자 제한(CLAUDE.md 4장)을 **한 장으로 즉시 초과**한다. 게다가 정화가 `src`를 지우므로 이미지가 조용히 사라진다.

### 10-2. 업로드 경로 3가지

`useEditor`의 `editorProps`에 핸들러를 단다.

- 툴바 이미지 버튼 (`lucide-react`의 `ImageIcon`, 기존 툴바 9개 옆에 추가)
- `handlePaste` — 클립보드의 이미지 파일을 가로챈다
- `handleDrop` — 드래그앤드롭

세 경로 모두 같은 업로드 함수를 호출한다.

### 10-3. 업로드 UX

1. 파일 선택 즉시 **로컬 blob URL로 임시 미리보기** 노드 삽입(업로드 중 표시)
2. presign → PUT → complete 순차 호출
3. 성공 시 노드의 `attachmentId`를 채우고 `src`를 실제 URL로 교체, **blob URL은 `URL.revokeObjectURL()`로 해제**
4. 실패 시 노드 제거 + `sonner` 토스트

### 10-4. API 함수

`src/hooks/useAttachments.ts` — React Query 뮤테이션으로 감싼다. (원본의 `lib/api/attachments.ts`는 이 프로젝트에 없는 디렉토리다)

```ts
presignUpload(...)     // apiClient.post
uploadFile(...)        // ⚠️ 순수 fetch — 유일한 예외
completeUpload(...)    // apiClient.post
getViewUrls(ids)       // apiClient.get
deleteAttachment(id)   // apiClient.delete
```

```ts
// uploadFile — apiClient를 쓰지 않는 유일한 함수.
// apiClient는 Authorization 헤더와 credentials:"include"를 자동으로 붙이는데,
// S3 presigned URL(Phase 13)은 헤더가 붙으면 서명이 깨지고 쿠키도 불필요하다.
// 로컬·S3 양쪽에서 동일하게 동작하려면 헤더 없는 순수 PUT이어야 한다.
await fetch(uploadUrl, {
  method: "PUT",
  body: file,
  headers: { "Content-Type": file.type },
  credentials: "omit",
});
```

**이 주석을 반드시 남긴다.** 없으면 "왜 여기만 apiClient를 안 쓰지" 하고 나중에 정리되어 Phase 13에서 깨진다.

### 10-5. 조회 URL과 캐시

```ts
// src/lib/queryClient.ts — queryKeys에 추가
attachmentUrls: (ids: number[]) => ["attachments", "urls", [...ids].sort((a, b) => a - b)] as const,
```

- `staleTime`은 토큰 만료(30분)보다 **짧게(20분)** 둔다
- `<img onError>`에서 **1회만** URL을 재요청한다(무한 루프 금지)

### 10-6. 적용 화면

| 화면 | 처리 |
|---|---|
| `/todos/[id]` (확인) | `collectAttachmentIds` → URL 조회 → `sanitizeHtml` → `injectAttachmentSrc` → `dangerouslySetInnerHTML` |
| `/todos/[id]/edit` (수정) | 같은 순서로 만든 HTML을 `TodoEditor`의 `content`로 넘긴다 |
| `/todos/new` (작성) | 초기 본문이 비어 있으므로 주입 불필요 |

### 10-7. ⚠️ dirty 판정 오염을 막을 것

`TodoForm`은 `TodoEditor.onReady`가 넘겨준 **정규화된 HTML**을 dirty baseline(`contentBaseline`)으로 잡는다. 첨부 URL 주입이 baseline 확정 **이후**에 일어나면, 사용자가 아무것도 고치지 않아도 HTML이 달라져 dirty가 된다.

**증상**: 수정 화면에 들어갔다 그냥 나가려는데 "변경 사항이 있습니다" 확인창이 뜬다.

**대응**: URL이 모두 해석된 뒤에 `TodoForm`을 마운트한다. 해석 전에는 기존 `FormSkeleton`을 보여준다.

### 10-8. 클라이언트 사전 검증

업로드 전에 파일 타입·크기를 미리 확인해 불필요한 요청을 막는다. `src/lib/validation.ts`에 `validateImageFile(file)`을 추가한다. **서버 검증은 그대로 유지한다.**

### 10-9. ⚠️ 저장 왕복 시 HTML이 다른 것은 정상이다

에디터가 보내는 HTML에는 `src`가 있고, 서버가 저장하는 HTML에는 없다. 저장 후 서버 응답으로 캐시를 덮어쓰므로(`useUpdateTodo`) 동작은 정상이다. **"보낸 것과 응답이 다르다"를 버그로 오인하지 말 것.**

---

## 11. 로컬 테스트 시나리오

구현 후 아래를 직접 확인하고 결과를 보고한다. **5번과 11번은 이번에 새로 추가된 항목**이다.

| # | 확인 항목 |
|---|---|
| 1 | 서버 기동 시 `todo-project/upload` 디렉토리가 자동 생성되는가 |
| 2 | 이미지 첨부 시 `upload/todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}`에 파일이 실제로 생기는가 |
| 3 | `attachments` 테이블에 `status=TEMP` 행이 생기는가 |
| 4 | 할 일 저장 후 `status=LINKED`, `todo_id`가 채워지는가 |
| **5** | **저장된 HTML에 `src`가 없고 `data-attachment-id`만 있는가 — DB에서 직접 확인** |
| 6 | 확인 화면(`/todos/[id]`)과 수정 화면 양쪽에서 이미지가 정상 표시되는가 |
| 7 | 본문에서 이미지를 지우고 저장하면 `deleted_at`이 채워지고 **파일은 남아 있는가** |
| 8 | 5MB 초과 파일, `.exe` 파일 업로드가 거부되는가 |
| 9 | 다른 사용자의 `attachmentId`로 조회 시 **404**가 나는가 (403 아님) |
| 10 | `storageKey`에 `../`를 넣은 요청이 차단되는가 |
| **11** | **아무것도 고치지 않고 수정 화면을 나갈 때 이탈 확인창이 뜨지 않는가** (10-7 회귀) |
| 12 | `upload/` 폴더가 `git status`에 잡히지 않는가 |

### 자동화 테스트

기존 패턴을 따른다(실제 PostgreSQL 사용, H2 아님).

- `HtmlSanitizerTest` — `img` 허용, **`src` 제거**, `data-attachment-id` 보존
- `AttachmentIntegrationTest` (`@SpringBootTest` + `@AutoConfigureMockMvc`) — presign → upload → complete 전 흐름, 타인 접근 404, 경로 탈출 차단
- `AttachmentRepositoryTest` (`@DataJpaTest`) — UTC 저장 검증은 `UserRepositoryTest`처럼 **JDBC로 컬럼 원본값을 재조회**해 대조

### 실행 명령

```bash
# 백엔드 — test는 local 프로파일을 상속하지 않으므로 환경변수 필요
cd todo-backend && ./mvnw test
cd todo-backend && ./mvnw spring-boot:run

# 프론트엔드
cd todo-frontend && npm run validate   # typecheck + lint + format:check
cd todo-frontend && npm run dev
```

---

## 12. S3 전환 시 계약 (Phase 13에서 수행)

**이번에 구현하지 않는다.** 다만 지금 만드는 코드가 아래를 만족해야 Phase 13이 설정 교체만으로 끝난다.

### 지켜야 할 계약

1. `AttachmentService`는 **`StorageService` 인터페이스만** 의존한다. `instanceof`나 타입 분기를 두지 않는다.
2. 프론트의 `uploadFile`은 **헤더 없는 순수 PUT**이다(4-2). 로컬용 헤더를 추가하지 않는다.
3. `viewUrl` · `uploadUrl`은 **서버가 완성된 절대 URL로 내려준다.** 프론트가 경로를 조립하지 않는다.
4. 저장 키 규칙(`todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}`)은 로컬·S3 동일하다.

### Phase 13 작업 목록 (예고)

- `S3StorageService` 구현 + AWS SDK v2(`software.amazon.awssdk:s3`) 의존성 추가 (BOM으로 버전 관리)
- `config/S3Config` — `S3Client` · `S3Presigner` 빈
- `application-prod.properties`에 `app.storage.type=s3`, 버킷·리전
- **프론트 코드는 변경이 없어야 한다.** 수정이 필요하다면 추상화가 잘못된 것이므로 **구현하지 말고 보고**한다

### 자격증명 원칙 (지금부터 적용)

- **AWS 키를 코드나 설정 파일에 절대 넣지 않는다**
- 로컬 시험 시에만 환경변수 `AWS_ACCESS_KEY_ID` · `AWS_SECRET_ACCESS_KEY`
- EC2 배포 시 IAM Role (키 없음)
- 둘 다 `DefaultCredentialsProvider`가 자동 처리하므로 **코드에 분기를 두지 않는다**

---

## 13. 진행 순서

각 단계가 끝나면 **완료 조건을 실제로 실행해 확인한 뒤 커밋**하고 멈춘다.

| 단계 | 내용 | 저장소 |
|---|---|---|
| 1 | **문서 정본 수정** (1장 선행 조건 표) | todo-project |
| 2 | `attachments` 엔티티 · 리포지토리 · `ErrorCode` | todo-backend |
| 3 | **정화 화이트리스트** + `HtmlSanitizer` 테스트 | todo-backend |
| 4 | `StorageService` + `LocalStorageService` + 서명 토큰 | todo-backend |
| 5 | `AttachmentService` · `AttachmentController` · SecurityConfig | todo-backend |
| 6 | `TodoService` 연결 처리 + 정리 배치 | todo-backend |
| 7 | 통합 테스트 (11장 자동화 테스트) | todo-backend |
| 8 | `sanitize.ts` · `attachmentHtml.ts` · 타입 · 에러 메시지 | todo-frontend |
| 9 | Tiptap 이미지 노드 + 업로드 UX | todo-frontend |
| 10 | 확인·수정 화면 URL 주입 | todo-frontend |
| 11 | **로컬 통합 테스트 (11장 시나리오 12항목)** | 전체 |

> 3단계를 2단계보다 먼저 하지 말 것. 엔티티 없이 정화만 바꾸면 검증할 대상이 없다.
> 반대로 **9단계를 8단계보다 먼저 하면 이미지가 저장되지 않아 원인을 찾느라 시간을 쓴다.**

---

## 규칙

- **코드 주석은 한글로 작성한다** (식별자는 영문)
- 기존 Soft Delete · 페이지네이션 · 예외 처리 컨벤션을 그대로 따른다
- **엔티티를 컨트롤러에서 직접 반환하지 않는다.** 항상 DTO(record)
- 생성자 주입 + `@RequiredArgsConstructor`. 필드 주입 금지
- 조회 메서드에 `@Transactional(readOnly = true)`
- 자격증명은 어떤 형태로도 소스에 남기지 않는다
- 설정 파일은 **`.properties`** 형식만 사용한다
- **승인되지 않은 라이브러리를 추가하지 않는다.** 이번에 승인된 것은 `@tiptap/extension-image` 하나뿐이다
- 요구사항이 모호하면 **추측하지 말고 질문한다**
