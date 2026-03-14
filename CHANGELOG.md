# Changelog

All notable changes to the Iterative Planner project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [2.9.2] - 2026-03-14

### Changed
- **Normalize REPLAN naming** — all variants (`RE-PLAN`, `RE_PLAN`, `Re-plan`, `re-plan`) unified to `REPLAN`/`Replan`/`replan` across all files. Mermaid diagrams, prose, validator, references, and changelog all use the same form. Removed mermaid naming convention notes (no longer needed).

## [2.9.1] - 2026-03-14

### Fixed
- **build.ps1 silent success on unknown commands** — unknown commands now exit with code 1 instead of silently showing help and exiting 0. Cherry-picked from PR #1.

### Added
- **README merge edge case docs** — documented consolidated file merge behavior (heading extraction, boilerplate stripping, link rewriting). Cherry-picked from PR #1.

## [2.9.0] - 2026-03-06

### Fixed
- **stripHeader H1 injection** — `stripHeader()` in bootstrap.mjs could inject a stale H1 heading into consolidated files. Fixed heading removal logic.
- **verification.md template** — corrected the verification.md bootstrap template formatting.
- **INDEX.md pipe escaping** — pipe characters in INDEX.md table entries are now properly escaped to prevent broken markdown tables.
- **Validator numbered findings** — `validate-plan.mjs` now correctly parses numbered findings lists instead of only bullet-style findings.
- **Test counts and README project tree updated** — CLAUDE.md and README.md now reflect accurate test count and project structure.

## [2.8.0] - 2026-03-06

### Fixed
- **extractSection() only captured first line** — regex `([\\s\\S]*?)(?=\\n## |$)` with multiline flag caused `$` to match end-of-line, making lazy quantifier stop after first line. Replaced with indexOf-based approach. This broke the findings count gate (≥3 before PLAN) — `checkFindings()` always reported ≤1 finding regardless of actual count.

### Added
- **Bootstrap transition shortcuts documented** — SKILL.md Transitions section now documents that `bootstrap close` allows any-state→CLOSE (EXPLORE→CLOSE, PLAN→CLOSE, EXECUTE→CLOSE, REPLAN→CLOSE).
- **Mermaid naming convention note** — SKILL.md and README.md added note about `RE_PLAN` vs `RE-PLAN` naming (later removed in v2.9.2 when all variants were normalized to `REPLAN`).
- **7 new validator tests** — extractSection multi-line capture, findings count thresholds (0/2/3/5), summary.md at CLOSE, iteration/version mismatch, last-section edge case. 97 tests total (was 90).

## [2.7.2] - 2026-03-06

### Fixed
- **CRITICAL: Validator regex mis-parsed REPLAN transitions** — `validate-plan.mjs` line 122 regex `[→\->]` char class included literal `-`, causing `REPLAN → PLAN` to be split as `RE` + `-` (arrow) + `PLAN` and flagged as invalid. Fixed with `\s+(?:→|->)\s+`.
- **Orphan warning false positive** — `bootstrap.mjs new` warned about "orphaned directories from a previous crash" whenever closed plans existed without an active pointer (normal state after `close`). Now only warns when pointer file exists but points to a non-existent directory.
- **Validator missing summary.md check** — added WARN-level check for `summary.md` existence when plan state is CLOSE.
- **Resume missing verification.md** — `bootstrap.mjs resume` now lists `verification.md` in recovery files output.
- Updated orphan warning test to simulate corrupted pointer (correct scenario) + added test for no false warning after normal close. 90 tests total.

## [2.7.1] - 2026-03-06

### Changed
- **REFLECT → CLOSE requires user confirmation** — agent no longer auto-closes. Must present completed items, remaining work, verification summary, and recommendation, then wait for user to confirm close. Transition rule, REFLECT routing table, and User Interaction table updated.

## [2.7.0] - 2026-03-06

### Added
- **Protocol compliance validator** (`src/scripts/validate-plan.mjs`) — new script that checks state transition validity, mandatory plan.md sections, findings count, cross-file consistency (state/plan/progress/verification), and consolidated files existence. Read-only and advisory. Exit 0 on pass, exit 1 on errors. Warnings are non-blocking. Run during REFLECT or at any time. 12 new tests added (89 total).
- **Plan topic index** (`plans/INDEX.md`) — topic-to-directory mapping file, created on first `new`, updated on each `close`. Survives sliding window trim. Extracted topics come from findings.md index entries. Enables finding old plan data when consolidated files have been trimmed.
- **Lessons snapshot** (`lessons_snapshot.md`) — `close` now copies `plans/LESSONS.md` to `plans/{plan-dir}/lessons_snapshot.md` before removing the pointer. Makes old lesson states recoverable — previously, LESSONS.md rewrites were lossy and irrecoverable.
- **Protocol tiering** — checks marked *(EXTENDED)* in SKILL.md per-state rules may be skipped for iteration 1 single-pass plans. EXTENDED checks: prediction accuracy, devil's advocate, adversarial subagent review, ghost constraint scan. All other checks are CORE (always enforced).
- **Build validation expanded** — Makefile and build.ps1 now validate INDEX.md reference in bootstrap.mjs and validate-plan.mjs syntax.

### Changed
- **SKILL.md Filesystem Structure** updated with `INDEX.md` and `lessons_snapshot.md`.
- **SKILL.md Recovery** expanded with step 10 for INDEX.md.
- **SKILL.md Bootstrapping** expanded with validate-plan.mjs command.
- **SKILL.md EXPLORE** now includes INDEX.md in cross-plan context reads.
- **file-formats.md** now documents INDEX.md template and lessons_snapshot.md.
- **CLAUDE.md** updated: validation checklist expanded, tree includes validate-plan.mjs, test count updated to 89.

## [2.6.0] - 2026-03-06

### Added
- **Criteria adequacy check in REFLECT** — before running verification, ask: do these criteria test what matters, or what was easy to test? Notes gaps in `verification.md` Not Verified section.
- **Not-verified list in REFLECT** — mandatory "Not Verified" section in `verification.md`: what wasn't tested and why (no coverage, out of scope, untestable). Forces honesty about coverage gaps. Template and explanatory note added to `file-formats.md`.
- **Devil's advocate in REFLECT** — before routing to CLOSE, name one reason this might still be wrong despite passing verification. Recorded in `decisions.md`. Combats confirmation bias and sunk cost.
- **Adversarial subagent review in REFLECT** — for iteration ≥ 2, optional Task subagent reviews `verification.md`, `plan.md` criteria, and `decisions.md` for adequacy and blind spots. Main agent must address concerns before CLOSE. Adds genuine independence from anchoring bias on multi-iteration plans.
- **Phase Balance Heuristic expanded** — REFLECT warning in `planning-rigor.md` now requires justification in `decisions.md` when routing CLOSE after <5% REFLECT effort.

## [2.5.0] - 2026-03-05

### Added
- **Planning rigor reference** (`src/references/planning-rigor.md`) — new reference file with 7 techniques: assumption tracking, pre-mortem & falsification signals, exploration confidence, prediction accuracy, ghost constraint hunting, phase balance heuristic, decomposition at iteration limit.
- **Assumptions in plan.md** — mandatory bullet list: what you assume, which finding grounds it, which steps depend on it. On surprise discovery during EXECUTE, check assumptions first to identify invalidated steps. Template added to bootstrap and file-formats.
- **Pre-Mortem & Falsification Signals in plan.md** — mandatory section combining "assume the plan failed, why?" with concrete STOP IF triggers checked during EXECUTE. Covers approach validity (distinct from Failure Modes which cover dependencies). Template added to bootstrap and file-formats.
- **Exploration Confidence gate** — quality check before EXPLORE → PLAN transition: problem scope, solution space, risk visibility must each be at least "adequate." Recorded in state.md transition log, not as a separate file section.
- **Prediction Accuracy in verification.md** — during REFLECT, compare plan.md predictions (step count, file count, line delta) against actuals. Builds calibration data for LESSONS.md. Template added to bootstrap and file-formats.
- **Ghost constraint scan in REPLAN** — before designing a new approach, actively check if the constraint that led to the failed approach is still valid. 3-question checklist in SKILL.md, detailed guidance in planning-rigor.md.
- **Decomposition analysis at iteration 5** — mandatory analysis in decisions.md identifying 2-3 independent sub-goals before the iteration 6 hard stop. Gives users actionable next steps.
- **Step risk/dependency annotations** — `[RISK: low/medium/high]` and `[deps: N,M]` recommended on each plan step. Enforces risk-first ordering and reveals parallelization opportunities.
- **Phase balance heuristic** — rough effort distribution guideline (EXPLORE 20-30%, EXECUTE 40-50%, etc.) with warning signs for imbalance.

### Changed
- **"Risks" section removed from plan.md** — subsumed by Failure Modes (dependencies) and Pre-Mortem (approach validity). No unique purpose remaining. Removed from bootstrap template, file-formats template, and test assertions.

## [2.4.0] - 2026-03-05

### Added
- **Constraint classification in EXPLORE** — guidance to classify findings as hard constraints (non-negotiable), soft constraints (negotiable preferences), or ghost constraints (past constraints that no longer apply). Sourced from Hohpe's constraint identification framework, generalized for any domain.
- **Problem decomposition in PLAN** — 5-point process for breaking goals into steps: understand the whole first, identify natural boundaries, minimize dependencies, start with riskiest part, split/merge criteria.
- **Essential vs accidental complexity in Simplification Checks** — new check #3: "Is this inherent in the problem, or did we create it?" Adds analytical depth to REFLECT. Simplification Checks now 6 (was 5). Sourced from Brooks' essential/accidental complexity model.

## [2.3.0] - 2026-03-03

### Added
- **Sliding window for consolidated files** — bootstrap auto-trims `plans/FINDINGS.md` and `plans/DECISIONS.md` to the 8 most recent plan sections on each close. Keeps files naturally bounded at ~300-450 lines. Old plan data remains in per-plan directories. Compression rarely triggers. 3 new tests added.

### Fixed
- **Consolidated merge corrupted files after compression** — `prependToConsolidated()` inserted new plan content inside `<!-- COMPRESSED-SUMMARY -->` markers when a compressed summary existed, because `indexOf("\n## ")` found `## Summary (compressed)` before `## plan_*`. Now skips past the closing marker before finding the insertion point.
- **`stripCrossPlanNote` regex mismatch** — regex matched old format (`...and plans/DECISIONS.md`) but not current format (`...plans/DECISIONS.md, and plans/LESSONS.md`). Updated to wildcard match `[^*]*` after `plans/FINDINGS.md`.
- **No deduplication guard on close** — closing the same plan twice produced duplicate sections. Added existence check in `prependToConsolidated()`.
- **Blank line accumulation in consolidated files** — each prepend cycle added an extra blank line to the header area. Fixed by trimming header whitespace before insertion.

## [2.2.0] - 2026-03-02

### Added
- **Cross-plan institutional memory (`plans/LESSONS.md`)** — new consolidated file for capturing user corrections, recurring mistakes, and workflow preferences across plans. Bootstrap creates it on first `new`. Referenced in SKILL.md at 5 protocol points: EXPLORE (read at start), PLAN gate check, REPLAN (review before pivot), CLOSE (merge lessons learned), and Recovery. 9 new tests added (73 total).

### Fixed
- **README badge updated** — was `v2.1.2`, now matches VERSION.
- **Test count corrected** — CLAUDE.md and README.md said "64 tests"; actual is 73.
- **build.ps1 header comment completed** — listed 7 of 11 commands; now lists all 11.
- **Test file excluded from packages** — `*.mjs` glob in Makefile and build.ps1 was including `bootstrap.test.mjs` (~58KB) in distribution packages. Now explicitly copies only `bootstrap.mjs`.
- **LESSONS.md added to build validation** — Makefile and build.ps1 now check that `bootstrap.mjs` references `LESSONS.md`, matching existing checks for `FINDINGS.md` and `DECISIONS.md`.

## [2.1.4] - 2026-02-24

### Fixed
- **Read-before-write coverage completed** — v2.1.3 missed `plan.md` and other files on first write after bootstrap. Now: (1) bootstrap section requires reading all 6 plan files before starting EXPLORE, (2) PLAN gate check expanded to include `state.md`, `plan.md`, `progress.md`, `verification.md` alongside existing findings/decisions reads, (3) EXPLORE reads `state.md` at start. Covers every bootstrap-created file.

## [2.1.3] - 2026-02-24

### Fixed
- **Read-before-write rule added** — Claude Code's Write tool rejects writes to files not yet read in the current session. Added explicit "read-before-write" rule to File Lifecycle Matrix, EXPLORE (`findings.md`), PLAN (`verification.md`, `state.md`, `progress.md`), and REFLECT (`verification.md`). Prevents "failed to write file" errors on first update after bootstrap.
- **Mandatory re-reads expanded** — added `verification.md` to the "Before any REFLECT" row in the Mandatory Re-reads table.

## [2.1.2] - 2026-02-24

### Fixed
- **`.gitignore` cleaned** — removed ~200 lines of Python boilerplate from a non-Python project. Only project-relevant entries remain (build/, dist/, .claude/, plans/, nul).
- **SKILL.md `close` description corrected** — previously said "removes pointer only"; now accurately describes the full behavior (merge findings/decisions to consolidated files, update state.md, remove pointer).
- **Revert-First step count aligned** — `complexity-control.md` had 6 steps while SKILL.md had 5. Harmonized to 5.
- **SKILL.md duplication trimmed** — REPLAN keep-vs-revert decision tree and irreversible operations procedure now summarize and point to `references/code-hygiene.md` instead of duplicating full content.
- **Iteration 5 / Nuclear Option consolidated** — removed duplicate from "Iteration Limits" section; single definition in "Complexity Control" section.
- **`build.ps1` default command** — changed from `help` to `package` to match Makefile behavior.
- **`build.ps1` combined build ordering** — added `Sort-Object Name` for deterministic reference file ordering (Makefile already sorted).
- **Redundant tests removed** — removed 2 tests that were strict subsets of other tests; added `## Verification Strategy` to `requiredSections` validation array; removed unused `before` import. Test count: 66 → 64.

### Added
- **`bootstrap.test.mjs` in project trees** — README.md and CLAUDE.md now include the test file in their project structure listings.

## [2.1.1] - 2026-02-19

### Changed
- **Quick Start reordered** — Option 1 is now zip package install to `~/.claude/skills/` (recommended). Single-file moved to Option 2.
- **README badge** bumped to v2.1.0.

## [2.1.0] - 2026-02-19

### Added
- **Verification feedback loop** — new `verification.md` per-plan artifact for recording objective verification results during REFLECT. Ensures REFLECT and CLOSE transitions are grounded in evidence (test results, lint output, behavioral diffs, smoke tests) rather than subjective assessment.
- **Verification Strategy in PLAN** — mandatory section in `plan.md` mapping each success criterion to a test/check method and expected result. Plans with no testable criteria must write "N/A — manual review only" (proves you checked). Documented in SKILL.md PLAN rules and file-formats.md template.
- **REFLECT verification gate** — REFLECT rules now require running each check from the Verification Strategy and recording results in `verification.md` (criterion, method, command, result PASS/FAIL, evidence). REFLECT → CLOSE transition strengthened from "All success criteria met" to "All criteria verified PASS in `verification.md`".
- **File Lifecycle Matrix expanded** — added `verification.md` row: W in PLAN (initial template), W in EXECUTE (per-step results), W in REFLECT (full verification pass), R in REPLAN and CLOSE.
- **Structured Simplification Checks** — `complexity-control.md` Simplification Checks now have a recording template with blocker flag. If any check reveals a blocker, it must be addressed before CLOSE.
- **Bootstrap creates verification.md** — `bootstrap.mjs` `new` command creates `verification.md` with initial template (criteria table, additional checks, verdict sections).
- **Build validation expanded** — Makefile and build.ps1 now validate that `bootstrap.mjs` creates `verification.md`.

## [2.0.0] - 2026-02-19

### Changed (BREAKING)
- **Plan storage moved from `.claude/` to `plans/`** — plan directories are now visible (not hidden) and decoupled from Claude Code's own `.claude/` config directory. Directory prefix changed from `.plan_` to `plan_` (no leading dot). Pointer file moved from `.claude/.current_plan` to `plans/.current_plan`. Gitignore pattern simplified from `.claude/.plan_*` + `.claude/.current_plan` to `plans/`.

### Added
- **Consolidated cross-plan files** — `plans/FINDINGS.md` and `plans/DECISIONS.md` persist across plans. Created on first `new`, updated on each `close`. Enables cross-plan knowledge transfer: findings and decisions from previous plans are available to subsequent plans.
- **Merge-on-close** — when `close` is run, per-plan `findings.md` and `decisions.md` are merged into consolidated files. Content is prepended (newest first) so the most recent context is immediately accessible. Headings are demoted (## → ###) and nested under a `## plan_YYYY-MM-DD_XXXXXXXX` section. Relative `findings/` links are rewritten to include the plan directory name.
- **Cross-plan context seeding** — when consolidated files exist, new per-plan `findings.md` and `decisions.md` include a cross-plan context reference note.
- **Consolidated files in resume output** — `resume` command now shows `plans/FINDINGS.md` and `plans/DECISIONS.md` paths.
- **EXPLORE reads consolidated files** — EXPLORE rules now include reading consolidated files at start for cross-plan context.
- **PLAN gate check expanded** — PLAN gate check now includes `plans/FINDINGS.md` and `plans/DECISIONS.md`.
- **File Lifecycle Matrix expanded** — added `plans/FINDINGS.md` and `plans/DECISIONS.md` rows: R in EXPLORE/PLAN/REPLAN, W(merge) in CLOSE.
- **Recovery protocol expanded** — added step 8 for consolidated cross-plan context files.
- **Consolidated file templates** — `file-formats.md` now documents `plans/FINDINGS.md` and `plans/DECISIONS.md` formats.
- **Build script validation** — Makefile and build.ps1 validate that bootstrap.mjs references `FINDINGS.md` and `DECISIONS.md`.
- **Build script tests** — round-trip test verifies consolidated files exist after `close`.

## [1.9.0] - 2026-02-18

### Fixed
- **Goal regex first-line capture** — removed `m` flag from goal extraction regex in bootstrap.mjs; `^` could match mid-content. Changed to `\n` anchor. `resume` and `status` now truncate goal to first line (matching `list` behavior).
- **EXECUTE → REFLECT trigger clarification** — Mermaid diagram and transition table wording updated from "step done" to "phase ends" to reflect that REFLECT triggers when all steps complete, not after each individual step.
- **File Lifecycle Matrix legend incomplete** — expanded R/W/— legend to define R+W (distinct read and write operations), removing ambiguity.
- **Makefile test cleanup on failure** — wrapped round-trip test in `bash -c` with `trap` for guaranteed temp directory cleanup even on test failure.
- **CLI `close` vs protocol CLOSE confusion** — added note to `cmdClose` output and SKILL.md bootstrapping section clarifying that `close` is administrative (pointer removal only) and protocol CLOSE (summary.md, decision audit) should happen first.
- **Recovery protocol missing pointer fallback** — added step 0 to Recovery from Context Loss: if `.current_plan` is missing, use `bootstrap.mjs list` to find plan directories and recreate the pointer.
- **Silent error swallowing in cmdNew cleanup** — added `WARNING:` messages to the three catch blocks in cmdNew's error path. Added explanatory comments to two other intentional empty catches (checkpoints dir, TOCTOU-safe unlink).
- **CLAUDE.md missing build commands** — replaced incomplete 4-command list with all 11 commands for both PowerShell and Make (build, build-combined, package, package-combined, package-tar, validate, lint, test, clean, list, help).
- **Orphaned plan directory warning** — `cmdNew` now detects plan directories with no active pointer and emits a non-blocking warning suggesting `list` to inspect.

## [1.8.0] - 2026-02-18

### Fixed
- **CRITICAL: ensureGitignore failure no longer destroys plan** — `ensureGitignore()` moved outside the plan-creation try/catch. Failure is now a warning, not a rollback. Also cleans up the pointer file on creation failure.
- **make test is no longer a no-op** — replaced `|| true` swallowed exit code with actual round-trip test (new → status → close in temp directory). Help command exit code now checked.
- **SKILL.md Mermaid diagram now has initial/terminal state markers** — added `[*] --> EXPLORE` and `CLOSE --> [*]` to match README diagram.
- **Validation now checks PLAN → PLAN self-transition** — both Makefile and build.ps1 validate all 9 transition table entries (was 8).
- **Validation now checks checkpoints/ and findings/ directory creation** — bootstrap.mjs directory creation verified by both build scripts.
- **RE_PLAN/RE-PLAN validation regex tightened** — `RE.PLAN` (matches anything) → `RE[-_]PLAN` (matches only hyphen or underscore). Later normalized to `REPLAN` in v2.9.2.
- **cmdClose TOCTOU race** — `unlinkSync(pointerFile)` wrapped in try/catch to handle concurrent removal.
- **ensureGitignore now uses atomic write** — temp file + rename, consistent with pointer file write.
- **Empty goal prevented on backward-compat path** — `node bootstrap.mjs ""` now defaults to "No goal specified".
- **Goal extraction regex handles ## Goal as last section** — lookahead changed from `(?=\n## )` to `(?=\n## |$)`.
- **build.ps1 path separator portability** — `Invoke-List` now uses `[IO.Path]::DirectorySeparatorChar` instead of hardcoded backslash.

### Added
- **build.ps1 `test` command** — mirrors Makefile test target with lint + round-trip test.
- **build.ps1 `package-tar` command** — mirrors Makefile package-tar target. Closes parity gap.
- **Combined package bootstrap limitation documented** — combined single-file build now appends a note about missing `bootstrap.mjs`. README Quick Start also notes this.

### Changed
- **Iteration limits clarified** — replaced ambiguous "If iteration > 5 → STOP" with explicit two-tier: iteration 5 = Nuclear Option if bloated, iteration 6+ = unconditional hard stop.

## [1.7.0] - 2026-02-17

### Added
- **`list` subcommand** — `bootstrap.mjs list` shows all plan directories under `.claude/` (active and closed) with state, goal, and active marker. Useful for reviewing plan history. Documented in SKILL.md, CLAUDE.md, and README.md.
- **Findings subagent naming convention** — `findings/{topic-slug}.md` (kebab-case, descriptive). Prevents filename collisions when parallel subagents write simultaneously. Documented in SKILL.md EXPLORE rules and file-formats.md.

### Changed
- **Atomic pointer write** — `bootstrap.mjs` now writes `.current_plan` via temp file + rename, preventing partial pointer on crash between directory creation and pointer write.
- **Multi-line goal support** — `extractField` regex for `## Goal` now captures until the next heading, not just the first line. `resume` and `status` display the first line; full goal preserved in plan.md.
- **Enhanced `validate` target** — Both Makefile and build.ps1 now verify: (1) all `references/` cross-references in SKILL.md resolve to actual files, (2) bootstrap.mjs creates all expected plan directory files, (3) state machine transition pairs appear in SKILL.md.

## [1.6.0] - 2026-02-17

### Added
- **Pre-Step Checklist in state.md** — New `## Pre-Step Checklist` section in state.md, reset before each EXECUTE step. Converts memory-dependent mandatory re-read rules into file-based enforcement: re-read state.md, plan.md, progress.md, decisions.md (if fix), checkpoint (if risky/irreversible). Bootstrap creates it; file-formats.md documents it.
- **Minimum EXPLORE depth** — ≥3 indexed findings required in `findings.md` before EXPLORE → PLAN transition. Findings must cover: problem scope, affected files, existing patterns/constraints. PLAN gate check also enforces this — <3 findings sends you back to EXPLORE.
- **Post-Step Gate failure case clarified** — Gate heading changed from "MANDATORY — all 3" to "successful steps only — all 3". Added explicit line: on failed step, skip gate and follow Autonomy Leash.
- **Irreversible-operation protocol** — Steps with side effects git cannot undo (DB migrations, external API calls, service config, non-tracked file deletion) must be tagged `[IRREVERSIBLE]` in plan.md. Before executing: (1) explicit user confirmation, (2) rollback plan in checkpoint, (3) dry-run if available. Added to SKILL.md EXECUTE rules, file-formats.md plan.md template, and code-hygiene.md as new section.

## [1.5.1] - 2026-02-17

### Fixed
- **Missing state transitions formalized** — Added PLAN → EXPLORE (can't state problem, can't list files, insufficient findings) and PLAN → PLAN (user rejects, revise and re-present) to both Mermaid diagram and transition table. Prose already described these behaviors but the formal spec omitted them.
- **File Lifecycle Matrix CLOSE column corrected** — `findings.md`, `findings/*`, and `progress.md` changed from `—` to `R` during CLOSE. Writing summary.md requires reading these files.
- **`.gitignore` update moved from CLOSE to bootstrap** — `bootstrap.mjs` now idempotently ensures `.claude/.plan_*` and `.claude/.current_plan` patterns in `.gitignore` on plan creation. Prevents plan files from being committed during EXECUTE step commits. Previously this was a manual instruction at CLOSE — by which point plan files may have already been committed.

## [1.5.0] - 2026-02-17

### Changed
- **Checkpoint lifecycle expanded** — File Lifecycle Matrix: REFLECT gains R (read checkpoints to know rollback options before deciding transition)
- **Checkpoint naming encodes iteration** — `cp-NNN.md` → `cp-NNN-iterN.md` (e.g. `cp-001-iter2.md`). NNN increments globally.
- **Checkpoint "Git State" clarified** — explicitly documented as the commit BEFORE changes (the restore point), not after
- **REPLAN keep-vs-revert decision criteria** — keep when steps are valid under new approach + tests pass; revert when fundamentally different approach or commits would conflict; default when unsure = revert to latest checkpoint
- **REFLECT reads checkpoints** — notes available restore points in `decisions.md` when transitioning to REPLAN
- **Autonomy leash includes checkpoints** — on leash hit: revert uncommitted first, present available checkpoints to user
- **3-strike rule specifies rollback** — revert to checkpoint covering the struck area
- **Nuclear option allows later checkpoint** — default is `cp-000` but user may choose a later checkpoint if partial progress is worth keeping
- **Recovery protocol includes checkpoints** — `checkpoints/*` now listed as step 7 (rollback points and git hashes)
- **Git integration REPLAN line expanded** — clarifies keep/revert logic and requires logging choice in `decisions.md`
- **code-hygiene.md REPLAN section** — added decision criteria, "read checkpoints first", default-to-revert guidance
- **complexity-control.md** — 3-strike adds checkpoint rollback step; nuclear option clarifies checkpoint selection
- **file-formats.md checkpoint template** — updated naming, clarified git state semantics, added parenthetical examples for risky change triggers

## [1.4.0] - 2026-02-17

### Changed
- **findings.md lifecycle expanded** — File Lifecycle Matrix updated: REFLECT gains R (read to check contradictions), REPLAN gains R+W (can now correct wrong findings)
- **EXPLORE subagent coordination** — main agent owns `findings.md` index; subagents write only to `findings/`. Correction format: `[CORRECTED iter-N]`
- **PLAN gate check enforced** — "read first" → explicit gate: "If not read → read now. No exceptions."
- **EXECUTE surprise discovery rule** — unexpected findings noted in `state.md`, step finishes or reverts, then transitions to REFLECT. No silent findings updates during EXECUTE.
- **REFLECT reads findings** — explicitly reads `findings.md` + `findings/*` to detect contradictions from EXECUTE. EXPLORE transition now triggers on contradicted findings.
- **REPLAN can correct findings** — if earlier findings proved wrong, update with `[CORRECTED iter-N]` + reason. Append-only (don't delete original text).
- **file-formats.md updated** — findings.md template adds `## Corrections` section and documents index ownership

## [1.3.1] - 2026-02-17

### Fixed
- **Build scripts now include `src/scripts/` in packages** — both `Makefile` and `build.ps1` were globbing for `*.sh` instead of `*.mjs`, causing `bootstrap.mjs` to be missing from release artifacts
- **Lint/test targets updated** — replaced `bash -n src/scripts/bootstrap.sh` with `node --check src/scripts/bootstrap.mjs` in both build scripts
- **Fixed Makefile target conflict** — removed directory rules that shadowed the phony `build` target, eliminating "overriding recipe" warnings

## [1.3.0] - 2026-02-17

### Changed
- **Restructured project to use `src/` directory** — moved `SKILL.md`, `references/`, and `scripts/` into `src/` to separate skill source files from project-level files (README, build scripts, etc.)
  - Updated all cross-references in `Makefile`, `build.ps1`, `README.md`, `CLAUDE.md`, `CHANGELOG.md`
  - Internal relative paths within `src/` (SKILL.md ↔ references/ ↔ scripts/) unchanged
- **README badge**: "Protocol v1.1" → "Skill v1.3.0"; replaced "protocol" wording with "skill"

## [1.2.3] - 2026-02-17

### Changed
- **Unified language style across all agent-facing files** to match SKILL.md's terse, imperative, operator-manual voice
  - `src/references/complexity-control.md`: conversational phrasing → imperative fragments (5 edits)
  - `src/references/code-hygiene.md`: explanatory sentences → compressed directives (7 edits)
  - `src/references/decision-anchoring.md`: narrative intro → arrow-notation style (2 edits)
  - `src/references/file-formats.md`: redundant prose → compressed phrasing (3 edits)
  - `CLAUDE.md`: verbose prose sections → terse fragments (8 sections rewritten)
  - Net result: −12 lines, zero semantic changes

## [1.2.2] - 2026-02-17

### Added
- **Problem Statement requirement in PLAN**: Before designing steps, plan.md must now define expected behavior, invariants (what must always be true), and edge cases. Can't state the problem clearly → back to EXPLORE.
- **Failure Mode Analysis in PLAN**: For each external dependency or integration point, plan.md now requires a Failure Modes table (Slow / Bad Data / Down / Blast Radius). "None identified" if no dependencies.
- **Trade-off framing in decisions.md**: Every decision entry must now state "X at the cost of Y" — never recommend without stating what it costs.
- **Updated file-formats.md templates**: plan.md template includes Problem Statement and Failure Modes sections; decisions.md template includes Trade-off lines with examples across all three sample entries.

## [1.2.1] - 2026-02-17

### Changed
- **Reference files compressed**: 621 → 480 lines (−23%), 3,520 → 2,482 words (−29%)
  - `src/references/complexity-control.md`: −34% lines / −45% words — removed motivational preambles, tightened rule descriptions
  - `src/references/code-hygiene.md`: −30% lines / −34% words — compressed procedure steps, removed redundant explanations
  - `src/references/decision-anchoring.md`: −29% lines / −30% words — tightened trigger list and rules
  - `src/references/file-formats.md`: −14% lines / −17% words — trimmed prose around templates (code blocks preserved)
  - All rules, thresholds, code templates, procedures, and cross-references preserved

## [1.2.0] - 2026-02-17

### Changed
- **src/SKILL.md compressed**: 386 → 244 lines (−37%), 3,007 → 1,697 words (−44%)
  - ASCII state diagram replaced with mermaid `stateDiagram-v2`
  - Per-state prose sections replaced with terse bullet lists
  - Post-Step Gate compressed to 3-line numbered checklist
  - Bootstrapping prose eliminated (code comments suffice)
  - Complexity Control and Autonomy Leash compressed to bold one-liner rules
  - User Interaction section converted to table
  - File Lifecycle Matrix simplified to R/W/— notation
  - YAML frontmatter description shortened to 3 lines
  - All protocol semantics preserved, zero functional changes

## [1.1.0] - 2026-02-14

### Changed
- Plan directory moved from `.plan/` in project root to `.claude/.plan_YYYY-MM-DD_XXXXXXXX/`
  - Dynamic naming with date + 8-char hex seed (e.g. `.plan_2026-02-14_a3f1b2c9`)
  - Only one plan directory allowed at a time
  - Discovery via `.claude/.current_plan` pointer file (contains the plan directory name)
  - Bootstrap writes pointer; protocol reads it to find the active plan
  - `.gitignore` patterns: `.claude/.plan_*` and `.claude/.current_plan`

## [1.0.0] - 2026-02-14

### Added
- **Core Protocol (src/SKILL.md)**: Complete state-machine driven iterative planning and execution protocol
  - EXPLORE: Context gathering with parallel subagent support
  - PLAN: Structured approach design with complexity budgets
  - EXECUTE: Step-by-step implementation with change manifests
  - REFLECT: Result evaluation against written success criteria
  - REPLAN: Evidence-based pivoting with decision logging
  - CLOSE: Summary writing with decision-anchored comment auditing
- **State Machine**: Full transition rules with mandatory re-read protocol
- **Autonomy Leash**: 2-attempt limit per plan step, then STOP and present to user
- **Complexity Control** (`src/references/complexity-control.md`):
  - Revert-First Policy (revert → delete → one-liner → REFLECT)
  - 10-Line Rule (>10 lines = not a fix)
  - 3-Strike Rule (same area breaks 3x = wrong approach)
  - Complexity Budget tracking (files, abstractions, lines)
  - Forbidden Fix Patterns (wrapper cascades, config toggles, exception swallowing, etc.)
  - Nuclear Option (full revert at iteration 5 if bloat > 2x scope)
- **File Formats Reference** (`src/references/file-formats.md`):
  - Templates for state.md, plan.md, decisions.md, findings.md, progress.md
  - Checkpoint and summary file formats
  - Examples for each file type
- **Bootstrap Script** (`src/scripts/bootstrap.mjs`):
  - Initializes `.claude/.plan_YYYY-MM-DD_XXXXXXXX/` directory structure under `.claude/`
  - Creates state.md, plan.md, decisions.md, findings.md, progress.md
  - Writes `.claude/.current_plan` pointer file for plan directory discovery
  - Idempotent-safe (refuses if `.claude/.current_plan` already points to an active plan)
- **Code Hygiene Protocol**:
  - Change manifest tracking in state.md
  - Revert-on-failure with forbidden leftover checks
  - Clean state guarantees between iterations
- **Decision Anchoring**:
  - Code comments referencing decisions.md entries
  - Rules for when to anchor and when not to
  - Format guidelines with decision IDs
- **Git Integration**: Commit conventions (`[iter-N/step-M]`), checkpoint support
- **Recovery Protocol**: Full session recovery from plan directory files
- **Build Scripts**: Makefile (Unix/Linux/macOS) and build.ps1 (Windows)
- **CLAUDE.md**: AI assistant guidance for working with the codebase
- **README.md**: User documentation with install instructions and protocol overview
