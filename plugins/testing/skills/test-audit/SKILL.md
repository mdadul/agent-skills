---
name: test-audit
description: >
  Review tests before adding, audit existing ones for junk patterns, or prune a whole subsystem's test surface.
  Use when a test feels brittle, redundant, or tightly coupled to implementation; when asked to judge whether a test is worth keeping; or to sweep a module/plugin and delete low-value tests or test-only production seams.
---

# Test Audit

Three modes: **authoring gate** (vet new/changed tests before landing), **audit** (hunt junk patterns in focused test batches), **sweep** (prune one subsystem's test surface in one PR). All use the same value bar: tests must independently protect observable behavior, not duplicate proof or couple to implementation.

Optimize for **confidence in test value**, not deletion count. When audits get broad, continue as separate PRs.

## Authoring Gate

Before adding any test, answer these four questions. A missing answer = do not add it yet:

1. **What observable behavior, invariant, or independent contract does it protect?**
2. **What credible regression makes it fail?**
3. **Why existing coverage doesn't catch that failure.** Each contract has one strongest-boundary owner; other layers test only distinct risks (transport, lifecycle, ordering). Consolidate setup; prefer extending a table-driven case over near-duplicates.
4. **Does it need a production seam (export, flag, wrapper, injection) no real caller uses?** Yes → move to the real boundary instead.

Then match against every [junk pattern](#junk-patterns). A match fails the gate unless the [retention bar](#retention-bar) guards an independent contract.

**Regression test rule:** Code that breaks on behavior-preserving refactoring is testing implementation, not behavior. Rewrite at the owning boundary. For bug regressions: must fail on pre-fix code (for the intended reason) and pass after the owner-boundary fix. One regression per bug at its owner; do not replay across layers.

## Junk Patterns

Checklist for authoring gate and audits. The gate rejects a new test matching one; audits hunt for existing ones. See [REFERENCE.md](REFERENCE.md#junk-pattern-catalog) for signature, cost, and fix for each.

- Assertion-free probes (calls code, asserts nothing or "didn't throw")
- Self-comparisons (asserts value == itself, or re-reads what it just set)
- Copied fixtures or export lists (test data mirrors the source it checks)
- Exact source/import greps (breaks on rename; tests implementation text, not behavior)
- Private tests duplicating real boundaries (asserts internals the public boundary already covers)
- Duplicate contract invocations (multiple tests exercise the same contract identically)
- Provider-local helper replays (re-tests a shared helper the provider merely calls)
- Seam-preserving tests (only reason an export/flag/wrapper exists is this test)
- Test-only dead code (production path reachable only from tests)
- Self-derived expectations (expected value produced by code under test)
- Behavior-implementing or shared mocks (mock contains the logic it claims to verify)
- Owner-supplied fixtures or wrong-store assertions (pre-supplies what owner should produce)
- Flag-restating tests (asserts declared capability, not the delivery it gates)
- False negative controls (test passes because a different guard denies, not the one under test)
- Over-promising names/fixtures (name claims behavior the input never triggers)

## Value Bar

A test earns its maintenance cost by protecting behavior, catching a credible regression, or guarding an independent contract. A test that breaks on behavior-preserving refactoring is implementation-coupled and fails the gate (authoring) or audit (existing).

Before judging: read the complete test, production owner (entry, callers, callees, siblings), overlapping tests, CI routing, and history. For dependency-backed behavior claims, inspect the dependency source directly.

## Discovery

Keep discovery read-only. Report evidence before editing. For broad scope, run parallel lanes:

- Core, packages
- Plugins, extensions
- UI, apps, scripts, tooling
- Cross-cutting patterns

Outside sweep mode, prioritize high-confidence candidates over large speculative lists. Hunt [junk patterns](#junk-patterns).

## Retention Bar

Keep a test that independently guards a public API, SDK, protocol, config, migration, storage, security, platform, default, or architecture contract. Also keep:

- **Call ordering** when order is observable behavior
- **Regressions** with a credible failure mode
- **Source inspection** when it's the cheapest guard: fails on contract change (user-facing key, byte, path), survives refactoring
- **Failing baseline tests**: treat as possible product bugs; repair the owner, don't delete

Speed and static analysis are not deletion reasons. Prove a test is implementation-coupled, don't assume.

## Candidate Evidence

Before editing, record every field. Missing field = candidate not ready for deletion:

- Test name and exact location
- Actual detectable failure (not its name; read the assertions)
- Non-test callers of covered production paths or test seams
- Remaining owner-boundary proof (or evidence none is needed)
- History: why the test/seam exists (git blame, commit msgs)
- Unlocked simplifications (deleted exports, flags, wrappers, dead code)
- Risk assessment + focused validation command

## Edit Shape

Choose one coherent owner-boundary batch. Delete obsolete test-only exports, globals, wrappers, dead code—don't preserve aliases. Move retained regressions to their owners. Consolidate repeated package/dependency assertions into one contract.

Prefer net-negative production LOC. Don't add replacement tests or pad deletion counts with uncertain candidates.

## Validation Workflow

Never edit source or tests while your suite runs.

1. **Run owner + sibling tests** via your suite's module filter (`npm test src/module`, `pytest -k module`, `go test ./module`).
2. **For removed greps or contract assertions**, run the real contract's executable or integration test.
3. **Run linting, type checks**, then diff the tree (production vs. test, core vs. extension).
4. **Count impact**: LOC deleted (production separately from test), test count before/after, deleted exports/flags/wrappers.
5. **Final run**: full subsystem suite + integration tests on your project's CI command.

## Landing

Land one coherent PR at a time. After landing, refresh from main and restart read-only discovery for the next batch.

## Handoff

After audit or sweep, summarize:
- Root cause and removed low-value categories
- Production owner simplifications unlocked
- Retained false positives (and why they matter)
- Proof actually run: test counts, coverage impact
- Production vs. test LOC impact
- PR merge state and follow-ups
