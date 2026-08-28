# Next.js 15 가이드

> 참고 자료. 스펙은 `CLAUDE.md`다(`docs/guides/README.md` 참조). 버전은 `CLAUDE.md` 3장에서 **Next.js 15로 확정**됐다(Amplify Hosting SSR 지원 범위가 12~15이기 때문 — 16 사용 금지).

## 이 프로젝트의 App Router 구조

`src/` 아래에 App Router가 있다(`src/app/`). Next.js 16의 전역 타입(`LayoutProps<...>`)이나 `next typegen` CLI는 15에 없으므로 사용하지 않는다 — 레이아웃의 `children` prop은 `{ children: React.ReactNode }`로 명시하고, 라우트 타입은 `next dev`/`next build` 시 `.next/types/`에 자동 생성된다.

```
src/
├── app/
│   ├── layout.tsx        # 루트 레이아웃(서버 컴포넌트) + Provider 분리
│   ├── page.tsx
│   ├── globals.css
│   ├── (auth)/            # 비인증 라우트 그룹
│   │   ├── login/page.tsx
│   │   └── signup/page.tsx
│   ├── (main)/             # 인증 필요 라우트 그룹
│   │   ├── layout.tsx      # 클라이언트 인증 가드 + 공통 헤더
│   │   └── todos/
│   │       ├── page.tsx
│   │       ├── new/page.tsx
│   │       └── [id]/page.tsx
│   └── oauth/callback/page.tsx
├── components/
├── lib/
├── hooks/
├── providers/
└── types/
```

## 이 프로젝트는 사실상 전부 클라이언트 컴포넌트다

인증 토큰이 localStorage에 있고 데이터를 React Query로 가져오므로, 서버 컴포넌트에서 데이터를 미리 가져올 수 없다(`CLAUDE.md` 9장). `/login`·`/signup`·`/oauth/callback`·`/todos`·`/todos/new`·`/todos/[id]`의 `page.tsx`에는 `"use client"`를 붙인다. `app/layout.tsx`(루트)만 서버 컴포넌트로 두고 Provider는 별도 클라이언트 컴포넌트로 분리해 감싼다.

## `useSearchParams`는 `<Suspense>`가 필수다

`useSearchParams`를 쓰는 컴포넌트를 `<Suspense>`로 감싸지 않으면 `npm run build`가 프리렌더 단계에서 실패한다(개발 서버에서는 통과하므로 늦게 발견되기 쉽다). 이 프로젝트에서 해당되는 곳은 `/oauth/callback`(`?token=` 읽기)과 `/todos`(검색어·필터·페이지를 쿼리로 관리)다.

## 라우트 보호는 middleware로 하지 않는다

토큰이 localStorage에 있어 서버에서 실행되는 middleware는 접근할 수 없다. `middleware.ts`를 만들지 말고, `(main)` 클라이언트 레이아웃 + `useAuth`(JWT `exp` 클레임 검사)로 구현한다.

## Amplify 배포 제약

- `public/static` 경로를 만들지 않는다(Amplify 예약 경로).
- `main`·`develop` 브랜치 모두 동일한 렌더링 방식(SSR/CSR 혼합, `next build`)이어야 한다.
- 빌드 출력은 `.next`. `next.config.ts`에 `distDir`을 설정하지 않는다.
