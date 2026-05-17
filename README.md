# Veritae-workflow

iOS 앱 + Java Spring 서버를 함께 다루는 제품팀을 위한 **AI 네이티브 워크플로** Claude Code 마켓플레이스.

기획 → 디자인 → API 계약 → 양쪽 구현 → 테스트 → 리뷰가 같은 산출물(PRD, 디자인 명세, OpenAPI, ADR, 테스트 플랜, lessons-learned)을 단일 진실 원본으로 삼아 움직이게 만듭니다.

## 설치

```
/plugin marketplace add IMCHO/Veritae-workflow
/plugin install veritae-workflow@veritae-workflow
```

설치 후 새 Claude Code 세션에서:

```
@planner 사용자가 프로필 이메일을 수정할 수 있게 해주세요
```

## 포함된 플러그인

### `veritae-workflow` (v0.4.0)

**10개 역할** 기반 서브에이전트. 역할별 모델 선택, ADR + lessons-learned 히스토리 컨벤션.

| 에이전트 | 모델 | 역할 |
|---|---|---|
| `planner` | opus | PRD + 작업 분해 |
| `designer` | opus | 디자인 명세 (Figma MCP 활용) |
| `api-architect` | opus | OpenAPI 스펙 + ADR |
| `ios-dev` | sonnet | Swift 코드 + 테스트 |
| `spring-dev` | sonnet | Java 코드 + 테스트 |
| `qa` | opus | 테스트 시나리오 + 커버리지 매트릭스 |
| `convention-checker` | haiku | 산출물 경로·파일명·금지 패턴·LL-NNN Detection 사전 패스 |
| `ios-reviewer` | opus | Swift 6 동시성·메모리·SwiftUI 라이프사이클·접근성 |
| `spring-reviewer` | opus | N+1·트랜잭션·보안·validation |
| `api-contract-reviewer` | opus | BREAKING 판별·prior ADR 정합성 |

자세한 내용은 [`plugins/veritae-workflow/README.md`](plugins/veritae-workflow/README.md), 컨벤션은 [`plugins/veritae-workflow/docs/conventions.md`](plugins/veritae-workflow/docs/conventions.md).

## 설계 철학

1. **산출물이 핸드오프 인터페이스** — 에이전트끼리 직접 호출 대신 PRD, 디자인 명세, OpenAPI, 테스트 플랜 같은 파일로 주고받습니다.

2. **역할별 컨텍스트 격리 + 모델 선택** — 기획자가 보는 정보와 iOS 개발자가 보는 정보가 다릅니다. 서브에이전트로 분리하고, 각 역할에 맞는 모델을 고정 (Opus = 추론/리뷰/추측, Sonnet = 코딩).

3. **3단계 히스토리** — Commit message(WHY/WHAT/RISK) / ADR(아키텍처 결정) / Lessons learned(반복 실수 카탈로그)가 함께 자랍니다. 6개월 뒤 "왜 이렇게 했더라"에 답할 수 있게.

4. **반복 실수 방지 (LL-NNN)** — 리뷰어가 같은 패턴 2회 이상 보면 `docs/lessons-learned.md`에 LL-NNN으로 등록. 다음부터 모든 dev 에이전트가 그 LL을 읽고 회피. qa는 LL마다 회귀 테스트 1개씩.

5. **Figma는 무료 계정으로 가능** — Figma 공식 Remote MCP 또는 Framelink가 Free 플랜에서 동작.

6. **리뷰는 도메인 전용 + 기존 도구 병행** — 내장 `/review`, `/security-review`, `/ultrareview`는 그대로. 추가로 iOS/Spring/API 전용 리뷰어가 도메인 특유 함정에 집중.

7. **점진적 채택** — 한 역할만 써도 됩니다. 파이프라인 전체 자동화 강요 없음.

## 로드맵

- [x] ~~`designer` 서브에이전트 (Figma MCP 연동)~~ — v0.3.0
- [x] ~~`qa` 서브에이전트~~ — v0.3.0
- [x] ~~Haiku 기반 빠른 lint/convention 검사 에이전트~~ — v0.4.0 (`convention-checker`)
- [ ] `/feature` 슬래시 명령 (10단계 체인 자동 실행)
- [ ] PRD 템플릿, OpenAPI 체크리스트를 별도 **Skill**로 분리
- [ ] 리포별 `CLAUDE.md` 표준 템플릿
- [ ] 토이 기능 dogfood 결과 → 친로그 → 다음 버전 입력

## 기여

이슈 / PR 환영합니다. 새 역할 에이전트를 제안하실 때는 다음 3개를 명시해 주세요:
- 산출물
- 입력 (어떤 산출물을 소비하는가)
- 기존 에이전트와의 핸드오프 지점

## 라이선스

미정 (사용 전 저자에게 문의).
