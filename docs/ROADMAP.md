# 모임 이벤트 관리 웹 MVP 개발 로드맵

소/중규모 모임 주최자가 공지, 참여자 관리, 카풀, 정산을 한 곳에서 처리하도록 돕는 올인원 모임 운영 플랫폼

## 개요

모임 이벤트 관리 웹은 **정기적으로 소/중규모 모임(2~30명)을 운영하는 주최자와 참여자**를 위한 **통합 모임 운영 도구**로 다음 기능을 제공합니다:

- **모임 관리(CRUD)**: 모임 생성/수정/삭제 및 상태 관리(draft/open/closed/cancelled)
- **참여자 관리**: 참여 신청, 주최자 승인/거절, 참여자 목록 조회
- **공지 관리**: 공지 작성/조회 및 승인된 참여자 전원에게 이메일 일괄 발송
- **카풀 매칭**: 운전자 등록(좌석 수), 탑승 신청, 운전자 확정/거절
- **정산**: 비용 항목 입력, 1/N 자동 계산, 납부 여부 체크
- **이메일 알림**: Resend 기반 6가지 트리거 자동 알림

> **참고**: 인증(F010)과 사용자 기본 프로필(F012)은 이미 구현 완료되어 있습니다. (이메일/비밀번호 로그인, 회원가입, Google OAuth, 비밀번호 재설정, 프로필 페이지). 본 로드맵은 신규 도메인 기능에 집중합니다.

## 기술 스택

- **프레임워크**: Next.js 15 (App Router) + React 19 + TypeScript
- **백엔드/DB**: Supabase (PostgreSQL + Auth, SSR 쿠키 기반 인증)
- **스타일링**: Tailwind CSS + shadcn/ui
- **이메일**: Resend
- **데이터 변경**: Next.js Server Actions
- **테스트**: Playwright MCP (E2E / 통합 테스트)

## 개발 워크플로우

1. **작업 계획**
   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - 새로운 작업을 포함하도록 `ROADMAP.md` 업데이트
   - 우선순위 작업은 마지막 완료된 작업 다음에 삽입

2. **작업 생성**
   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - 고수준 명세서, 관련 파일, 수락 기준, 구현 단계 포함
   - API/비즈니스 로직(Server Actions) 작업 시 "## 테스트 체크리스트" 섹션 필수 포함 (Playwright MCP 테스트 시나리오 작성)
   - 새 작업의 경우 문서에는 빈 박스와 변경 사항 요약이 없어야 함

3. **작업 구현**
   - 작업 파일의 명세서를 따라 기능을 구현
   - API 연동 및 비즈니스 로직 구현 시 Playwright MCP로 테스트 수행 필수
   - 각 단계 후 작업 파일 내 단계 진행 상황 업데이트
   - 구현 완료 후 Playwright MCP를 사용한 E2E 테스트 실행
   - 테스트 통과 확인 후 다음 단계로 진행
   - 각 단계 완료 후 중단하고 추가 지시를 기다림

4. **로드맵 업데이트**
   - 로드맵에서 완료된 작업을 ✅로 표시

## 상태 표시 범례

- `✅ - 완료`: 완료된 작업
- `- 우선순위`: 즉시 시작해야 할 작업
- 표시 없음: 대기 중인 작업

---

## 개발 단계

### Phase 0: 인증 및 프로필 (완료) ✅

- **Task 000: 인증 및 기본 프로필 시스템** ✅ - 완료
  - ✅ 이메일/비밀번호 로그인 및 회원가입 (`components/login-form.tsx`, `sign-up-form.tsx`)
  - ✅ Google OAuth 로그인 (`components/google-login-button.tsx`)
  - ✅ 비밀번호 재설정 플로우 (`forgot-password-form.tsx`, `update-password-form.tsx`)
  - ✅ SSR 쿠키 기반 세션 관리 (`proxy.ts`, `lib/supabase/*`)
  - ✅ 사용자 기본 프로필 페이지 (`app/protected/profile/page.tsx`, `lib/profile.ts`)

---

### Phase 1: 애플리케이션 골격 구축

전체 라우트 구조, 타입 정의, DB 스키마를 먼저 완성하여 이후 모든 작업의 기반을 마련합니다.

- **Task 001: 라우트 구조 및 빈 페이지 골격 생성** - 우선순위
  - `app/(protected)/` 라우트 그룹 생성 및 네비게이션 레이아웃 골격 (`layout.tsx`)
  - 모임 관련 빈 페이지 껍데기 생성:
    - `events/page.tsx` (내 모임 목록), `events/new/page.tsx` (모임 생성)
    - `events/[eventId]/layout.tsx` (탭 바), `events/[eventId]/page.tsx` (개요), `events/[eventId]/edit/page.tsx` (수정)
    - `events/[eventId]/participants/page.tsx` (참여자 관리)
    - `events/[eventId]/announcements/page.tsx`, `announcements/new/page.tsx`
    - `events/[eventId]/carpools/page.tsx`, `carpools/new/page.tsx`
    - `events/[eventId]/settlements/page.tsx`, `settlements/[settlementId]/page.tsx`
  - 기존 `app/protected/profile/page.tsx`를 `(protected)` 그룹으로 정리/유지
  - 모든 페이지에 "준비 중" placeholder 표시 및 라우팅 네비게이션 동작 확인

- **Task 002: TypeScript 타입 및 인터페이스 정의**
  - `types/database.ts`에 8개 테이블 타입 추가 정의:
    - `Event`, `EventParticipant`, `Announcement`
    - `Carpool`, `CarpoolPassenger`
    - `Settlement`, `SettlementItem`, `SettlementParticipant`
  - 모임 상태 enum 타입 정의 (`EventStatus`: draft/open/closed/cancelled)
  - 참여 상태(pending/approved/rejected), 카풀 탑승 상태, 정산 납부 상태 enum 정의
  - Server Action 입력/응답 공통 타입(`ActionResult<T>` 등) 정의
  - 기존 `Profile` 타입과의 관계(주최자, 참여자) 타입 연결

- **Task 003: 데이터베이스 스키마 설계 및 마이그레이션 작성**
  - Supabase 마이그레이션 SQL 작성 (8개 테이블, 구현은 Phase 3에서 적용):
    - `events` (주최자 FK, 제목/설명/일시/장소/상태)
    - `event_participants` (모임 FK, 사용자 FK, 신청 상태)
    - `announcements` (모임 FK, 제목/본문)
    - `carpools` (모임 FK, 운전자 FK, 좌석 수, 출발지/시간)
    - `carpool_passengers` (카풀 FK, 탑승자 FK, 확정 상태)
    - `settlements` (모임 FK, 제목, 총액)
    - `settlement_items` (정산 FK, 항목명/금액)
    - `settlement_participants` (정산 FK, 참여자 FK, 분담액, 납부 여부)
  - 외래 키 관계 및 인덱스 설계 (모임-참여자, 모임-공지 등)
  - Row Level Security(RLS) 정책 초안 작성 (주최자/참여자 권한 구분)

---

### Phase 2: UI/UX 완성 (더미 데이터 활용)

shadcn 컴포넌트와 더미 데이터로 모든 화면을 완성하여 전체 사용자 플로우를 체험 가능하게 합니다.

- **Task 004: 추가 shadcn 컴포넌트 설치 및 더미 데이터 유틸 작성** - 우선순위
  - 필요한 shadcn 컴포넌트 추가: `Tabs`, `Select`, `Dialog`, `Textarea`, `Separator`, `Avatar`
  - `lib/dummy-data.ts` 작성 (모임/참여자/공지/카풀/정산 더미 데이터 생성기)
  - 상태 배지/날짜 포매팅 등 공통 표시 유틸 작성
  - 디자인 시스템 및 스타일 가이드(간격, 타이포그래피) 확립

- **Task 005: 공통 도메인 컴포넌트 라이브러리 구현**
  - `components/events/`: `event-card`, `event-form`, `event-status-badge`, `event-tabs`
  - `components/participants/`: `participant-list`, `participant-card`, `join-button`, `approve-reject-buttons`
  - `components/announcements/`: `announcement-list`, `announcement-card`, `announcement-form`
  - `components/carpools/`: `carpool-list`, `carpool-card`, `carpool-form`
  - `components/settlements/`: `settlement-list`, `settlement-detail`, `settlement-form`, `payment-check-button`
  - 모든 컴포넌트는 props 기반으로 더미 데이터를 받아 렌더링 (로직 미포함)

- **Task 006: 네비게이션 및 모임 목록/상세 페이지 UI 완성**
  - `(protected)/layout.tsx` 네비게이션 바 완성 (로고, 모임 목록 링크, 프로필/로그아웃)
  - 내 모임 목록 페이지: 주최/참여 탭, 상태 필터(`Select`), 모임 카드 그리드
  - 모임 상세 탭 바(`event-tabs`): 개요/참여자/공지/카풀/정산 탭 네비게이션
  - 개요 탭 UI: 모임 정보, 참여자 요약, 빠른 액션
  - 반응형 디자인(모바일/데스크톱) 및 빈 상태(Empty State) 처리

- **Task 007: 모임 생성/수정 및 기능별 탭 페이지 UI 완성**
  - 모임 생성/수정 폼 UI (`event-form`, 일시/장소/상태 입력, `Dialog` 확인)
  - 참여자 관리 탭 UI (신청 목록, 승인/거절 버튼, 참여 신청 버튼)
  - 공지 목록/작성 탭 UI (`announcement-list`, `announcement-form` + `Textarea`)
  - 카풀 목록/등록 탭 UI (운전자 카드, 좌석 표시, 탑승 신청 버튼)
  - 정산 상세 탭 UI (항목 입력, 1/N 분담 표, 납부 체크박스)
  - 전체 사용자 플로우 클릭 검증 및 네비게이션 일관성 확인

---

### Phase 3: 핵심 기능 구현 (DB 연동 + Server Actions)

더미 데이터를 실제 Supabase 데이터와 Server Actions로 교체하여 비즈니스 로직을 완성합니다.

- **Task 008: 데이터베이스 마이그레이션 적용 및 RLS 설정** - 우선순위
  - Phase 1에서 설계한 8개 테이블 마이그레이션을 Supabase에 적용
  - RLS 정책 적용 (주최자만 수정/삭제, 참여자만 조회 등)
  - Supabase 타입 자동 생성 및 `types/database.ts`와 동기화
  - 시드 데이터로 기본 조회 동작 검증
  - ## 테스트 체크리스트
    - Playwright MCP로 로그인 후 빈 모임 목록이 정상 표시되는지 확인
    - RLS로 타 사용자 데이터 접근이 차단되는지 검증

- **Task 009: 모임 CRUD Server Actions 구현 (F001)**
  - `lib/actions/events.ts`: `createEvent`, `updateEvent`, `deleteEvent`, `getMyEvents`
  - 모임 상태 전환 로직 (draft → open → closed/cancelled)
  - 모임 목록/생성/수정/삭제 UI를 실제 데이터로 연결 (더미 데이터 제거)
  - 권한 검증 (주최자만 수정/삭제 가능)
  - ## 테스트 체크리스트
    - Playwright MCP: 모임 생성 → 목록 노출 → 수정 → 상태 변경 → 삭제 전체 플로우
    - 주최/참여 탭 및 상태 필터 동작 검증
    - 비주최자의 수정/삭제 시도 차단 검증

- **Task 010: 참여자 관리 Server Actions 구현 (F002)**
  - `lib/actions/participants.ts`: `applyToEvent`, `approveParticipant`, `rejectParticipant`, `cancelParticipation`
  - 참여 상태 전이(pending → approved/rejected) 및 중복 신청 방지
  - 참여자 관리 탭 UI를 실제 데이터로 연결
  - 개요 탭 참여자 요약 실데이터 반영
  - ## 테스트 체크리스트
    - Playwright MCP: 참여자 계정으로 신청 → 주최자 계정으로 승인/거절 플로우
    - 신청 취소 및 중복 신청 방지 검증
    - 승인 인원/정원 표시 정확성 검증

- **Task 011: 공지 관리 Server Actions 구현 (F003)**
  - `lib/actions/announcements.ts`: `createAnnouncement`, `updateAnnouncement`, `deleteAnnouncement`
  - 공지 목록/작성 UI를 실제 데이터로 연결
  - 주최자 권한 검증 (작성/수정/삭제는 주최자만)
  - (이메일 발송 연동은 Phase 4 Task 014에서 처리)
  - ## 테스트 체크리스트
    - Playwright MCP: 공지 작성 → 목록 노출 → 수정 → 삭제 플로우
    - 참여자는 공지 조회만 가능한지 권한 검증

- **Task 012: 카풀 매칭 Server Actions 구현 (F004)**
  - `lib/actions/carpools.ts`: `registerCarpool`, `applyForRide`, `confirmPassenger`, `rejectPassenger`
  - 좌석 수 기반 잔여석 계산 및 초과 신청 방지
  - 카풀 목록/등록 UI를 실제 데이터로 연결
  - ## 테스트 체크리스트
    - Playwright MCP: 운전자 등록 → 탑승 신청 → 확정/거절 플로우
    - 좌석 초과 신청 차단 및 잔여석 표시 검증

- **Task 013: 정산 Server Actions 구현 (F005)**
  - `lib/actions/settlements.ts`: `createSettlement`, `addSettlementItem`, `calculateSettlement`, `markAsPaid`
  - 1/N 자동 계산 로직 (총액 ÷ 승인 참여자 수, 단수 처리 규칙 정의)
  - 정산 상세 UI를 실제 데이터로 연결 (항목 입력, 분담액, 납부 체크)
  - ## 테스트 체크리스트
    - Playwright MCP: 정산 생성 → 항목 추가 → 1/N 계산 → 납부 체크 플로우
    - 분담액 합계가 총액과 일치하는지 검증 (단수 처리 포함)
    - 납부 완료/미납 상태 토글 검증

- **Task 013-1: 핵심 기능 통합 테스트**
  - Playwright MCP로 전체 사용자 플로우 E2E 테스트 (모임 생성 → 참여 → 공지 → 카풀 → 정산)
  - 주최자/참여자 2개 계정 시나리오 교차 검증
  - 에러 핸들링 및 엣지 케이스 테스트 (권한 없음, 정원 초과, 중복 신청, 잘못된 입력)
  - RLS 및 Server Action 권한 검증 종합

---

### Phase 4: 이메일 알림 및 고급 기능

- **Task 014: Resend 이메일 알림 시스템 구현 (F011)** - 우선순위
  - Resend SDK 설정 및 `lib/email/` 발송 유틸 작성, 이메일 템플릿 구성
  - 6가지 트리거를 Server Actions에 연동:
    1. 참여 신청 접수 → 주최자
    2. 참여 승인/거절 → 신청자
    3. 공지 등록 → 승인된 참여자 전원
    4. 정산 요청 → 승인된 참여자 전원
    5. 카풀 탑승 확정 → 탑승 신청자
  - 발송 실패 시 비즈니스 로직에 영향 없도록 에러 격리 처리
  - ## 테스트 체크리스트
    - Playwright MCP로 각 트리거 액션 수행 후 발송 호출 검증 (테스트 모드/모킹)
    - 일괄 발송(공지/정산) 시 승인 참여자 전원 대상 검증
    - 이메일 미발송이 핵심 플로우를 막지 않는지 검증

- **Task 015: 사용자 경험 향상 기능**
  - 모임 검색/필터 강화, 로딩 스켈레톤, 토스트 알림
  - 폼 유효성 검사 강화 및 낙관적 업데이트
  - 빈 상태/에러 상태 UX 개선
  - 접근성(키보드 네비게이션, ARIA) 점검

---

### Phase 5: 폴리싱 및 배포

- **Task 016: 성능 최적화 및 품질 보증**
  - Server Component 캐싱 전략 및 데이터 페칭 최적화
  - 튜토리얼/스타터 잔여 컴포넌트(`components/tutorial/`, `deploy-button` 등) 정리
  - `npm run lint`, `npm run type-check`, `npm run format:check` 통과 확인
  - 주요 사용자 플로우 Playwright MCP 회귀 테스트

- **Task 017: 배포 및 모니터링 구성**
  - 프로덕션 환경변수 구성 (Supabase, Resend 키)
  - Vercel 배포 파이프라인 및 빌드 검증
  - 에러 로깅/모니터링 기본 구성
  - 배포 후 스모크 테스트 (인증 → 모임 생성 → 핵심 플로우)
