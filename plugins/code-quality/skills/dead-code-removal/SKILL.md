---
name: dead-code-removal
description: Finds dead code in any-language project (unused exports, files, dependencies, unreachable code, stale feature flags, commented-out code, unused endpoints, config keys, DB columns) using static-analysis tools plus grep/usage verification, then produces a risk-ranked, phased plan for safe removal without editing code. Use when the user asks to find dead code, unused code, orphaned files or dependencies, clean up a codebase, or plan safe deletion of legacy code.
---

# Dead Code Removal

## Purpose
Produce an evidence-backed dead-code report and a phased, reversible removal plan. This skill analyzes and plans only. Do not delete or edit source files unless the user separately asks.

## When To Use
- "find dead / unused code", "what can we delete", "clean up unused dependencies"
- Auditing a repo, package, or folder before a refactor or migration
- Planning removal of stale feature flags, endpoints, config keys, or DB columns

## Inputs
- Required: target scope (repo root, package, or folder). Default: the whole workspace.
- Optional: languages, entry points, known public API surface, deploy/runtime constraints.

## Workflow
Copy this checklist and track progress:

```
- [ ] 1. Map scope and entry points
- [ ] 2. Detect candidates (tools)
- [ ] 3. Verify each candidate (usage + dynamic-use checks)
- [ ] 4. Classify confidence and risk
- [ ] 5. Write report
- [ ] 6. Write phased removal plan
```

### 1. Map scope and entry points
- Identify languages and build files (`package.json`, `composer.json`, `go.mod`, `pyproject.toml`, `pom.xml`, `Cargo.toml`, etc.).
- List entry points: `main`/`bin`/`exports` fields, route registrations, CLI commands, cron/queue/Kafka consumers, DI module roots, test setup files, build/config files.
- Note public API surfaces (published packages, HTTP/gRPC contracts, event schemas). Anything externally consumed is never "dead" from this repo alone.

### 2. Detect candidates
Prefer an installed or `npx`-runnable tool; fall back to LSP find-usages and grep. Run tools in read-only/report mode. Never use auto-fix flags (`--fix`, `--write`) during detection.

| Ecosystem | Unused code / files / exports | Unused dependencies |
|-----------|-------------------------------|---------------------|
| TS/JS | `knip`, `ts-prune`, `unimported`, ESLint `no-unused-vars` | `knip`, `depcheck` |
| Python | `vulture`, `ruff --select F401,F841` | `deptry` |
| Go | `staticcheck` (U1000), `deadcode`, `go vet` | `go mod tidy -diff` |
| PHP | `phpstan` (level 4+), `psalm --find-unused-code` | `composer-unused` |
| Java/Kotlin | IDE inspections, `detekt`, `pmd` unused rules | `mvn dependency:analyze` |
| Rust | compiler `dead_code` warnings, `cargo udeps` | `cargo udeps` |
| Other | LSP find-references + grep | manifest vs import scan |

Also scan for non-tool categories:
- Commented-out code blocks (grep for commented statements, `TODO remove`, `deprecated`, `legacy`, `unused`).
- Feature flags: list flag keys, then check whether each is still read, is fully on/off in all environments, or has no remaining call sites.
- Config keys and env vars: defined but never read.
- Endpoints/routes/handlers: registered but with no callers in the repo, logs, or API gateway metrics (if accessible).
- DB columns/tables/indexes: referenced in migrations or models but never selected/written in code.

### 3. Verify each candidate
A tool hit is a candidate, not a fact. For each one:
1. Search for the symbol name across the repo, including tests, docs, configs, scripts, CI, IaC, and templates.
2. Check dynamic-use patterns that static tools miss:
   - reflection, string-based lookup, `require(variable)`, `getattr`, `Class::forName`
   - DI containers, decorators/annotations, auto-discovery by file path or naming convention
   - serialization/ORM field mapping, template/view references, route files loaded by glob
   - CLI/cron/queue/Kafka consumer registration by name or topic
   - framework hooks (lifecycle methods, migrations, `__magic__` methods, exported for plugins)
3. Check external consumers: published package exports, public HTTP/gRPC/event contracts, other repos (search org code if available), mobile clients.
4. Check history: `git log -S <symbol>` / `git blame` for recent additions or in-flight work; check open branches/PRs when possible.
5. Record the evidence (commands run, results) per candidate.

### 4. Classify confidence and risk

| Confidence | Meaning |
|------------|---------|
| High | Tool + grep agree, no dynamic-use pattern, internal-only, no recent activity |
| Medium | Tool flags it, but dynamic/framework use is plausible or evidence is partial |
| Low | Possibly externally consumed, runtime-only signal needed (logs, metrics), or data-bearing (DB, flags) |

| Risk | Examples |
|------|----------|
| Low | Unused local variables, private functions, commented-out code, unreferenced internal files |
| Medium | Unused exports, unused dependencies, unused config keys |
| High | Public API/endpoints, event consumers/producers, feature flags in prod, DB columns/tables, anything with a rollback cost beyond `git revert` |

Never list Low-confidence items as "safe to delete". Route them to a verification step in the plan.

### 5. Write the report
Use the Output Format below. Report only verified candidates and clearly mark unverified ones. Do not pad with style nits.

### 6. Write the phased removal plan
Order by risk ascending. Each phase must be independently shippable and revertible.

| Phase | Content | Gate to proceed |
|-------|---------|-----------------|
| 0. Baseline | Record green build, tests, lint, type-check, and tool output snapshot | All pass before any change |
| 1. Trivial | Commented-out code, unused locals/imports/private symbols | Build + tests + lint green |
| 2. Internal | Unreferenced files, unused internal exports, unused dependencies | Build + tests + bundle/install check green |
| 3. Config & flags | Dead flags (remove flag branch that lost), unused config/env keys | Flag confirmed 100% one-way in all envs; deploy config updated in same PR |
| 4. Externally visible | Endpoints, events, public exports | Deprecate first: add logging/metrics, wait an agreed observation window, notify consumers, then remove |
| 5. Data | DB columns/tables/indexes | Stop writes -> stop reads -> observe -> backup/archive -> drop in separate migration |

Plan rules:
- One category per PR; small, reviewable, no mixed refactors.
- Each PR: what is removed, evidence link, verification commands, rollback (usually `git revert`).
- Remove dependencies in their own PR and run install + build + tests + lockfile diff review.
- After each phase, rerun the detection tools; removals can expose new dead code (iterate until stable).
- Where runtime evidence is needed, specify the exact signal (log query, metric, access log) and observation window; do not invent a number, ask the user or propose a default and label it as an assumption.

## Output Format

```markdown
# Dead Code Report: <scope>

## Summary
- Scope, languages, tools run (with versions if known), tools unavailable
- Counts by category and confidence

## Candidates
| # | Category | Location | Evidence | Confidence | Risk | Recommended action |
|---|----------|----------|----------|------------|------|--------------------|

## Not Safe To Remove (needs verification)
- <item>: <why: dynamic use / external consumer / runtime data needed> -> <how to verify>

## Removal Plan
### Phase 0 ... Phase N
For each phase: items, PR split, verification commands, gate, rollback.

## Assumptions and Gaps
- <anything not checkable from the repo>
```

## Validation
Before handing off:
- Every "High confidence" item has recorded grep/tool evidence and was checked against the dynamic-use list.
- No item touching public API, events, flags in prod, or DB is placed before Phase 3.
- No source file was modified during analysis.
- Each phase has a gate and a rollback step.
- Gaps (tools not installed, no runtime metrics access) are stated, not guessed.

If a check fails: fix the report, re-verify, then proceed. Only present the plan when all checks pass.

## Examples
Candidate row:

```
| 4 | Unused export | src/utils/date.ts:`formatLegacyDate` | knip: unused export; grep: 0 refs (src, tests, docs); not in package `exports`; last touched 2023-02 | High | Medium | Phase 2: delete with its test |
```

Not-safe row:

```
- `FEATURE_NEW_CHECKOUT` flag: code path is always on in repo config, but prod value lives in the flag service -> confirm 100% rollout for the agreed window, then remove the flag and the old branch (Phase 3).
```
