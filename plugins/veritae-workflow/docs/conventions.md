# Veritae-workflow conventions

이 플러그인 에이전트들이 따르는 작업 결과물 컨벤션입니다. 타겟 리포(iOS/Spring/기타) 안에서 다음 파일들이 자동으로 만들어지고 유지됩니다.

## 디렉토리 인벤토리

| 경로 | 작성자 | 용도 |
|---|---|---|
| `docs/prd/{date}-{slug}.md` | planner | 기능 요건서 |
| `docs/design/{slug}.md` | designer | 디자인 명세 (화면·토큰·a11y) |
| `docs/test-plans/{date}-{slug}.md` | qa | 테스트 시나리오 + 커버리지 매트릭스 |
| `api/openapi.yaml` | api-architect | API 단일 진실 원본 |
| `api/CHANGES-{date}.md` | api-architect | API 변경 영향 노트 |
| `docs/decisions/{NNNN}-{slug}.md` | 다수 (planner/api/dev/designer/qa) | ADR — 비자명한 결정 |
| `docs/lessons-learned.md` | 리뷰어 (ios/spring/api-contract) | 반복 실수 카탈로그 |

## 3단계 히스토리 모델

| 레벨 | 위치 | 작성자 | 용도 |
|---|---|---|---|
| **Commit message** | git log | 모든 dev 에이전트 | 모든 변경의 WHY를 짧게 |
| **ADR** | `docs/decisions/{NNNN}-{slug}.md` | 결정 내리는 모든 에이전트 | 비자명한 결정의 맥락·대안·결과 |
| **Lessons learned** | `docs/lessons-learned.md` | reviewer 에이전트 | 반복된 실수의 카탈로그 |

읽기 흐름: 작업 시작 시 → 관련 ADR + lessons 먼저 읽기. 결과물 끝에 → 필요하면 ADR 추가 + 커밋 메시지 작성 + (리뷰어인 경우) lessons 업데이트.

## ADR (Architecture Decision Record)

비자명한 결정을 내릴 때마다 작성. 다음 중 하나라도 해당하면 ADR 필수:

- 명시적으로 alternative를 거절한 경우
- 스코프를 인위적으로 줄이거나 늘린 경우
- prior decision을 의도적으로 어긴 경우
- BREAKING change를 도입한 경우 (api-architect)
- 새 라이브러리/프레임워크/패턴 채택한 경우 (dev)
- 새 디자인 시스템 컴포넌트 추가 (designer)
- 알려진 테스트 갭을 의도적으로 남기는 결정 (qa)

**파일명 규칙**: `docs/decisions/{NNNN}-{kebab-slug}.md`
- `NNNN`: `docs/decisions/` 안 최대 번호 + 1, 4자리 zero-pad
- `slug`: 결정 제목을 kebab-case로 (예: `breaking-rename-user-id-to-uid`, `design-otp-input-new-component`)

**포맷**:

```markdown
# {NNNN}. {제목}

- Date: YYYY-MM-DD
- Status: accepted | superseded by ADR-NNNN | deprecated
- Decided by: {agent name} (on behalf of {user})

## Context
왜 결정이 필요한가? 어떤 forces가 작용하는가?

## Decision
무엇으로 결정했는가? 한 문장.

## Alternatives considered
- A: ... — 거절 사유
- B: ... — 거절 사유

## Consequences
- Positive: ...
- Negative / trade-off: ...
- Follow-ups: ...
```

api-architect의 API ADR은 추가 섹션:
- `## API surface change` (전/후 비교)
- `## iOS impact`
- `## Spring impact`
- `## Migration steps` (BREAKING일 경우)

## Lessons learned

리뷰어 에이전트(`ios-reviewer`, `spring-reviewer`, `api-contract-reviewer`)가 관리. 카탈로그된 함정은 dev 에이전트들이 다음 작업 시작 시 읽어 회피.

**규칙**:
1. 리뷰 시작 시 전체 읽기.
2. 이미 등록된 패턴을 보면 리뷰 코멘트에 `LL-NNN 참조` 인용.
3. 같은 패턴이 **2회 이상** 발견되면 신규 엔트리 추가.
4. qa 에이전트는 LL 엔트리마다 회귀 테스트 1개씩 작성.

**LL-NNN 의미**: `LL` = Lessons Learned 접두사. `NNN` = 3자리 zero-pad 순번 (001, 002, ...). 카테고리 무관 단일 시퀀스. ID는 영원히 안정 — 항목이 deprecate돼도 번호 재사용 안 함.

**파일 구조** (`docs/lessons-learned.md`):

```markdown
# Lessons Learned

자주 반복되는 실수와 그 대응. 리뷰어 에이전트가 자동으로 갱신.

## iOS

### LL-001: {제목}
- **Category**: iOS
- **First seen**: YYYY-MM-DD (PR/commit 링크)
- **Repeated**: N회
- **Pattern**: 코드 패턴 (코드 예시)
- **Why bad**: 어떤 문제를 일으키는가
- **Fix**: 어떻게 고쳐야 하는가 (코드 예시)
- **Detection**: 자동 검출 방법 (lint rule, grep 등)

## Spring

### LL-NNN: ...

## API

### LL-NNN: ...
```

## 커밋 메시지

dev 에이전트(`ios-dev`, `spring-dev`)가 commit/PR을 만들 때 따르는 형식:

```
{type}: {imperative subject under 70 chars}

WHY:
- 무엇을 해결하려는 변경인가
- 어떤 alternative를 거절했는가 (있다면)
- 어떤 PRD / ADR / OpenAPI 변경에 대응하는가

WHAT:
- 핵심 변경 파일/타입 (3~5개)

RISK:
- 회귀 위험 / 테스트 안 된 영역 (있다면)
```

`type`: `feat` / `fix` / `refactor` / `test` / `docs` / `chore`

## 타겟 리포에 처음 적용할 때

```bash
mkdir -p docs/{prd,design,test-plans,decisions} api
touch docs/lessons-learned.md
```

또는 첫 에이전트 실행이 자동으로 만들어 줍니다.

## 모델 선택 표

| 역할 | 모델 | 빈도 | 이유 |
|---|---|---|---|
| planner | opus | 드묾 | 모호성 처리, 구조화된 추론 |
| api-architect | opus | 드묾 | cross-cutting 결정, BREAKING 판별 |
| designer | opus | 드묾 | 디자인 명세 → 코드 품질 cascade |
| qa | opus | 드묾 | 엣지 케이스 추론 |
| ios-dev | sonnet | 잦음 | 코딩 작업, 비용 합리화 |
| spring-dev | sonnet | 잦음 | 코딩 작업 |
| ios-reviewer | opus | 잦음 | 놓친 버그 캐치 = 고레버리지 |
| spring-reviewer | opus | 잦음 | 동일 |
| api-contract-reviewer | opus | 보통 | BREAKING 판별 책임 |

사용자 override: `CLAUDE_CODE_SUBAGENT_MODEL` 환경 변수 또는 invocation 시점 `model` 파라미터.
