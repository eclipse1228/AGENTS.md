# reference — 원본 파일 사본

[roboco-io/the-software-factory-demo](https://github.com/roboco-io/the-software-factory-demo) (커밋 `5b1927f8328266070c3d0bf98bac194dc21df012`, 2026-10-13)에서 **내용을 수정하지 않고** 복사한 학습용 사본입니다. 원본 라이선스는 Apache-2.0이며 [LICENSE](LICENSE)를 함께 둡니다. 원본이 갱신되면 이 사본은 낡을 수 있으니 최신 내용은 원본 repo를 보세요.

| 사본 경로 | 원본 경로 |
|---|---|
| `AGENTS.md` | `AGENTS.md` |
| `CHANGELOG.md` | `CHANGELOG.md` |
| `LICENSE` | `LICENSE` |
| `docs/design-spec.md` | `docs/superpowers/specs/2026-10-11-software-factory-design.md` |
| `docs/operations.md` | `docs/operations.md` |
| `factory/risk-policy.yml`, `factory/agents.yml` | 동일 |
| `factory/prompts/*.md` | 동일 |
| `factory/evals/**` | 동일 |
| `agents-skills/project-changelog/SKILL.md` | `.agents/skills/project-changelog/SKILL.md` |
| `github/CODEOWNERS` | `.github/CODEOWNERS` |
| `github/ISSUE_TEMPLATE/*.yml` | `.github/ISSUE_TEMPLATE/*.yml` |
| `github/workflows/{ci,factory-*}.yml` (13개) | `.github/workflows/` (앱 배포용 `deploy.yml`, `preview.yml` 제외) |
| `runtime-src/state/machine.ts` | `packages/factory/src/state/machine.ts` |
| `runtime-src/policy/{risk,evidence,eval-case,improve-paths}.ts` | `packages/factory/src/policy/` |
| `runtime-src/stages/{gate,guard,protect,command}.ts` | `packages/factory/src/stages/` |
| `runtime-src/budget.ts` | `packages/factory/src/agents/budget.ts` |

**복사하지 않은 것**: `apps/`, `tests/`, `scripts/`, `pnpm-lock.yaml`, `docs/superpowers/plans/`(약 880KB), `packages/factory`의 나머지 소스·테스트, `factory/test-fixtures/`.

사본에는 원본 repo의 내부 링크(예: `docs/superpowers/specs/...`, `[AGENTS.md](../AGENTS.md)`)가 그대로 들어 있어 이 폴더 안에서는 깨질 수 있습니다.
