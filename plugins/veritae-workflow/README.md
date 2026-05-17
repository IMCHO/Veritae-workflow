# veritae-workflow plugin

iOS 앱 + Java Spring 서버를 함께 다루는 팀을 위한 **역할 기반 AI 워크플로** Claude Code 플러그인.

## 포함된 서브에이전트

| 에이전트 | 산출물 | 입력 |
|---|---|---|
| `planner` | PRD, 수락기준, 작업 분해 | 한 줄짜리 요구사항 |
| `api-architect` | OpenAPI 3.x 스펙 | PRD §5 (API 초안) |
| `ios-dev` | Swift 코드 + 테스트 | OpenAPI + 디자인 명세 |
| `spring-dev` | Java 코드 + 테스트 | OpenAPI + 도메인 요건 |

## 파이프라인

```
한 줄 요구 ─▶ planner ─▶ PRD ─▶ api-architect ─▶ OpenAPI
                                                    │
                              ┌─────────────────────┴─────────────────────┐
                              ▼                                           ▼
                          ios-dev                                     spring-dev
                              │                                           │
                              └─────────────▶ /review, /security-review ◀─┘
```

산출물(파일)이 핸드오프 인터페이스입니다 — 에이전트끼리 직접 호출하지 않고 사람이 중간에 개입할 수 있게 설계됐습니다.

## 사용 예

```
# 단계별 실행
Agent(subagent_type="planner", prompt="사용자가 프로필 이메일을 수정할 수 있게 해주세요")
# → docs/prd/2026-05-17-edit-profile-email.md 생성

Agent(subagent_type="api-architect", prompt="docs/prd/2026-05-17-edit-profile-email.md §5를 OpenAPI로 확정")
# → api/openapi.yaml 갱신

# iOS와 Spring 작업은 병렬 실행 가능
Agent(subagent_type="ios-dev", prompt="openapi.yaml의 PATCH /users/me/email를 iOS에 구현")
Agent(subagent_type="spring-dev", prompt="openapi.yaml의 PATCH /users/me/email를 Spring에 구현")
```

## 정착 팁

각 리포의 `CLAUDE.md`에 다음 한 줄을 추가하면 에이전트의 추측 정확도가 크게 올라갑니다:

```markdown
# 이 리포는 SwiftUI + TCA + Tuist 기반입니다. 네트워크는 URLSession + async/await.
```

iOS·Spring 리포 모두 동일하게 적용 가능합니다.

## 한계

- `designer`, `qa` 역할은 아직 없습니다 (Figma MCP / 테스트 시나리오 생성기 의존).
- 산출물 경로 (`docs/prd/`, `api/openapi.yaml`)는 기본값입니다. 리포 컨벤션이 다르면 각 리포 `CLAUDE.md`에서 오버라이드하세요.
- 리뷰 단계는 Claude Code 기본 `/review`, `/security-review`, `/ultrareview`를 재사용합니다.
