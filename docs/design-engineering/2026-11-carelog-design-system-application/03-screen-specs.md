# 03 화면별 디자인 명세 (Carelog, Foundation v1.0.0 · B Teal · C 위계 · Compact)

- 근거: 01 인벤토리(화면 ID), 02 계약 차이. 시각 근거는 설계안(`ds/project/06-screens.md`의 Carelog 3화면)과 Finance 적용 분석(`finance-design/03`)의 규칙. 렌더·구현 모두 **NOT_RUN**.
- **"B+C" 해석**: B = Carelog 브랜드 **Teal**(`#007974` 계열, Owner 지정), C = 설계안 시각 방향 **C**(머리 = 핵심 정보, 진한 버튼 1개, 카드 최소, 배지는 상태에만, 메타는 텍스트, 섹션 간격 큼). 둘 다 설계안의 *추천*이며 확정 근거는 이번 Owner 승인이다 [추정: "B+C"를 이렇게 읽음. 다르면 알려줘].
- **제품 고유 UX 유지**: 시간축(일정/고객 2 탭), 상태 레일 3px + 텍스트, Overdue는 PLANNED의 표현, 적응형 2단(AdaptiveHost) 구조·breakpoint, 입력 보존·복귀 경계 흐름은 **바꾸지 않는다.** 이 문서는 시각·위계·컴포넌트 선택만 정한다.
- **금지**: 계약에 없는 기능(검색·필터·정렬·새 확인 Dialog·sticky 액션바 신규 도입·성공 토스트 등) 추가, route·내비 변경, 문구 새로 쓰기. 필요한 것은 "제안(요 Owner 확인)"으로만 적는다.

## 0. 공통 규칙

### 0.1 위계(C) — Carelog 변환

| 규칙 | 내용 |
|---|---|
| 머리 | 페이지 제목 `type-page-title`(Compact 22/ 700). **eyebrow 삭제**(시간축·"Carelog 워크스페이스" 등 브랜드색 장식). 부제는 `type-supporting` 1줄, 없어도 되면 삭제 |
| Key 영역 | 화면당 1~2곳(시간·핵심 값). 사용자가 쓴 글(메모·일정 메모)은 `type-reading`(15/24), 시각 값은 `type-key-value`(20/28) |
| 진한 버튼 | **화면당 1개**(strong). 같은 의도의 두 번째는 subtle/neutral/text |
| 배지 | **상태에만**: `예정`·`준비됨`·`정리 필요`·`취소됨`. descriptor·워크스페이스명·건수는 텍스트 |
| 카드 | 테두리·그림자 없음. page(#f2f4f6) 위 card(#fff). 카드 안 구획은 `inset`(#f2f4f6, 반경 12)만. 제목만 든 카드 금지 → 섹션 제목(`SectionHeader`)은 카드 밖 |
| 브랜드색 | 행동·선택·포커스·활성 탭만. 장식 막대·eyebrow·강조 숫자에 쓰지 않음 |
| 간격 | 섹션 간 32, 제목→카드 8, 카드 패딩 16, 관련 항목 8 (Compact) |
| 안내 | 섹션당 1문장. 중립 안내는 inset, 상태 안내는 Notice(상태색 fg on bg) |
| 위험 | 삭제류 없음(고객/일정 삭제 UI는 현재 없음, **추가하지 않음**). `취소 처리`는 danger가 아니라 중립 텍스트 버튼(상태 전환이지 파괴 아님: 예정 시각 보존) — 단 비가역성은 Q4 |

### 0.2 컴포넌트 사용 지도 (공통 자산 이식 후, 04 참조)

Button(strong/subtle/neutral/text/text-neutral/danger-text/danger-strong) · Badge(tone: neutral/information/success/warning/danger/conflict + Carelog 별칭 planned/prepared/overdue/cancelled) · Card(+inset) · SectionHeader · InfoList/InfoRow · SelectChip · ErrorState · Dialog(**신규 사용처 없음**, 이식만) · Checkbox(**사용처 없음**, 이식만 또는 보류).
신규 필요(Finance 계획 중): TextField/Textarea/FieldMessage, Notice, EmptyState·PageHeader 재정렬, StateView(loading/auth). → 04 §4.

### 0.3 반응형 (구조 불변, 값만)

| 구간 | 현재 구조 [확인] | 명세 |
|---|---|---|
| 375(폰) | 단일 열, 하단 내비(탭 2 + 계정 링크) | 좌우 여백 16. 컨트롤 높이 44. 본문 14/21 |
| 768(태블릿 세로) | 단일 열 md max-w 520, 좌측 레일 w-20 | 레일 라벨 12px로(11px 제거). 레일 항목 높이 56 유지 |
| 1024(세로) | 단일 열 lg max-w 560(readable) / 목록·일정은 none(scan) | scan 화면은 목록 행 `xl:` 가로 배치 유지 |
| ≥1100 가로 | 2단: 마스터 \| 선택 상세 (일정·고객 root) | 패널 사이 구분은 **면 차이**(page vs card)로, 현재 `border-r`는 divider 1px로 축소 허용. 패널 안 스크롤은 각자 |
| 1180 / 1440 가로 (e2e 프로젝트) | 동일 2단 | 그대로. 상세 패널 폭이 커져도 카드 1열 max-w 640 유지(reading 줄 길이) |

폭 기준 전환(D-3)은 이 문서 범위 밖.

### 0.4 Light / Dark

| 용도 | Light | Dark |
|---|---|---|
| page / card / inset / control | #f2f4f6 / #fff / #f2f4f6 / #e9ecef | #0f1114 / #1a1d21 / #22262b / #2a2f35 |
| text 강/본문/보조/3차 | #191f28 / #333d4b / #4e5968 / #5b6673 | #f2f4f6 / #d1d6db / #b4bcc5 / #98a2ad |
| 브랜드 strong / deep / subtle / on-strong | #007974 / #00605c / #d9f1ef / #fff | #3cb7ab / #8fe0d6 / #0f2e2c / #0b1220 (**D-2 제안**) |
| 상태 | success #0a6b3b, warning #8a4a00, danger #c42336, conflict #6b3fb4, info #2a4f86 (fg) | #5fd08f, #f0b44c, #ff8a95, #b79cf0, #8fb2ea |
| 대비(직접 계산 hex) | strong on card 5.27, on-strong(#fff) on strong 5.27, deep on subtle 6.29, deep on card 7.43, tertiary on inset 5.30, warning on warning-bg 6.15, info on info-bg 7.09, danger on card 5.77, **strong on subtle 4.46(미달 → 금지)** | strong on page 7.70, on-strong on strong 7.63, deep on subtle 9.53, strong on subtle 5.91 |

규칙: brand-subtle 위 텍스트는 **deep만**. 다크는 Light 값을 상속하지 않는다(전 토큰 `.dark` 재정의). 다크 전환 방식은 `.dark` class(`@custom-variant dark`) 유지. **현재 앱 어디에서도 `.dark`를 켜지 않는다** [확인: src·index.html에서 classList·prefers-color-scheme·'dark' 설정 코드 grep 결과 0건, SettingsPage에 테마 항목 없음] → 다크 대비 결함은 지금은 잠재 결함이고, 다크 토큰은 사용자에게 노출되기 전까지 검증 대상(contrast 테스트)이다. → 다크를 사용자에게 켜는 경로는 이번 범위 밖(제안: 토큰은 준비, 노출은 Owner 결정 Q6).

### 0.5 상태 표현 표준 (모든 화면)

로딩 `role=status` + 텍스트(스켈레톤은 `aria-hidden`) / 오류 `ErrorState`(아이콘+제목+복구 액션, `role=alert`, 원인 기술정보 미노출 — 현행 `getCarelogMessageKey` 매핑 유지) / 빈 `EmptyState`(제목+설명+CTA 최대 1, CTA는 strong 1개 규칙에 포함) / 입력 오류 필드 `aria-invalid` + 메시지 연결.

---

## 1. 셸·내비게이션

### 1.1 BottomNavigation / SideNavigationRail
| 항목 | 명세 |
|---|---|
| 위계 | 활성 탭만 강조: 활성 = brand-subtle 면 + brand-deep 글자 + 아이콘(레일), 하단 탭은 현재 `text-primary`만 → **아이콘+라벨+굵기**로 색 외 단서 추가(1.4.1) |
| 컴포넌트 | 기존 `Link` 유지. 라벨 `type-label`(12) (11px 제거). 탭 높이 56 유지 |
| 폰 하단 구조 | 탭 56 + 계정 링크 44 + safe-area = 약 100px+ 상시 [확인]. **관찰**: 375×667에서 콘텐츠 영역을 크게 줄인다. 계정을 하단 두 번째 줄로 둔 것은 제품 결정이라 변경하지 않음. Q5로 기록 |
| 포커스 | 전역 포커스 규칙으로 통일, inline `outlineColor` 제거 |
| Light/Dark | 비활성 text-secondary(4.5+), 배경 card, 상단 divider 1px(`--border`) |
| 보호 | `BOTTOM_TABS`·`routes.ts` 불변. `aria-current`·`aria-label` 불변 |

### 1.2 인증 배너 (A5)
복구 중 배너 = info Notice(아이콘+텍스트), 작업 오류 = warning/danger Notice + 재시도(text)·닫기(text-neutral). 현재 `border-b border-warning bg-warning-bg`는 Notice 시각으로 대체, 구조·문구·콜백 불변.

---

## 2. 일정 (S1) `/app`, `/app/schedule`

| 항목 | 명세 |
|---|---|
| 목적 | 오늘 기준 시간순 agenda에서 정리 필요·예정·기록을 훑고 열기 |
| Key | **정리 필요 cue**(있을 때)와 **선택 날짜의 일정 목록**. 제목은 page-title 1줄, eyebrow("시간축")·부제는 삭제 후보(부제는 접근성 설명 용도로 유지 가능, 시각 숨김 `sr-only` 옵션) |
| 진한 버튼 | `+ 일정` strong **1개**(헤더 우측) — 현행 유지 |
| 구성 | 헤더 / WeekStrip(sticky) / OverdueCue / "오늘로 돌아가기"(text) / DateSection[ AgendaRow ] |
| WeekStrip | 활성 일 = brand-subtle + deep 글자 + `aria-current=date`. **요일 11px → 12px**. 일정 점(●)=brand, 오늘 표시(○) = brand 테두리 → 색 외 단서는 `오늘` 라벨(스크린리더 `aria-label` 현행) 유지. 셀 최소 44×56 |
| OverdueCue | warning Notice형(warning-bg + warning 글자 + 건수 텍스트). `정리 필요 N개` 텍스트가 이미 있으므로 `Badge` 중복 제거 가능(배지=상태에만 규칙과 충돌 안 함: 건수는 텍스트) |
| DateSection 헤더 | `inset` 면 + `type-supporting` 700. 오늘 라벨은 brand strong → **inset(#f2f4f6) 위 strong 4.78 OK**, 다크 7.7 OK. 단 `bg-subtle`이 `accent-bg`로 대체되면 미달이므로 헤더 배경은 inset 고정 |
| AgendaRow | 상태 레일 3px(PLANNED=info, Overdue=warning, OCCURRED/CANCELLED=중립) + 시각(key-value 20/ tabular) + 제목(content-title) + 요약 1~2줄 + 고객 링크(deep, 44 터치). **배지는 `정리 필요`·`취소됨`만**(descriptor 배지 → 텍스트). 행 전체 클릭 영역 현행 구조(absolute button) 유지. 행 구분은 divider 1px, 카드로 감싸지 않음 |
| CANCELLED | 헤더(시각·제목)만 약화(text-secondary), 요약·메모는 원래 불투명도 [계약 유지] |
| 하이라이트 | `event-highlight` 애니메이션: brand-subtle → card. `prefers-reduced-motion` 시 정적 유지(현행 유지) |
| 상태 | 로딩 AgendaSkeleton(`aria-hidden`+`role=status` 텍스트) / 오류 ErrorState+재시도(text) / 고객 0명 EmptyState + `첫 고객 추가`(strong 1 — 이때 헤더 `+ 일정`은 **비활성이 아니라 현행대로 표시**하므로 strong 2개가 된다 → 고객 0명 상태에서는 `+ 일정`을 subtle로 낮추는 상태 의존 variant 제안. 코드가 이미 `hasNoCustomers` 분기를 가짐) / 일정 0 EmptyState(neutral `고객 보기`) / 선택 날짜 일정 없음 텍스트 |
| 반응형 | 375: 단일 열 / 768–1024: scan 정책(max-w 없음 lg) / ≥1100 가로: 마스터 agenda + 오른쪽 선택 일정 E2 |
| Light/Dark | 레일 색 3개 모두 다크 별도값(info #8fb2ea, warning #f0b44c, 중립 #2a2f35) |
| 접근성 | `role=status` 로딩, WeekStrip `aria-pressed`+`aria-current`, 행 `aria-label` 현행 |
| 보호 계약 | `data-*` 속성(`data-schedule-page`, `data-agenda-row`, `data-event-status` 등) e2e가 의존 → 변경 금지 |

## 3. 고객 목록 (C1)

| 항목 | 명세 |
|---|---|
| 목적 | 내 고객 훑어 열기 |
| Key | 고객 이름 |
| 진한 버튼 | `+ 고객 추가` strong 1(목록이 있을 때 헤더, 0명이면 헤더 CTA 숨김 + EmptyState CTA 1) — **현행 규칙과 동일** [확인] |
| 행 | 카드 1장 안에 divider 목록(`ListRow` 제안, Finance P1 계획). 이름(content-title) / 최근 연락일(meta) / 맥락 요약 1줄(supporting, tertiary 이상). **워크스페이스 Badge → 텍스트**(메타). 레일/상태 배지는 목록 행에 쓰지 않음(상태는 일정 쪽) — 설계안은 "상태 레일+배지+다음 일정"을 제안했지만 **계약 없음·데이터 필요(다음 일정 조회는 목록 응답에 없음)** → 이번 범위에서 채택 안 함 [보류 제안] |
| 선택(2단) | 선택 행 = brand-subtle 면 + `aria-current=true`. 색 외 단서: 굵은 이름. 현행 유지 |
| 검색·필터·정렬 | **추가하지 않음**(계약 없음) |
| 상태 | 로딩 `role=status` 텍스트 / 오류 EmptyState→ErrorState + 재시도 / 빈 EmptyState + 첫 고객 추가 |
| 반응형 | 375: 행 세로 쌓기. xl: 행 가로 배치 현행 유지. ≥1100: 마스터 목록 + 상세 |
| 개인정보 | 요약 1줄 clamp 유지. 증거 캡처는 합성 데이터 |

## 4. 고객 추가·수정 (C2, C4)

| 항목 | 명세 |
|---|---|
| Key | 고객 이름 입력 |
| 진한 버튼 | 제출 `고객 추가`/`저장` strong 1. 취소 text-neutral |
| 구성 | PageHeader(back) / 폼 카드(card) 안 TextField(이름 필수) + Textarea(메모) + FieldMessage(오류 `role=alert`, 첫 오류 포커스는 현행 유지) |
| 시각 | 입력 fill=card가 아니라 폼이 card 위이므로 **입력 fill = inset 또는 card + 외곽선 `--input`(3:1)** — Finance 규칙 "fill=card, 외곽선 --input"을 따르되 폼 카드 배경이 page이면 입력만 card. → 폼은 카드로 감싸지 않고 page 위에 직접 배치(제목만 든 카드 금지 규칙 준수) |
| 상태 | 제출 중 `aria-busy` + 버튼 라벨 불변, 이름 오류, 서버 오류 `role=alert` |
| 접근성 | `label for`, 높이 44, 메모 최소 120 |
| 보호 | `CustomerFormDraftContext`·검증 로직·`data-customer-form-submit` 불변 |

## 5. 고객 상세 (C3) — 계약상 순서 유지: 맥락 → 다음 일정 → 메모 → 타임라인

| 항목 | 명세 |
|---|---|
| 목적 | 다음 방문 전에 맥락·예정·지난 기록을 한 곳에서 보기 |
| Key | 고객 이름(헤더) + **다음 일정**(예정 시각 key-value) |
| 진한 버튼 | **0개**. `일정 추가` subtle 1(헤더 아래 행동 행), `AI에게 물어보기`는 **[보류] 현행 라벨·위치 유지**, neutral로 낮춤(두 버튼이 같은 outline/secondary로 동급이던 것 해소). `고객 정보 수정` text(헤더 trailing) |
| 구성 | PageHeader / (워크스페이스명 텍스트) / 고객 맥락(SectionHeader `고객 맥락` + 요약 reading + `마지막 업데이트` meta) / 행동 행 / (일정 추가 인라인 폼: card) / 다음 일정 섹션 / 메모 섹션(inset, 3줄 clamp 유지) / 타임라인 섹션 |
| 다음 일정 | 카드 1(PLANNED 레일 info) = 제목·`예정`(상태 배지)·시각. 이후 예정은 InfoList형 링크 행(`이후 예정 N건`, deep 링크, 44). 없으면 inset `다음 일정 없음`(안내 1문장) |
| 타임라인 | 항목 = divider 행, 중립 레일 + 시각 + 제목, CANCELLED만 `취소됨` 배지(과거 항목 전체를 흐리게 하지 않음 — 계약). `더보기` neutral(전폭 대신 자연폭 가능) |
| 일정 추가 폼 | 현행 인라인 폼 유지. `예정으로 추가 / 지금 기록`은 라디오(Choice) — SelectChip이 아니라 **native radio 그룹**(단일 선택, 필수) → ChoiceGroup 스타일(P1) |
| 상태 | 읽기 로딩 `role=status` / 오류 `role=alert` warning→danger Notice + 현행 재시도 흐름 / 생성 실패 입력 보존(현행) |
| 반응형 | 375/768/1024 단일 열 readable(max-w 560). ≥1100: 오른쪽 상세 패널, 설계안 "1360 2열(다음 일정·메모 \| 타임라인)"은 **현행 AdaptiveHost 구조에 없는 새 2열**이라 이번 범위 밖(제안만) |
| Light/Dark | 메모 inset, 타임라인 중립 레일 |
| 보호 | `data-customer-detail-page`·`data-assistant-entry`·`data-add-event`·메모 `data-testid`, 어시스턴트 복귀 상태 |
| 미병합 충돌 | dev/visual-polish/design-handoff가 같은 파일을 수정함(01 §6) → 이 화면 PR은 통합 대상 확정 후 |

## 6. 일정 추가 (E1) / 일정 폼

| 항목 | 명세 |
|---|---|
| E1 1단계 | 대상 고객 선택 [확인: 현재 고객마다 `<button aria-pressed>`, 선택 시 `variant=primary`(=진한 버튼), 나머지 outline, 화살표 키 이동 구현]. **선택된 고객이 strong이 되어 "진한 버튼 1개" 규칙과 충돌**(선택이 행동처럼 보임). 명세: SelectChip 의미(`aria-pressed`, 체크 아이콘 + brand-subtle 면)를 **전폭·좌정렬·줄바꿈 허용** 변형으로 쓴다(Finance SelectChip은 `rounded-full whitespace-nowrap`라 그대로는 불가 → 04 §3에서 변형 합의). 키 이동·`data-customer-selector-option`·`aria-pressed` 불변 |
| 폼(E1·C3 공통 EventForm) | 입력 4개: 방식(라디오 2), 제목(descriptor), 예정/기록 시각(datetime-local), 메모. `원래 예정 시각은 기록에 보존됩니다.`는 inset 안내 1문장. 제출 strong 1, 닫기 text-neutral. 오류 `role=alert` Notice |
| 보호 | `EventCreateDraftContext`(입력 보존), 복귀 경계, `fixedCreateStatus` |

## 7. 일정 상세 (E2)

| 항목 | 명세 |
|---|---|
| 목적 | 한 일정의 상태·시각·메모 확인, 예정이면 기록 완료 |
| Key | **상태 + 시각**(PLANNED: 예정 시각 / OCCURRED: 실제 시각 + 보존된 예정 시각 / CANCELLED: 원래 예정 시각) |
| 진한 버튼 | PLANNED·Overdue 일 때 `기록 완료` strong **1개**. 나머지 `수정`(neutral), `취소 처리`(text-neutral), `미리 준비하기`[보류](text). OCCURRED는 strong 0, `기록에 대해 질문`[보류](text), `수정` neutral |
| 위치 | **본문 끝 인라인 행동 그룹**(현행 구조 유지). 설계안의 "폰 하단 고정 Primary"는 **채택 보류**: 하단 내비 100px+와 겹쳐 화면이 눌린다(§1.1) + StickyActionBar 미존재. 필요하면 후속 Pilot에서 비교 |
| 구성 | PageHeader(back) / 상태 Badge(예정·정리 필요·취소됨·준비됨) + descriptor 텍스트 / InfoList(고객[링크], 시각들) / Overdue면 warning Notice `예정 시각 지남` / 메모(inset, SectionHeader 아님 — 소제목 `메모` supporting) / 행동 그룹 / 기록 완료 확인 패널(실제 시각 입력: **인라인 inset 패널**, 현행 구조 유지, Dialog 신규 도입 안 함) |
| 상태 | 시각 오류 `role=alert` / 저장 실패 Notice(입력 보존, 현행 동작) |
| 접근성 | 확인 패널 열림 시 첫 입력 포커스·닫힘 시 트리거 복귀(현행 e2e 보호), 배지 텍스트 필수 |
| 반응형 | 375 단일 열 / ≥1100 오른쪽 패널. 설계안 "같은 고객의 최근 기록" 보조 카드는 **신규 기능 → 제안만** |
| 보호 | `data-event-actions`·상태 `data-event-status`, 예정 시각 보존 안내 문구 |

## 8. Assistant (AS1) — **[보류]** 시각 정렬만

| 항목 | 명세 |
|---|---|
| 원칙 | 문구·버튼 라벨·동작·fixture 안내(`프로토타입용 예시 결과입니다`)·메모 저장 확인 흐름 **불변**. 계약 확인 전에는 시각 토큰만 따라간다 |
| 시각 | 문맥 블록 = inset, 추천 질문 = 세로 neutral 버튼(좌정렬, 줄바꿈 허용 `h-auto min-h-11` 현행), 직접 질문 입력 = TextField, `질문 보내기` strong **1**, 결과 카드 = card(요약 reading), `메모에 추가` = subtle, 메모 확인 편집 = inset 패널, 저장 확인 `role=status`·포커스 이동 현행 |
| 개인정보 | 문맥에 고객 이름·메모가 노출. 외부 전송 계약 없음 → **캡처·문서에 실데이터 금지**. 이 화면의 새 안내 문구를 만들지 않는다 |
| 진입 버튼 | 라벨 `AI에게 물어보기`·`미리 준비하기`·`기록에 대해 질문` 그대로(Finance 금지어 규칙은 Carelog에 적용하지 않음 — 다른 제품, 설계안 §리뷰) |

## 9. 계정 (ST1)

언어 선택 = SelectChip 2개(`aria-pressed`, 선택=체크+subtle, 라벨 불변 — 현행 `aria-pressed` Button 대체). 로그아웃 = neutral(파괴 아님). 제목 page-title. 현재 언어 표기 텍스트 현행 유지. **테마 토글 추가 안 함**(계약 없음, §0.4).

## 10. 인증 (A1–A5)

| 화면 | 명세 |
|---|---|
| A1 Entry | eyebrow 삭제. 제목 page-title(현 `text-3xl/4xl` 제거), 설명 reading. `로그인` strong **1**, `회원가입` neutral. 콘텐츠 중앙 정렬 컬럼 max-w 440 |
| A2/A3 폼 | 카드 없이 page 위 직접 폼(border+rounded-xl 제거) 또는 card 1장. TextField 3개(계정·비밀번호·확인), 오류 `role=alert` + 첫 오류 포커스(현행), 제출 strong 1(`aria-busy`), 하단 전환 링크 deep. 3.3.8: 비밀번호 붙여넣기 허용(현행 `input` 속성 확인 NOT_RUN) |
| A4 상태 화면 | `AuthStatusSurface`: 아이콘 원(brand-subtle + **deep** 글자로 변경, strong 금지) + 제목 + 설명 + 재시도. `shadow-sm`·`rounded-xl`·border 제거 → card |
| A5 배너 | §1.2 |

## 11. 공개 (P1–P3), 404, 자리표시

- P1/P2: **카피가 템플릿 상태**("Customer Relationship Foundation")이며 화면 계약이 없다. 이번에는 **토큰·타입만 적용**(헤더·CTA 2개·본문 폭), 새 카피·섹션 추가 금지. 공개 랜딩 디자인은 별도 트랙(Finance도 동일). CTA `앱 열기` strong 1, `기능 보기` neutral. 정적 prerender 결과에 JS 없이 보이는 스타일이어야 함 → `pnpm build`의 prerender 검증을 PR AC로.
- P3/N1: EmptyState형(제목·설명·링크 deep).
- 자리표시(C5·C6·R1·F1): `EmptyState` 공통 + PageHeader 재정렬만. **새 기능·문구 없음.** 사용자에게 보이는 `이 화면은 다음 단계에서 채워집니다.`는 현행 유지(계약 없음).

---

## 12. 화면별 공통 수용 기준 요약 (05에서 PR AC로 복사)

진한 버튼 ≤1/상태 · 카드 테두리·그림자 0 · 12px 미만 텍스트 0 · 브랜드색은 행동·선택·포커스 · 배지=상태만 · 터치 ≥44 · Light/Dark 모두 텍스트 4.5:1·비텍스트 3:1 · 375/768/1024/1180/1440 가로 스크롤 0 · `data-*` e2e 훅·route·BOTTOM_TABS 불변 · 문구 변경 0(삭제 eyebrow 제외) · Assistant 보류 항목 동작 불변 · 합성 데이터 증거.
