# 03. 공부할 폴더·파일 가이드

데모 repo(`roboco-io/the-software-factory-demo`)에서 **결과물 소프트웨어(`apps/`, `tests/realworld/`, 앱 배포 워크플로)를 제외**하고 읽을 곳을 우선순위로 정리했습니다. 로컬 사본은 [reference/](reference/README.md)에 있습니다.

## 제외하는 것

| 경로 | 이유 |
|---|---|
| `apps/api`, `apps/web` | 예제 서비스(RealWorld) 본체 |
| `tests/realworld/`, `tests/smoke/` | 예제 서비스 검증 자료(스모크는 개념만 참고) |
| `deploy.yml`, `preview.yml` | Cloudflare/GitHub Pages 배포 전용. 우리 스택과 다를 가능성이 큼 |
| `scripts/` | 데모 앱·벤더링 보조 스크립트 |

## 1순위 — 설계와 규칙 (먼저 읽기)

| 경로 | 왜 | 사본 |
|---|---|---|
| `docs/superpowers/specs/2026-10-11-software-factory-design.md` | "왜 이렇게 만들었나." 위험 등급 표(4.3), 역할/예산(5.3), 가드레일(5.4), 완료 조건(9장), 수용한 위험(10장) | [reference/docs/design-spec.md](reference/docs/design-spec.md) |
| `AGENTS.md` | 에이전트 지침 원본. 구조, 명령, 병렬 작업·워크트리, 마이그레이션 주의, 규칙의 쓰는 방식 | [reference/AGENTS.md](reference/AGENTS.md) |
| `factory/risk-policy.yml` | 경로→T0~T3 정책. **우리 SaaS의 2단계에 직결** | [reference/factory/risk-policy.yml](reference/factory/risk-policy.yml) |
| `.github/CODEOWNERS` | 위험 등급과 사람 승인을 연결 | [reference/github/CODEOWNERS](reference/github/CODEOWNERS) |
| `factory/agents.yml` | 역할별 엔진·모델·예산 | [reference/factory/agents.yml](reference/factory/agents.yml) |
| `docs/operations.md` | 설치, 시크릿·변수, 운영 절차, 리허설 런북, 롤백, 보안 한계 | [reference/docs/operations.md](reference/docs/operations.md) |

## 2순위 — 프롬프트와 평가

| 경로 | 내용 |
|---|---|
| `factory/prompts/{triage,implement,review,fix,improve}.md` | 역할 분리, 신뢰할 수 없는 입력 처리, 구조화 JSON 출력 계약. 분석은 [04 문서](04-prompts-and-evals.md) |
| `factory/evals/` | 자기개선의 핵심. 실패 사례를 `case.yml` + 픽스처로 박제 |
| `factory/test-fixtures/` | 에이전트 응답을 흉내 낸 JSON(`triage-feature`, `review-approve`, `review-changes`, `implement-budget-exceeded`). 사본 없음 — 원본 repo 참고 |
| `.agents/skills/project-changelog/SKILL.md` | repo에 있는 유일한 스킬. 스킬 작성 예시 |

## 3순위 — 자동화 배선 (`.github/workflows/`)

읽는 순서(사본: [reference/github/workflows/](reference/github/workflows/)):

1. `factory-intake` → `factory-triage` → `factory-command` : 유입, 정리, `@factory go`
2. `factory-implement` → `factory-review` → `factory-fix` : 구현, 다른 모델 리뷰, 수정 라운드(≤3)
3. `ci` → `factory-merge` : 필수 체크와 등급별 병합
4. `factory-weekly` → `factory-eval-live` : 주간 집계와 개선 PR, 효과 비교
5. `factory-guard`, `factory-smoke`, `factory-contract` : 차단기, 프로덕션 확인, 엔진 계약 테스트

> 읽을 때 볼 것: `prepare / agent / publish` 3단 구조, `permissions:` 최소화, `concurrency.group`, 시크릿을 `needs.prepare.outputs.engine == 'claude' && secrets.X || ''`로 **엔진 하나만** 주입하는 식, 코멘트 본문을 `env`로만 전달(스크립트 인젝션 방지).

## 4순위 — 런타임 코드 (`packages/factory/src/`)

그대로 가져오기보다 **개념 이해용**입니다(Cloudflare Workers·pnpm 스택에 맞춘 구현). 핵심 10개 파일은 [reference/runtime-src/](reference/runtime-src/)에 사본을 두었고, 해설은 [07 문서](07-runtime-code-notes.md).

| 파일 | 개념 |
|---|---|
| `state/machine.ts` | 허용 전이표와 actor별 권한. 라벨 충돌 처리 |
| `policy/risk.ts` | glob 판정(첫 일치, T3 대소문자 무시, rename은 양쪽) |
| `policy/evidence.ts` | evidence 대상 테스트 파일 정의, 안전한 경로 검사 |
| `stages/gate.ts` | 병합 판단: 체크 상태, 등급, verdict, skip.txt 우회 방어 |
| `stages/guard.ts` | 실패율 차단기, 일일 상한 재개 |
| `agents/budget.ts` | 예산 초과 판정 |
| `stages/protect.ts`, `stages/command.ts`, `policy/improve-paths.ts`, `policy/eval-case.ts` | 보호 경로, 명령 파싱, improve 허용 경로, 평가 케이스 스키마 |

사본이 없는 나머지 디렉토리(필요할 때만 원본에서): `agents/{claude,codex,pi,fake,registry,env,prompt,types}.ts`, `github/{port,octokit,memory,forbidden}.ts`, `exec/{process,checks}.ts`, `git/port.ts`, `report/{pr,comments,commit,metrics,text}.ts`, `runs/store.ts`, `stages/{triage,implement,review,fix,evidence,eval,eval-live,improve,weekly,weekly-marker,collect,intake,smoke,board,deploy,bootstrap,worktree,common,deps,phased,risk}.ts`, `config/agents.ts`, `cli.ts`, `main.ts`.

## 5순위 — 구축 이력 (배우는 순서 참고)

| 경로 | 내용 |
|---|---|
| `docs/superpowers/plans/2026-10-11-p1-foundation.md` | P1 기반(앱, CI 게이트, 배포, 프리뷰) 15개 태스크 |
| `…/p2-factory-core.md` | P2 팩토리 코어 18개 태스크 |
| `…/p3-evidence-intake.md` | P3 증거·유입 16개 태스크 + **판정 후보 10개** |
| `…/2026-10-12-p4-report-improve.md` | P4 보고·자기개선 17개 태스크 |
| `CHANGELOG.md` | 기능이 붙은 순서 ([사본](reference/CHANGELOG.md)) |

계획서 4개는 합계 약 880KB(대부분 코드 블록)라 복사하지 않았습니다. 목차와 판단 기록은 [06 문서](06-build-history-and-lessons.md)에 있습니다.

## 단계별로 어디를 읽을까 (내 로드맵 대응)

| 내 SaaS 도입 단계 | 읽을 곳 |
|---|---|
| 1. 기반 | `AGENTS.md`, `docs/operations.md`, `ci.yml` |
| 2. 위험 등급 | `risk-policy.yml`, `CODEOWNERS`, `policy/risk.ts` |
| 3. 반자동 유입 | `prompts/triage.md`, `factory-intake/triage/command/implement/review` |
| 4. 증거 게이트 | `policy/evidence.ts`, `ci.yml`의 `evidence-plan`/`evidence` job |
| 5. 환류 | `factory/evals/`, `prompts/improve.md`, `factory-weekly`, `stages/guard.ts` |

로드맵 전체는 [08 문서](08-adoption-roadmap.md).
