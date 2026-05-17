# Veritae-workflow

iOS 앱 + Java Spring 서버를 함께 다루는 제품팀을 위한 **AI 네이티브 워크플로** Claude Code 마켓플레이스.

기획 → API 계약 → 양쪽 구현이 같은 산출물(PRD, OpenAPI)을 단일 진실 원본으로 삼아 움직이게 만듭니다.

## 설치

Claude Code에서 다음을 실행하세요:

```
/plugin marketplace add IMCHO/Veritae-workflow
/plugin install veritae-workflow@veritae-workflow
```

설치 후 서브에이전트가 자동으로 등록됩니다. 다음과 같이 호출할 수 있습니다:

```
@planner 사용자가 프로필 이메일을 수정할 수 있게 해주세요
```

## 포함된 플러그인

### `veritae-workflow` — 역할 기반 서브에이전트

| 에이전트 | 역할 | 입력 → 산출물 |
|---|---|---|
| `planner` | PM/기획 | 한 줄 요구 → PRD + 작업 분해 |
| `api-architect` | API 계약 설계자 | PRD §5 → OpenAPI 3.x 스펙 |
| `ios-dev` | iOS 개발자 (Swift 6) | OpenAPI + 디자인 → Swift 코드 + 테스트 |
| `spring-dev` | Java Spring 개발자 | OpenAPI + 도메인 → Java 코드 + 테스트 |

자세한 내용은 [`plugins/veritae-workflow/README.md`](plugins/veritae-workflow/README.md).

## 설계 철학

1. **산출물이 핸드오프 인터페이스** — 에이전트끼리 직접 호출하는 대신 PRD, OpenAPI 같은 **파일**을 주고받습니다. 사람이 중간에 개입하기 쉽고, 디버깅·재실행이 명확합니다.

2. **역할별 컨텍스트 격리** — 기획자가 보는 정보와 iOS 개발자가 보는 정보가 다릅니다. 서브에이전트로 분리해 컨텍스트 오염을 막습니다.

3. **리뷰는 기존 도구 재사용** — Claude Code 내장 `/review`, `/security-review`, `/ultrareview`를 그대로 씁니다. 바퀴를 다시 발명하지 않습니다.

4. **점진적 채택** — 한 역할만 써도 됩니다. 파이프라인 전체를 자동화하지 않아도 됩니다.

## 로드맵

- [ ] `designer` 서브에이전트 (Figma MCP 연동)
- [ ] `qa` 서브에이전트 (수락 기준 → 테스트 시나리오)
- [ ] PRD 템플릿, OpenAPI 체크리스트를 별도 **Skill**로 분리
- [ ] `/feature` 슬래시 명령 (planner → api-architect → dev 체인 자동 실행)
- [ ] 리포별 `CLAUDE.md` 표준 템플릿

## 기여

이슈 / PR 환영합니다. 새 역할 에이전트를 제안하실 때는:
- 어떤 산출물을 만드는가
- 어떤 산출물을 입력으로 받는가
- 기존 에이전트와의 핸드오프 지점

을 명확히 적어주세요.

## 라이선스

미정 (사용 전 저자에게 문의).
