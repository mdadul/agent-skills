---
name: doc-drift
description: >-
  Detect function-level mismatches between documentation and code. Report over-promises and direct mismatches, and leave under-promises out of the report. Use when the user asks to check docstrings, comments, JSDoc, Javadoc, or other function docs against the implementation — including
  stale comments, doc drift, wrong @param or @returns, and "does this comment match the code" — on a selection, a file, or a project. On a large codebase, fan the check out to several low-cost subagents.
---

# Doc Drift

Check function-level documentation against the implementation. Report incorrectness only: a documented claim the code does not implement, or a claim the code contradicts. Leave out anything that is merely undocumented.

Models treat "the comment is shorter than the code" as a bug. The check gives that observation its own answer, then deletes that answer. Both steps matter. Skipping the undocumented-code question, or telling the checker not to mention it, pushes those notes into the real findings.

Keep the JSON key names. They are the procedure.

Do not edit the code or the comments unless the user asks.

## What the three answers mean

- **Over-promise.** The documentation promises a behavior the function never performs.
- **Direct mismatch.** The documentation and the code describe the same behavior and disagree about it.
- **Under-promise.** The function does extra work the comment never mentions, and that extra work still agrees with the comment. A comment is supposed to be shorter than the code.

## Scope

Run on whichever the user gives:

- **Selected code:** documented functions inside the selection.
- **File:** documented functions in that file.
- **Project:** documented functions in project source. Skip dependencies, generated files, lockfiles, and vendored trees.

Use the docstring, JSDoc, Javadoc, or block comment attached to the function, including parameter and return tags in that block. Skip functions with no attached documentation.

## Step 1 — Categorize one function at a time

For each documented function, fill one JSON object. Complete the keys in order. Do this before you decide what to show the user.

```json
{
  "function": "name",
  "file": "path",
  "Documentation_Summary": "...",
  "Code_Summary": "...",
  "Is_any_core_part_of_the_documentation_not_implemented_in_the_code?": "Yes or No",
  "If_yes_to_Is_any_core_part_of_the_documentation_not_implemented_in_the_code": [],
  "Does_the_code_correctly_implement_what_is_mentioned_in_the_documentation?": "Yes or No",
  "If_no_to_Does_the_code_correctly_implement_what_is_mentioned_in_the_documentation": [],
  "Is_some_code_not_documented_or_mentioned_in_the_documentation?": "Yes or No",
  "If_yes_to_Is_some_code_not_documented_or_mentioned_in_the_documentation": []
}
```

Summaries are one or two sentences. Check-in values are exactly `Yes` or `No`.

Follow-up arrays use these objects, and only when the check-in says there is something to explain:

- Over-promise, only if the check-in is `Yes`: `{"original_documentation_snippet_that_is_not_implemented_in_the_code": "...", "explanation": "..."}`
- Direct mismatch, only if the check-in is `No`: `{"original_documentation_snippet_that_has_conflicting_information_with_some_code_snippet": "...", "original_code_snippet_that_has_conflicting_information_with_the_identified_documentation_snippet": "...", "explanation": "..."}`
- Under-promise, only if the check-in is `Yes`: `{"original_code_snippet_that_is_not_documented_or_mentioned_in_the_documentation": "...", "explanation": "..."}`

Quote the original documentation and code. An empty array means that category is clean. If a check-in and its follow-up disagree, or the object is not valid JSON, record no finding for that function.

While you fill the first two check-ins:

- Behavior implemented in a callee is implemented. If the body only delegates, read that callee when it is in the repo before calling the claim missing.
- A plain realization of the words is consistent. Setting a handle to `undefined` can be how the code expires it.
- A note about when to call the function, or what to call it with, is context. It is a missing feature only when the body contradicts it.
- Extra behavior the comment omits belongs in the under-promise array, not in the other two.

## Step 2 — Filter after the JSON exists

Delete the under-promise check-in and its follow-up. Do not bring them back into the report.

Keep a finding only when:

- the over-promise check-in is `Yes` and its follow-up is non-empty, or
- the direct-mismatch check-in is `No` and its follow-up is non-empty.

If both are clean, say nothing about that function.

## Who checks it

- A selection, one file, or up to 5 source files: do both steps yourself.
- A project, or more than 5 source files: do not categorize them yourself when you can fan out. Split the work, then do step 2 on what comes back.

If the host cannot run subagents, still categorize in batches of about 8 files, one function at a time, and filter only after each batch's JSON exists.

## Fan-out

1. List the source files in scope that can contain documented functions.
2. Split that list into batches of about 8 files.
3. Launch every batch together so they run at the same time. Use whatever subagent mechanism the host provides.
4. Run each batch on a low-cost model the host actually lists. Prefer a name containing haiku, then mini, then flash, then luna. Do not invent an id. If none of those are listed, use the host's default.
5. Paste step 1, including the JSON shape and the filling rules, plus that batch's file paths. The subagent cannot see this skill. Tell it to return the JSON objects and not to filter.
6. You do step 2. Merge by file, drop duplicates, and present one report. Do not re-check a batch that is clean after filtering.

## Report

Group findings by file. For each finding:

- **function:** name and location
- **kind:** over-promise or direct mismatch
- **documentation:** the quoted snippet
- **code:** the quoted snippet when it is a direct mismatch
- **why:** one or two sentences

If nothing remains after filtering, say the checked functions have no over-promises or direct mismatches. Do not list dropped under-promises.

## Examples

### Over-promise (keep)

```python
def set_test_mode(v: bool = True) -> None:
    """Keeps track of whether numexpr was used. Stores an additional
    True for every successful use of evaluate with numexpr since the
    last get_test_result."""
    global _TEST_MODE, _TEST_RESULT
    _TEST_MODE = v
    _TEST_RESULT = []
```

The comment says a True is stored for every successful numexpr use. The body only sets the flag and clears the list. Over-promise check-in: `Yes`.

### Direct mismatch (keep)

```typescript
/**
 * Returns the count of root collections for a teamID.
 * The count returned is highest OrderIndex + 1.
 */
private async getRootCollectionsCount(teamID: string) {
  return rootCollectionCount[0].orderIndex;
}
```

The comment says highest OrderIndex plus 1. The code returns `orderIndex` unchanged. Direct-mismatch check-in: `No`.

### Under-promise (answer, then delete)

```cpp
/** Return the floating point value of the given index into the list. */
float SORTED_FLOATS::operator[](int32_t index) {
  it.move_to_first();
  return it.data_relative(index)->entry;
}
```

The comment never mentions moving the iterator first. That goes in the under-promise array. Step 2 deletes it. The other two check-ins stay clean.
