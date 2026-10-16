# 07. 런타임 코드 읽기 노트

대상: `packages/factory/src/` 중 핵심 파일. 사본은 [reference/runtime-src/](reference/runtime-src/). 이 문서는 코드를 읽으며 얻은 구조 정리입니다(코드에 있는 사실만 적음).

## 1. 상태 머신 (`state/machine.ts`)

상태 8개: `new, triaged, approved, in-progress, in-review, needs-human, merged, deployed`
actor 8개: `triage, command, implement, review, fix, gate, github, deploy`

### 허용 전이표 (이 표 밖의 전이는 `TransitionError`)

| from → to | 허용 actor |
|---|---|
| new → triaged | triage |
| new → approved | command |
| new → needs-human | triage, command |
| triaged → approved | command |
| triaged → needs-human | command |
| needs-human → approved | command |
| approved → in-progress | implement |
| approved → needs-human | implement, command |
| in-progress → in-review | implement, fix |
| in-progress → needs-human | implement, fix, command |
| in-review → in-progress | fix |
| in-review → needs-human | review, fix, gate, command |
| in-review → merged | **github** |
| needs-human → merged | **github** |
| merged → deployed | **deploy** |
| merged → needs-human | deploy |

**읽을 점**
- `merged`는 `github`(PR 병합 이벤트)만, `deployed`는 `deploy`만 만든다 → **에이전트·gate가 "완료"를 선언할 수 없다**(패턴 2: 판단과 실행 분리). `needs-human → merged`가 있는 이유: T2·T3 PR은 `needs-human` 상태에서 사람이 병합하므로.
- `approved`로 가는 전이는 **오직 `command` actor**(사람의 `@factory go`)뿐.
- `stateFromLabels`: `factory:*` 라벨이 없으면 `new`, 하나면 그 상태. 여러 개이고 `approved`가 섞여 있으면 나머지가 모두 `approved`로 전이 가능할 때만 `approved`(나머지는 `stale`로 반환 → 정리), 그 외 충돌은 예외.

## 2. 위험 정책 (`policy/risk.ts`)

- 정책 스키마(zod): `default`(등급), `requiredChecks[]`, `rules[{tier, patterns[]}]`
- `tierOf(file)`: **규칙 목록 위에서부터 첫 일치**. picomatch `{ dot: true }`, **T3 규칙만 `nocase: true`**(대소문자 무시 파일시스템의 `Agents.md` 우회 차단). 안 맞으면 `default`.
- `classifyFiles`: 중복 제거 후 파일별 등급 → **최고 등급이 PR 등급**.
- `normalizePath`: 역슬래시→슬래시, 앞의 `./`·`/` 제거.
- `ChangedFile`에 `previousFilename`(rename), `additions`(추가된 줄), `patchMissing`(바이너리/큰 diff), `removed`가 있고, **`addedLinesFromPatch`**가 `@@` 이후의 `+` 줄만 추가로 센다(`++`로 시작해도 추가 줄). 용도: 봇 implement PR이 `tests/realworld/skip.txt`에 줄을 **추가**해 게이트를 우회하는 것을 금지.
- `gate.ts`의 `flattenFiles`는 rename이면 새 경로와 **이전 경로 모두** 판정 대상에 넣는다.

## 3. 병합 게이트 (`stages/gate.ts` `runGate`)

판정 순서(위에서 아래로, 해당하면 즉시 종료):

1. PR이 없거나 닫힘/draft/**작성자가 봇이 아님** → `ignored`. base가 main이 아니어도 `ignored`.
2. PR 본문의 팩토리 표식(`parsePrMeta`)이 없으면 `ignored`. `role=improve`면 **`ignored`(사람이 병합)**.
3. 이슈 상태가 `in-review`가 아니면 `ignored`.
4. 필수 체크 요약(`summarizeChecks`): 체크별 상태 — `success/skipped/neutral`=pass, 미완료·**cancelled/stale/null**=pending, 그 외=fail.
   - fail이고 `risk-tier`가 실패 목록에 있으면 → `needs-human`(구현 역할이 T3 경로를 바꿨거나 판정 실패), 기록 `risk_tier`
   - fail인데 리뷰 verdict가 `REQUEST_CHANGES`면 `pending`(이미 fix 흐름), 아니면 `factory-fix.yml`을 `reason=ci`로 dispatch → `fix-dispatched`
   - pending이면 `pending`
5. 변경 파일 목록을 읽는다. 너무 많아 끝까지 못 읽으면(`TooManyFilesError`) **T3로 간주**하고 `needs-human`(`too_many_files`) — **fail closed**.
6. 등급 판정. implement 역할이 `skip.txt`에 줄을 추가했거나 줄 단위 확인이 불가능하면 `needs-human`(`skip_txt`) — risk-tier 체크와 **이중 방어**.
7. **T2 또는 T3 → `needs-human`**, 사유에 등급을 정한 파일 목록을 적고 `risk_t2`/`risk_t3` 기록.
8. verdict가 `REQUEST_CHANGES`면 `pending`. **T1은 `APPROVE`가 있어야** 진행(T0은 체크만으로 진행).
9. `mergePull(sha = 현재 head SHA)`로 squash 병합, 커밋 메시지에 `Refs #이슈` + 서명 트레일러. 거부되면(충돌·보호 규칙) `needs-human`(`merge_refused`) → `merge-failed`. 성공이면 `merged`.

`latestVerdict(reviews, botLogin, headSha)`: **현재 head SHA에 대한 최신 verdict만** 인정(이전 커밋 승인 무시).

`runOnClosed`: 봇 PR이 병합되면 이슈를 `merged`(actor `github`)로, **병합 없이 닫히면 `needs-human`**. 개선 PR(`improve`)은 리포트 이슈에 "병합되었다/닫혔다" 코멘트만 남기고 상태 라벨은 안 바꿈.

## 4. 가드 (`stages/guard.ts`)

- 차단기 상수: `{ windowDays: 7, minRuns: 10, minSuccessRate: 0.5 }`
- `evaluateBreaker`: 최근 7일(그리고 마지막 `factory:alert` 이슈 생성 시각 이후)의 **implement·fix 런만** 집계. `total >= 10`이고 성공률 `< 0.5`이면 `tripped`.
- 작동하면: `FACTORY_PAUSED=true` 설정. **열린 알림 이슈가 있으면 새로 만들지 않고 코멘트로 갱신**, 없으면 `factory:alert` 이슈 생성(멈춘 것/계속 동작하는 것/재개 방법 명시).
- 멈추지 않은 상태라면: 오늘 사용량(`usedToday`)이 일일 상한 미만인 동안 `factory:approved` 이슈를 번호순으로 재개(이미 열린 `factory/issue-N` PR이 있으면 건너뜀).
- 마지막에 `reconcileBoard`로 Projects 보드를 라벨 기준으로 맞춤.
- **읽을 점**: "알림 이후 실패만 다시 센다"로 사람이 재개한 직후 즉시 재차단되는 것을 막음.

## 5. evidence 판정 (`policy/evidence.ts`)

- 테스트 파일 정의 `TEST_FILE_PATTERNS`: `apps/*/test/**`, `**/*.test.ts`, `**/*.test.tsx`, `apps/web/e2e/**`
- **`isSafeRelativePath`**: 길이 ≤300, 문자 집합 `[A-Za-z0-9._/@+-]`만 허용(공백·제어·bidi·비ASCII·역슬래시 거부), 절대경로 거부, 세그먼트가 `""`/`.`/`..`이거나 **`-`로 시작하면 거부**(vitest 옵션으로 해석될 수 있음).
- **`planSuites`**: 파일을 패키지별 실행 계획으로 변환 — `apps/api/`→`--filter api`, `apps/web/`→`--filter web`, `packages/factory/`→`--filter factory`. `apps/web/e2e/`와 그 밖의 위치는 `unsupported`. `*.test.ts(x)`가 아닌 도우미 파일은 **덮어쓰기만 하고 직접 실행하지 않음**.
- 실행 방식은 [02 문서](02-demo-architecture.md)와 워크플로(`ci.yml`)에서: base 트리를 `git worktree`로 만들고 PR이 추가·수정한 테스트 파일만 head 버전으로 덮어쓴 뒤 → base에서 **실패**, head에서 **통과**. 판정은 종료 코드가 아니라 **Vitest JSON 보고서의 단언 결과**(base는 모든 대상 파일을 수집 오류 없이 읽고 1개 이상 실패, head는 모든 대상 파일이 있고 1개 이상 통과·0개 실패). 그래서 **PR이 새로 추가한 모듈을 import해 base에서 수집 오류가 나는 테스트는 재현으로 인정하지 않는다.**

## 6. 그 밖의 파일 (사본 포함)

| 파일 | 읽은 내용 |
|---|---|
| `agents/budget.ts` | `runWithBudget`: 어댑터를 `AbortController` 타임아웃(`Promise.race`)으로 감싸고, 끝난 뒤 `enforceLimits`가 `turns > maxTurns` → `max_turns`, `costUsd > maxUsd` → `budget` 실패로 전환. 어댑터가 던지는 예외는 `engine_error`로 흡수. 타임아웃이 걸리면 성공 결과도 `timeout`으로 덮어씀 |
| `stages/protect.ts` | **`clampResult`**: agent VM이 쓴 결과의 `ok`·턴·시간·비용은 신뢰하지 않고, ok는 publish가 검증한 값으로, 턴·시간·비용은 역할 예산으로 자름. **`importAndVerify`**: publish가 번들을 가져온 뒤 ①결과 파일의 SHA가 번들 head와 같은지 ②번들 head가 준비 시점 main 커밋의 **후손**인지 ③변경 파일을 다시 구해 T3 파일(또는 improve면 허용 경로 밖 파일)이 있거나 변경이 0개면 거부 |
| `stages/command.ts` | `parseCommand`: **코드 펜스(``` / ~~~) 안의 줄은 명령으로 보지 않고**, 줄 시작 `@factory go\|stop\|retry`만 인정. 봇·`[bot]` 사용자는 무시. PR 코멘트면 "연결된 이슈에 남기라"고 안내하고 거부. `isWriter`가 아니면 거부 코멘트 + `permission_denied` 기록 |
| `policy/improve-paths.ts` | `IMPROVE_PATHS = ["AGENTS.md", "factory/prompts/**", "eslint.config.*", "factory/evals/**"]`. `isSafeRelativePath`를 통과한 경로만 허용. `addedCaseSlugs`가 변경 파일에서 `factory/evals/<slug>/case.yml`의 slug를 추출 |
| `policy/eval-case.ts` | 평가 케이스 스키마(`lint`/`prompt-contains`)와 `SLUG` 검증 — 이 노트에서는 상세 미독 |

## 7. 코드에서 배울 구조적 아이디어 (해석)

1. **정책은 데이터(YAML), 판정은 순수 함수**(`classifyFiles(files, policy)`) → 단위 테스트가 쉽고 정책 변경이 코드 변경이 아님.
2. **상태 전이를 한 곳의 표로** 두고 actor를 제한 → "누가 무엇을 선언할 수 있나"가 코드로 고정됨.
3. **fail closed 습관**: 파일 목록을 못 읽으면 T3, 계획이 없으면 실패, 체크가 취소됐으면 대기, 건너뛴 필수 체크가 통과로 오인되지 않게 `!cancelled()`.
4. **이중 방어**: `risk-tier` 체크와 gate, 프롬프트의 금지 경로와 publish 재검증.
5. **입력 경로 검증**(`isSafeRelativePath`)처럼 외부에서 온 값을 파일 시스템·CLI 인자로 쓰기 전에 엄격히 거름.
