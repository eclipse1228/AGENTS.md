# sw-factory — 소프트웨어 팩토리 학습·적용 노트

로보코(ROBOCO)의 소프트웨어 팩토리 연구와 데모 구현체를 읽고, 내가 개발·유지보수하는 SaaS에 **단계별로 도입**하기 위해 정리한 자료입니다.

- 정리일: 2026-10-16
- 원본 1: [오픈소스 프로젝트들로부터 배우는 소프트웨어 팩토리 모범사례 (1/4)](https://roboco.io/posts/software-factory-best-practices-from-open-source/)
- 원본 2: [roboco-io/the-software-factory-demo](https://github.com/roboco-io/the-software-factory-demo) (정리 기준 커밋 `5b1927f`, 2026-10-13)
- 데모 라이선스: Apache-2.0. `reference/`의 사본에는 원본 `LICENSE`를 함께 두었고, 파일 내용은 수정하지 않았습니다.

## 읽는 순서

| 순서 | 문서 | 내용 |
|---|---|---|
| 1 | [01-concepts-and-criteria.md](01-concepts-and-criteria.md) | 블로그 1편 정리: 팩토리 판별 기준 6개, 12개 저장소, 반복 패턴 6개, 미해결 문제 |
| 2 | [02-demo-architecture.md](02-demo-architecture.md) | 데모 설계 문서 정리: 상태 머신, 위험 등급, 워크플로, 에이전트 어댑터, 가드레일 |
| 3 | [03-folder-guide.md](03-folder-guide.md) | 데모 repo에서 공부할 폴더·파일과 읽는 우선순위 |
| 4 | [04-prompts-and-evals.md](04-prompts-and-evals.md) | 프롬프트 5종 분석, 평가 케이스 형식, 변경 기록 스킬 |
| 5 | [05-security-and-limits.md](05-security-and-limits.md) | 보안 설계(3단 job 분리, 시크릿 격리)와 데모가 수용한 잔여 위험 |
| 6 | [06-build-history-and-lessons.md](06-build-history-and-lessons.md) | P1~P4 구축 이력, 실제로 겪은 실패와 교훈 |
| 7 | [07-runtime-code-notes.md](07-runtime-code-notes.md) | 런타임 코드 읽기 노트(상태 전이표, 게이트 로직, 차단기, evidence 계획) |
| 8 | [08-adoption-roadmap.md](08-adoption-roadmap.md) | **내 SaaS 적용 로드맵**: 0~5단계, 졸업 기준, 데모 파일 대응표 |
| 9 | [09-glossary-and-checklists.md](09-glossary-and-checklists.md) | 용어집, SaaS별 진단표, 도입 체크리스트 |

원본 파일 사본은 [reference/](reference/README.md)에 있습니다.

## 한 문단 요약

소프트웨어 팩토리는 "AI 에이전트를 쓰는 개발"이 아니라 **작업 유입 → 구현 → 검증 → 병합 → 배포 → 실패의 규칙화**가 사람의 매번 지시 없이 이어지는 체계입니다. 핵심은 에이전트의 능력이 아니라 **경계 설계**입니다. 무엇을 자동으로 하고(위험 등급), 무엇을 증거로 막고(evidence gate), 어디서 사람이 승인하며(`@factory go`, CODEOWNERS), 실패를 어떻게 규칙으로 바꾸는지(eval + improve)를 정합니다. 데모는 이를 GitHub Issues/Actions만으로 구현했고, 모든 로직을 테스트 가능한 TypeScript CLI에 두었습니다.

## 이 자료에서 지킨 원칙

- 원본에 있는 사실만 적고, 내 해석은 "해석:" 또는 08번 문서(적용 로드맵)에 분리했습니다.
- 1편 블로그 글은 원문을 직접 읽지 않고 요약 추출본으로 정리했습니다. 수치(203/300 등)는 원문과 대조해 재확인하세요.
- 블로그 2~4편과 「바이브 코딩을 넘어서 소프트웨어 팩토리로」는 아직 읽지 않았습니다. 링크만 [01 문서](01-concepts-and-criteria.md#후속-읽을-자료)에 있습니다.
- 데모의 구현 계획서(`docs/superpowers/plans/` p1~p4, 합계 약 880KB, 대부분 코드)는 복사하지 않고 목차·판단 기록만 [06 문서](06-build-history-and-lessons.md)에 정리했습니다. 전체는 원본 repo에서 보세요.
