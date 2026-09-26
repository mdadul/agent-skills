---
name: doc-drift
description: >-
  Check function docs against the implementation and report only claims the
  code misses or contradicts. Use when the user asks whether a function
  comment matches the code — stale comments, doc drift, a wrong parameter
  or return note — on a selection, a file, or a named folder.
  Scan a whole project only when the user explicitly asks. Leave
  under-promises out of the report.
---

# Doc Drift

A cheap, high-precision contract check. Report a documented claim the function never performs, or a claim the code contradicts. Leave out anything that is merely undocumented.

The comment is shorter than the code on purpose. Give that observation its own answer, then delete it. Skipping the undocumented-code question pushes those notes into the real findings.

Do not edit the code or the comments unless the user asks.

A good pass can say: only functions with attached docs were opened, every kept finding was opened in source by you, callees were read before calling something missing, and a comment that already admits the gap was not reported.

## What the three answers mean

- **Over-promise.** The documentation promises a specific behavior the function never performs.
- **Direct mismatch.** The documentation and the code describe the same behavior and disagree about it.
- **Under-promise.** The function does extra work the comment never mentions, and that extra work still agrees with the comment.

## What counts as a claim

A claim names a concrete behavior a caller could rely on: a returned value, a count, a stored flag, a formula. A vague verb with no concrete behavior is not a claim.

- Extra metrics, retries, logging, and optional filters are under-promises.
- If the same block already says the body is a stub, temporary, or intentionally incomplete, the comment is admitting the gap. That roadmap is not drift.
- One sentence is one kind. When it could be either an over-promise or a mismatch, keep the mismatch. Over-promise means the behavior is absent. Mismatch means the code does that thing differently.

## Scope

Start small. Widen only when the user asks.

| Mode | When | What to scan |
|------|------|----------------|
| Selection / file | The user has a buffer or a range, or names a file | Documented functions there |
| Module | The user names a folder | That folder, plus in-repo callees of those functions |
| Repo | The user explicitly says whole project | Exported functions and exported class methods only, then stop |

Do not start by scanning the whole source tree.

On a repo pass, the doc block must sit immediately above the signature. Private helpers, data fields, and tests are out unless the user opts in.

Open a file only when a doc comment is immediately followed by a function or method. A comment on a type, a constant, or a field is not a function doc.

Honor paths and ignore rules the user names. Skip tests, dependencies, generated files, and vendored trees.

Skip functions with no attached documentation.

## Step 1 — Categorize one function at a time

Fill one object per documented function, keys in order, before you decide what to show. Short keys, so a weak model can finish them.

```json
{
  "fn": "name",
  "file": "path",
  "line": 1,
  "doc": "one or two sentences",
  "code": "one or two sentences",
  "over_promise": "Yes or No",
  "over_promise_why": [],
  "implements": "Yes or No",
  "mismatch_why": [],
  "under_promise": "Yes or No",
  "under_promise_why": []
}
```

Questions, in order:

1. `doc` — what the comment claims.
2. `code` — what the function does.
3. `over_promise` — is a specific claim absent from this function? `Yes` or `No`.
4. `implements` — does the code do what the documentation says? `Yes` or `No`.
5. `under_promise` — is some code undocumented? `Yes` or `No`. Answer this even though it will be deleted.

Follow-ups only when the check-in says there is something to explain. Otherwise `[]`.

- `over_promise` is `Yes`: `{"doc": "quoted claim", "why": "what is missing"}`
- `implements` is `No`: `{"doc": "quoted claim", "code": "quoted lines", "why": "the conflict"}`
- `under_promise` is `Yes`: `{"code": "quoted lines", "why": "undocumented extra"}`

Quote the original text. If a check-in and its array disagree, drop that function. Do not invent a finding from broken JSON.

Before `over_promise` is `Yes`, open the in-repo callee, one hop. Behavior implemented there is implemented. A callee outside the repo is not evidence the claim is missing.

A batch returns this envelope, not a bare list:

```json
{
  "files_opened": ["path"],
  "documented_fn_count": 0,
  "objects": []
}
```

`objects` holds the per-function records. An empty list with no `files_opened` and no `documented_fn_count` means the batch was skipped. That is not a clean result.

## Step 2 — Filter, then open the source

Delete `under_promise` and `under_promise_why`. Do not bring them back.

Keep a finding only when:

- `over_promise` is `Yes` and `over_promise_why` is non-empty, or
- `implements` is `No` and `mismatch_why` is non-empty.

If both are clean, say nothing about that function.

You report a finding only after you open that file at that line and the quote is there. If you cannot find the quote, drop the finding. This check is yours, including findings a subagent returned.

A skipped batch is redone, by you or in a later wave. Do not call it clean.

## Who checks it

- A selection or one file: do both steps yourself.
- One module: do it yourself when it is a handful of documented functions. Otherwise one subagent for that module.
- Several modules, or an explicit repo pass: fan out by module.

If the host cannot run subagents, do the modules yourself, one module at a time, and filter only after that module's JSON exists.

## Fan-out

Batch by module, not by a fixed file count. Callees live next to the function. One batch is one module.

1. List modules in scope that contain a block comment immediately above a function or method.
2. Launch 4 to 6 modules at a time. Wait, merge, then start the next wave.
3. Use the cheapest model the host actually lists. If you cannot choose, use the host's default. Do not invent a model id.
4. Paste the three answers, the claim rules, Step 1, and that module's paths. The subagent cannot see this skill. It returns the envelope and does not filter.
5. You do Step 2, including opening every kept finding in source. Merge by file. Drop duplicates. Do not re-check a module that is clean after you have verified it.

## Report

Write a review, not the JSON. One finding per claim. Group by file.

- **function:** name and line
- **kind:** over-promise or direct mismatch
- **documentation:** the quoted claim
- **code:** the quoted lines, and only for a mismatch
- **why:** one sentence

If nothing remains, say the checked functions have no over-promises or direct mismatches. Do not list under-promises. Do not list files you did not open.

## Examples

### Over-promise (keep)

The comment says a success flag is stored for every use. The body only sets a mode flag and clears a list. `over_promise`: `Yes`.

### Direct mismatch (keep)

The comment says the result is the highest index plus one. The code returns the index unchanged. `implements`: `No`. That sentence is a mismatch, not also an over-promise.

### Under-promise (answer, then delete)

The comment says to return the value at an index. The body also resets a cursor first. That goes in `under_promise_why`. Step 2 deletes it.

### Not drift

A comment that already says the body is temporary or intentionally incomplete is telling you the gap. Do not report it. A comment that only names a vague action, with no concrete behavior, is not a claim.
