# Code Search and Navigation Policy

This repository uses three complementary code-navigation tools:

* `ripwire` for architectural/contextual questions, call relationships, impact analysis, task preparation, and post-edit analysis.
* `tgrep` for fast repeated text and regex searches when an index/server is available.
* `rg` (ripgrep) as the always-correct live-filesystem search fallback and for post-edit verification.

They are not interchangeable. Select the tool according to the question.

## Core rule

Do not begin a non-trivial task by recursively reading large parts of the repository.

First determine whether the task needs:

1. code understanding,
2. exact text lookup, or
3. verification of the current filesystem state.

Then use the appropriate tool below.

## 1. Task orientation and architecture: use ripwire

For a new feature, bug, refactor, or unfamiliar subsystem, prefer:

`ripwire . --pack-task="<task described using the repository's own vocabulary>"`

For lighter orientation:

`ripwire . --for="<task described using the repository's own vocabulary>"`

Use code identifiers and terminology already present in this repository whenever possible.

If the location is already known and the question can be answered with one exact text search, skip ripwire and search directly.

Do not use ripwire merely to locate a literal string in a known file.

## 2. Exact strings, identifiers, config keys, errors, and regexes: use tgrep or rg

For ordinary repeated repository search, prefer `tgrep` when installed.

Literal symbol/string:

`tgrep -F -- "symbolOrLiteral" .`

Whole word:

`tgrep -w -F -- "symbolName" .`

Regex:

`tgrep -- "pattern.*here" .`

Find matching files first when a query is broad:

`tgrep -l -F -- "symbolOrLiteral" .`

Then narrow the next search to the relevant directory or files.

Request only a small amount of context when needed:

`tgrep -F -C 3 -- "symbolOrLiteral" path/to/relevant/dir`

Always put tgrep flags before `--`, put the pattern after `--`, and provide an explicit search path.

If `tgrep` is unavailable, errors, or its indexed result is unsuitable, use equivalent `rg` commands.

Examples:

`rg -F -- "symbolOrLiteral" .`

`rg -w -F -- "symbolName" .`

`rg -l -F -- "symbolOrLiteral" .`

`rg -F -C 3 -- "symbolOrLiteral" path/to/relevant/dir`

Do not fall back to `grep -R` when `rg` is available.

## 3. Relationships and blast radius: use ripwire

When the question is about code relationships rather than text occurrence, prefer ripwire.

Callers:

`ripwire . --callers=SYMBOL`

Callees:

`ripwire . --callees=SYMBOL`

Direct use sites:

`ripwire . --uses=SYMBOL`

Potential blast radius:

`ripwire . --impact=SYMBOL`

Read a specific function only when necessary:

`ripwire . --expand=SYMBOL --top-k=0`

A text occurrence is not necessarily a call or dependency. Do not substitute grep results for call-graph analysis when the distinction matters.

## 4. Treat ripwire results as a map, not as proof

Inspect ripwire's confidence and completeness signals.

If output reports ambiguity, unresolved relationships, truncation, low confidence, a count floor, or otherwise incomplete information, verify the relevant result against source code with `rg`/`tgrep` and direct file reads.

Do not infer that a missing ripwire edge proves that no runtime relationship exists.

Be especially cautious around dynamic dispatch, callbacks, registries, dependency injection, framework conventions, generated code, and string-based lookup.

## 5. Freshness rule: indexed search is not final verification

`tgrep` is an acceleration layer, not the source of truth after edits.

Immediately after creating, deleting, renaming, or modifying code, never rely exclusively on an indexed tgrep query to prove that the new filesystem state is correct.

For post-edit verification use:

`rg ...`

or explicitly force a live tgrep scan:

`tgrep --no-index ...`

Examples:

Verify that an old symbol no longer exists:

`rg -F -- "oldSymbol" .`

Verify that a new symbol exists where expected:

`rg -F -- "newSymbol" path/to/scope`

Use indexed tgrep again for normal exploration after verification.

## 6. Editing workflow

For a non-trivial change, follow this sequence:

BEFORE EDITING

Use `ripwire --pack-task` or `--for` to locate the relevant subsystem.

For important symbols, inspect callers, uses, and impact before changing their contracts.

Use tgrep/rg to confirm exact occurrences and configuration references.

DURING EDITING

Keep searches scoped.

Prefer exact literal searches with `-F`.

Do not repeatedly read whole large files when a focused search or symbol expansion is sufficient.

AFTER EDITING

Verify important textual invariants with live `rg`.

For an edited API or significant symbol, run:

`ripwire . --edit-check=SYMBOL`

Review the current change context with:

`ripwire . --situ`

Ask ripwire for test obligations with:

`ripwire . --test-gate`

Then run the repository's actual formatter, lint, typecheck, tests, and build commands relevant to the changed area.

Never treat ripwire as a replacement for compiler, type checker, test suite, or runtime validation.

## 7. Test-gate exit code

`ripwire --test-gate` returning exit code 4 means that test obligations remain.

Do not treat exit code 4 as a tool crash.

Resolve the named obligations and run the actual relevant tests.

## 8. Search efficiency rules

Prefer this search progression:

broad file discovery → narrow directory/file search → small context read → direct source inspection.

Avoid dumping hundreds or thousands of search matches into model context.

For broad queries, first use file-only output (`-l`), then inspect a smaller set.

Prefer `-F` for identifiers, URLs, error strings, config names, and user-provided literal text.

Use regex only when regex semantics are actually required.

Prefer repository-relative paths and narrow scopes.

Do not search dependency/build directories unless the task specifically requires them.

## 9. tgrep server/index handling

If `tgrep status .` succeeds, use the existing server/index.

Do not start duplicate tgrep servers.

If no server can be kept alive, an on-disk index may be used, but remember that it represents the last published index state.

Never commit `.tgrep/`.

Do not rebuild an index merely to perform one freshness-sensitive verification; use `rg` or `tgrep --no-index` instead.

## 10. Tool failure and fallback order

If ripwire cannot answer a structural question reliably:

ripwire → focused tgrep/rg → direct source inspection.

If tgrep is unavailable:

tgrep → rg.

If indexed tgrep freshness is uncertain:

tgrep → rg or `tgrep --no-index`.

Never let tool availability block progress.

## 11. Validation command discovery

Before inventing a test, build, lint, or typecheck command, inspect the repository's existing configuration and scripts.

Prefer project-defined commands from files such as:

`package.json`, lockfiles, `Makefile`, task runners, CI workflows, language manifests, and repository documentation.

Run the smallest relevant validation first, followed by broader validation when the change warrants it.

## 12. Definition of done

A code change is not complete merely because search results look correct.

Completion requires:

* the intended code path was identified,
* relevant callers/use sites were considered,
* stale names/references were checked using live filesystem search,
* relevant tests/type checks/build checks were run,
* and failures or uncertainty from static analysis were explicitly investigated.
