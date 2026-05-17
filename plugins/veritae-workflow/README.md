# veritae-workflow plugin

iOS 앱 + Java Spring 서버를 함께 다루는 팀을 위한 **역할 기반 AI 워크플로** Claude Code 플러그인. v0.2.0부터 전용 리뷰어, 역할별 모델 선택, ADR + lessons-learned 히스토리 컨벤션을 포함합니다.

## 포함된 서브에이전트

### Pipeline

| 에이전트 | 모델 | 산출물 | 입력 |
|---|---|---|---|
| `planner` | opus | PRD, 수락기준, 작업 분해 | 한 줄짜리 요구사항 |
| `api-architect` | opus | OpenAPI 3.x 스펙 + ADR | PRD §5 (API 초안) |
| `ios-dev` | sonnet | Swift 코드 + 테스트 | OpenAPI + 디자인 명세 |
| `spring-dev` | sonnet | Java 코드 + 테스트 | OpenAPI + 도메인 요건 |

### Review gate

| 에이전트 | 모델 | 범위 |
|---|---|---|
| `ios-reviewer` | opus | Swift 6 동시성, retain cycle, SwiftUI 라이프사이클, 접근성 |
| `spring-reviewer` | opus | N+1, 트랜잭션 경계, 보안, validation |
| `api-contract-reviewer` | opus | BREAKING 판별, iOS·Spring 영향 검증, prior ADR 정합성 |

리뷰어는 Claude Code 내장 `/review`, `/security-review`와 **병행** 사용합니다 — 도메인 전용 함정에 집중합니다.

## 파이프라인

```
한 줄 요구 ─▶ planner ─▶ PRD ─▶ api-architect ─▶ OpenAPI ─▶ api-contract-reviewer
                                                    │
                              ┌─────────────────────┴─────────────────────┐
                              ▼                                           ▼
                          ios-dev                                     spring-dev
                              │                                           │
                              ▼                                           ▼
                       ios-reviewer                              spring-reviewer
                              └──────────▶ /review, /security-review ◀────┘
```

산출물(파일)이 핸드오프 인터페이스입니다.

## 모델 선택 정책

- **Opus**: 모호성 처리/cross-cutting 결정/놓친 버그 캐치가 중요한 곳 (planner, api-architect, 모든 reviewer)
- **Sonnet**: 코드 작성 (ios-dev, spring-dev) — 빈도 높고 충분히 capable
- `inherit`로 두지 않은 이유: 역할별 적합도가 다르고, 사용자가 메인 세션을 어느 모델로 띄우든 결과 품질이 일관되도록

사용자가 모델을 override하려면 `CLAUDE_CODE_SUBAGENT_MODEL` 환경 변수 또는 호출 시점 `model` 파라미터 사용.

## 히스토리·반복 실수 방지 컨벤션

타겟 리포(iOS·Spring·기타)에 다음 3종 자산이 생성·유지됩니다.

| 레벨 | 위치 | 작성자 |
|---|---|---|
| Commit message | git log | 모든 dev 에이전트 (WHY/WHAT/RISK 포맷) |
| ADR | `docs/decisions/{NNNN}-{slug}.md` | planner, api-architect, dev (아키텍처 결정 시) |
| Lessons learned | `docs/lessons-learned.md` | reviewer 에이전트 (반복 실수 카탈로그) |

자세한 포맷은 [`docs/conventions.md`](docs/conventions.md) 참조.

### 반복 실수 방지 메커니즘

1. 모든 dev 에이전트는 작업 시작 시 `docs/lessons-learned.md`를 읽고 알려진 함정을 회피
2. 리뷰어가 같은 패턴을 2회 이상 발견하면 신규 LL 엔트리를 추가
3. 다음 PR부터 dev 에이전트가 그 LL을 학습 → 재발 방지

## 사용 예

```
# 단계별 실행
@planner 사용자가 프로필 이메일을 수정할 수 있게 해주세요
# → docs/prd/2026-05-17-edit-profile-email.md + 필요 시 docs/decisions/NNNN-*.md

@api-architect docs/prd/2026-05-17-edit-profile-email.md §5를 OpenAPI로 확정
# → api/openapi.yaml + docs/decisions/NNNN-api-*.md (BREAKING이면)

@api-contract-reviewer 방금 변경된 api/openapi.yaml 검토
# → BREAKING 판별 + iOS·Spring 영향 보고

# iOS와 Spring 작업은 병렬 실행
@ios-dev openapi.yaml의 PATCH /users/me/email를 iOS에 구현
@spring-dev openapi.yaml의 PATCH /users/me/email를 Spring에 구현

@ios-reviewer 현재 브랜치 검토
@spring-reviewer 현재 브랜치 검토
```

## 정착 팁

각 리포의 `CLAUDE.md`에 한 줄만 추가해도 에이전트 추측 정확도가 크게 올라갑니다:

```markdown
# 이 리포는 SwiftUI + TCA + Tuist. 네트워크는 URLSession + async/await.
# History 컨벤션: veritae-workflow 플러그인을 따른다 (docs/conventions.md).
```

## 한계

- `designer`, `qa` 역할은 다음 라운드. designer는 Figma MCP 의존 (Free 플랜 + Remote MCP 가능).
- 산출물 경로 (`docs/prd/`, `api/openapi.yaml`, `docs/decisions/`, `docs/lessons-learned.md`)는 기본값. 리포 컨벤션이 다르면 각 리포 `CLAUDE.md`에서 오버라이드.
- 리뷰는 도메인 전용. 일반 코드 리뷰는 Claude Code 내장 `/review`, `/security-review` 병행 사용.
