 OMP  ida  + 고급 사용법

 1.  ida  — IDA Pro가 에이전트 툴로 내장

  packages/coding-agent/src/tools/ida.ts  +  src/ida/  (host/client/supervisor/worker.py). 마케팅이 아니라 idalib 워커를 진짜 돌림.

 동작 방식

   •  read ./bin/server  → IDB 자동 생성/오픈, 개요 출력. 바이너리 자체는 건드리지 않고  ~/.omp/agent/idbs/<sha16>-<name>/ 에 스토어 IDB 생성.  .i64 / .idb 는 in-place
   • 각 오픈 DB =  omp.ida.<id>  데몬 —  omp ps 에 표시, 프로젝트 내 모든 에이전트/서브에이전트/omp 프로세스가 공유
   •  ida.maxOpen  기본 4 — 초과 시 LRU idle DB 저장+닫기.  ida.idleCloseSec  기본 15분
   • dirty DB는 idle 후 ~10초에 autosave

 모델이 쓰는 인터페이스


   read ./bin/server:main          → 의사코드
   read ./bin/server:main:asm      → 디스어셈블리
   read ./bin/server:imports       → 임포트 테이블
   read ./bin/server:strings:crypt → 스트링 regex
   read ./bin/server:xrefs:main    → xref
   ida(action:"rename",   target:"sub_401000", name:"parse_header")
   ida(action:"set_type", target:"parse_header", decl:"int __fastcall parse_header(...)")
   ida(action:"comment"/"make_function"/"open"/"close"/"save")
   ida(action:"exec", code:"[hex(ea) for ea,n in functions('aes|crypt')]")


  exec 는 DB별 지속 Python 네임스페이스 —  ida_domain ,  idautils  프리로드 + 헬퍼 ( pseudocode() ,  xrefs_to() ,  callers() ,  hexdump()  등). 모든 에이전트가 같은 네임스페이스 공유.  => 에 마지막 expr repr 반
 환.

 왜 중요한가

 리버싱이 read 툴 표면에 통합됨. IDA 있으면 read 프롬프트에 executable-view 가이드만 자동 추가 — 모델이 "바이너리 읽기"를 파일 읽기처럼 함. 구현자 관점: IDA 프로세스 수명주기·공유 네임스페이스·자동저장을 에이전트
 프리미티브로 강등시킨 설계.

 ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

 2. 잘 알려지지 않은 기능들

 데몬 모델 ( omp ps 가 진짜 프로세스 매니저)

 프로젝트당 공유 데몬들:  ida.*  (IDB 워커), blob daemon (아웃바운드 이미지용 stable URL — Anthropic byte-identical 요구 + OpenAI 2GET/quirk 대응),  ovnfs  (overlay VFS — glob/grep을 데몬이 수행),  predict  (입력
 자동완성 엔진). 컴팩션/세션 경계를 넘어 살아남음.

  eval  = 오케스트레이션 언어


   review = workpool(name="review", context="evidence only, no edits")
   review.push(*items)                        # 단계당 하나의 named pool
   h = agent(prompt, {schema, isolated, apply, merge})   # schema-validated 리턴
   j = judge(state, questions)                # choice/bool/score + 확률
   b = judgeBatch(states, questions, {concurrency: 32})  # 대량 분류
   await wait(handles)                        # ordered barrier
   @tool                                      # 커널 로컬 툴 (공유 캐시/dedup)
   budget.remaining()


   •  workpool  두 모드:  freshAgents  (item당 새 에이전트, 무공유) vs keep-alive (least-loaded 워커 재사용, 워커가 대화 유지)
   • 결과 auto-deliver.  wait  tool은 완전 블록 시에만
   • JS/Python 커널 모두 persistent — 한 셀에서 로드한 CSV를 다음 셀에서 차팅
   •  @tool 로 커널 로컬 함수를 서브에이전트 툴로 승격 — 풀 워커들이 공유 캐시/중복제거 집합에 접근

 서브에이전트 운용

   •  Alt+A  Agent Hub: 라이브 로스터 → 트랜스크립트 열람, steer 메시지 주입, revive, kill.  t  flat/tree 전환,  /  필터
   •  task  배치, read-only 에이전트, worktree 격리 (file-disjoint 변환 시 merge conflict 없음)
   • 서브에이전트 IRC DM (피어간 직접 조율)
   •  agent://<id>/findings.0.path  — 서브에이전트 리턴의 필드를 read로 직접 참조

 컨텍스트 다이어트 4종

 ┌───────────────────────────────────────┬────────────────────────────────────────────────────────────┐
 │ 메커니즘                              │ 뭘 하는가                                                  │
 ├───────────────────────────────────────┼────────────────────────────────────────────────────────────┤
 │  checkpoint  → 탐색 →  rewind(report) │ 중간 메시지 전부 제거, 리포트만 남김                       │
 ├───────────────────────────────────────┼────────────────────────────────────────────────────────────┤
 │ TTSR dormant rules                    │ 규칙은 컨텍스트에 없다가 위반 시에만 주입, compaction 생존 │
 ├───────────────────────────────────────┼────────────────────────────────────────────────────────────┤
 │ advisor 재주입 collapse               │ verbatim 재주입 컨텍스트를 접어서 advisor 전송 비용 절감   │
 ├───────────────────────────────────────┼────────────────────────────────────────────────────────────┤
 │  read  요약 모드 + artifact spill     │ 전체 파일 대신 구조 요약                                   │
 └───────────────────────────────────────┴────────────────────────────────────────────────────────────┘

  /seance  — 죽은 세션 소환

 과거 세션을 id/prefix로 찾아 서브에이전트로 부활.  buildSessionContext 로 컨텍스트 재구성 → fork → task 지시. "지난주 세션에서 그 버그 뭐였더라"를 메인 컨텍스트 오염 없이 질의.

 세션 necromancy 풀세트

   •  /collab  — 라이브 세션을 릴레이에 올려 링크+QR 발급. view/read-write. 프레임은 클라이언트측 봉인
   • fork/resume/export/share — 세션 트리 브랜칭
   •  /fresh  — stale 프롬프트 캐시/웨지된 스트림만 리셋, 로컬 트랜스크립트 유지
   •  claude-auto-reset / codex-auto-reset  — 구독 쿼터 윈도우 감지해 자동 세션 리셋
   • foreign session import — 다른 툴의 세션 파일을 읽어서 재개

 Magic keywords (턴 단위 오버라이드)

 standalone 소문자 단어만 트리거 (코드블록/식별자 안에서는 무시):

 ┌──────────────┬──────────────────────────────────────────────────┐
 │ 키워드       │ 효과                                             │
 ├──────────────┼──────────────────────────────────────────────────┤
 │  ultrathink  │ 최고 thinking effort                             │
 ├──────────────┼──────────────────────────────────────────────────┤
 │  chillax     │ 최저 thinking effort, 직행                       │
 ├──────────────┼──────────────────────────────────────────────────┤
 │  orchestrate │ 병렬 subagent 팬아웃 + 단계 검증                 │
 ├──────────────┼──────────────────────────────────────────────────┤
 │  workflowz   │  workpool  기반 결정적 멀티서브에이전트 워크플로 │
 ├──────────────┼──────────────────────────────────────────────────┤
 │  jevify      │  judge()  대량 분류 모드                         │
 └──────────────┴──────────────────────────────────────────────────┘

 각각  <word>-notice  hidden 시스템 주입 + TUI 그라디언트.

 모델 라우팅 심화

   • 9개 role:  default/smol/slow/plan/commit/vision/task/advisor/tiny
   •  Ctrl+P  — 현재 role의 설정 모델 순환
   • path-scoped models:  enabledModels / disabledProviders 에  path:  프리픽스 — 리포별 모델셋 고정
   • fallback chains:  retry.fallbackChains  — 429/쿼터 시 다음 엔트리가 턴 이어받기, 쿨다운 후 복귀
   • round-robin creds: provider당 API키 스택, session affinity + per-cred backoff
   • service tier by family — 모델 패밀리별 속도/비용 티어를 서브에이전트 단위로
   •  smol / tiny  role이 내부 작업 담당: 세션 타이틀 생성, job widget 요약, auto-graph 차트 선택 (judge)

 규칙/감시 체계

   • TTSR ( omfg ): dormant regex rule → 매칭 시 stream abort → rule 주입 → 재시도. LLM이 생성한 규칙을 judge가 최근 8개 output 대상 검증 + regex 자동 리페어
   • advisor: 두 번째 모델이 매 턴 델타 감시. note/concern/blocker 인라인 주입. advisor 자체 컨텍스트 관리(모델 promote/clear) + quota 고갈 시 credential fallback + 롤백
   •  /review  — 전용 reviewer 서브에이전트 병렬 스윕, P0~P3 + 신뢰도 + ship/no-ship 판정.  /annotate 로 diff 라인에 핀 메모 선탑재
   •  security_scan  — 기본 off (설정 게이트). Codex Security 클라우드 스캔 구동

 읽기 표면의 깊이

  read  하나로: 파일/디렉토리/아카이브( x.tar.gz:inner/file )/SQLite/PDF/ipynb/프로파일러 리포트/URL/ pr:// / issue:// / agent:// / skill:// / ssh:// / conflict:// / xd:// . 셀렉터  :50-100 ,  :raw , hashline/
 line-number/raw 자동 디스플레이 모드. MCP 서버는 스킴 추가 가능.

 메모리 — 에이전트가 큐레이트

   •  retain  (런 중 팩트),  recall ,  reflect  (뱅크 전체 종합),  memory_edit  (id별 수정/무효화)
   •  learn  — 재사용 레슨 → managed skill로 승격 가능
   • sharpshooter: 대화에서 typed delta 추출 —  architecture_decision | product_decision | style_decision | constraint | rejected_approach | correction , 각각 evidence + friction 점수. 스케줄러가 배치 추출 →
     consolidate가 머지
   • 백엔드: local / Hindsight / Mnemopi. project-scoped 기본

 기타 깊은 것들

   •  checkpoint / rewind  (위 표)
   •  vibe  모드 ( /vibe ): persistent  fast / good  워커 세션 — 감독 패턴 (vibe_spawn/vibe_send/vibe_wait)
   •  stt  push-to-talk — MLX ASR 워커, endpointer
   •  predict  — 입력 자동완성 (ngram + SmolLM 블렌드, 외래 Claude/Codex 히스토리 학습)
   •  ssh  —  read ssh://host/path  + sshfs mount + 커넥션 매니저
   •  blob-broker  — 외래 이미지 안정 URL화 (provider 캐시 미스 대응)
   •  auto-graph  — judge role이 차트 종류/컬럼/방향성까지 골라줌
   •  spotlight  — 특정 tool call 시점의 "에이전트가 뭘 하려 했나" 컨텍스트 재구성
   •  auto-prewalk  — 턴 시작 전 코드베이스 선탐색 여부 분류
   •  context-notes  — 세션 스크래치 메모
   •  aside  — 사이드 채널 검색/검색+컨텍스트 합성
   •  approval-scope ,  approval-cache ,  file-mutation-queue  — 승인 세분화
   •  native-delegation  — Rust 네이티브로 위임되는 작업 표시
   •  secrets  — 아웃바운드 메시지 obfuscation (advisor/MCP/외래 세션 대상)

 ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

 3. 파워유저 레시피

 대규모 리뷰:


   workflowz 로 진입 → workpool push로 파일별 reviewer →
   judge로 P0~P3 심각도 + false-positive 확률 분류 → 메인에서 통합


 리버싱 워크플로:


   read ./bin:main          # 의사코드 개요
   read ./bin:strings:flag  # 힌트 스트링
   ida exec로 함수 목록/xref 배치 추출 → subagent에 "crypt* 함수 리네임+타입" 위임
   ida save로 스냅샷


 취약한 diff 검증:


   checkpoint → 서브에이전트들이 variant 패치 시도 → rewind(report)로 컨텍스트 회수 →
   승자만 메인에서 적용


 무인 장시간 작업:


   advisor = gpt-5.5-codex (second-opinion) +
   TTSR 규칙으로 알려진 안티패턴 차단 +
   workpool keep-alive로 반복 작업 +
   budget 하드캡 + wait로 주기적 체크인


 세션 회수/이어하기:


   /seance로 어제 세션 소환 → "이 접근 왜 버렸지?" 질의 → 그 결과만 메인에 인용
   (메인 컨텍스트는 청정 유지)


 크리덴셜 번아웃 방지:


   provider당 키 3개 스택 + fallbackChains로 codex→anthropic→gemini +
   path: ./legacy-repo에 다른 모델셋 +
   advisor는 저렴한 모델로


 ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

 한 줄 요약

 OMP의 숨은 축은 "에이전트가 운영체제다" —  omp ps  데몬, eval 커널,  xd://  가상 디바이스,  agent:// / ssh:// / conflict://  스킴, daemon-shared IDB/overlay VFS. 도구가 아니라 프로세스·파일시스템·IPC를 에이전트
 프리미티브로 만든 설계. GJC는 게이트·증거·외부제어에 같은 깊이를 쏟았고, OMP는 런타임 표현력에 쏟음.
