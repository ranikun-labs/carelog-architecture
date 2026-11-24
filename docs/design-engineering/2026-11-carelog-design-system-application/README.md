# Carelog 디자인 시스템 적용 분석 (비정본)

> **정본 아님 · Owner 승격 전 참고 자료.** 이 폴더의 문서는 Carelog Product Architecture, ADR, 교차 저장소 계약이 아니다. Product 정본(`carelog-architecture`의 ADR·contracts)과 Finance Product/UX 정본(`finance-harness-docs`)을 대체하지 않는다. Owner 승격 전까지 F1 설계의 입력 자료로만 사용한다.

## 목적

Foundation Design System v1.0.0을 Carelog FE(`carelog-fe`)에 적용하기 위해, 현재 구현·LOCK 계약·Foundation 값·Finance 참조 구현 사이의 차이를 분석한 결과를 보존한다. 분석이 한 세션의 로컬 작업 디렉터리에만 남아 유실되는 것을 막고, 후속 설계(F1)와 구현 인계(PR 순서)가 같은 근거를 보도록 한다.

## 파일

| 파일 | 내용 |
| --- | --- |
| `01-inventory.md` | Carelog 화면·흐름·계약 인벤토리 (화면 ID 정의: S1, C1, A2/A3 등) |
| `02-contract-gap.md` | 현재 구현 vs Carelog LOCK vs Foundation v1.0.0 계약 차이, 격차 번호(#n)와 Owner 결정 항목(D-n) |
| `03-screen-specs.md` | 화면별 디자인 명세 (Foundation v1.0.0, B Teal, C 위계, Compact 밀도) |
| `04-shared-assets.md` | Finance와 공유 가능한 자산 및 중복 구현 방지 기준 |
| `05-handoff-plan.md` | 구현 인계 계획: 작은 PR 순서, 범위·AC·검증·롤백 |

## 상태

- 분석 작성 완료.
- 렌더·테스트·CI 검증: **NOT_RUN**. 문서 내 대비 수치(WCAG 등)는 hex 값 기준 직접 계산이며 실제 DOM 렌더 결과가 아니다.
- 문서 내 표기: `[확인]` = 파일·git에서 직접 확인, `[추정]` = 근거가 부족한 해석, `[보류]` = 계약 확인 대기.
- 원본 로컬 작업 경로와 로컬 전용 clone 경로는 `scratchpad(비보존)`, `design-system(로컬 clone, 비보존)`으로 표기했다.

## 기준 ref

| 대상 | 기준 |
| --- | --- |
| `carelog-fe` | `origin/main` `c4d67d4` |
| Finance 참조 (`finance-harness-fe`) | `origin/dev` `7dd9590` |
| `carelog-architecture` | `main` `0689422` |

ref는 분석 시점의 재현 기준이며 영구 불변 값이 아니다.

## 관련 작업

- Jira: `RPL-146`, `RPL-148`
- PR: `ranikun-labs/carelog-fe#37`

## Owner 결정 (2026-11-24 승인)

> **우선순위**: 01~05 본문은 아래 Owner 결정 **이전에** 작성됐다. 본문의 미결 항목은 아래 결정으로 대체한다. 본문과 이 절이 충돌하면 이 절을 따른다. 대체되는 본문 항목은 다음과 같다.
> - `01-inventory.md` Q2: "첫 PR은 main"
> - `02-contract-gap.md` D-1·D-2: Owner 확인 대기·확정 전 제안
> - `05-handoff-plan.md` PRE-2·PRE-3과 PR-1~PR-6의 base=main
> - `05-handoff-plan.md` PRE-5: 제품 어휘 질의
>
> 실제 진행 기록: PR-1(`ranikun-labs/carelog-fe#37`)은 base `dev`로 진행했고 merge `963f173`으로 병합됐다.

- **범위**: Carelog 디자인 LOCK 해제 및 Design System v1.0.0 적용 착수. 해제 범위는 *시각 계약*(색·타입·반경·그림자·다크·밀도·반응형 값)이다.
- **화면 어휘**: 화면 표기는 `고객` / `일정`을 유지한다.
- **내부 도메인 명칭**: 변경하지 않는다.
- **적용 기준 브랜치**: dev 기준.
- **Foundation 확정값**: 반경 16/10 등 Foundation 확정값 적용을 승인한다.
- **유지되는 제품 의미 계약**: CustomerEvent 상태 3개(`PLANNED`, `OCCURRED`, `CANCELLED`), 시간축, Overdue 파생, 상태 문법의 텍스트 동반은 디자인 LOCK 해제 대상이 아니다 [추정: 승인 문구가 "디자인 LOCK"만 언급. 해석이 다르면 `02-contract-gap.md` §1 "유지" 열을 재검토].

## 판단 근거 (배치)

- `carelog-architecture` 저장소에는 검토 대기·인계 성격의 정식 경로(`review`, `handoff`, `legacy` 등)가 없다. `README.md`·`AGENTS.md`의 문서 배치 규칙도 이 분석의 위치를 정하지 않는다.
- 기존 디렉터리(`adr/`, `architecture/`, `contracts/`)는 정본 Product 문서 전용이므로 비정본 분석을 두지 않는다.
- 따라서 `docs/design-engineering/2026-11-carelog-design-system-application/`에 두고, 이 README 상단에 비정본 상태를 명시한다. 정본 승격 시 `adr/` 또는 `contracts/`로 옮기고 이 폴더는 참고 자료로 남기거나 정리한다.
