# 05 구현 인계 계획 (작은 PR 순서)

- 구현은 **Writer**가 한다. 이 문서는 범위·AC·검증·순서·롤백만 정한다. 사실 기준: carelog-fe `origin/main` c4d67d4, Finance 참조 `origin/dev` 7dd9590. 이 단계의 렌더·테스트·CI는 전부 **NOT_RUN**.
- 용어: 화면 ID(S1, C1…)는 01·03, 격차 번호(#n)와 결정 ID(D-n)는 02, 자산·상류 규칙은 04.
- PR 작성은 `carelog-fe/AGENTS.md`와 `.github/PULL_REQUEST_TEMPLATE.md`(공통 표준 v1)를 따른다. Writer 검증과 독립 Reviewer PASS를 분리하고, 미실행은 NOT_RUN. Source Merge와 Production Gate는 별도.

## 0. 전제·순서 원칙

1. **작게, 한 목적, revert 단독 가능.** 데이터 마이그레이션 없음. 각 PR은 route·`BOTTOM_TABS`·`data-*` e2e 훅·CustomerEvent 의미 계약을 바꾸지 않는다.
2. **첫 PR = 토큰 적용 + 공통 컴포넌트 이식**(Owner 지시). 화면 파일은 건드리지 않고 별칭 호환층으로 화면이 자동으로 새 값을 받는다. **그래서 PR-1은 모든 화면의 시각을 바꾼다**(색·반경·버튼 높이·페이지 배경). 이를 가리지 않고 AC에 전 화면 Before/After를 넣는다.
3. **PR-1은 미병합 브랜치와 파일이 겹치지 않는다**(01 §6 확인). 화면 PR 중 C3·E1/EventForm·Auth 상태는 `dev` 계열과 겹치므로 통합 대상 확정(Q2) 후.
4. 다크는 앱이 켜지 않으므로(`.dark` 설정 코드 없음) **테스트·캡처에서 `document.documentElement.classList.add('dark')`로 강제**한다.
5. 시각 증거는 **합성 fixture 데이터만**(e2e fixture `박세입` 등). 실제 고객 데이터·`e2e-real`·BE 실DB 캡처 금지.

## 1. 선결 (PR 아님)

| ID | 항목 | 담당 | 추천 |
|---|---|---|---|
| PRE-1 | LOCK 해제 범위 확인: 시각 계약만 해제, 제품 의미 계약 유지 (02 머리) | Owner | 예 |
| PRE-2 | 화면 PR base: main 직행 vs dev(+9) 먼저 병합 (Q2) | Owner/Writer | PR-1~PR-5는 main, PR-8 이후는 통합 확정 뒤 |
| PRE-3 | D-1(반경 16/10), D-2(다크 on-strong), D-3(2단 조건 유지), D-4(PLANNED=info 레일) | Owner/Architect | 02 §3 가정 채택 |
| PRE-4 | 상류 선행: 밀도 변수화 Finance PR 여부(S-2), TextField/Notice 작성 순서(04 §4-3) | Owner | 변수화 먼저, TextField는 Finance PR-1 후 복사 |
| PRE-5 | 제품 어휘(Q1), Assistant 계약(Q3), 취소 처리 비가역 UX(Q4), 하단 내비 계정 줄(Q5), 다크 노출(Q6) | Owner | 이번 PR 범위에서 모두 "현행 유지" |

## 2. PR 목록

| PR | 이름 | 범위 | 선행 | base |
|---|---|---|---|---|
| **PR-1** | 토큰 + 공통 컴포넌트 이식 | `globals.css` Foundation 토큰, `ui/*` 7종, 대비·드리프트 테스트, README 개정 | PRE-1,3,4 | main |
| PR-2 | 셸·공통 표면 | 내비(11px 제거·포커스 정리), PageHeader, EmptyState/ErrorState 사용, 자리표시 4화면, 404 | PR-1 | main |
| **PR-3** | **Pilot A: 고객 목록 C1** | CustomersPage + CustomerCard | PR-2 | main |
| **PR-4** | **Pilot B: 일정 S1** | SchedulePage·WeekStrip·DateSection·AgendaRow·OverdueCue·AgendaSkeleton | PR-3 승인 | main |
| PR-5 | 일정 상세 E2 | EventDetail(+확인 패널) 위계·Notice | PR-3 | main |
| PR-6 | 입력 공통 + 고객 폼·로그인 폼 | TextField/Notice 복사(상류 후), CustomerForm(C2/C4), AuthFormPage(A2/A3) | PRE-4, PR-3 | main |
| PR-7 | Auth 진입·상태·배너 | AuthEntryPage, AuthStatusSurface(A4/A5), AuthLayout | PR-6, PRE-2 | dev 반영 후 |
| PR-8 | 고객 상세 C3 + 일정 폼 E1 | CustomerDetailPage·Context·Upcoming·Memo·Timeline, EventForm, EventCreatePage(SelectChip block) | PR-6, PRE-2 | dev 반영 후 |
| PR-9 | Assistant 시각 + 계정 + 공개 | AssistantSurface(시각만), SettingsPage(SelectChip), PublicHome/Features/NotFound 토큰 적용 | PR-8 | dev 반영 후 |
| PR-10 | 정리 | 호환 별칭·Button `primary` alias 삭제, 11px·`rounded-lg`·`text-sm` grep 0, README 최종, 중복 키 정리 | 전부 | 마지막 |
| (선택) PR-C | CI 보강 | `pnpm format:check`, `verify:leakage`, (경량) e2e를 CI에 추가 | - | 시각 PR과 독립 |

병렬: PR-3·PR-4·PR-5는 파일이 겹치지 않아 병렬 가능(`SchedulePage`·`EventDetail`·`CustomersPage`는 dev 계열 변경 목록에 없음 [확인]). 단, 같은 `globals.css`를 건드리지 않는다는 약속 필요.

## 3. 공통 AC (모든 화면 PR에 복사)

| # | 기준 | 확인 |
|---|---|---|
| 1 | 진한 버튼(strong/danger-strong) ≤ 1 / 화면 상태 | DOM·variant 검사 |
| 2 | 카드 테두리·그림자 0(Dialog 제외), 중첩은 inset만 | `git diff`에서 `border`·`shadow-` grep |
| 3 | 브랜드색 = 행동·선택·포커스·활성 탭, 장식 eyebrow 0 | 시각 + grep |
| 4 | 배지는 상태(예정·준비됨·정리 필요·취소됨)만 | 시각 |
| 5 | `type-*` 역할 사용, **12px 미만 텍스트 0**, `rounded-lg`·`text-sm`·`text-xs`·`text-[` 신규 0 | grep |
| 6 | WCAG 2.2 AA: 텍스트 4.5:1, 비텍스트 3:1, 터치 ≥44, 포커스 가림 없음(2.4.11), 상태를 색만으로 전달 안 함 | contrast 테스트 + 수동 |
| 7 | Light·Dark 모두 렌더(다크는 class 강제), brand-subtle 위 글자는 deep | 캡처 + 테스트 |
| 8 | 375·768·1024(세로)·1180·1440(가로) 가로 스크롤 0, 정보 손실 0 | Playwright 6 프로젝트 기존 구성 활용 |
| 9 | 보호 계약 무변경: `routes.ts`·`BOTTOM_TABS`·`data-*` 훅·CustomerEvent 상태 3개·Overdue 파생·예정 시각 보존·AdaptiveHost 판정(`>=1100 && landscape`) | diff + e2e |
| 10 | 문구 변경 0(삭제된 eyebrow 키 제외), ko/en parity 테스트 통과 | i18n 테스트 |
| 11 | 계약에 없는 기능·route·Dialog·sticky 바·검색/필터/정렬 추가 0 | PR 설명 체크리스트 |
| 12 | **Assistant 보류 항목 동작·문구 불변** | e2e `rpl-87-ai-assistant` |
| 13 | 증거: Before(main)/After 캡처(합성 데이터), 개인정보 0, 미실행 NOT_RUN | PR 본문 |

## 4. PR별 범위·AC·검증·롤백

기본 검증(원문 확인된 스크립트): `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build && pnpm verify:leakage`(= `pnpm verify`), 시각·접근성은 `pnpm test:e2e`(또는 `pnpm verify:full`). **CI는 typecheck·lint·test·build만 실행**하므로 나머지는 Writer가 로컬 수행하고 PR 본문에 명령·결과를 적는다.

### PR-1 토큰 + 공통 컴포넌트 이식

- **범위**
  - `src/styles/globals.css`: Foundation v1.0.0 토큰(중립·글자·상태 5종·반경 세트·포커스·버튼 리셋·`.dark` 전 토큰 재정의) + **Teal 브랜드(Light/Dark)** + **Compact 밀도 변수**(`--control-h: 44px`, 타입 역할 Compact) + **RPL-56 별칭**(`--accent-primary*`·`--bg-*`·`--text-*`·`--warning*`·`--border-*`을 Foundation 값으로, `--status-rail-width`·레일 3종 매핑 D-4). `@theme inline`에 Foundation 이름과 기존 이름을 모두 노출.
  - `ui/*`: Button(7 emphasis + 기존 alias + `primary`), Badge(Foundation tone + `planned/prepared/overdue/cancelled/info` 로컬 tone), Card(+inset), **신규 이식** SectionHeader, InfoList/InfoRow, SelectChip, ErrorState(+각 테스트).
  - **이식하지 않음**: Dialog, Checkbox(사용처 없음, 04 §1.2), TextField/Notice(상류 선행 필요).
  - 대비 테스트: Finance `contrast.test.ts` 패턴을 Carelog 토큰 이름으로(쌍 목록: text×surface, brand-deep on subtle, on-strong on strong, 상태 fg on bg, **Teal strong on subtle 금지 쌍은 "사용 금지 목록"으로 주석**).
  - 드리프트 테스트 + `design-foundation.lock.json` + 복사 파일 provenance 헤더(04 §4).
  - 테스트 갱신(02 §2): `globals.test.ts`·`button.test.tsx`·`badge.test.tsx`·`CustomersPage.test.tsx`·`SchedulePage.test.tsx`의 값/클래스 단언, `e2e/foundation.spec.ts`의 tertiary 값, `e2e/rpl-86-customer-cta.spec.ts`.
  - `docs/design/carelog/README.md` 개정: Status를 "LOCK 해제, Foundation v1.0.0 적용 중"으로, **제품 계약 절은 그대로**, 시각 값 표는 Foundation 참조 + 차이 표(04 §4-5).
- **AC**
  1. 공통 AC 6·7·9 + 아래.
  2. 대비 테스트가 Light·Dark 전 쌍에서 4.5:1(텍스트) 통과, 02의 실패 쌍(다크 상속 2.4 이하 등)이 **테스트에서 재현되지 않음**.
  3. 별칭 이름(`bg-accent-primary` 등)이 모두 존재해 화면 파일 diff가 **0줄**(grep으로 `src/pages`·`src/components/{customers,schedule,layout,auth,assistant}` 무변경 확인).
  4. `Button size` 높이가 44 계열(min-h 유지: **텍스트 확대 시 clip 안 함** 의도 보존 — `h-*` 고정이 아니라 `min-h-*` 유지 여부를 Writer가 결정하고 `e2e/foundation.spec.ts`의 "generic text controls expand" 시나리오 통과).
  5. 전 라우트(S1·C1·C2·C3·E1·E2·AS1·ST1·A1~A3·P1·P2·404) Before/After 캡처 375/1180 × Light/Dark: **레이아웃 깨짐·잘림·대비 붕괴 없음**(색·반경·높이 변화만).
  6. Provenance·lock·드리프트 테스트 동작(복사 파일을 임의 수정하면 실패하는지 1회 확인).
  7. 공개 prerender 산출물(`pnpm build`의 `verify:prerender`) 통과, 폰트 변경 없음.
- **검증**: `pnpm verify` + `pnpm test:e2e`(6 프로젝트) + 캡처. NOT_RUN 가능 항목(실기기·스크린리더·Capacitor)은 명시.
- **위험**: 별칭이 새 값을 받아 `text-accent-primary on accent-bg` 같은 쌍이 숨는다(AuthStatusSurface:37 — Teal strong on subtle 4.46 미달). → 대비 테스트에 DOM 레벨은 없으므로 **수동 캡처 점검 항목에 명시**, 화면 PR(PR-7)에서 해소하되 PR-1에서 해당 클래스만 `deep`으로 바꾸는 1줄 예외를 허용(Writer 판단, PR 본문 기록).
- **롤백**: 단일 revert(토큰·ui·테스트 동반). 데이터 영향 0.
- **크기 가드**: diff가 리뷰 한도를 넘으면 PR-1a(토큰+테스트+README) / PR-1b(ui 이식+테스트)로 분할. 1a 단독으로도 화면은 정상.

### PR-2 셸·공통 표면
- **범위**: BottomNavigation·SideNavigationRail(라벨 12px, `aria-current`, 활성 = brand-subtle+deep, 개별 inline outline 제거), PageHeader(lucide 뒤로가기 아이콘, `type-page-title`), `common/EmptyState`·ErrorState 사용 정렬, C5·C6·R1·F1·N1·AppHome(자리표시) 정렬.
- **AC**: 공통 AC + 내비 링크 ≥44×44 유지(`e2e/foundation.spec.ts`), `data-bottom-navigation`·`data-side-navigation` 훅 불변, 11px 0, `BOTTOM_TABS`·`routes.ts` diff 0, 하단 내비 높이 증가 없음(Q5: 현행 유지).
- **검증**: `pnpm verify`, `e2e/foundation.spec.ts`·`e2e/adaptive-host.spec.ts`, 375/768/1024/1180 캡처.
- **롤백**: revert.

### PR-3 Pilot A 고객 목록 C1
- **범위**: `CustomersPage.tsx`, `CustomerCard.tsx`(+테스트). 03 §3. **검색·필터·정렬·다음 일정·상태 레일 추가 금지.**
- **AC**: 공통 AC + `+ 고객 추가` strong 1(0명이면 헤더 CTA 없음·빈 상태 CTA 1, 현행 동작 유지), 워크스페이스명 Badge → 텍스트, 카드 테두리 0, 선택 행 `aria-current` 유지, 오류 = ErrorState + 재시도, 기존 `CustomersPage.test`·`e2e/customer-lifecycle`·`rpl-86-customer-cta`·`adaptive-host` 의미 보존.
- **검증**: `pnpm verify` + 해당 e2e + 375/768/1180 × Light/Dark 6장 + **Owner 시각 승인**(Go/No-Go for PR-4 이후).
- **롤백**: revert(2 파일).

### PR-4 Pilot B 일정 S1
- **범위**: 03 §2 전체. WeekStrip 12px, AgendaRow 레일 + 시각 위계, `OverdueCue` Notice형, DateSection inset 헤더, EmptyState/ErrorState, `+ 일정` strong 1(고객 0명 상태 variant 낮춤은 **제안**: Owner 승인 시).
- **AC**: 공통 AC + 상태 문법 표(01 §4) 전부 렌더 확인(예정·정리 필요·준비됨·기록됨·취소됨), 색 외 단서(텍스트 배지·메타) 존재, 레일 3px 유지, `event-highlight` reduced-motion 동작 유지, `data-agenda-row`·`data-event-status`·`data-week-strip`·`data-today-anchor` 불변, `e2e/schedule*.spec.ts`·`rpl-79-surface-composition.spec.ts` 통과, 레일 3색 다크 값 확인.
- **검증**: `pnpm verify`, 해당 e2e, 캡처(정리 필요 포함 데이터), IntersectionObserver 기반 "오늘로 돌아가기" 동작 회귀 확인(코드는 안 건드림).
- **롤백**: revert.

### PR-5 일정 상세 E2
- **범위**: `EventDetail.tsx`(+테스트). 상태 배지·InfoList·메모 inset·행동 그룹 위계(기록 완료 strong 1)·Overdue Notice·확인 패널 inset(인라인 유지).
- **AC**: 공통 AC + 기록 완료 → 실제 시각 확인 → OCCURRED(예정 시각 보존), 취소 처리 → CANCELLED(예정 시각 보존) 동작 불변, **취소 확인 Dialog 추가 안 함**, 확인 패널 포커스·복귀 e2e(`event-mutations`) 통과, `미리 준비하기`·`기록에 대해 질문` 라벨 불변.
- **검증**: `pnpm verify` + `e2e/event-mutations.spec.ts`, `schedule-origin-event.spec.ts`.

### PR-6 입력 공통 + 고객 폼·로그인 폼
- **선행**: Finance TextField/Notice/FieldMessage 병합 후 **복사**(없으면 Carelog가 먼저 작성하고 Finance가 복사 — 한 곳에서만, 04 §4-3).
- **범위**: `ui/text-field`·`field-message`·`notice`(+테스트), `CustomerForm`, `AuthFormPage`(+테스트).
- **AC**: 공통 AC + label↔input 연결, `aria-invalid`+`aria-describedby`, 오류 `role=alert`, 첫 오류 포커스·`CustomerFormDraftContext` 입력 보존 불변, 높이 44, 입력 외곽선 `--input` 3:1, 비밀번호 붙여넣기 허용(3.3.8) 확인(`type`·`autoComplete` 속성 diff 0), `e2e/auth*.spec.ts`·`customer-lifecycle` 통과.

### PR-7 Auth 진입·상태·배너 / PR-8 고객 상세·일정 폼 / PR-9 Assistant·계정·공개
- 각 03 §10 / §5–6 / §8–11. **선행 조건: PRE-2(dev 반영)**. 겹치는 파일을 이 PR이 새 base 위에서 다시 읽고 시작(이 문서는 main c4d67d4 기준이라 dev의 변경을 반영하지 않음).
- PR-8 특이 AC: E1 고객 선택 SelectChip 변형이 `aria-pressed`·화살표 키 이동·`data-customer-selector-option` 유지, 라디오(예정/지금 기록) native 유지, 타임라인 CANCELLED만 배지, 일정 추가 인라인 폼 `EventCreateDraftContext` 입력 보존, 어시스턴트 복귀 상태(`adaptiveCustomerId`) 불변.
- PR-9 특이 AC: **Assistant 문구·fixture 안내·메모 저장 확인 흐름 불변**, 설정 언어 선택 SelectChip `aria-pressed` 유지, 공개 P1/P2는 prerender HTML에서 스타일 깨짐 없음(JS 비활성 캡처 1장), 새 카피 0.

### PR-10 정리
- **범위**: RPL-56 별칭·Button `primary` alias·`bg-subtle` 계열 이름 제거 → Foundation 이름으로 일원화, `globals.test.ts`의 별칭 단언 삭제, README 최종.
- **AC**: `git grep -nE "accent-primary|bg-subtle|text-text-|border-(subtle|default)"` 0건(테스트 설명 제외), 11px/`rounded-lg`/`text-sm`/`text-xs` 0, `pnpm verify` + `pnpm test:e2e` 전체, lock 최신.

### (선택) PR-C CI 보강
- 현재 CI는 `format:check`·`verify:leakage`·e2e를 실행하지 않아 토큰 PR의 시각·포맷 회귀를 CI가 못 잡는다 [확인]. **추천**: `pnpm format:check`와 `verify:leakage` 추가(저비용), e2e는 `mobile-375`+`desktop-1180-landscape` 두 프로젝트만. `carelog-fe-ci.yml`은 `codex/carelog-visual-polish-20261121`도 수정하므로 충돌 주의. 시각 PR의 선행 조건은 아님.

## 5. 증거 수집 방법(합성 데이터)

- Playwright `build:e2e`(test mode)로 fixture 앱 구동 → `page.addInitScript`로 `.dark` 강제 → 라우트별 `page.screenshot`. 고객 이름은 fixture 값만. 캡처 파일은 PR 첨부, 레포에 커밋하지 않는다(대용량·PII 방지).
- 대비는 `contrast.test.ts`(토큰 파일 파싱)가 정본. 렌더 대비(axe 등)는 도구 미도입이라 **NOT_RUN 기본**, 필요 시 별도 합의.

## 6. 이번 인계가 모르는 것

- Foundation DTCG 원본(`tokens/*.tokens.json`)과 Compact 줄 높이 일부는 접근하지 못했다(UNKNOWN, 04 §2).
- 미병합 `dev`·`visual-polish`·`design-handoff` 브랜치의 구현 품질과 의도는 검토하지 않았다(NOT_RUN). 특히 `design-handoff`는 치료 경과 어휘를 쓴다(Q1).
- 모든 렌더·접근성·테스트·CI는 NOT_RUN. Capacitor 실기기, 스크린리더, 색각 시뮬레이션, 사용자 테스트(Teal 구분) 미수행.
- BE의 실제 고객·일정 API 필드(목록에 다음 일정 포함 여부 등)는 확인하지 않았다. 03에서 이를 필요로 하는 제안(고객 목록의 다음 일정·상태)은 **채택 안 함**으로 처리했다.
- 제품 어휘(고객/환자)와 Assistant, 취소 처리 비가역 UX, 다크 노출 경로는 Owner 질의로 남아 있다(PRE-5).
