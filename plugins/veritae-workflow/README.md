# veritae-workflow plugin

**Veritae 앱**(iOS + Java Spring 서버)을 개발하기 위해 만든 **역할 기반 AI 워크플로** Claude Code 플러그인. Veritae 안에서 실제로 굴려본 구조를 부산물로 공개합니다. v0.4.0에서 Haiku 기반 `convention-checker`가 추가되어 무거운 리뷰어 호출 전 빠른 사전 패스를 제공합니다 (총 10개 역할).

## 포함된 서브에이전트 (10개)

### 기획·설계

| 에이전트 | 모델 | 산출물 | 입력 |
|---|---|---|---|
| `planner` | opus | PRD, 수락기준, 작업 분해 | 한 줄짜리 요구사항 |
| `designer` | opus | 디자인 명세 (화면·토큰·a11y) | PRD §4 + Figma (MCP) |
| `api-architect` | opus | OpenAPI 3.x 스펙 + ADR | PRD §5 (API 초안) |

### 구현

| 에이전트 | 모델 | 산출물 | 입력 |
|---|---|---|---|
| `ios-dev` | sonnet | Swift 코드 + 테스트 | OpenAPI + 디자인 명세 |
| `spring-dev` | sonnet | Java 코드 + 테스트 | OpenAPI + 도메인 요건 |

### 검증

| 에이전트 | 모델 | 범위 |
|---|---|---|
| `qa` | opus | 테스트 시나리오 + 커버리지 매트릭스 |
| `convention-checker` | haiku | 산출물 경로·파일명, 금지 패턴, LL-NNN Detection 정규식의 빠른 기계적 점검 (의미 판단 X) |
| `ios-reviewer` | opus | Swift 6 동시성, retain cycle, SwiftUI 라이프사이클, 접근성 |
| `spring-reviewer` | opus | N+1, 트랜잭션 경계, 보안, validation |
| `api-contract-reviewer` | opus | BREAKING 판별, iOS·Spring 영향 검증, prior ADR 정합성 |

`convention-checker`는 Haiku로 빠르게 1차 패스를 끊어 무거운 Opus 리뷰어가 의미·아키텍처에 집중하도록 돕습니다. 리뷰어는 Claude Code 내장 `/review`, `/security-review`와도 **병행** — 도메인 전용 함정에 집중합니다.

## 파이프라인

```
한 줄 요구
    │
    ▼
[planner] ──▶ PRD ──┬──▶ [designer]  ──▶ design spec ────┐
                     │                                     │
                     ├──▶ [api-architect] ──▶ OpenAPI ─────┼──▶ [ios-dev]    ──▶ [convention-checker] ──▶ [ios-reviewer]
                     │              │                      │
                     │              ▼                      └──▶ [spring-dev] ──▶ [convention-checker] ──▶ [spring-reviewer]
                     │     [api-contract-reviewer]
                     │
                     └──▶ [qa] ──▶ test plan ──▶ (dev 에이전트가 자동화)
```

`convention-checker`는 무거운 리뷰어 직전에 빠른 사전 패스로 들어갑니다 (선택 단계 — 건너뛰어도 됨, 비용 절감용).

산출물(파일)이 핸드오프 인터페이스입니다 — 에이전트끼리 직접 호출하지 않고 사람이 중간에 개입할 수 있게 설계됐습니다.

## 모델 선택 정책

- **Opus**: 모호성 처리/cross-cutting 결정/놓친 버그 캐치/엣지 케이스 추론이 중요한 곳 (planner, designer, api-architect, qa, 모든 reviewer)
- **Sonnet**: 코드 작성 (ios-dev, spring-dev) — 빈도 높고 충분히 capable
- **Haiku**: 기계적 정규식·glob 기반 사전 패스 (convention-checker) — 빠르고 저렴, Opus 리뷰어 전 1차 필터
- 사용자 override: `CLAUDE_CODE_SUBAGENT_MODEL` 환경 변수 또는 호출 시점 `model` 파라미터

## Figma MCP (designer 전용)

designer 에이전트는 Figma MCP 도구가 있으면 활용. **유료 계정 불필요**:

- **공식 Remote MCP Server**: Free 포함 모든 plan
- **Framelink** (커뮤니티): PAT 기반, Free 가능

MCP 미설정 시 사용자에게 Figma URL 또는 디자인 export를 요청. 추측으로 채우지 않음.

## 히스토리·반복 실수 방지 컨벤션

타겟 리포에 다음 자산이 생성·유지됩니다.

| 경로 | 작성자 | 용도 |
|---|---|---|
| `docs/prd/` | planner | 기능 요건서 |
| `docs/design/` | designer | 디자인 명세 |
| `docs/test-plans/` | qa | 테스트 시나리오 |
| `api/openapi.yaml` | api-architect | API 단일 진실 원본 |
| `docs/decisions/` | 다수 | ADR (비자명한 결정) |
| `docs/lessons-learned.md` | reviewer | 반복 실수 카탈로그 |

자세한 포맷은 [`docs/conventions.md`](docs/conventions.md).

### 반복 실수 방지 메커니즘 (LL-NNN)

1. 리뷰어가 같은 패턴을 2회 이상 보면 `docs/lessons-learned.md`에 신규 엔트리 추가 (`LL-NNN` 접두사 + 3자리 zero-pad)
2. 다음 PR부터 dev 에이전트가 작업 시작 시 이 파일을 읽어 알려진 함정 회피
3. qa 에이전트는 모든 LL 엔트리마다 회귀 테스트 1개씩 작성
4. ID는 영원히 안정 — 항목이 deprecate돼도 번호 재사용 안 함

LL-NNN은 commit message, PR 코멘트, 코드 주석 어디서나 1단어로 인용 가능 → "이거 LL-007 케이스야"로 끝.

## 사용 예

```
# 단계별 실행
@planner 사용자가 프로필 이메일을 수정할 수 있게 해주세요
# → docs/prd/2026-05-17-edit-profile-email.md

@designer docs/prd/2026-05-17-edit-profile-email.md의 §4를 구체화
# → docs/design/edit-profile-email.md

@api-architect docs/prd/2026-05-17-edit-profile-email.md §5를 OpenAPI로 확정
# → api/openapi.yaml + 필요시 docs/decisions/NNNN-api-*.md

@api-contract-reviewer 방금 변경된 api/openapi.yaml 검토

@qa docs/prd/2026-05-17-edit-profile-email.md 기반 테스트 플랜
# → docs/test-plans/2026-05-17-edit-profile-email.md

# 구현은 병렬
@ios-dev openapi.yaml + design spec 기반 iOS 구현
@spring-dev openapi.yaml 기반 Spring 구현

# 무거운 리뷰 전 빠른 사전 패스 (선택)
@convention-checker 현재 브랜치의 변경 파일 검사
# → Fail 0건이면 ios-reviewer / spring-reviewer 진행, 1건 이상이면 먼저 수정

@ios-reviewer 현재 브랜치 검토
@spring-reviewer 현재 브랜치 검토
```

## 정착 팁

각 리포의 `CLAUDE.md`에 한 줄만 추가해도 추측 정확도가 크게 올라갑니다:

```markdown
# 이 리포는 SwiftUI + TCA + Tuist. 네트워크는 URLSession + async/await.
# History 컨벤션: veritae-workflow 플러그인 (docs/conventions.md 참조).
```

## 한계 / 다음 라운드

- `/feature` 슬래시 명령으로 10단계 파이프라인 일괄 실행 (v0.5 후보)
- PRD/OpenAPI/디자인 체크리스트를 Skill로 분리 가능
- 산출물 경로는 기본값 — 리포 컨벤션이 다르면 `CLAUDE.md`에서 오버라이드
- 리뷰는 도메인 전용. 일반 코드 품질은 Claude Code 내장 `/review`, `/security-review` 병행
