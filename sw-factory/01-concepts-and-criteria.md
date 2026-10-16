# 01. 개념과 판별 기준 (블로그 1/4편 정리)

> 출처: [roboco.io — 오픈소스 프로젝트들로부터 배우는 소프트웨어 팩토리 모범사례 (1/4)](https://roboco.io/posts/software-factory-best-practices-from-open-source/)
> 데이터 기준일: 2026-10-10 (GitHub API). 이 문서는 원문 요약 추출본을 바탕으로 했으므로 수치는 원문 확인을 권장합니다.

## 1. 정의

소프트웨어 팩토리 = 개발 흐름이 **"사람의 반복적인 직접 지시 없이 이어지는"** 체계. 에이전트에게 매번 지시하는 상호작용형 코딩(바이브 코딩)과 구분됩니다.

## 2. 판별 기준 6개

| # | 기준 | 뜻 |
|---|---|---|
| 1 | 작업 유입 자동화 | 이슈, 피드백, 에러가 수동 트리거 없이 에이전트 작업이 된다 |
| 2 | 다중 병렬 흐름 | 여러 에이전트 작업이 동시에 돌고, 충돌 방지 규칙이 있다 |
| 3 | 자동 검증 게이트 | 테스트, 리뷰 봇, 품질 검사가 병합 가능 여부를 정한다 |
| 4 | 위험 기반 위임 범위 | 변경 종류별로 승인 요건이 다르다 |
| 5 | 보고 체계 | 사람이 모든 로그를 읽지 않아도 상태를 판단할 수 있다 |
| 6 | 자기 개선 루프 | 실패가 테스트, 지침, 워크플로 수정으로 돌아온다 |

판정 4단계: **충족**(인프라와 증거 모두 있음) / **부분**(하나만 있거나 핵심 단계를 사람이 처리) / **미충족**(반대 증거 있음) / **미확인**(공개 repo로 판단 불가).

## 3. 분석 대상 12개

- **팩토리로 판정(7개 프로젝트, 8개 repo)**: OpenClaw, Beads, Gas Town, mastra, Bun, Astro, Vercel AI SDK, OpenHands
- **고급 바이브 코딩**: Aider
- **판단 불가**: OpenAI Codex(내부 미러링이 2026-08 시작), Claude Code(소스 비공개), StrongDM(운영 비공개)

## 4. 핵심 발견: 병합 권한은 여전히 사람이 쥔다

봇이 main 병합의 과반을 하는 곳은 **OpenClaw 하나**(최근 병합 300건 중 203건). 나머지는 최종 승인을 사람이 합니다.

| 프로젝트 | 병합 집중도 |
|---|---|
| Bun | 한 사람이 robobun PR의 78% 병합(2026-09 기준 446건) |
| mastra | 한 사람이 봇 PR의 82% 병합(9월 1,531건 중 434건) |
| Astro | 한 메인테이너가 팩토리 PR 31건 중 28건 승인·병합 |

**병목 사례**: Aider는 에이전트가 최근 릴리스의 21~88%를 썼는데도 메인테이너가 자리를 비우자 멈췄습니다. 해석: 팩토리의 처리량은 병합 승인자의 가용성에 묶입니다.

## 5. 반복 패턴 6개

### 패턴 1 — 고치기 전에 증거 (Evidence Before Fix)
5개 이상 프로젝트가 "실패 증거 없이 수정 금지"를 요구합니다.
- **Bun**: PR 설명에 수정 전 실패 로그 필수, 이전 릴리스에서 통과하는 테스트는 거부
- **mastra**: CI가 변경된 테스트를 원래 코드에서 돌려 실패를 확인
- **Vercel AI SDK**: 커밋을 재현 → 실패 테스트 → 수정 → 재현 재확인 순으로 쌓음
- **OpenClaw, Astro**: 에이전트에게 회귀 테스트부터 만들도록 지시

### 패턴 2 — 판단은 에이전트, 실행은 결정적 코드
되돌릴 수 없는 행동은 규칙 기반 코드가 수행합니다.
- **Claude Code**: Claude가 중복 이슈를 판단하고, 규칙 스크립트가 대기 기간 후 닫음
- **Astro**: 에이전트에게 쓰기 토큰이 없고 워크플로 코드만 commit/push
- **mastra**: 에이전트는 카드를 "완료"로 옮길 수 없고 병합만 상태를 바꿈

### 패턴 3 — 실패를 출처 있는 규칙으로 (Failure→Rule with Provenance)
- **Beads**: 각 규칙에 사건 날짜와 PR 번호를 인용
- **Astro**: changeset 누락을 "모호한 지시" 탓으로 진단하고 지침을 수정
- **Bun**: 약 2,500개 PR 리뷰에서 병합 차단 사유를 뽑아 `REVIEW.md`로 정리
- 한계: 규칙 준수를 자동 측정하는 곳은 드뭅니다(Astro의 스킬 평가도 CI에 연결되지 않음).

### 패턴 4 — 시작과 확인 지점의 경계
완전 자율 파이프라인 대신 사람의 개입 지점을 둡니다.
- **OpenAI Codex**: 2025-08에 이슈→PR 자동화를 "설계 문제"로 제거
- **mastra**: 자동→수동 모드로 전환, 9/22 이후 봇 PR의 82%에 책임자 표기
- **Astro**: 에이전트가 수정을 만들어도 사람이 프리뷰 릴리스를 확인한 뒤 PR 생성
- **OpenClaw**: 자동 구현을 엄격한 버그로 한정, **실패율 서킷 브레이커** 추가

### 패턴 5 — 에이전트 신원을 사람의 책임에 연결
- **OpenHands**: PR 본문을 `HUMAN:`/`AGENT:` 섹션으로 나누고 CI가 사람 섹션 존재를 검증
- **Beads**: 에이전트 행동에 `Agent-Signature:` 트레일러 필수, 불확실하면 `unknown-model`
- **mastra**: PR에 "Opened by @사람", **OpenClaw**: "Worked on by @사람"

### 패턴 6 — 위험 등급별 권한 (OpenClaw 모델)
| 변경 종류 | 처리 |
|---|---|
| 문서 | Docs 에이전트가 허용 목록 검사 후 main 직접 push |
| 일반 코드 | 선택 검사 + 봇 리뷰로 병합 |
| 민감 경로 | 정책 파일을 SecOps 승인으로 관리 |
| 릴리스 | 수동 실행만 |

## 6. OpenClaw의 변화 (2026-03 → 2026-10)

- 추가: **ClawSweeper** 유지보수 봇(2026-04, 별도 공개 repo; 리뷰·라벨·수리·병합), **실패율 서킷 브레이커**(7일간 성공률 50% 미만이면 자동 구현 중단, 9/3~9/9에 16.9%로 발동), **증거 우선**(AI 사용 공개 의무를 "동작 증명" 요구로 대체)
- 제거: `CLAUDE.md` 심볼릭 링크(9/19 폐기, `AGENTS.md`가 유일한 기준), CONTRIBUTING의 AI 사용 공개 요건(8/30)

해석: **지침 파일은 `AGENTS.md` 하나로 통일**하는 흐름입니다. 데모도 `CLAUDE.md`가 `AGENTS.md`를 가리키는 11바이트 파일입니다.

## 7. 미해결 문제

1. **의도 vs 실행**: Astro의 팩토리가 "의도된 동작"을 버그로 고쳐 병합했다가 릴리스 전에 되돌림. 같은 이슈가 다시 들어오자 같은 수정이 재생성되었고 PR은 미해결로 닫힘.
2. **교차 repo 맹점**: OpenHands의 PR이 AI 리뷰 봇 승인을 받았지만 의존하는 `enterprise` repo의 기능을 깨뜨림. 봇은 다른 repo를 보지 못함.
3. **한 사람 처리량 한계**: 병합 권한 집중이 팩토리 속도와 품질을 결정.

## 8. 관찰된 도구·구성요소

GitHub Actions(검증 게이트, 자동 병합 자격), 봇 계정(robobun, ai-sdk-factory, mastra-platform), 정책 파일(YAML/Markdown), changeset 추적(Astro), 수정 전 실패 확인 CI, 심볼릭 링크·README 지시문, 트레일러 메타데이터(Agent-Signature, Opened by), 허용/차단 경로 설정.

## 9. 후속 읽을 자료

연재 4편 구성(원문 기준): 1편 판별 기준과 12개 repo → **2편 위험 기반 위임과 병합 권한** → **3편 증거 요구와 테스트 게이트** → **4편 실패→규칙 루프와 자기개선의 한계**. 데모 README가 링크한 글:

- [바이브 코딩을 넘어서 소프트웨어 팩토리로](https://roboco.io/posts/beyond-vibe-coding-software-factory/)
- [(2/4) 누가 병합하는가](https://roboco.io/posts/software-factory-who-merges/)
- [(3/4) 증거로 거르는 팩토리](https://roboco.io/posts/software-factory-evidence-before-fix/)
- [(4/4) 실패를 규칙으로](https://roboco.io/posts/software-factory-failures-into-rules/)
- 책: [factory.roboco.io](https://factory.roboco.io/), 집필 repo [roboco-io/the-software-factory](https://github.com/roboco-io/the-software-factory)

## 10. 패턴 ↔ 데모 구현 대응

| 패턴 | 데모에서의 구현 |
|---|---|
| 1 증거 | 필수 체크 `evidence`: 버그 PR의 재현 테스트가 base에서 실패, head에서 통과 |
| 2 판단/실행 분리 | 에이전트는 commit·push 불가, `publish` job(코드)이 검증 후 push·PR·라벨 |
| 3 실패→규칙 | `collect` → `improve` 에이전트 → ESLint 규칙 + `factory/evals/` 케이스 → `eval` 체크 |
| 4 경계 | `@factory go`(시작), T2/T3 CODEOWNERS 승인(병합), 수정 라운드 ≤3, 차단기 |
| 5 신원-책임 | 커밋 트레일러 `Agent-Signature`, `Factory-Issue`, PR 보고서 실행 정보 |
| 6 위험 등급 | `factory/risk-policy.yml` T0~T3 + `.github/CODEOWNERS` |
