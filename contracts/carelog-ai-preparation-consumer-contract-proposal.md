# Carelog — 방문 준비 AI Consumer 구현 계약 v0.2 (제안)

- 상태: **PROPOSED / NOT_EFFECTIVE**. CAI-01~03 설계 후보. [RPL-62](https://ranikunlabs.atlassian.net/browse/RPL-62)는 HOLD — BLOCKED_BACKEND_CONTRACT 유지.
- 검토일: 2026-11-27. 소유: Carelog Product/BE(업무 의미·인가·저장), Shared AI(실행).
- 기존 [RPL-87](https://ranikunlabs.atlassian.net/browse/RPL-87) Mock-first UX 및 [RPL-62 Product Scope Sync](https://ranikunlabs.atlassian.net/browse/RPL-62?focusedCommentId=10362)를 보존한다.
- 운영 Memory OFF. 실제 사용자·민감 데이터 및 실모델 사용 금지(별도 승인).

## 1. 목적 및 범위

고객의 예정 방문을 선택하면 이미 기록한 내용을 근거로 방문 목적, 이전 맥락, 확인할 사항과 중립적인 질문을 제공한다. AI 결과를 CRM 원본으로 자동 저장하지 않는다. 첫 구현 후보는 PLANNED 방문 준비에 집중하지만 세 진입점은 하나의 Assistant Surface를 재사용한다.

| 진입 | 제품 범위 | 첫 Slice |
|---|---|---|
| PLANNED CustomerEvent → 미리 준비하기 | 기존 RPL-62 | 우선 합성 검증 |
| OCCURRED CustomerEvent → 기록에 대해 질문 | 기존 RPL-62 | 후속 계약/검증 필요 |
| Customer Detail → 이 고객에 대해 AI에게 물어보기 | RPL-87 및 RPL-62 Scope Sync | 후속 계약/검증 필요 |

첫 Slice 완료를 전체 RPL-62 완료 또는 HOLD 해제로 간주하지 않는다. CustomerEvent 상태 PLANNED/OCCURRED/CANCELLED를 유지하며 PREPARED canonical enum/AI Briefing History Domain을 신설하지 않는다. RelationJournal과 CustomerEvent를 임의 1:1로 매핑하지 않는다.

## 2. 사용자 흐름

고객 → 예정 방문 → 미리 준비하기 → 현재 사용자 및 Workspace/Customer/Event 인가 → 허용된 원본 최소 조회 → 제품 정책 및 Shared AI 실행 → 출처/결과/현재 인가 검증 → 한국어 브리핑 표시 → 선택적으로 Event 메모에 추가 → 사용자 확인/편집 → 기존 Event Memo 저장. Suggested Actions 우선, 직접 질문은 기존 UX 목표로 보존하되 첫 Preparation Slice에서는 고정 작업으로 시작하는 제안이다. Desktop co-visible/standalone Context 표시 규칙은 기존 UX를 유지한다.

## 3. Provider-neutral 입력 및 데이터 범위

아래는 논리 의미이며 승인된 물리 API/DTO/SkillId 이름이 아니다.

| 항목 | 계약 후보 |
|---|---|
| 작업 유형 | Preparation: 선택 PLANNED Event. Event-level question과 Customer-level question은 후속 작업 유형으로 분리하며 범위를 넓히지 않는다. |
| 현재 actor | Shared Identity/Gateway의 승인 인증 경계를 통해 확보, Carelog-local linkage 및 Workspace membership/권한 검증. 클라이언트·모델 주장의 user/owner ID를 권한 증거로 사용하지 않는다. |
| 대상 | 하나의 Workspace에 속한 Customer와 그 Customer에 속한 Event의 참조. 모든 관계를 서버에서 확인한다. |
| Context | 선택 예정 Event의 목적/계획 시각/허용된 메모, 제품이 승인한 고객 메모와 동일 고객의 완료 방문 기록 중 필요한 정보. 원본 필드·버전/시각·조회 시각의 참조를 보존한다. |
| 실행 제어 | 기존 request/execution correlation, product/tenant/resource scope, capability allowlist, deadline, 사용자 언어(첫 인수 ko). Provider/model 선택을 FE 계약에서 숨긴다. |

**조회 범위 Owner 결정:** '최근 완료 방문 3건 + 더보기'는 미확정이다. 초기 합성 실험은 명시적으로 지정한 같은 고객의 승인된 Fixture 기록만 쓴다. 실제 기록 조회 기본 건수/기간, 정렬(동일 시각 tie-break 포함), pagination, 고객 메모 field allowlist/크기/민감도, 더보기 UX는 Product Owner 승인 전 고정하거나 임의 전체 조회하지 않는다. 기록이 없으면 기록 없음과 조회 실패를 구분한다. 과거 기록은 OCCURRED 여부와 발생 시각을 제품 정본에서 검증하며 CANCELLED/다른 고객/삭제/비인가 기록을 포함하지 않는다. 예정 시간과 실제 발생 시간을 혼용하지 않는다. 같은 고객의 허용 완료 기록과 타 고객·동일 고객의 비인가 기록·취소 기록이 섞여 있으면 Carelog가 모델 실행 전에 제외하고, 제외된 내용과 참조는 모델 입력 및 결과 출처 모두에서 나타나지 않아야 한다.

권한은 조회 때마다 현재 authoritative Carelog 권한을 확인한다. Shared AI는 scope 무결성만 집행하고 Carelog DB 직접 접근 금지. 동일 Workspace라도 접근할 수 없는 고객은 차단한다. provider payload에 identity·연락처·민감 의료 원문 등 불필요한 값을 포함하지 않는다. 외부 제공자 이용·고지·민감정보 허용 목록·보존 및 로그 정책은 별도 승인 Gate다.

## 4. 출력 및 Evidence

| 논리 구역 | 의미 |
|---|---|
| 방문 목적 | 선택 Event에서 확인된 내용, 없으면 명시적으로 부족하다고 표시 |
| 이전 맥락 | 실제 제공된 이전 기록의 요약, 출처 참조로 결속; 기록에 없는 사건·치료 반응을 사실처럼 생성 금지 |
| 확인할 사항 | 기존 기록에 기반한 비의료적 준비/업무 확인 후보, 확정된 일정 변경/업무 명령과 구별 |
| 후속 질문 | 사용자가 고객에게 확인할 중립 질문; 의료 진단·처방·개별 치료 판단 금지 |
| 출처 및 한계 | 사용한 Event/메모별 유효한 원본 참조·기록 시점·필요한 조회 시점, 정보 부족·누락 표시. 모델 해석 자체는 원본 근거가 아님 |

Carelog는 결과의 업무 의미, Evidence admissibility 및 의료 안전 검사를 소유한다. Shared AI는 요청/실행과 결과의 기계적 schema·scope·correlation·Evidence 무결성 검사를 담당한다. 결과와 현재 요청/Workspace/Customer/Event가 일치하지 않거나 허용된 원본에 결속할 수 없으면 미노출. 유효한 출처 ID만으로 주장의 의미가 검증되는 것은 아니다. 일정 변경 '요청'을 '확정'으로 표현하는 등 원본보다 강한 사실 주장은 해당 출처를 인용했더라도 결과 검증 실패로 전체 브리핑을 숨긴다. 미확정 사실은 미확정이라고 표시하고 확인 질문으로만 제안한다. 원본 메모는 지시가 아닌 untrusted data이다. 정상 응답은 기존 Assistant 결과 UI에서 한국어로 표시한다. exact DTO, HTTP 오류·status enum, 공통 schema 변경은 소유자 승인 필요.

## 5. 결과 수명과 명시 저장 (중요 Owner Gate)

**추천 최소 계약:** 첫 합성 Slice에서는 현재 선택된 Assistant session/view에 메모리상 임시 결과를 유지한다. 화면의 '준비됨'은 현재 유효한 결과에서 파생하는 표현이며 DB canonical 상태가 아니다. 같은 유효 세션에서 다시 열 때 현재 권한과 원본/선택 일치를 확인할 수 있을 때만 표시한다. 다시 확인할 수 없으면 무효화하고 재준비를 안내한다. 새로고침·프로세스 재시작·다른 기기까지 결과가 살아남는다고 보장하지 않는다. 이것이 기존 '준비 내용 다시 보기' UX에 충분한지는 Carelog Owner가 결정한다. 더 긴 재열람이 필수라면 별도 저장 목적·권한·TTL·삭제·백업·민감정보 정책 및 구현 계약 승인 전 활성화하지 않는다. AI History 도메인 신설은 추천하지 않는다.

Event Memo 저장은 결과 표시와 별개다. 기존 사용자 확인·편집·명시 Save를 통해 기존 mutation 경계를 사용하는 방향이며, 아래 불변식을 만족할 때만 실제 저장을 연동한다. AI 모델은 save 권한을 갖지 않고, 요청이 성공해도 자동 CRM write=0이다. 사용자 편집 문구와 AI 원문/출처의 영속 수명은 동일하다고 가정하지 않는다.

- 기존 메모 보존: 사용자가 확인·편집한 추가 문구만 기존 메모에 반영하며 기존 본문을 삭제/대체하지 않는다. 저장 직전 현재 권한·Event 관계/상태를 확인한다. 확인창이 열렸을 때 본 메모와 저장 시점의 원본이 다르면 stale 편집으로 간주하고 덮어쓰지 않는다. 변경된 메모를 보여 준 뒤 사용자 재확인으로만 새 저장을 허용한다.
- 동일 저장 재전송: 서버에서 동일한 승인된 저장 의도·대상·내용의 재전송은 한 번만 반영한다. 같은 의도의 다른 내용/대상은 충돌로 거부한다. 응답 유실 후 재전송에서도 중복 추가 금지; 정확한 동시성·멱등성 구현 방식/식별자는 BE Owner 승인 대상이다.
- 확인창 취소(제출 전)는 쓰기 0건. 저장 제출 후 취소/응답 유실은 성공·실패를 추정하거나 이미 완료된 저장을 자동 롤백하지 않는다. 현재 상태 확인으로 해결한다.
- 기존 [Event PATCH 갱신 경로](https://github.com/ranikun-labs/carelog-be/blob/5d6aa88cdf44d01ca07a8fa8265a8c7cb0c2a995/carelog-be/src/main/java/carelog/carelog/customerevent/app/CustomerEventServiceImpl.java#L108-L127)는 조회/잠금 후 전달된 note를 전체 교체한다는 검수 근거가 있다. 행 잠금만으로 이전 화면의 stale note 교체나 재전송 중복 방지를 보장할 수 없다. 기존 PATCH가 위 계약을 충족한다는 검증 전에는 실제 AI 메모 저장 연동 금지. 필요한 BE 변경·충돌 응답/재확인 UX·멱등 처리 방식은 BE Owner 및 Product Owner의 승인과 구현 검증 Gate로 남긴다. 신규 endpoint/DTO/테이블/프로토콜을 여기서 확정하지 않는다.

## 6. 실패·취소·안전성

- 미인증/권한 없음/다른 Workspace/다른 Customer/관계 불일치/존재하지 않음은 AI 호출 전 차단. 외부 존재 여부와 내부 오류 상세를 노출하지 않는다. 구체 상태 코드 매핑은 기존 Carelog API와 별도 승인된 Consumer 계약을 따른다.
- PLANNED 준비 요청에서 OCCURRED/CANCELLED/삭제된 Event는 실행할 수 없다. 응답 전 현재 인가 및 사용 원본의 변경 여부를 다시 검증한다. 현재 권한/버전 확인 불능이면 이전 결과를 제공하지 않는다. 이벤트 변경, 화면 전환, 선택 변경, 로그아웃, 권한 철회 시 이전 요청 결과는 폐기/비활성화한다.
- timeout, provider failure, invalid/missing Evidence, correlation mismatch: 실패 상태·안전한 안내. 불완전 생성 내용 미노출. 실제 제공자 취소를 보장하지 않는다; 취소 이후 늦은 응답은 무시한다. 명시 재시도는 새 권한/원본 검증 후; 자동 재시도는 별도 결정 전 금지한다.
- 정상 CRM 등록·일정 관리·Event Memo 작성은 AI 오류에 의존하지 않는다. background AI, 자동 Event/Customer 변경, 자동 의료 판단·처방 금지. 합성 결과를 운영 실모델로 표시하지 않는다.

## 7. 한국어 합성 예시 및 완료 기준

합성 입력: 고객 A의 예정 방문 목적은 이전 상담 내용 확인. 허용된 이전 완료 방문에는 '일정 변경 요청, 다음 방문에서 이전 상담 내용 재확인'이라고 기록되어 있다. 기대 표시: '지난 기록에는 일정 변경 요청과 다음 방문의 상담 내용 재확인이 남아 있습니다. 요청하신 일정 변경이 확정됐는지, 달라진 요청 사항이 있는지 확인해 보세요.' 출처: 해당 예정 방문·실제 제공된 완료 방문. 어떤 의료적 판단이나 다른 고객 정보를 포함하지 않는다.

| AC | 합성 검증 기대 |
|---|---|
| C-01 | 현재 인가된 Workspace/Customer/PLANNED Event → 한국어 브리핑, 정확한 원본 참조 및 Context label |
| C-02 | 타 Workspace/타 고객, 잘못된 Event 관계, 비인가·미인증·삭제·취소·완료 상태의 준비 요청은 model/tool 0회 |
| C-03 | 빈 이전 기록과 조회 오류 구분, 입력의 지시문 무효, 허위 출처/의료 조언/자동 변경 제안 등 정책 위반 출력 차단. 대조 Fixture: 일정 변경 요청만 있는 원본을 유효한 출처 ID로 인용하면서 변경 확정으로 단정하는 출력은 전체 미노출. 같은 고객의 허용 완료 기록에 타 고객·비인가·취소 기록을 섞어도 제외된 기록의 본문·참조가 모델 입력과 응답 출처에 모두 없어야 함. |
| C-04 | timeout, cancel, 잘못된 result/correlation/evidence, 늦은 응답, 처리 중 선택 변경·권한 철회·원본 수정 시 안전한 미노출 |
| C-05 | 브리핑 반복/정상/실패 및 제출 전 취소에서 CRM 쓰기 0건. 명시 저장은 현재 권한/대상 확인 후 기존 메모를 보존하고 편집한 문구만 추가. 확인창 이후 다른 사용자의 메모 변경 시 stale 저장은 기존 메모를 덮지 않고 충돌/재확인 처리(쓰기 0건). 동일 저장 재전송 및 응답 유실 후 재전송은 한 번만 반영, 같은 저장 의도의 내용/대상 불일치는 거부. 위 조건을 실제 BE 경계가 충족하는지 검증되기 전 저장 연동 금지. |
| C-06 | 같은 유효 Assistant 세션에서 최신 준비 결과 다시 보기(Owner 승인 수명 내), 만료/원본/권한 변경 후 재사용 금지. 새 세션 지속은 미승인. |
| C-07 | 기존 RPL-87 Assistant UX, standalone/co-visible 및 비AI CRM 회귀 유지. Fake Model을 합성 환경에 한정하며 고정 HEAD Native/CI/독립 검수와 실제 모델/인증/E2E를 별도로 기록 |

## 8. RPL-62 Re-entry Gate / 인수 계획

| 결정자 | 승인 또는 검증 항목 | 현재 상태 |
|---|---|---|
| Carelog Product Owner | 조회 데이터 허용 목록, 최근 완료 방문 3건+더보기, 빈 기록 의미, 작업 유형별 scope, 준비 결과 수명과 재열람, 한국어 결과 구성/의료 안전 | 미확정/검토 후보 |
| Carelog BE + Shared AI Owner | provider-neutral preparation 및 event-level question (Customer-level scope sync 포함), 실제 신뢰된 인증·transport/wire/API·DTO, request/result correlation, 결과 오류·timeout/cancel, source ref 계약 | 물리 계약 미확정; 임의 schema 금지 |
| Privacy/Security/Owner | 외부 데이터 처리·최소화·보존/삭제/로그, 허용 Provider/실모델 승인 | PENDING. 합성만 |
| AI/Normal Lane | 기존 Event Memo mutation 및 UI 계약/실제 인가 확인, 기존 메모 보존·동시 편집 충돌·멱등성 Gate에 대한 BE/Product Owner 결정 및 구현 검증, Writer 인계, 테스트/CI/독립 검수 | NOT_RUN; 실제 저장 연동 BLOCKED |

첫 PLANNED 합성 Slice를 위한 제한된 계약을 독립적으로 검토할 수 있으나, 기존 RPL-62의 모든 Re-entry 항목(다른 진입점, result lifetime/transport 등)이 확정되거나 Owner가 별도로 승인된 범위를 조정하기 전에는 HOLD를 자동 해제하지 않는다. 문서 작성만으로 코드 구현/실데이터 사용을 위임하지 않는다.

## 9. 기존 정본과 근거

- [RPL-62](https://ranikunlabs.atlassian.net/browse/RPL-62), [Customer-level 범위 동기화](https://ranikunlabs.atlassian.net/browse/RPL-62?focusedCommentId=10362), [RPL-87](https://ranikunlabs.atlassian.net/browse/RPL-87).
- [Authenticated Carelog Context](https://github.com/ranikun-labs/carelog-architecture/blob/main/contracts/authenticated-carelog-context.md): 의미 계약, 물리 전송/필드 별도.
- [CustomerEvent Lifecycle](https://github.com/ranikun-labs/carelog-architecture/blob/main/contracts/customer-event-lifecycle.md): Accepted lifecycle semantics, bootstrap 구현 상태는 과거 스냅샷.
- [Shared AI Consumer Matrix](https://github.com/ranikun-labs/shared-ai-architecture/blob/main/docs/consumer-use-case-matrix.md): 설계 trace, 실제 Runtime 통합 PASS 아님.
- [Shared AI ADR-0001](https://github.com/ranikun-labs/shared-ai-architecture/blob/main/docs/adr/ADR-0001-skill-lens-workflow-execution-ownership.md), [ADR-0002](https://github.com/ranikun-labs/shared-ai-architecture/blob/main/docs/adr/ADR-0002-evidence-provenance-core-domain-boundary.md), [ADR-0004](https://github.com/ranikun-labs/shared-ai-architecture/blob/main/docs/adr/ADR-0004-execution-context-api-tenant-isolation.md).

이번 설계 Native Test/CI/Independent Review/Runtime E2E = NOT_RUN. Owner Gate = PENDING. RPL-62 HOLD 유지.
