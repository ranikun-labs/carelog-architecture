# 01 Carelog 화면·흐름·계약 인벤토리

- 역할: Carelog Design Architect. 읽기 전용(레포 수정·커밋·푸시 없음). 코드·렌더·테스트 실행은 전부 **NOT_RUN**.
- 표기: [확인]=파일·git에서 직접 확인 / [추정]=근거 부족한 해석 / [보류]=계약 확인 대기 / NOT_RUN=실행·렌더 안 함.
- 기준 ref: `carelog-fe` **origin/main = c4d67d4** (`gh api`로 default branch = `main` 확인). `carelog-architecture` main 0689422. `carelog-be` CLAUDE.md. Finance 참고 `finance-harness-fe` origin/dev **7dd9590**.

## 0. 먼저 알아야 할 사실 4개

1. **main은 가장 앞선 브랜치가 아니다.** [확인] 미병합 브랜치가 main보다 앞서 있다. 화면 PR이 겹칠 수 있다(§6). 토큰·`ui/*` 파일은 이들과 겹치지 않는다(아래 §6).
2. **FE와 BE의 제품 어휘가 다르다.** [확인] FE = 고객(Customer)·일정(CustomerEvent: PLANNED/OCCURRED/CANCELLED). BE `CLAUDE.md` = 환자·치료 일지·관계(`relations`, `RelationJournal`). `carelog-architecture`는 "Relation/RelationJournal은 legacy 자산이며 CustomerEvent와 1:1 매핑하지 않는다"고 적었다. → 디자인은 **FE 어휘(고객·일정)** 로 한다. "환자/치료 일지" 라벨을 UI에 새로 넣지 않는다. 미병합 `codex/claude-design-handoff-20261123`은 치료사·증상·시행·반응 어휘를 쓴다 → 제품 어휘 Owner 확인 필요(§7 Q1).
3. **현재 FE에는 "치료 일지 목록/작성" 화면이 없다.** [확인] `src/pages`에 일지·템플릿 화면 없음. 일정(CustomerEvent)의 `descriptor`·`note`가 가장 가까운 개념이다. 일지 UI를 상상으로 만들지 않는다.
4. **개인정보가 화면에 직접 나온다.** [확인] 고객 이름(`displayName`), 고객 메모, 고객 맥락 요약, 일정 메모가 목록·상세·Assistant 문맥에 렌더된다. 스크린샷·테스트 데이터·PR 증거는 합성/마스킹 데이터만 쓴다(§5).

## 1. 기술 스택·검증 경로 [확인]

| 항목 | 값 |
|---|---|
| 앱 | Vite 8 + React 19 + react-router 8 + TypeScript 6 SPA, Capacitor 8(iOS/Android) |
| 스타일 | Tailwind 4(`@tailwindcss/vite`, `@theme inline`), class-variance-authority, tailwind-merge, lucide-react, tw-animate-css |
| 폰트 | Pretendard Variable, 레포 내 `src/assets/fonts/PretendardVariable.woff2` self-host (`fonts.css`). Finance는 npm `pretendard` dynamic-subset(다른 방식) |
| 런타임 | Node >=22.22 <23 (CI 22.23.1), pnpm 11.15.1 |
| 빌드 | `pnpm build` = build:client + build:ssr + prerender + verify:prerender (공개 라우트 prerender) |
| 단위 테스트 | vitest 4 + jsdom + Testing Library. 테스트 파일 53개. `pnpm test` |
| E2E | Playwright `e2e/` 12개 spec, 프로젝트 6개: 375/480/768p/1024p/1180L/1440L. 실제 API용 `e2e-real/`(별도 config) |
| 정적 | `pnpm lint`(eslint 10), `pnpm typecheck`(tsc -b), `pnpm format:check`(prettier + tailwind plugin) |
| 통합 | `pnpm verify` = format:check + lint + typecheck + test + build + verify:leakage. `pnpm verify:full` = verify + test:e2e (scripts/verify-full.mjs) |
| CI | `.github/workflows/carelog-fe-ci.yml` (PR→main): install → typecheck → lint → test → build. **format:check, verify:leakage, test:e2e는 CI에 없다.** `rpl64-redirect-safety.yml`은 특정 브랜치 push 전용 |
| 컨텍스트 | i18n ko/en(`src/i18n`), 로케일 ko 기본. `docs/design/carelog/README.md`가 UI 계약(LOCK). `docs/provenance.md`: `react-product-foundation` v0.1.0에서 파생(Finance도 같은 템플릿 계열) |
| PR 규칙 | `AGENTS.md`: PR 템플릿 확인, `/핵심정리` 형식, 미실행은 NOT_RUN, Source Merge와 Production Gate 분리, PII 금지 |

## 2. 라우트·화면 인벤토리 [확인: `src/constants/routes.ts`, `src/app/AppRouter.tsx`]

구현 상태: **실화면**=기능 있음 / **자리표시**=`placeholder.comingSoon` 문구만.

### 2.1 공개 (Prerender 대상, 앱 셸 없음)

| ID | 경로 | 컴포넌트 | 상태 | 계약 근거 |
|---|---|---|---|---|
| P1 | `/:locale` (`/`는 `/ko` redirect) | PublicHomePage | 실화면(문구는 "Customer Relationship Foundation" 템플릿 문구) | 화면 계약 없음. README(LOCK)는 공개 화면 미언급 |
| P2 | `/:locale/features` | FeaturesPage | 실화면(항목 4개 템플릿 문구) | 없음 |
| P3 | `/:locale/*`, `*` | PublicNotFoundPage | 실화면 | route-architecture 성격 |

### 2.2 인증 (`/auth`, 앱 셸·하단 내비 없음)

| ID | 경로 | 컴포넌트 | 상태 | 비고 |
|---|---|---|---|---|
| A1 | `/auth/entry` (`/auth`, `/auth/*` → entry) | AuthEntryPage + AuthLayout | 실화면 | 로그인/회원가입 2 CTA |
| A2 | `/auth/login` | AuthFormPage mode=login | 실화면 | 계정·비밀번호, 오류 `role=alert`·포커스 |
| A3 | `/auth/signup` | AuthFormPage mode=signup | 실화면 | + 비밀번호 확인 |
| A4 | (경계 화면) | AuthBootstrapScreen / AuthBootstrapErrorScreen / AuthRecoveryScreen / CustomerDataLoadingScreen | 실화면 | 세션 확인 중·실패(재시도)·복구 중·고객 데이터 로딩 |
| A5 | (셸 위 배너) | AuthRecoveryBanner / AuthOperationErrorSurface | 실화면 | 복구 중 안내, 403·5xx·네트워크 오류(재시도·닫기) |

로그인 후 이동 [확인 `postAuthRouting.ts`]: 고객이 1명 이상이면 일정, 0명이면 고객 목록(첫 고객 추가). 실제 고객 읽기 실패 시 `/app/customers`로 보내 재시도를 보이게 한다 [확인].

### 2.3 앱 (`/app/*`, AppShell + AdaptiveHost)

| ID | 경로 | 컴포넌트 | 내비 | 상태 | 계약 근거 |
|---|---|---|---|---|---|
| S1 | `/app`, `/app/schedule` | SchedulePage | 일정 탭 | 실화면: 주간 스트립·Overdue cue·날짜 섹션 agenda·로딩 skeleton·오류·고객 없음·일정 없음·"오늘로 돌아가기" | README: PLANNED/OCCURRED/CANCELLED, overdue 파생, 상태 레일 |
| C1 | `/app/customers` | CustomersPage + CustomerCard | 고객 탭 | 실화면: 목록·로딩·오류(재시도)·빈 상태("첫 고객 추가") | README "Customer List Row final component = UNDEFINED" |
| C2 | `/app/customers/new` | CustomerCreatePage + CustomerForm | 고객 탭 | 실화면: 이름 필수·메모 | Input/Textarea 최종 시각 UNDEFINED |
| C3 | `/app/customers/:id` | CustomerDetailPage | 고객 탭 | 실화면: 고객 맥락·일정 추가 폼(인라인)·다음 일정·메모·타임라인(더보기)·로딩/오류 | README: Timeline=OCCURRED 투영, Follow-up=PLANNED 표현 |
| C4 | `/app/customers/:id/edit` | CustomerEditPage | 고객 탭 | 실화면 | 동일 |
| C5 | `/app/customers/:id/import` | CustomerImportPage | - | **자리표시** | 계약 없음 |
| C6 | `/app/customers/:id/handoff` | CustomerHandoffPage | - | **자리표시** | architecture: "Handoff는 Deferred" |
| R1 | `/app/reviews/:id` | ReviewDetailPage | - | **자리표시**(제목 "검토") | 계약 없음 |
| F1 | `/app/follow-ups` | FollowUpsPage | 내비에서 제거됨 | **자리표시** | README RPL-58: 내비는 일정/고객 2개 |
| E1 | `/app/events/new` | EventCreatePage + EventForm | 일정 탭(복귀 경계) | 실화면: 대상 고객 선택 → 폼(예정/지금 기록)·고객 0명 안내 | CustomerEvent 생성 |
| E2 | `/app/events/:id` | EventDetailPage + EventDetail | 일정/고객 탭 | 실화면: 상태 배지·고객 링크·시각·메모·수정·취소 처리·기록 완료(실제 시각 확인) | README 상태 문법 |
| AS1 | `/app/assistant` | AssistantPage + AssistantSurface | context | 실화면(고정 fixture 결과 + 메모에 추가 확인) **[보류]** | 계약 확인 대기. e2e `rpl-87-ai-assistant` |
| ST1 | `/app/settings` | SettingsPage | 계정 링크 | 실화면: 언어 ko/en, 로그아웃 | 없음 |
| N1 | `/app/*` | AppNotFoundPage | - | 실화면 | - |

### 2.4 셸·레이아웃 [확인]

- **BottomNavigation**(<md, `BOTTOM_TABS` 2개: 일정·고객) + 아래 줄 "계정" 링크. **SideNavigationRail**(md 이상, w-20, 라벨 11px).
- **AdaptiveHost**: `shouldUseTwoPane = availableWidth>=1100 && viewportWidth>viewportHeight`. 그 외 단일 프레임(md `max-w 520`, lg `max-w 560`(readable) 또는 none(scan)). 2단은 일정·고객 두 root에서만(마스터 | 선택 상세). 상세는 `data-major-surface`로 공존 판정.
- 라우트 문자열은 `routes.ts` 한 곳, 내비는 `navigation.ts` `BOTTOM_TABS`만 [확인]. 디자인 PR은 둘 다 건드리지 않는다.

## 3. 사용자 흐름 [확인/추정 표시]

```text
공개 P1 ─앱 열기→ /app ─(미인증)→ A1 entry ─→ A2 login | A3 signup ─성공→ (고객 0 → C1 첫 고객 추가 / 그 외 S1 일정)
S1 일정 ─+일정→ E1 ─고객 선택→ 폼 ─저장→ 원래 화면 복귀(복귀 경계) ; 행 열기 → E2 ; Overdue cue → 최신 정리 필요 일정으로 점프
C1 고객 ─행→ C3 ─일정 추가(인라인 폼)→ 저장 ; ─다음 일정/타임라인 항목→ E2 ; ─AI에게 물어보기[보류]→ AS1 ─돌아가기→ C3
E2 PLANNED ─기록 완료→ 실제 시각 확인 → OCCURRED(예정 시각 보존) ; ─취소 처리→ CANCELLED(예정 시각 보존) ; ─수정 ; ─미리 준비하기[보류]→ AS1
E2 OCCURRED ─기록에 대해 질문[보류]→ AS1
ST1 ─언어 변경 / 로그아웃
```

2단(≥1100 가로): 일정 root = agenda | 선택 일정 상세, 고객 root = 목록 | 선택 고객 상세. Assistant는 원래 상세를 마스터 쪽에 유지하는 `AssistantReturnSurface` 규칙 [확인].

## 4. 상태 모델과 표현 계약 [확인: `docs/design/carelog/README.md`]

- `PLANNED` scheduledAt / `OCCURRED` occurredAt(+원래 scheduledAt 보존 가능) / `CANCELLED` 원래 scheduledAt 보존, occurredAt 합성 금지.
- Overdue = `PLANNED && scheduledAt < now` 파생, 상태 아님. Prepared는 PLANNED의 표현.
- 표현: PLANNED 레일(accent) + `예정`, Overdue 레일(warning) + `정리 필요`·메타 `예정 시각 지남`, OCCURRED 레일(neutral), CANCELLED 레일(neutral) + `취소됨`(헤더만 약화, 본문·메모는 원래 불투명도), Prepared `준비됨`. 색만으로 전달 금지.
- **이 의미 계약은 디자인 LOCK 해제와 무관하게 유지**한다(제품 계약이지 시각 계약이 아님). 색·크기 값만 Foundation으로 바뀐다.

## 5. 개인정보 노출 지점 (화면 설계·증거·테스트 주의) [확인]

| 위치 | 표시되는 데이터 | 설계 규칙 |
|---|---|---|
| C1 CustomerCard | 이름, 최근 연락일, 맥락 요약 1줄, 워크스페이스명 | 요약 `line-clamp-1` 유지. 목록 스냅샷에 요약을 늘리지 않는다 |
| C3 | 이름, 맥락 요약, 메모(3줄 clamp), 일정 메모 | 메모 전문은 상세에서만 |
| E2 | 고객 이름 링크, 일정 메모 | - |
| AS1 | 고객 이름·맥락 요약·메모를 Assistant 문맥에 표시, `memoPlaceholder`로 메모 저장 | **[보류]** 외부 AI 전송 여부·보존·동의 계약 없음. 화면은 시각만 정렬, 데이터 흐름 변경 금지 |
| 브라우저 탭 제목·URL | `customerId`가 경로에 노출 | 이름을 경로·title에 넣지 않는다(현재도 없음) |
| 증거 이미지 | - | **합성 데이터만**(fixture `박세입` 등). 실제 고객 데이터·BE 실DB 화면 캡처 금지. `e2e-real/`은 실 API용이라 증거 첨부 금지 |
| 다크/라이트 비교 캡처 | - | PR 본문·Jira에 PII 금지(AGENTS.md) |

## 6. 미병합 브랜치 (화면 PR 충돌 위험) [확인: `git diff --name-only origin/main...<branch>`]

| 브랜치 | main 대비 | 마지막 | 겹치는 파일 |
|---|---|---|---|
| origin/dev | +9 | 11-21 | CustomerDetailPage, CustomerTimeline, CustomerUpcoming, EventForm, AuthStatusSurface, i18n, 공개 홈·기능, EventStoreContext, wrangler.jsonc |
| codex/carelog-design-recovery-20261121 | +12 | 11-21 | dev + 상세 복구 테스트 |
| codex/carelog-visual-polish-20261121 | +14 | 11-21 | dev + "Polish Carelog customer and appointment screens"(c32a664), carelog-fe-ci.yml, EventForm 포커스 |
| codex/carelog-visual-evidence-20261121 | +14 | 11-21 | polish와 같은 범위 + 비교 캡처 |
| codex/claude-design-handoff-20261123 | +10 | 11-23 | dev + `design-handoff/`(치료 경과 이어보기 프로토타입 요청서 + 참고 HTML) |

- **`src/styles/globals.css`, `src/components/ui/*`, `navigation.ts`, `layout/*`(BottomNavigation·SideNavigationRail·AppShell)는 위 브랜치 어디서도 바뀌지 않았다** [확인: 파일 목록 대조]. → 첫 PR(토큰+공통 컴포넌트)은 충돌이 없다.
- 화면 PR(고객 상세·타임라인·일정 폼·Auth 상태)은 dev 계열과 겹친다. **통합 대상(dev를 먼저 main에 병합할지)을 Owner가 정해야 한다**(§7 Q2). 이들 브랜치의 내용 품질은 이번에 검토하지 않았다 [NOT_RUN].

## 7. 이 문서가 모르는 것 / 확인 요청

| ID | 내용 | 추천 |
|---|---|---|
| Q1 | 제품 어휘: 고객/일정 vs 환자/치료 일지/치료 경과. 시각 시스템은 어휘와 무관하나 라벨·빈 상태 문구가 달라진다 | FE 현행 어휘 유지, 어휘 변경은 별도 계약 PR |
| Q2 | 화면 PR의 base: main인가, dev(+9) 병합 후인가 | 첫 PR은 main. 화면 PR 전에 dev 병합 여부 확정 |
| Q3 | Assistant("AI에게 물어보기"·"미리 준비하기"·"기록에 대해 질문") 계약 | **[보류]** 문구·동작 유지, 시각만 |
| Q4 | 일정 `취소 처리`는 CANCELLED로 가며 되돌리는 UX가 없다. 확인 단계를 둘지 | 계약 없음 → 새 Dialog 추가하지 않고 현행 유지, 위험만 기록 |
| Q5 | 폰 하단 chrome(탭 56 + 계정 줄 44 + safe-area ≈ 100px+)이 콘텐츠를 누른다. 계정을 헤더로 옮길지 | IA 변경이라 이번 범위 밖, 현행 유지 |
| Q6 | 다크 모드를 사용자에게 노출할지(현재 `.dark`를 켜는 코드 없음) | 토큰은 준비·검증, 노출은 별도 결정 |
| NOT_RUN | `pnpm verify`, e2e, 렌더, 대비 계산(Carelog 실제 DOM), 스크린리더, 실기기·Capacitor | 구현 PR에서 수행 |
| UNKNOWN | `EventCreateReturnBoundary`·`EventStoreContext` 세부, BE 실제 API 응답 필드 | 화면 PR에서 필요 시 읽기 |
