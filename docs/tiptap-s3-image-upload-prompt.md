# Tiptap 에디터 이미지 첨부 기능 추가 (로컬 → S3)

> 클로드 코드에 그대로 붙여넣는 프롬프트입니다.

---

## 작업 목표

할일(Todo) 작성/수정 시 Tiptap 에디터의 상세 설명란에 **이미지 파일을 첨부**할 수 있게 한다.
파일 실체는 스토리지(로컬 디렉토리 또는 S3)에 저장하고, 메타데이터는 PostgreSQL에서 관리한다.

**개발 순서: 로컬 파일 저장으로 먼저 완성하고 테스트한 뒤, S3로 전환한다.**
프론트엔드 코드는 두 방식에서 **동일하게 동작해야 한다.** 스토리지 교체는 백엔드 설정만 바꿔서 이뤄진다.

`todo-backend`와 `todo-frontend`는 별도 저장소다. 백엔드를 먼저 완성하고 프론트로 넘어간다.

---

## 0. 먼저 확인할 것 (코드 수정 전)

아래를 파악한 뒤 **어떻게 바꿀지 계획만 보고하고 멈춰라.** 승인 후 구현한다.

1. `todo-backend`의 현재 엔티티 구조 — 특히 `Todo`, `User`(또는 Member) 엔티티와 Soft Delete 처리 방식
2. DB 스키마 관리 방식 — Flyway/Liquibase 마이그레이션인지, `ddl-auto`인지
3. `Todo`의 상세 설명 컬럼(`description`) 타입과 길이 — Tiptap HTML을 담기에 충분한지 (`TEXT` 권장)
4. 현재 설정 파일 형태 — `application.yml`이 있다면 **`application.properties`로 전환**한다
5. 프로파일 분리 여부 — 없다면 `local` / `prod` 프로파일을 새로 만든다
6. 기존 인증 방식(JWT) 하에서 파일 소유자를 어떻게 식별할지
7. `todo-frontend`의 Tiptap 에디터 컴포넌트 위치와 현재 등록된 extension 목록

---

## 1. DB 스키마 설계

새 테이블 `attachment`를 추가한다.

| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | BIGSERIAL PK | |
| `todo_id` | BIGINT NULL, FK → todo(id) | **NULL 허용** (아래 이유 참고) |
| `user_id` | BIGINT NOT NULL, FK | 업로더 (권한 검증용) |
| `storage_type` | VARCHAR(20) NOT NULL | `LOCAL` / `S3` |
| `storage_key` | VARCHAR(512) NOT NULL UNIQUE | 로컬 상대경로 또는 S3 객체 키 |
| `original_filename` | VARCHAR(255) NOT NULL | 원본 파일명 |
| `content_type` | VARCHAR(100) NOT NULL | |
| `file_size` | BIGINT NOT NULL | bytes |
| `status` | VARCHAR(20) NOT NULL | `TEMP` / `LINKED` |
| `created_at` | TIMESTAMP NOT NULL | |
| `deleted_at` | TIMESTAMP NULL | Soft Delete (기존 정책과 동일하게) |

**`todo_id`를 NULL 허용으로 두는 이유**
에디터에서는 할일을 저장하기 *전에* 이미지가 먼저 업로드된다. 따라서 업로드 시점에는 `status = TEMP`, `todo_id = NULL`로 만들고, 할일 저장/수정 시 본문에 실제로 남아 있는 첨부만 `LINKED`로 전환하며 `todo_id`를 채운다.

**`storage_type`을 두는 이유**
로컬에서 테스트한 데이터와 S3 전환 이후 데이터가 섞여도 각각 올바른 방식으로 조회하기 위함이다.

**인덱스**: `todo_id`, `(status, created_at)` — 고아 파일 정리 배치용

**저장 키 규칙**: `todos/{userId}/{yyyy}/{MM}/{uuid}.{ext}` (로컬·S3 동일)

**마이그레이션 파일**은 프로젝트의 기존 방식에 맞춰 작성한다. 별도 방식이 없다면 Flyway를 도입하고 `V{n}__add_attachment.sql`로 만든다.

---

## 2. 스토리지 추상화 (핵심)

로컬과 S3를 같은 인터페이스로 다룬다.

```java
public interface StorageService {
    StorageType getType();
    // 업로드용 URL 발급 (S3: presigned PUT / 로컬: 백엔드 업로드 엔드포인트)
    String createUploadUrl(String storageKey, String contentType);
    // 조회용 URL 발급 (S3: presigned GET / 로컬: 백엔드 조회 엔드포인트)
    String createViewUrl(String storageKey);
    // 업로드 완료 후 실제 파일 존재 및 크기 확인
    long verifyUploaded(String storageKey);
    void delete(String storageKey);
}
```

구현체 두 개를 만들고 `app.storage.type` 값에 따라 `@ConditionalOnProperty`로 빈을 하나만 등록한다.

- `LocalStorageService` — 로컬 디스크 저장
- `S3StorageService` — AWS S3 저장

`AttachmentService`는 이 인터페이스만 의존하고, 어떤 구현체인지 알지 못한다.

---

## 3. 업로드 흐름 (로컬·S3 공통)

```
1. 프론트 → 백엔드   POST /api/attachments/presign  { filename, contentType, fileSize }
2. 백엔드            검증 후 attachment 레코드 생성(status=TEMP) + 업로드 URL 발급
                     ← { attachmentId, uploadUrl, storageKey }
3. 프론트 → uploadUrl  PUT (파일 본문, Content-Type 헤더 포함)
4. 프론트 → 백엔드   POST /api/attachments/{id}/complete
5. 백엔드            실제 파일 존재/크기 확인 후 확정
                     ← { attachmentId, viewUrl }
```

**`uploadUrl`이 무엇을 가리키는가**

| | uploadUrl | viewUrl |
|---|---|---|
| 로컬 | `http://localhost:8080/api/attachments/{id}/upload` | `http://localhost:8080/api/attachments/{id}/raw` |
| S3 | S3 presigned PUT URL | S3 presigned GET URL (유효기간 30분) |

프론트는 **이 URL이 어디를 가리키는지 신경 쓰지 않고** 그냥 PUT을 보낸다. 그래서 스토리지를 바꿔도 프론트 코드는 그대로다.

S3 버킷은 퍼블릭으로 열지 않는다.

---

## 4. 로컬 스토리지 구현

### 저장 위치
`todo-project/upload` 디렉토리를 사용한다. `todo-backend`, `todo-frontend`와 형제 레벨이며 **어느 저장소에도 커밋하지 않는다.**

```
todo-project/
├── todo-backend/
├── todo-frontend/
└── upload/          ← 여기
```

- 경로는 설정값으로 주입한다 (하드코딩 금지)
- 애플리케이션 시작 시 디렉토리가 없으면 생성한다
- 두 저장소의 `.gitignore`에 `upload/`, `../upload/`를 추가한다

### 경로 조작 방어 (필수)
`storageKey`로 상위 디렉토리 탈출(`../`)이 가능하면 서버 파일이 노출된다.
저장·조회 전에 최종 경로를 정규화(`Path.normalize()`)한 뒤 **base 디렉토리 하위인지 반드시 확인**하고, 벗어나면 예외를 던진다.

### 업로드 엔드포인트
`PUT /api/attachments/{id}/upload`
- 요청 본문을 그대로 파일로 기록 (`RequestBody`를 스트림으로 처리, 메모리 전체 적재 금지)
- `attachment.status`가 `TEMP`이고 요청자가 업로더 본인인지 확인
- 이미 파일이 존재하면 409

### 조회 엔드포인트
`GET /api/attachments/{id}/raw`
- 소유자 검증 후 `Content-Type`과 함께 파일 스트림 반환
- `Content-Disposition: inline`
- 브라우저 `<img>` 태그가 직접 호출하므로 **JWT 헤더를 못 실어 보낸다.** 아래 중 하나로 처리한다.
  - (권장) 단기 유효 서명 토큰을 URL 쿼리로 붙여 발급: `?token=...` — S3 presigned와 개념이 같아 전환 시 구조 변화가 없다
  - 또는 프론트에서 fetch로 받아 blob URL로 변환

---

## 5. 백엔드 설정 (`application.properties`)

### `application.properties` (공통)
```properties
spring.profiles.active=local

# 파일 업로드 제한
app.upload.max-file-size=5242880
app.upload.allowed-content-types=image/jpeg,image/png,image/gif,image/webp

# multipart 사용하지 않지만, 스트림 방식 PUT 요청 크기 제한
server.tomcat.max-swallow-size=6MB
```

### `application-local.properties`
```properties
app.storage.type=local
app.storage.local.base-dir=${LOCAL_UPLOAD_DIR:../upload}
app.storage.local.base-url=http://localhost:8080
app.storage.url-expiry-minutes=30
```

### `application-prod.properties`
```properties
app.storage.type=s3
app.storage.s3.bucket=${AWS_S3_BUCKET}
app.storage.s3.region=${AWS_REGION:ap-northeast-2}
app.storage.url-expiry-minutes=30
```

### 자격증명 원칙
**AWS 키를 코드나 설정 파일에 절대 넣지 않는다.**
- 로컬에서 S3를 시험할 때만 환경변수 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 사용
- EC2 배포 시에는 IAM Role 사용 (키 없이)
- 둘 다 `DefaultCredentialsProvider`가 자동 처리하므로 코드에 분기를 두지 않는다
- `application-prod.properties`, `.env`가 `.gitignore`에 있는지 확인한다

---

## 6. 백엔드 구현 목록

### 의존성
AWS SDK v2 (`software.amazon.awssdk:s3`) — BOM으로 버전 관리. **5단계에서 추가한다.**

### 클래스
- `domain/attachment/Attachment` (엔티티), `AttachmentStatus`, `StorageType` (enum), `AttachmentRepository`
- `storage/StorageService` (인터페이스)
- `storage/LocalStorageService` — `@ConditionalOnProperty(name="app.storage.type", havingValue="local")`
- `storage/S3StorageService` — `havingValue="s3"`
- `config/S3Config` — `S3Client`, `S3Presigner` 빈 (프로파일 조건부)
- `service/AttachmentService` — 검증, 상태 전환, 권한 체크
- `controller/AttachmentController`

### API
| 메서드 | 경로 | 설명 |
|---|---|---|
| POST | `/api/attachments/presign` | 업로드 URL 발급 |
| PUT | `/api/attachments/{id}/upload` | **로컬 전용** 파일 수신 |
| POST | `/api/attachments/{id}/complete` | 업로드 완료 확정 |
| GET | `/api/attachments/{id}/url` | 조회용 URL 발급 |
| GET | `/api/attachments/{id}/raw` | **로컬 전용** 파일 스트림 |
| DELETE | `/api/attachments/{id}` | Soft Delete + 실제 파일 삭제 |

### 검증 규칙
- 로그인 사용자만 업로드 가능
- `contentType` 화이트리스트 검증 (확장자만 믿지 않는다)
- 파일 크기 상한 초과 시 400
- 조회/삭제 시 `attachment.user_id`와 요청자 일치 확인 → 불일치 403

### 할일 저장 시 연결 처리
`TodoService`의 생성/수정 로직에 추가:
1. 저장될 `description` HTML을 파싱해 `data-attachment-id` 값을 모두 수집
2. 해당 첨부를 `LINKED` + `todo_id` 설정
3. 기존에 연결돼 있었으나 본문에서 사라진 첨부는 Soft Delete 처리
4. 다른 사용자의 첨부 ID가 섞여 있으면 거부

### 고아 파일 정리
`@Scheduled` 배치 — 하루 1회, `status = TEMP`이고 생성 후 24시간 지난 레코드를 실제 파일과 함께 삭제.

---

## 7. 프론트엔드 (`todo-frontend`)

### Tiptap 설정
- `@tiptap/extension-image`를 확장한 커스텀 노드를 만든다. `src` 외에 **`attachmentId` 속성을 추가**하고 `data-attachment-id`로 렌더링한다.
  → 조회 URL은 만료되므로 HTML에 URL을 박아두면 안 된다. ID를 저장하고 조회 시점에 URL을 다시 받는다.
- 툴바 이미지 버튼, 붙여넣기(paste), 드래그앤드롭 세 경로 모두 지원

### 업로드 UX
1. 파일 선택 즉시 로컬 blob URL로 **임시 미리보기** 노드 삽입 (업로드 중 표시)
2. presign → PUT → complete 순차 호출, 진행률 표시
3. 성공 시 노드의 `src`를 실제 URL로, `attachmentId`를 채워 교체
4. 실패 시 노드 제거 + 토스트로 에러 안내

### 조회
할일 상세를 불러올 때 본문의 `data-attachment-id`를 모아 조회용 URL을 일괄 요청하고 `src`에 주입한다.

### 클라이언트 검증
업로드 전 파일 타입/크기를 미리 확인해 불필요한 요청을 막는다. (서버 검증은 그대로 유지)

### API 함수
`lib/api/attachments.ts`에 `presignUpload`, `uploadFile`, `completeUpload`, `getViewUrl`, `deleteAttachment`를 추가하고 React Query 뮤테이션으로 감싼다.
`uploadFile`은 서버가 준 `uploadUrl`로 PUT을 보낼 뿐, **로컬인지 S3인지 판단하는 로직을 두지 않는다.**

---

## 8. 로컬 테스트 시나리오

구현 후 아래를 직접 확인하고 결과를 보고한다.

1. 서버 기동 시 `todo-project/upload` 디렉토리가 자동 생성되는가
2. 이미지 첨부 → `upload/todos/{userId}/...` 경로에 파일이 실제로 생기는가
3. `attachment` 테이블에 `status=TEMP`로 행이 생기는가
4. 할일 저장 후 `status=LINKED`, `todo_id`가 채워지는가
5. 저장된 할일을 다시 열었을 때 이미지가 정상 표시되는가
6. 본문에서 이미지를 지우고 저장하면 해당 첨부가 Soft Delete 되는가
7. 5MB 초과 파일, `.exe` 파일 업로드가 거부되는가
8. 다른 사용자의 `attachmentId`로 조회 시 403이 나는가
9. `storageKey`에 `../`를 넣은 요청이 차단되는가
10. `upload/` 폴더가 git status에 잡히지 않는가

---

## 9. S3 전환 시 추가 작업 (5단계)

- `S3StorageService` 구현 + AWS SDK 의존성 추가
- `application-prod.properties` 프로파일로 기동해 동일 시나리오 재확인
- 프론트 코드는 **변경 없어야 한다.** 수정이 필요하다면 추상화가 잘못된 것이므로 보고할 것

### AWS 콘솔 설정 (별도 안내 필요)
- 버킷 **CORS 설정** — presigned PUT을 위해 `PUT`, `GET`, `HEAD` 허용 / `AllowedOrigin`에 로컬·Amplify 도메인
- 퍼블릭 액세스 차단은 **켠 상태 유지**
- IAM 정책은 해당 버킷에 대한 `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`만 허용 (최소 권한)

---

## 10. 진행 순서

각 단계가 끝나면 멈추고 확인받는다.

1. 현황 파악 + 변경 계획 보고 ← **여기까지만 먼저**
2. `application.yml` → `application.properties` 전환, `local`/`prod` 프로파일 분리
3. DB 마이그레이션 + Attachment 엔티티/리포지토리
4. `StorageService` 인터페이스 + `LocalStorageService` + AttachmentService/Controller
5. TodoService 연결 로직 + 고아 파일 정리 배치
6. 프론트 API 함수 + 커스텀 Tiptap 이미지 노드 + 업로드 UX
7. **로컬 통합 테스트 (8번 시나리오 전체)**
8. `S3StorageService` 구현 및 전환 검증

---

## 규칙

- 코드 주석은 한글로 작성
- 기존 Soft Delete / 페이지네이션 / 예외 처리 컨벤션을 그대로 따른다
- 자격증명은 어떤 형태로도 소스에 남기지 않는다
- 설정 파일은 `.properties` 형식만 사용한다
