# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 명령어

```bash
npm run dev          # 개발 서버 실행 (localhost:3000)
npm run build        # 프로덕션 빌드
npm run start        # 프로덕션 서버 실행
npm run lint         # ESLint 검사
npm run lint:fix     # ESLint 자동 수정
npm run format       # Prettier 포매팅 적용
npm run format:check # Prettier 포매팅 검사 (CI용)
npm run type-check   # TypeScript 타입 검사 (tsc --noEmit)
```

shadcn/ui 컴포넌트 추가:

```bash
npx shadcn@latest add [component-name]
```

## Git 커밋 컨벤션

`npm install` 시 Husky Git hooks가 자동 설치된다. 커밋 메시지는 Conventional Commits 형식을 따라야 한다:

```
<type>: <subject>

예시:
feat: 사용자 프로필 페이지 추가
fix: 로그인 실패 시 오류 메시지 미표시 버그 수정
chore: 개발 도구 설정 (prettier, husky)
docs: CLAUDE.md 업데이트
```

허용 타입: `feat` `fix` `docs` `style` `refactor` `test` `chore` `perf` `ci` `revert`

- **pre-commit**: lint-staged (ESLint + Prettier를 스테이징된 파일에 적용)
- **commit-msg**: commitlint (커밋 메시지 형식 검사)
- **pre-push**: tsc --noEmit (TypeScript 타입 검사)

## 환경변수

`.env.local` 파일에 필요:

```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=...
```

## 아키텍처

**스택**: Next.js (App Router) + React 19 + TypeScript + Tailwind CSS + shadcn/ui + Supabase

### 인증 흐름

Supabase SSR 쿠키 기반 인증을 사용한다. 핵심 구조:

- `proxy.ts` (루트) — Next.js 미들웨어 역할. `lib/supabase/proxy.ts`의 `updateSession`을 호출해 모든 요청마다 세션을 갱신한다. 파일명이 `middleware.ts`가 아닌 `proxy.ts`임에 주의.
- `lib/supabase/server.ts` — Server Components / Route Handlers용 Supabase 클라이언트 생성 (`@supabase/ssr`의 `createServerClient`).
- `lib/supabase/client.ts` — 브라우저(Client Components)용 Supabase 클라이언트 생성 (`createBrowserClient`).
- `app/auth/confirm/route.ts` — 이메일 OTP 검증 처리.

**중요**: Supabase 클라이언트를 전역 변수로 선언하지 말 것. 각 함수/요청마다 새로 생성해야 한다 (Fluid Compute 환경 때문).

세션 확인은 `getClaims()` 사용 (`getUser()` 아님):

```typescript
const { data } = await supabase.auth.getClaims();
const user = data?.claims;
```

### 라우팅 구조

- `/` — 공개 홈페이지 (환경변수 설정 여부에 따라 튜토리얼 or 인증 단계 표시)
- `/auth/login`, `/auth/sign-up`, `/auth/forgot-password`, `/auth/update-password` — 인증 페이지
- `/auth/error` — 인증 오류 페이지
- `/auth/sign-up-success` — 회원가입 완료 페이지
- `/protected/*` — 인증 필요 페이지. `proxy.ts`에서 미인증 시 `/auth/login`으로 리다이렉트.

### 컴포넌트 구조

- `components/ui/` — shadcn/ui 기반 기본 UI 컴포넌트 (직접 수정하지 말 것)
- `components/` — 인증 폼(`login-form.tsx`, `sign-up-form.tsx` 등) 및 레이아웃 컴포넌트
- `components/tutorial/` — 초기 설정 가이드 컴포넌트 (개발 완료 후 제거 가능)

### 주요 규칙

- 모든 컴포넌트는 기본적으로 Server Component. 상태/이벤트 핸들러가 필요한 경우에만 `'use client'` 사용.
- Next.js 15에서 `params`, `searchParams`, `cookies()`, `headers()`는 모두 async. 반드시 `await` 해야 한다.
- 경로 별칭 `@/`는 프로젝트 루트를 가리킨다 (예: `@/lib/supabase/server`).
- 파일명은 kebab-case, 컴포넌트명은 PascalCase.
- Named export 사용 권장; 페이지 컴포넌트에는 default export 사용.
