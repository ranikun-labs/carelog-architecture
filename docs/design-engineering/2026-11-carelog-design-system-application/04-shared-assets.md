# 04 Finance와 공유 가능한 자산과 중복 구현 방지

- 사실 기준: Finance `origin/dev` **7dd9590**(`src/styles/globals.css`, `src/components/ui/*`, `contrast.test.ts`, `fonts.*`), Carelog `origin/main` c4d67d4. 전제: **공유 패키지 배포 없음**(`design-system`(로컬 clone, 비보존)은 원격 저장소가 있으나 로컬 clone에 커밋 0개 = 빈 레포 [확인]).
- 두 FE는 같은 템플릿(`react-product-foundation` v0.1.0)에서 파생 [확인: `docs/provenance.md`]이라 `cn`, `@/` alias, shadcn 구조, Tailwind 4 `@theme inline`이 같다 → 복사 이식 비용이 낮다.
- 렌더·테스트·이식 시도 모두 NOT_RUN.

## 1. 공유 자산 목록과 이식 판단

판정: **복사**(파일 그대로, 로컬 수정 금지) / **파생**(구조는 따르되 제품 값을 얹음, 차이를 문서로 고정) / **제품 전용**(공유 대상 아님) / **보류**.

### 1.1 토큰 (`globals.css`)

| 자산 | Finance 7dd9590 | Carelog 처리 | 판정 | 이유·차이 |
|---|---|---|---|---|
| 중립 면 `--page/--inset/--control/--card`, scrim, overlay-shadow | 값 정의 | 그대로 | **복사(값)** | Foundation 공통. Light/Dark 동일 |
| 글자 4단 `--text-body/-secondary/-tertiary/-disabled` | 정의 | 그대로 | **복사(값)** | 공통 |
| 상태 5종 `success/warning/danger/conflict/info`(+`-bg`) + `danger-solid`/`on-danger-solid` | 정의 | 그대로 | **복사(값)** | 브랜드와 독립(D3) |
| 브랜드 `--brand-deep/-subtle/-border`, `--primary`, `--primary-foreground` | Finance Blue | **Teal로 교체** | **파생** | 제품별 4~5개 값만 다름. 이 블록을 별도 선언으로 분리 |
| shadcn 호환 이름(`--background`, `--primary` …) | Foundation 값으로 매핑 | 동일 매핑 | **복사(구조)** | 기존 shadcn 코드 호환 |
| 반경 `--radius-badge 6/control 10/inset 12/card 16/sheet 24` | `@theme` | 동일(D-1) | **복사(값)** | 의미 이름 |
| 포커스 전역 규칙(`:where(...):focus-visible`) | 있음 | 동일 | **복사** | specificity 0 |
| 버튼 리셋 `:where(button…){cursor…}` | 있음 | 동일 | **복사** | - |
| 타입 `@utility type-*` 9개 | **Comfortable px 하드코딩**(24/18/16/22/16·27/15·24/14/13/13) | **Compact 값 필요**(22/18/15/20/15·24/14·21/13/12/12) | **파생(차이 해소 필요)** | 아래 §2 문제 1 |
| 컨트롤 높이 `--control-h: 48px`, `--touch-min: 44px` | 있음 | `--control-h: 44px`(Compact) | **파생** | 변수만 다름. 단 Finance `Button`이 `h-12` 하드코딩 → §2 문제 1 |
| 어댑티브 셸 CSS(`.adaptive-*`, `.journal-workspace*`) | Finance 전용 | 사용 안 함 | **제품 전용** | Carelog는 `AdaptiveHost.tsx` 구조가 다름 |
| 다크 `.dark` 전 토큰 | 정의 | 동일 값 + Teal 다크 | **복사+파생** | `.dark` 별칭 재선언 주석 유의(조상에 .dark) |
| Carelog 별칭(`--accent-primary*`, `--bg-*`, `--warning-rail` …) | 없음 | **Carelog만**(02 §4 호환층) | **제품 전용(임시)** | PR-9에서 삭제 |
| 상태 레일 3종·`--status-rail-width: 3px` | 없음 | 유지(Foundation 토큰으로 매핑) | **제품 전용** | Carelog 의미 문법 |
| 폰트 `fonts.css` | npm `pretendard` dynamic subset | self-host woff2 유지 | **복사 안 함** | 로딩 방식만 다름. Capacitor 오프라인 전제 변경 금지 |
| `contrast.test.ts` 패턴 | 토큰을 읽어 쌍별 4.5:1 검사 | 같은 파일을 Carelog 토큰 이름에 맞춰 이식 | **복사(로직)+파생(쌍 목록)** | Carelog `globals.test.ts`는 대비를 검사하지 않는 약점 [확인] |
| `fonts.test.ts` | 있음 | 내용 미확인 | **보류** | 이식 전 읽기 필요 |

### 1.2 컴포넌트 (`src/components/ui/*`)

| 컴포넌트 | Finance | Carelog 현재 | 판정 | Carelog 필요 변경 |
|---|---|---|---|---|
| Button (7 emphasis + 호환 alias) | strong/subtle/neutral/text/text-neutral/danger-text/danger-strong, alias default/destructive/secondary/outline/ghost/link, size default 48 / sm 36+터치 44 / lg 54 / icon 44 | variant default/primary/destructive/outline/secondary/ghost/text/link, size min-h 46/44/54/44 | **복사 + 높이 변수화** | `h-12` → `h-(--control-h)` 한 줄(상류 반영) 후 복사. Carelog 호출부의 `variant="primary"`는 alias로 추가해 호환(Finance는 `primary` alias 없음 — **추가 필요**) |
| Badge | tone neutral/information/success/warning/danger/conflict (+ info, destructive 호환) | tone neutral/planned/prepared/overdue/cancelled/info/destructive | **파생** | Foundation tone은 복사. `planned`·`prepared`·`overdue`·`cancelled`는 **Carelog 의미 tone**이라 로컬 추가(= `information`/`warning`/`neutral` 별칭). 상류에 넣지 않음 |
| Card (+inset) | `inset` prop, 테두리 없음 | 테두리·`rounded-lg`·`shadow-none` | **복사** | `rounded-card/inset` 반경 필요(토큰 PR에서 제공). `border` 제거에 따른 시각 확인 |
| SectionHeader | 있음(h2/h3/h4, action) | 없음 | **복사** | C3·C1·S1에서 사용 |
| InfoList/InfoRow | dl/dt/dd row·stack | 없음(`dl`을 EventDetail에서 직접) | **복사** | E2·C3 |
| SelectChip | button+aria-pressed, 체크 아이콘, `rounded-full whitespace-nowrap` | 없음(언어 선택은 Button aria-pressed, 고객 선택은 primary 버튼) | **복사 + 전폭 변형 합의** | E1 고객 선택용 `block`/`justify-start`/줄바꿈 변형 필요 → 상류에 `layout="block"` 제안(§4) |
| Checkbox | native checkbox + label | 사용처 없음 | **복사하지 않음(보류)** | 사용처 생길 때. 미사용 코드 이식 금지(YAGNI) |
| ErrorState | alert 아이콘+제목+action | `EmptyState`로 오류도 대행 | **복사** | 오류 화면 3곳(S1·C1·C3)에서 사용 |
| Dialog | native `<dialog>`, 시트/모달, danger 초기 포커스 취소 | 없음 | **복사하지 않음(보류)** | Carelog에 모달 사용처 없음(기록 완료 확인은 인라인 패널). 계약 없이 사용처 신설 금지. 필요 시 그때 복사 |
| TextField/Textarea/FieldMessage/Notice | **없음**(Finance 계획 PR-1 [05-handoff-plan], 미구현 — 미병합 브랜치 `feat/step8-design-system-foundation` 등에도 `ui/text-field` 없음 [확인]) | 입력 클래스 복붙 | **선행 합의 필요** | §4 |
| EmptyState/PageHeader | Finance `common/*`(재정렬 대상, 미재정렬) | Carelog `common/*` 동일 형태 | **제품 전용 → 상류 합의** | 두 레포가 각자 재정렬하면 중복. §4 |
| ListRow, StateView, StickyActionBar, Disclosure, ChoiceGroup | 없음(Finance P1 계획) | 없음 | **보류** | 필요 화면 PR 직전에 한 곳에서만 구현 |

### 1.3 Carelog 전용 (공유 대상 아님, 상류에 올리지 않음)

`AgendaRow`·`DateSection`·`WeekStrip`·`OverdueCue`·`AgendaSkeleton`(시간축), 상태 레일 문법, `CustomerCard`, AdaptiveHost 2단, Assistant 표면. 이들은 공통 토큰/컴포넌트를 **소비**만 한다.

## 2. 공유를 막는 구조 문제 2개

1. **밀도(Compact)가 Finance 컴포넌트·유틸리티에 하드코딩되어 있지 않고 반대로 Comfortable만 하드코딩.** [확인] `@utility type-page-title { font-size: 24px }`, `Button default: h-12 … text-[15px]`. Carelog는 22px·44·14/21이 필요하다. 그대로 복사하면 Carelog 밀도가 틀리고, 값을 Carelog에서 고치면 **복사 금지 규칙이 깨진다**.
   - 해결(나라면): 유틸리티·컴포넌트를 CSS 변수 참조로 바꾸는 **상류 변경 1건**(Finance PR 1개: `type-*`는 `font-size: var(--type-page-title-size, 24px)`, `Button`은 `h-(--control-h)`)을 먼저 하고, Carelog는 변수 값만 `[data-density=compact]`(또는 `:root` 제품 테마 파일)에서 지정. 상류 변경이 늦으면 Carelog가 **같은 변경을 로컬에 선반영하고 그 diff를 상류에 제안**(차이 문서에 기록).
   - Compact 줄 높이 중 `reading 15/24`, `body 14/21`만 설계안에 숫자가 있고 나머지(page-title·section-title·content-title·key-value·supporting·meta·label)는 **UNKNOWN**(DTCG 원본 파일 미접근). 이 문서의 제안값: 22/30, 18/26, 15/22, 20/28, 13/19, 12/17, 12/17 [제안, Owner/Design Architect 확정 필요].
2. **Finance `globals.css`가 한 파일에 Foundation + Finance 브랜드 + Finance 셸 CSS를 섞고 있다.** 파일 단위 복사가 불가능. 해결: Carelog는 `globals.css`를 **파생**하되 구획 주석(`/* Foundation v1.0.0 — upstream: finance-harness-fe@7dd9590 */`)으로 복사 구간과 제품 구간을 나눈다. 장기적으로 상류가 `foundation.css`(공통) / `theme.<product>.css`로 분리하면 Carelog 파일이 그대로 대응된다.

## 3. 복사 vs 정본 파생 결정

**결정**: 정본(canonical)은 **디자인 시스템 레포(`design-system`, 현재 빈 레포)** 로 정하되, 레포가 채워지기 전(= 지금)은 **Finance `origin/dev` 7dd9590을 임시 기준(de facto)** 으로 삼는다. Carelog는 다음처럼 소비한다.

| 대상 | 방식 | 근거 |
|---|---|---|
| `ui/*` 컴포넌트(밀도 변수화 반영본), `contrast.test` 로직 | **복사**(verbatim + provenance 헤더, 로컬 수정 금지, 로컬 요구는 상류 PR 또는 wrapper) | 변경 빈도 낮음, API가 안정적, 패키지 부재 |
| 토큰 값(중립·글자·상태·반경·포커스) | **복사(값)** | 숫자 동일해야 의미 있음 |
| 브랜드·밀도·레일·호환 별칭 | **파생**(제품 테마 구획) | 의도적으로 다름 |
| Carelog 도메인 컴포넌트 | 제품 전용 | - |

버리는 것: 패키지(`@ranikun/*`) 방식의 즉시성 — 아직 배포 인프라 없음(사실), 그 선결은 이 작업 범위 밖. 되돌림: 패키지가 생기면 `@/components/ui/*` alias를 유지한 채 파일 삭제 + import 경로 교체 → 복사 레이아웃을 패키지 export 구조와 1:1로 맞춰 둔다.

## 4. 중복 구현 방지 방안

1. **Provenance 헤더 + lock 파일.** 복사 파일 맨 위 한 줄 주석 `// upstream: finance-harness-fe@<sha>:<path> (verbatim)`, 레포 루트 `design-foundation.lock.json` = `{source, sha, files:[{path, sha256}]}`. [제안, 파일명은 Writer 선택]
2. **드리프트 테스트(vitest, CI 포함).** lock의 sha256과 현재 파일 해시를 비교해 "verbatim" 파일이 수정되면 실패. 의도적 갱신은 lock을 같이 바꾸는 PR만 통과(리뷰어가 상류 diff를 확인). CI(`carelog-fe-ci.yml`)는 이미 `pnpm test`를 실행하므로 별도 workflow 불필요 [확인].
3. **상류 선행 규칙(둘 중 먼저 만든 쪽이 상류).** TextField/Notice/FieldMessage, EmptyState·PageHeader 재정렬, SelectChip block 변형, 밀도 변수화는 **Finance 계획과 겹친다**. 같은 컴포넌트를 양쪽에서 동시에 새로 만들지 않도록 Owner가 순서를 정한다. 추천: **밀도 변수화 → Finance PR(소)로 상류 선반영**, TextField/Notice는 Finance PR-1(계획) 병합을 기다려 복사, Carelog 화면 PR 중 입력이 필요한 것은 그 뒤. 상류가 지연되면 Carelog가 먼저 만들고 Finance가 복사하는 역방향도 허용(규칙은 "한 곳에서만 작성").
4. **로컬 수정 금지 구역.** `src/components/ui/*`(복사본)는 로컬에서 직접 편집하지 않는다. Carelog 의미(상태 tone 등)는 `src/components/ui/`가 아니라 `src/components/common/` wrapper 또는 별칭 구획에 둔다. 예외 = 상류 PR 링크가 있는 선반영.
5. **차이 문서.** `docs/design/carelog/README.md`(개정)에 "Foundation에서 Carelog가 다르게 가는 것" 표 1개(브랜드 Teal, Compact, 레일, 별칭)만 둔다. 나머지는 "Foundation 그대로"로 선언(정책 전문 복사 금지 — Finance FE `CLAUDE.md`와 같은 원칙).
6. **토큰 이름은 Foundation 의미 이름만 신규 사용.** `rounded-lg`·`text-sm`·`text-xs` 신규 사용 금지(grep AC, 05 공통 AC). 이름이 곧 중복 방지 장치다.
7. **리뷰 체크.** PR 템플릿에 "복사 파일 수정 없음 / 상류 sha / lock 갱신" 3줄(AGENTS.md PR 규칙에 맞춰 `Finding`에 기록).
8. **마이그레이션 종료 조건.** 별칭(RPL-56 이름)과 호환 alias(Button `primary` 등)는 PR-9에서 삭제하고 grep 0건을 AC로 둔다.

## 5. 열린 질문

| ID | 질문 | 추천 |
|---|---|---|
| S-1 | 정본 위치: `design-system` 레포를 이번에 시드할지, Finance FE를 정본으로 둘지 | 레포 시드는 별도 작업. 지금은 Finance 7dd9590 + lock으로 충분 |
| S-2 | 밀도 변수화를 Finance PR로 먼저 낼지 | 예(소규모, Finance 회귀는 값이 같으므로 무변) |
| S-3 | Button `primary` alias를 상류에 추가할지 | Carelog 로컬 alias로 시작(상류 오염 방지), 종료 조건 PR-9 |
| NOT_RUN | 복사 후 `pnpm typecheck/lint/test/build`, Finance 테스트가 Carelog 토큰에서 통과하는지 | 첫 PR에서 수행 |
