# Test Audit Reference

Read [SKILL.md](SKILL.md) first. This file expands the **junk-pattern catalog** (signature, cost, fix for each pattern) and the **sweep workflow** (8-step process for pruning one subsystem).

## Junk Pattern Catalog

Each row expands one SKILL [junk pattern](SKILL.md#junk-patterns). A match fails the authoring gate or is a prime audit target—unless the [retention bar](SKILL.md#retention-bar) names an independent contract it guards.

| Pattern | Signature | Why it is low value | Fix |
|---|---|---|---|
| Assertion-free coverage probe | Calls the code but asserts nothing, or only that it "did not throw". | Raises coverage while passing even when the output is wrong. | Assert the observable result, or delete. |
| Self-comparison / identity copier | Asserts a value equals itself, or re-reads the variable it just set. | Cannot fail; proves nothing. | Assert against an independently computed expectation, or delete. |
| Copied fixture, inventory, or export list | Test data is a copy of the production list, manifest, or export set it checks. | Restates source; two copies must be edited together and can never disagree. | Assert a property of the list, or delete. |
| Exact source / import / string grep | Greps production files for a literal identifier, import, or code shape. | Breaks on identifier-only refactors; asserts implementation text, not behavior. | Exercise the behavior. Keep only when source inspection is the cheapest guard of a user-facing key, byte, or path. |
| Private predicate or call-shape test | Tests a private helper, or that an internal method was called, when a public boundary already covers it. | Couples to internals and duplicates the boundary's proof. | Assert at the public boundary; drop the internal test. |
| Duplicate contract invocation | Several tests exercise the same contract with the same risk. | Extra maintenance, no extra proof. | Consolidate into one table-driven case at the owner. |
| Provider-local replay of a shared helper | A provider or package re-tests a shared helper it merely calls. | The helper's owner already proves it. | Delete; keep one generic contract at the helper's owner. |
| Seam-preserving test | The only reason a production export, flag, or wrapper exists is this test. | Keeps a seam no production caller needs. | Move the test to the real boundary and delete the seam. |
| Test-only dead code | A production path is reached only from tests. | Tests keep dead code alive. | Delete the code and its tests together. |
| Self-derived expectation | The expected value is produced by the same helper or renderer under test. | Tautology; expectation and code move together. | Hard-code an independently derived expectation. |
| Behavior-implementing / shared mock | The mock contains the logic the test claims to verify, or one identical mock stands in for several distinct APIs. | Proves the mock, not production; hides real contract differences. | Assert against the real boundary, or a faithful fake per API. |
| Owner-supplied fixture / wrong-store assertion | The test pre-supplies the receipt, admission, or ordering the owner should produce, or checks a store the path never writes. | Passes without the owner doing its job. | Let the owner produce the value; assert the store the path actually writes. |
| Flag-restating capability test | Asserts a declared flag or capability value instead of the delivery it promises. | Restates config; the promised behavior stays unproven. | Exercise the delivery or acknowledgement the flag gates. |
| False negative control | A "rejects X" test passes because a different guard denies, or via a path production never reaches. | Green for the wrong reason; would miss the real regression. | Route the denial through the owning guard and assert the specific reason. |
| Over-promising name or fixture | The name claims a behavior the input never triggers (e.g. "retires the window" while asserting it was *not* cleared). | False confidence; the test is judged by its name, not its assertion. | Make the input exercise the promised behavior, or rename to what it proves. |

## Sweep Mode: Pruning a Subsystem's Test Surface

One PR, one subsystem (plugin, extension, module, package, or core area). The value bar, retention bar, candidate evidence, and validation in [SKILL.md](SKILL.md) apply throughout. This section orders the 8-step workflow and completion criteria for each. Do not start the next step early.

## 1. Baseline

Pin a `main` SHA. Record test/support LOC and every test file's pass/fail state. Separate baseline failures: they're stale infrastructure or delivery bugs, not stale tests.

**Done:** Every in-scope test file has a recorded baseline result.

## 2. Lanes and Inventory

Split into **lanes** by production owner boundary, not file prefix. Example (payment subsystem): accounts, balances, transactions, settlements, webhooks, auth, persistence, transport, shared, QA/integration.

**Done:** Every test file and QA scenario belongs to exactly one lane.

## 3. Read-Only Ledger Per Lane

Read every test in full (params, fixtures, owners, callers, history, CI routing). Write a **ledger** for each declaration; mark each row:

- `R`: Retain (name contract + bug it catches); move-only stays `R`
- `F`: Fix assertion (repair vacuous negatives, etc.)
- `C`: Consolidate (name absorbing owner)
- `D`: Delete (name remaining proof or why no contract exists)

Table rows (`it.each`, `@pytest.mark.parametrize`) = one mark unless rows differ, then mark each row. Judge assertions, not names.

**Done:** Every declaration has a mark and evidence line.

## 4. Layer Plan Per Lane

Ledger is input, not the edit list. Second read-only pass: find redundant **layers**. Example: multiple transaction suites replay the same serialization through a mocked persistence, while stronger real-database and transport-fixture suites exist elsewhere. Name the **keeper** per contract; prefer real transport boundaries over mocked collaborators.

Fix ledger errors found.

**Done:** Each lane plan names retired files, keepers per contract, carried assertions, and unlocked test-only production seams.

## 5. Cutover

Edit lane by lane. Serialize shared harness changes through one owner. Per lane:

1. Apply deletions and consolidations from the plan
2. Remove test-only seams: injection params, getters, reset exports, indirection layers
3. Register moved suites in CI and test inventories
4. Run keepers, confirm all pass
5. Update baselines and line-cap rules if needed

Codify test-ownership rules in subsystem docs (drawn from real mistakes found).

**Done:** Every lane plan applied; all keepers pass.

## 6. Preservation Review

Independent reviewers compare deleted coverage vs. keepers (one reviewer per boundary group). Look for contracts that lost their only proof and assertions that cannot fail (e.g., rejection rows the code never reaches).

For each restored contract, **mutate** the owner and confirm the keeper fails. Restore byte-for-byte.

**Done:** All gaps restored or rejected with evidence; all restored contracts have a caught mutation.

## 7. Product Defects

Baseline failure surviving into a keeper = bug. Fix at owner (separate commit), prove via real user flow with a **control** run (revert fix, show old behavior). Log unrelated discrepancies as follow-ups.

**Done:** Each repaired defect has a failing control and passing candidate on the same harness.

## 8. Reconcile and Hand Off

Merge `main` (don't rebase). If `main` modified a file the sweep deleted, keep the deletion and port the new contract into the keeper. Confirm every new regression `main` added still has a keeper. Rerun full subsystem suite on merged head.

Hand off per [SKILL.md](SKILL.md) format, plus:
- Baseline and final test/support LOC (production separate)
- Lanes, retired layers, keepers
- Preservation gaps and their mutations
- Product defects with control and proof
