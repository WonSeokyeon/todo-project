# 폼 처리 가이드

> 참고 자료. 스펙은 `CLAUDE.md`다(`docs/guides/README.md` 참조).

## 이 프로젝트의 폼 방식

**폼 라이브러리를 쓰지 않는다.** `react-hook-form`·`zod`·`@hookform/resolvers`를 설치하지 않는다(`CLAUDE.md` 3장). 이 앱의 폼은 로그인(2필드)·회원가입(3필드)·`TodoForm`(4필드) 셋뿐이고 검증 규칙도 `CLAUDE.md` 4장 표로 고정되어 있어, `useState` + 수동 검증으로 충분하다.

`npx shadcn add form`을 실행하지 않는다 — 이 컴포넌트만 `react-hook-form` 위에 만들어져 있어 실행 시 `react-hook-form`·`@hookform/resolvers`가 의존성으로 함께 설치된다.

## 검증 로직은 한 곳에 모은다

화면마다 검증을 흩어 두지 않고 `src/lib/validation.ts`에 함수로 모은다(이메일 형식, 닉네임 1~50자, 제목 200자, 본문 50,000자, 비밀번호 6자 이상).

> ⚠️ **비밀번호 상한은 문자 수가 아니라 UTF-8 바이트다.** `maxLength={64}` 같은 문자 수 제한만 걸면 한글 25자(=75바이트)가 통과해 서버 BCrypt 단계에서 500이 난다(`CLAUDE.md` 4장). `new TextEncoder().encode(value).length`로 바이트를 센다.

## 기본 패턴

```tsx
"use client";

import { useState } from "react";
import { validateEmail, validatePasswordLength } from "@/lib/validation";

export function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [errors, setErrors] = useState<{ email?: string; password?: string }>({});

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    const nextErrors: typeof errors = {};
    if (!validateEmail(email)) nextErrors.email = "올바른 이메일 형식이 아닙니다.";
    if (!validatePasswordLength(password)) nextErrors.password = "비밀번호는 6자 이상이어야 합니다.";
    setErrors(nextErrors);
    if (Object.keys(nextErrors).length > 0) return;

    // apiClient 호출
  }

  return <form onSubmit={handleSubmit}>{/* ... */}</form>;
}
```

## `dirty` 판정 (이탈 확인용)

라이브러리의 `formState.isDirty`가 없으므로 초기값과 현재값을 직접 비교한다(`CLAUDE.md` 3장 ⚠️ 참조).

- 일반 필드: 초기값 문자열과 현재 상태를 그대로 비교한다.
- Tiptap 본문: 에디터가 HTML을 정규화하므로, 서버가 준 HTML 문자열을 그대로 초기값으로 쓰면 아무것도 고치지 않아도 다르게 나온다. **초기 스냅샷은 `editor.commands.setContent()` 직후의 `editor.getHTML()`로 잡는다**(정규화를 거친 값끼리 비교).

## 인증·CRUD는 Spring Boot REST API를 호출한다

이 프로젝트는 자체 백엔드(Spring Boot)를 쓴다. Server Actions에서 직접 해싱·DB 접근을 하지 않는다.

| 하는 일             | 방법                                                        |
| -------------------- | ------------------------------------------------------------ |
| 비밀번호 해싱        | 백엔드가 BCrypt로 처리 — 프론트는 평문을 HTTPS로 전달만 한다 |
| 회원가입             | `POST /api/v1/auth/signup`                                  |
| 로그인               | `POST /api/v1/auth/login` 후 Access Token을 `apiClient`가 저장 |
| 데이터 조회/저장     | `src/lib/apiClient.ts`를 경유한 REST 호출                    |

엔드포인트 전체 목록과 요청/응답 포맷은 `CLAUDE.md` 5장을 따른다.
