# 06. 구축 이력과 교훈 (P1~P4)

출처: `docs/superpowers/plans/` 4개(원본 repo), 설계 문서 7·9·10장, 운영 문서, `CHANGELOG.md`.
계획서는 대부분 코드라 복사하지 않았고, **목차·판단 기록·실제로 겪은 문제**를 정리했습니다. (날짜는 모두 2026-10, UTC)

## 1. 구축 순서가 말해주는 것

| 단계 | 목표 | 팩토리 판별 기준 기여 |
|---|---|---|
| **P1 기반** | 앱 + CI 게이트 + 프로덕션 배포 + PR 프리뷰 | 병렬 흐름(프리뷰), 검증 게이트(typecheck/lint/unit/realworld-api/e2e) |
| **P2 팩토리 코어** | 트리아지→`@factory go`→구현→교차 리뷰→등급별 자동 병합→배포 | 유입(이슈), 병렬(이슈 단위 concurrency), 위험 위임(T0~T3), PR 보고서 |
| **P3 증거·유입** | `evidence` 필수 체크 + 에러·피드백·스모크 유입 | 유입 자동화 완성, 검증 게이트 강화 |
| **P4 보고·자기개선** | collect → improve → eval, 보드·주간 리포트·대시보드 | 보고 체계, 자기 개선 루프 |

**해석**: 앱과 게이트(P1)가 먼저, 사람 승인 있는 반자동(P2)이 다음, 증거와 유입(P3)을 붙이고, 마지막에 자기개선(P4). 이 순서는 [08 로드맵](08-adoption-roadmap.md)의 단계 순서와 같은 이유입니다. **P4를 가장 늦게** 둔 것은 실패 기록이 쌓여야 개선할 재료가 생기기 때문입니다.

## 2. P1 기반 — 15개 태스크

1 모노리포 골격·린트·CI 기본 체크 → 2 API 골격(Hono, 오류 형식, Workers 테스트 환경) → 3 DB 스키마·마이그레이션 → 4 비밀번호 해시·JWT → 5 사용자·인증 → 6 프로필·stub 라우트 → 7 글 CRUD → 8 댓글 → 9 **RealWorld Hurl 게이트** → 10 웹 골격 → 11 웹 화면 → 12 Playwright E2E → 13 프로덕션 배포 → 14 PR 프리뷰 → 15 브랜치 보호·완료 검증

**고정한 제약**: Node 24, pnpm 10.25(`packageManager`로 고정), TypeScript `~6.0.3`(typescript-eslint peer 범위 때문), Hurl 스위트는 **업스트림 특정 커밋 SHA로 고정**, P1에서 skip하는 Hurl 파일은 4개(favorites, feed, profiles, tags).

**P1 리뷰 포커스(실제 버그가 숨기 쉬운 곳)**
1. 같은 밀리초의 연속 요청: `updatedAt`은 이전보다 커야 하고 목록은 `created_at DESC, id DESC`
2. 악성 마크다운(`<img onerror>`, `<script>`) 제거
3. 깨진 JSON 본문은 500이 아니라 422
4. 위조·만료 JWT는 401 `token: ["is invalid"]`, 선택 인증 엔드포인트는 익명 처리
5. 이메일 대소문자·공백 정규화, 대소문자 변형 중복 가입 방지

## 3. P2 팩토리 코어 — 18개 태스크

1 사전 준비(GitHub App, 변수, 시크릿 범위, pi provider 확인) → 2 `packages/factory` 골격+위험 정책 엔진 → 3 **상태 머신** → 4 에이전트 계약 타입·엔진 배정 → 5 공통 래퍼(타임아웃·예산·구조화 출력)·env 정화·FakeAdapter → 6 Claude 어댑터 → 7 Codex 어댑터 → 8 자식 프로세스 실행기·pi 어댑터 → 9 GitHubPort(Octokit·메모리·차단)+런 기록 → 10 프롬프트 템플릿·보고서 → 11 단계 공통 기반(GitPort, StageDeps, 3단 실행), CLI, triage → 12 command → 13 implement → 14 review·fix → 15 gate·on-closed·risk-tier·deploy 결과 → 16 guard + 전체 흐름 시나리오 테스트 → 17 워크플로·CODEOWNERS·이슈 템플릿·bootstrap·문서 → 18 병합·bootstrap·브랜치 보호·엔진 계약·라이브 완주 검증

**의존성 선택(2026-10-11 npm 확인)**: `@anthropic-ai/claude-agent-sdk`, `@openai/codex-sdk`, `@earendil-works/pi-coding-agent@1.1.0`(정확히 고정), `@octokit/rest`, `zod`, `yaml`, `picomatch`, `tsx`.

**P2 리뷰 포커스**
1. 표식/템플릿 변수가 섞인 에이전트 출력·이슈 본문 → 표식 위조로 이슈 번호·역할(`improve`)을 바꾸지 못해야 함, 템플릿 재확장 금지
2. **이름 변경으로 T3 파일을 옮기는 PR** → 이전 경로도 판정되어 `risk-tier` 실패
3. 취소/미완료 CI 체크, 미생성 `risk-tier`, **이전 head SHA에 대한 APPROVE**를 실패·승인으로 오인하지 않고 대기
4. 같은 head SHA 중복 이벤트(리뷰 2번, fix dispatch 2번, 늦은 라벨 이벤트) → 두 번째는 아무것도 안 함
5. 에이전트가 env·`.git/config`·부모 프로세스 env에서 토큰을 찾으려는 시도

**워크플로 설계 판단**
- `review`가 `pull_request` 이벤트를 쓰지 않는 이유: 상태 전이와의 경합 → implement/fix publish가 상태를 `in-review`로 옮긴 뒤 **dispatch**
- `check_suite`는 Actions가 만든 suite에 대해 워크플로를 트리거하지 않으므로 병합 판단은 **`workflow_run`**(ci 완료)
- GitHub concurrency 그룹은 워크플로를 가리지 않고 리포 전체에서 공유되고 대기 run은 그룹당 최신 1개만 남음 → **워크플로마다 별도 그룹**
- 같은 VM에서 phase만 나누면 에이전트가 publish용 CLI를 고쳐 토큰 탈취 가능 → **job 3개(VM 분리)**

## 4. P3 증거·유입 — 16개 태스크와 판정 후보 10개

1 P2 잔여(임시 분기 제거, 보류 PR의 리뷰 판정) → 2 증거 계획(`evidence` CLI) → 3 증거 실행기(`evidence-run`) → 4 버그 implement 프롬프트 규칙 → 5 API 기반(마이그레이션 0003, 해시) → 6 에러 기록기(fingerprint, upsert) → 7 익명 피드백 API+레이트 리밋 → 8 수집 엔드포인트(`INTAKE_TOKEN`, 404 은닉, ack, probe) → 9 웹 "피드백" 버튼 → 10 `intake` 단계·트리아지 예외 → 11 스모크 → 12 유입 워크플로·위험 정책·CODEOWNERS → 13 INTAKE_TOKEN 준비·배포·bootstrap → 14 evidence 활성화·필수 체크 등록 → 15 라이브 검증(음성 대조, 심은 버그 무개입 수정) → 16 라이브 검증(유입 3종)·spec 갱신

**P3가 정리한 "판정 후보" 10개(설계가 정하지 않은 값 — 근거와 틀렸을 때 비용을 함께 적음)**

| # | 결정 | 이유 |
|---|---|---|
| 1 | 테스트 파일 정의에 `**/*.test.tsx` 추가 | 웹 컴포넌트 테스트를 빼면 웹 버그를 판정할 수 없음 |
| 2 | Playwright E2E는 evidence 실행 범위에서 제외 | 서버 둘+브라우저가 필요해 base/head 두 번 돌리면 느림. E2E만 바꾼 버그 PR은 실패 처리, 프롬프트가 Vitest 재현을 안내 |
| 3 | evidence를 두 job·두 PR로 분리 | 토큰이 필요한 계획과 PR 코드를 실행하는 판정을 분리. 한 PR로 합치면 base에 없는 명령을 호출하거나 "CLI 없으면 건너뜀" 분기를 되살려야 함 |
| 4 | 오류 주입은 토큰으로 보호한 상시 경로(`probe`) | 대안(깨진 요청 찾기/임시 경로 배포)은 비용이 큼 |
| 5 | fingerprint의 스택 프레임에서 줄·열 번호 제거 | 배포마다 같은 에러가 새 이슈가 되는 것을 방지 |
| 6 | `INTAKE_TOKEN`을 deploy가 배포마다 environment secret에서 동기화 | 원본이 하나라 어긋나지 않음. secret이 비면 배포 실패(의도된 fail closed) |
| 7 | 수집 50건을 에러 25 + 피드백 25로 분할 | 한쪽이 많아도 다른 쪽이 밀리지 않음 |
| 8 | 새 경로의 등급: `tests/smoke/**`, `apps/web/e2e-smoke/**`는 T3, intake 미들웨어·라우트는 T2, feedback·error-events는 T1 | 스모크 기준을 약하게 만들거나 토큰 검사를 바꾼 PR이 자동 병합되지 않게 |
| 9 | 심은 버그: 글 목록 페이지 수를 `Math.floor`로 계산 | 단위·Hurl·E2E가 못 잡고 데이터를 안 바꿈. 단 글이 11개 이상이어야 보임 |
| 10 | 에러 메시지·경로를 정규화해 저장 | 퍼블릭 이슈에 개별 값(ID, slug)이 실리지 않게 |

**P3 리뷰 포커스**: 표식·멘션·`fixes #1`이 든 피드백, 고친 뒤 닫힌 에러의 재발(코멘트가 아니라 **새 이슈**로 회귀를 사람이 보게), `CF-Connecting-IP` 없는 요청·`INTAKE_TOKEN` 없는 환경(빈 Bearer도 못 닿게), 이름이 바뀌거나 삭제된 테스트 파일·`..` 경로, 스모크 실패/복구 반복 시 코멘트가 전이마다 한 번만.

## 5. P4 보고·자기개선 — 17개 태스크 (PR 5개)

- **PR 1 `p4-core`(Task 1~6)**: 리뷰 지적 범주와 런 기록 `findings`, GitHub 포트(커밋·병합 PR 목록), `collect`, 평가 케이스 형식·판정, `eval`/`eval-run` CLI, 라벨
- **PR 2 `p4-eval-gate`(Task 7)**: `eval-plan`·`eval` job과 필수 체크 등록
- **PR 3 `p4-improve`(Task 8~11)**: improve 프롬프트, improve 단계·개선 PR, 주간 리포트, `factory-weekly.yml`
- **PR 4 `p4-board-dashboard`(Task 12~14)**: Projects 보드 동기화, 대시보드 `/factory/`, `eval-live`
- **라이브 작업(Task 15~16)**: 보드 생성·bootstrap, 실패 심기 리허설
- **PR 5 `p4-docs`(Task 17)**: 문서·완료 대조

**계획에서 정한 사항 8개**: ①런 기록은 `findingCategories` 대신 `findings{category,file,message}`(대표 원문 필요) ②개선 PR 브랜치는 `factory/issue-<리포트 이슈>`(GitPort 정규식 때문) ③"삭제된 규칙"은 eslint diff의 `-` 줄로 대신(publish job에서 PR 코드를 실행할 수 없으므로) ④평가 케이스는 `lint`·`prompt-contains`만 구현 ⑤보드는 번호가 아니라 제목으로 찾음 ⑥"병합 집중도" = 작성자별 병합 PR 비중 ⑦대시보드의 개선 PR 병합 시점은 `improvements.json`에서(비인증 GitHub API 시간당 60회 한도) ⑧gate·on-closed는 개선 PR을 자동 처리하지 않음(항상 T3)

**P4 리뷰 포커스**: ESLint 설정에서 기존 규칙을 지우거나 끄는 경우, 사람 PR이 평가 케이스를 지우거나 약화, 리뷰 지적 원문의 펜스 탈출·구분 태그·멘션·닫기 키워드, 주간 워크플로가 같은 주에 두 번(예약+수동) 돌 때 중복 이슈·PR 방지, 보드 실패 시 단계 실행은 계속.

## 6. 실제로 겪은 문제와 교훈 (라이브 검증 기록)

| 날짜 | 문제 | 원인·조치 | 교훈 |
|---|---|---|---|
| 10-11 | 첫 계약 테스트에서 codex 쓰기 모드가 **파일을 못 만들었는데 성공으로 끝남** (run 38132807644) | ubuntu-24.04의 AppArmor가 비특권 user namespace를 막아 Codex 샌드박스(bubblewrap)가 조용히 실패. agent job에서 `sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0` (PR #6) | **조용한 실패**를 잡으려면 어댑터 계약 테스트가 "파일이 실제로 생겼는지"를 확인해야 함 |
| 10-11 | triage가 12턴을 쓰고 예산 초과로 실패 (이슈 #5, 비용 약 $0.015) | 리포 탐색에 턴을 씀. maxTurns 10→25 (PR #6) | 예산은 **실측 후 조정**하되 상한은 유지 |
| 10-11 | T2 PR(#10)의 AI 리뷰 판정이 남지 않음 | CI가 끝나자마자 gate가 `needs-human`으로 옮긴 뒤 리뷰 publish가 끝남 | P3 7.1.1에서 보류 상태 PR에도 판정을 남기도록 수정(상태는 안 바꿈) |
| 10-11 | `factory-intake` 예약 실행이 17:45까지 안 돎, guard도 22분 지연 | GitHub `schedule`은 정시 보장 없음 | 수동 `workflow_dispatch` 경로를 항상 준비 |
| 10-12 | `AGENTS.md`에서 오류 헬퍼 규칙 문장을 지워도 **에이전트가 실수하지 않음** (PR #31, 이슈 #32~#34, #38) | 에이전트가 **기존 코드 패턴**을 보고 계속 헬퍼를 씀. 리뷰 지적 `error-format` 0건 | **지침 문장보다 코드베이스의 기존 패턴이 에이전트 행동을 지배**한다. 그래서 실패 심기를 `implement.md`에 잘못된 규칙을 넣는 방식으로 바꿈(PR #40) → 이슈 #41~#43이 `error-format`을 각각 지적 |
| 10-12 | 개선 PR #47 병합 직후 **같은 근거로 같은 범주의 개선 PR #50이 또 열림** | collect가 병합 전 이벤트도 셌음 | PR #51: 병합된 개선 PR 이전의 같은 범주 이벤트는 세지 않음. 개선 뒤 서로 다른 이슈 3건에서 다시 나오면 재후보 |
| 10-12 | 연속 병합으로 대기 중이던 deploy run이 더 새 run으로 대체되자 이슈 #33이 `merged`에 남음 | 대체된 run의 `deployed` 보고가 사라짐 | PR #52: `deployed`가 배포 SHA보다 앞선 최근 14일·50개 커밋의 `merged` 이슈도 함께 옮김 |
| 10-12 | 이슈 #41의 implement prepare가 `GET /issues/41` 502로 실패해 `approved`에 남음 | GitHub API 일시 오류를 재시도하지 않음 | 실패한 job 재실행으로 복구. guard가 자동 복구하지 않음(알려진 한계) |
| 10-11 | CODEOWNERS 승인이 PR #13·#15에서 강제되지 않음 | 작성자가 유일한 코드 소유자면 GitHub가 자기 승인을 요구하지 않음 | T2·T3 게이트는 봇 PR에만 실질 적용. 소유자를 2명 이상으로 |

**실측된 성공 사례**
- 이슈 #7(태그 목록, T1): `triaged → approved → in-progress → in-review → merged → deployed`, **7분**, PR #8 codex 교차 리뷰 승인 후 봇이 squash 병합
- 이슈 #9(즐겨찾기, T2 마이그레이션): PR #10이 `risk:T2`, `BLOCKED`로 열려 있고 프리뷰 생략, 이슈는 `needs-human`
- 심은 버그 → 수정(이슈 #18, PR #16 버그 / PR #19 수정): `@factory go` 17:08 → 종료 17:16 UTC. `evidence` 로그 **"base 종료 코드 1(통과 2, 실패 4), head 종료 코드 0(통과 6, 실패 0)"**. 배포 후 페이지 버튼 0개 → 2개 복구
- 자기개선(PR #47): 후보 `review:error-format`(4회, 이슈 3건) → improve가 ESLint `no-restricted-syntax` 규칙 + `factory/evals/error-format/` 생성 → `eval` 로그 "error-format: base=못 잡음, head=잡음" → CODEOWNER 승인·병합(08:27 UTC) → 같은 실수를 넣은 draft PR #48이 `lint`에서 실패
- 음성 대조: 개선 PR #47의 head에서 `eslint.config.js`만 base로 되돌려 로컬 `eval-run` → "새 케이스 error-format를 head도 잡지 못한다", 종료 코드 1

**해석**: 이 이력의 가장 큰 교훈은 두 가지입니다. ①**검증기도 검증해야 한다**(음성 대조: 틀린 PR이 실제로 실패하는지 확인). ②**리허설로 실패를 만들어야 한다** — 자기개선 루프는 실패 기록이 있어야 동작하고, 의도한 방식으로는 실패가 생기지 않을 수 있습니다.

## 7. 한계로 남은 것 (데모 스스로 인정)

- 자기개선은 **정해진 실패 유형에 대한 PoC**이며 모든 실패 해결이나 장기 성능 향상을 보장하지 않음. 개선 규칙 병합에는 사람의 검토를 남김.
- 개선 PR 병합 후 관찰 기간이 0일이면 "규칙 효과" 비교는 **효과의 근거가 아님**(주간 리포트가 이를 명시).
- 실제 T2 봇 PR이 리뷰 판정 경로를 지난 사례는 2026-10-12 기준 아직 없음.
- GitHub 봇 PR에서 `evidence` 음성 판정이 난 사례는 없음(로컬 CLI와 단위 테스트로만 확인) — `evidence`는 봇이 연 PR만 판정하므로 사람 PR로 음성 대조를 만들 수 없음.
