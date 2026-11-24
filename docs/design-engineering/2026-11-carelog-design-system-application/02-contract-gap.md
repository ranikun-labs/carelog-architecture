# 02 계약 차이 분석: 현재 구현 vs Carelog LOCK vs Foundation v1.0.0

- 기준: carelog-fe origin/main c4d67d4. Foundation 값은 `ds/project/*`(vNext-2 설계안)와 Finance origin/dev 7dd9590 `globals.css`의 "Ranikun Labs Design Foundation v1.0.0" 구현, `SOL_REVIEW.md` §5, `token-baseline.md`.
- Owner 승인(이번 요청): **Carelog 디자인 LOCK 해제 + Design System v1.0.0 적용 착수.** 해제 범위는 *시각 계약*(색·타입·반경·그림자·다크·밀도·반응형 값)이다. *제품 의미 계약*(CustomerEvent 상태 3개·시간축·Overdue 파생·Prepared 표현·상태 문법의 텍스트 동반)은 해제 대상이 아니다 [추정: 요청문이 "디자인 LOCK"만 언급]. 이 해석이 틀리면 §1 "유지" 열이 바뀐다.
- 실행·렌더·테스트는 모두 NOT_RUN. 아래 대비 수치는 hex 기준 직접 계산(WCAG 상대 휘도, python)이며 Carelog 실제 DOM 렌더가 아니다.

## 1. 항목별 비교 (현재 | LOCK README | Foundation | 해소 방향)

| # | 항목 | 현재 구현 [확인] | Foundation v1.0.0 | 충돌/격차 | 방향 |
|---|---|---|---|---|---|
| 1 | 브랜드색 | `--accent-primary` oklch(52% .1 235) ≈ #1c719a(청) | Carelog **B Teal**: L strong #007974 / deep #00605c / subtle #d9f1ef / on-strong #fff. D #3cb7ab / #8fe0d6 / #0f2e2c / on-strong #0b1220 | 값 교체. Finance 파랑과 구분(OKLab ΔE 18.2 vs A 11.0, 설계안 수치) | 토큰 4개 교체. B는 Owner 지정, 시각 차별은 사용자 테스트 NOT_RUN |
| 2 | **브랜드 위 텍스트 규칙** | `text-accent-primary`를 accent 면 위에도 사용 가능 | strong on subtle **4.46:1 미달**(직접 계산). deep on subtle 6.29 | B Teal은 "strong 글자 on brand-subtle"이 불가 | 규칙: brand-subtle 위 글자는 deep만. `WeekStrip`·`SideNavigationRail`·`Badge`는 이미 deep을 쓴다 [확인]. `AuthStatusSurface:37`의 `text-accent-primary on bg-accent-primary-bg`가 위반 [확인] |
| 3 | 상태색 | warning 3종(+rail), success·conflict·info·danger 없음. `--destructive #f04452` | success/warning/danger/conflict/info 각 fg·bg, danger-solid | Destructive 흰 위 **3.71 미달**. Carelog에 success·info·conflict 없음 | Foundation 상태 5종 도입. `destructive`→`danger`(text #c42336 on card 5.77) |
| 4 | 상태 레일 | accent-rail / warning-rail / rail-neutral, 3px(`--status-rail-width`) | Foundation에 "rail" 토큰 없음. 설계안 SOL: "Carelog 예정 레일을 info로" | Carelog 고유 문법을 Foundation 토큰으로 매핑해야 함 | PLANNED rail=`info`, Overdue=`warning`, OCCURRED·CANCELLED=중립. 3px·텍스트 동반 규칙 유지. **[검토 필요]** 중립 레일 값은 Foundation에 없음 → 제안: `--rail-neutral: var(--border)` |
| 5 | 중립 면 | page oklch(97% .004 90) 웜, surface 흰, subtle oklch(96% .006 90) | page #f2f4f6 쿨, card #fff, inset #f2f4f6, control #e9ecef | 웜→쿨 (Foundation D1). 설계안 `07` #9 "예" | 교체. 카드는 page 대비 surface로 구분, **카드 테두리 제거** |
| 6 | 카드 테두리·그림자 | 카드 `border border-border-default`, 일부 `shadow-sm`(AuthStatusSurface:35), 정책 `--shadow-default: none` | 카드 테두리 없음, 그림자는 오버레이(Dialog/시트)에만 | 테두리 사용처 다수: Card, EventForm, CustomerForm, AuthFormPage form, AssistantSurface 블록, CustomerUpcoming/Timeline 항목, EventDetail 메모·확인 패널 | `Card` 컴포넌트가 정본, 화면은 `Card`/`inset`으로. 중첩은 inset만 |
| 7 | 반경 | badge 6 / control 8 / card 12 / pane 16 (`--radius`=card 12, `rounded-lg`=12) | badge 6 / control 10 / inset 12 / card 16 / sheet 24 | 같은 "lg"가 Finance 16 vs Carelog 12(token-baseline §4) | **Owner 확인(§3 D-1)**: 설계안은 "LOCK card 12는 라이브 제품에 그대로"라고도 적었고 `07` #1은 "Carelog 모든 카드 12→16"을 제안. LOCK 해제 승인으로 후자 채택 가능. 추천: card 16/control 10 채택(토큰만, 되돌림 쉬움). **의미 이름 사용**(`rounded-card`), `rounded-lg` 신규 사용 금지 |
| 8 | 타입 | 8단계 `--type-*`(display 24 … label 11). Tailwind 클래스 `text-3xl`·`text-lg` 등 혼용(CustomersPage h1 `text-3xl`, AuthEntry `text-3xl sm:text-4xl`) | 타입 역할 9개(page-title, section-title, content-title, key-value, reading, body, supporting, meta, label). Compact: 22/18/15/20/15·24/14·21/13/12/12. **12px 하한** | 11px 사용: `SideNavigationRail`(`text-[0.6875rem]`), `WeekStrip` 요일, `--type-label` 11 [확인]. 역할 이름·값 전부 다름 | `type-*` 유틸리티 + 밀도 변수. 11px 제거 |
| 9 | 밀도·컨트롤 높이 | Button default min-h 46, sm 44(`min-h-11`), lg 54, icon 44. 입력 `min-h-11` | Carelog **Compact**: 컨트롤 **44**, 터치 최소 44. (Comfortable 48 = Finance) | 46/54 → 44 계열로 정리. 설계안 주요 변경 3: "Compact 컨트롤 40→44" | `--control-h: 44px` 로 Carelog 오버라이드 |
| 10 | 간격 | `--space-1…8` = 4,8,12,16,20,24,32 (번호 이름) | 섹션 32(Compact)/40, 제목→카드 8/12, 카드 패딩 16/20, 관련 8 | 번호→의미 이름 재배열 위험(설계안 D6 "space-N 유지") | `--space-*` 유지 + 의미 유틸리티 추가. 이름 재배열 금지 |
| 11 | 다크 | **일부만 재정의**(token-baseline §2·§3). deep/bg/rail/warning 상속 → deep on surface **2.49, 글자 on 상속 accent-bg 1.06, warning 글자 2.38** 미달. 다크 Primary #3182f6 위 흰 글자 **3.71 미달**. README "not locked" | 전 토큰 Light/Dark 재정의, on-strong 다크 `#0b1220`(Teal strong 위 7.63) | 현 다크는 값이 WCAG 위반이지만 **앱이 `.dark`를 켜는 코드가 없다** [확인: grep 0건] → 지금은 잠재 결함. 해제 후 전체 재정의 | 토큰 PR에서 `.dark` 전 토큰 재정의 + contrast 테스트(Finance `contrast.test.ts` 패턴). Teal 다크 on-strong **[Owner 결정 대기: 설계안 "남은 과제"]** |
| 12 | 포커스 | `:focus-visible` 2px accent, offset 2, 대부분 컴포넌트가 개별 `focus-visible:outline-*` 중복, SideNavigationRail은 inline `style={{outlineColor}}` | 전역 2px `--ring`(=brand strong) offset 2, `:where` specificity 0 | 값 동일, 구현 중복 | 전역 1곳으로 정리, 개별 클래스·inline style 제거. 단 `button.test.tsx`가 `focus-visible:outline-accent-primary` 클래스를 단언 [확인] → 테스트 변경 |
| 13 | 비활성 | `disabled:opacity-40` (LOCK 명시 0.4) | Finance 구현: 50% + `aria-disabled`, SelectChip은 `aria-disabled`+이유 연결. 설계안 `07` #15는 opacity 대신 전용 토큰 제안 | LOCK 문구 vs Foundation | LOCK 해제로 Finance 방식 채택, **텍스트 이유는 제품 흐름이 제공**(README: "RPL-56 does not invent that copy" → 새 문구 만들지 않음) |
| 14 | 반응형 | 375 단일+하단 내비 / md 레일 w-20 / readable 520·560 / **2단 ≥1100 && 가로** | 설계안 `07` #7: 375–767 하단 / 768–1099 좁은 레일 / ≥1100 사이드. 2단 조건을 "가로"→"폭"으로 바꿀지 Owner 결정 대기 | 구조(AdaptiveHost)는 제품 로직 | **구조·breakpoint 유지**(첫 적용 범위 밖). 값(색·타입)만 교체. 폭 기준 전환은 별도 Owner 결정(§3 D-3) |
| 15 | 페이지 헤더 | `PageHeader` h-14, 아이콘 `←` 문자, `text-base font-semibold`, 화면별 `text-3xl font-bold` 혼재 | `type-page-title`, 카드 밖 헤더 | 일관성 없음 | `PageHeader`·화면 헤더를 역할 유틸리티로 통일(화면 PR) |
| 16 | Eyebrow(브랜드색 장식 문구) | `SchedulePage`("시간축"), `AuthEntryPage`, `AppHomePage`, 공개 홈 | 브랜드색은 행동·선택·포커스에만. 장식 eyebrow 삭제(Finance PR-2 AC와 동일 문법) | 문구 삭제는 **ko.ts 키 삭제**, 테스트 영향 | 화면 PR에서 시각 제거, 키는 즉시 삭제하지 말고 미사용 시 정리(i18n parity 테스트 확인 후) |
| 17 | 배지 | 메타(워크스페이스명·descriptor)도 `Badge tone=info/neutral` | "배지는 상태에만, 메타는 텍스트" | P1 문제(설계안) | 화면 PR: descriptor/워크스페이스명은 텍스트. 상태 배지(예정·정리 필요·취소됨·준비됨)만 유지 |
| 18 | 진한 버튼 수 | S1 `+ 일정` primary 1개 [확인] / C1 `+ 고객 추가` primary 1개 / C3 `일정 추가` outline·`AI에게 물어보기` secondary / E2 `기록 완료` primary(+수정·취소 처리 outline/secondary) / 폼 submit primary | strong 화면당 1개 | C3에는 strong 0 → 맞음. E2는 1개 → 맞음 | 위계는 화면 명세(03)에서 확정 |
| 19 | 입력 컴포넌트 | 입력마다 클래스 복붙(`min-h-11 rounded-md border px-3 ...`), `Input/Textarea final = UNDEFINED`(README) | TextField(label·hint·error·카운터), fill=card, 외곽선 `--input`(3:1), 높이 48/44, 반경 10 | **LOCK에서 UNDEFINED였던 영역** → 해제로 확정 가능 | Finance가 TextField를 아직 만들지 않음(origin/dev ui/* 9개에 없음). 중복 구현 방지(04 §4) |
| 20 | 폰트 | Pretendard Variable self-host woff2 | 동일 Pretendard | 로딩 방식만 다름 | 변경 없음. Carelog 방식 유지(Capacitor 오프라인) |

## 2. RPL-56 결과물 중 새 계약과 충돌하는 것 (테스트가 옛 값을 잠그고 있음) [확인]

토큰·클래스를 바꾸면 아래가 **의도대로 깨진다**. 첫 PR에서 같이 갱신해야 하며, "테스트를 통과시키려고 값을 되돌리는" 것은 금지다.

| 파일 | 잠그는 내용 | 처리 |
|---|---|---|
| `src/styles/globals.test.ts` | `--bg-page`·`--bg-surface`·`--bg-subtle`·텍스트 3종·border·`--accent-primary*`·`--warning*`·`--rail-neutral` 정확 문자열, `--radius-*` 6/8/12/16, `--status-rail-width: 3px`, 다크 `--status-neutral-foreground = var(--muted-foreground)` | Foundation 값으로 재작성 + **대비 테스트 추가**(현재 이 파일은 대비를 검사하지 않음) |
| `e2e/foundation.spec.ts` | `--text-tertiary === oklch(53%.01260)`, 내비 링크 ≥44, 포커스 2px/offset 2, 버튼 44 | tertiary 값 기대 갱신. 나머지는 **유지**(여전히 유효, 회귀 가드) |
| `src/components/ui/button.test.tsx` | `min-h-[46px]`, `size-11`, `disabled:opacity-40`, `focus-visible:outline-accent-primary`, `bg-accent-primary`, `border-border-default`, `text-accent-primary-deep` | variant·size 클래스 단언 갱신. **44 터치 최소·텍스트 확대 시 clip 안 함(min-h)** 의도는 보존 |
| `src/components/ui/badge.test.tsx` | tone→`bg-accent-primary-bg`/`text-accent-primary-deep` 등 | 새 tone(`planned`/`prepared`/`overdue`/`cancelled`) 이름 유지 + 클래스 갱신 |
| `src/pages/CustomersPage.test.tsx`, `SchedulePage.test.tsx`, `e2e/rpl-86-customer-cta.spec.ts` | CTA에 `bg-accent-primary`, `min-h-11`, `min-h-[54px]`, `min-h-[46px]` | **호환 별칭 전략**(§4): 첫 PR에서는 Tailwind 이름 `bg-accent-primary`가 계속 존재하게 해 이 3개는 변경 최소화. 높이 클래스 단언만 갱신 |
| `docs/design/carelog/README.md` | 모든 값, "Dark … not locked", "Default shadow: none" | 첫 PR에서 "LOCK 해제 + Foundation v1.0.0 적용" 상태로 개정. **제품 계약 절(시간축·상태 문법·a11y 기준·책임 경계)은 그대로 둔다** |

## 3. Foundation 쪽의 미결정 (Owner/Architect 결정 대기) — 적용 시 가정과 되돌림 비용

| ID | 결정 | 이번 문서의 가정 | 되돌림 |
|---|---|---|---|
| D-1 | 반경: Carelog card 12 → 16, control 8 → 10 (LOCK 해제 효과) | 채택(설계안 D3 "card 16 / control 10 유지 추천") | 토큰 2~3개 |
| D-2 | Teal 다크 `on-strong` `#0b1220`(7.63) | 채택, **Owner 확정 전 "제안" 표기** | 토큰 1개 |
| D-3 | 2단 조건 "가로 && ≥1100" 유지 vs 폭 기준 | **유지**(구조 무변경) | `adaptiveHostModel.ts` 한 함수 |
| D-4 | 상태 레일 PLANNED=info 매핑 | 채택 제안. 브랜드(Teal)와 info(#2a4f86 navy)는 색이 달라 혼동 위험 낮음. 다만 **일정 화면의 PLANNED 레일이 "파랑"이 되어 기존 "예정=브랜드색" 인상이 사라진다** | 레일 토큰 3개 |
| D-5 | 웜→쿨 page 배경 | 채택(Foundation 공통) | 토큰 3개 |
| D-6 | 상태 문법의 `준비됨`·`정리 필요` 배지 문구 | **유지**(제품 계약) | - |
| D-7 | Success·Conflict의 Carelog 사용처 | 토큰은 정의하되 **사용처를 새로 만들지 않는다**(저장 성공/충돌 UX는 README에 "Save failure UX UNDEFINED") | - |
| D-8 | 밀도: Compact 본문 14/21 | 채택(설계안 §5) | 변수 |

## 4. 호환 전략: 이름 두 층 (첫 PR의 핵심 설계 결정)

문제: 현재 코드는 `bg-accent-primary`·`text-accent-primary-deep`·`bg-subtle`·`text-text-tertiary` 등 RPL-56 이름을 172곳(비테스트 tsx 37개 파일)에서 쓴다 [확인: `git grep` 패턴 근사치]. 한 PR에서 이름까지 바꾸면 리뷰 불가·충돌 대형화.

결정(나라면): **Foundation 이름을 정본으로 추가하고, RPL-56 이름은 같은 PR에서 별칭으로 남긴다.** 별칭은 값이 Foundation으로 바뀐다(`--accent-primary: var(--primary)` 식). 화면 PR이 하나씩 Foundation 이름으로 옮기고, 마지막 PR에서 별칭과 RPL-56 이름 테스트를 삭제한다.
- 얻는 것: 첫 PR이 작고 revert 단독 가능, 의미 계약·e2e 대부분 유지.
- 버리는 것: 별칭 기간 동안 두 이름 공존(기술부채). **종료 조건을 PR-9에 명시**(05 문서).
- 위험: 별칭이 새 값을 받으면 `text-accent-primary on accent-bg`처럼 대비가 깨지는 쌍이 숨는다 → 첫 PR에 **대비 행렬 테스트 필수**(§2 `globals.test.ts`).

## 5. 구현 결함 중 Foundation/접근성과 직접 관련된 것 [확인, 첫 적용 범위 구분]

| 결함 | 위치 | 영향 | 처리 PR |
|---|---|---|---|
| 11px 텍스트 | SideNavigationRail 라벨, WeekStrip 요일 | 12px 하한 위반 | 토큰 PR(타입) + Shell/Week PR |
| `text-accent-primary` on accent-bg | AuthStatusSurface:37 | Teal 교체 시 4.46 미달 | Auth 상태 PR |
| 다크 상속 미정의 | globals.css `.dark` | 대비 1.06~2.5 | **토큰 PR** |
| 다크 Primary #3182f6 위 흰 글자 3.71 | Button primary(dark) | 대비 미달 | **토큰 PR** |
| 카드 `shadow-sm`·`rounded-xl` | AuthStatusSurface:35, AuthFormPage form | Foundation 위반 | Auth PR |
| 문자 `←` 아이콘 버튼 | PageHeader | SR은 aria-label로 읽음. 시각은 아이콘 아님 | 공통 PR(lucide ChevronLeft) |
| 입력 중복 클래스 | EventForm·CustomerForm·AuthFormPage·AssistantSurface | 유지보수·불일치 | TextField 도입 후 |
| 일정 `취소 처리`의 비가역성 | EventDetail | CANCELLED로 가면 되돌림 UX 없음. **계약 없음 → 새 확인 Dialog를 추가하지 않는다**(기능 추가 금지). 위험만 기록 | [보류] Owner 질의 Q4 |
| Overdue cue·rail에서 색 외 단서 | OverdueCue | 배지+텍스트 있음(OK) | 유지 |
| Assistant 문구 | 전체 | 계약 대기 | **[보류]** 문구·동작 불변 |

## 6. 계약에서 "UNDEFINED"였다가 이번에 확정이 필요한 것 (README §Explicitly undefined)

| README의 UNDEFINED | 이번 처리 |
|---|---|
| Customer List Row final component | 03 C1: 제안 명세. 검색·필터·정렬은 **추가하지 않음** (계약 없음) |
| Search, filter, sort | **추가 안 함** |
| Input/Textarea 최종 시각 | 03 §공통: Foundation TextField(시각만) |
| Save failure / Event Memo save failure UX | 현행 인라인 `role=alert` 유지, Foundation `Notice` 시각만. 새 재시도 흐름 **추가 안 함** |
| Skeleton delay threshold | 변경 없음 |
| Exact prop signatures | 변경 최소(variant 이름 추가, 기존 이름 별칭 유지) |
| Production API DTO | 범위 밖 |
| Dark-mode final visual contract | **이번에 확정**(토큰 PR) |

## 7. 이 문서의 한계

- 설계안(`ds/project`)은 Carelog 고객 목록·상세·일정 상세 3화면의 시안 근거만 있고 나머지 화면은 이 문서 03에서 처음 명세한다.
- Carelog 다크 Teal 값은 **새 후보**이며 사용자·색각 시뮬레이션·실기기 검증 NOT_RUN.
- `design-system.json`·`tokens.json`의 최종 파일 형식(DTCG)은 NOT_VERIFIABLE(원문 미접속). CSS 변수 구현은 Finance origin/dev 7dd9590을 사실 기준으로 삼는다.
