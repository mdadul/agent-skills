---
name: create-skills
description: Create and maintain custom Skills with correct SKILL.md metadata, strong trigger-aware descriptions, concise instructions, progressive disclosure, and validation loops. Use when the user asks to write, improve, review, or organize a Skill.
---

# Create Skills

## Purpose
Build production-ready custom Skills that AI agents can discover and apply reliably across real tasks.

## When To Use
Use this skill when the user asks to:
- create a new Skill
- improve an existing Skill
- review a SKILL.md for quality or compliance
- scaffold supporting files for a Skill
- tune a Skill for better activation/discovery
- add reference files or utility scripts

## Skill Authoring Workflow
Follow these phases in order.

### 1) Mandatory requirements clarification
Before creating or editing any Skill files, ask targeted clarifying questions and wait for answers.

Minimum question set:
- What is the exact end goal this skill must achieve?
- Which scope should be used (project-level or user-level)?
- What inputs will users provide, and what outputs are expected?
- Any constraints (tools, security, style, runtime, file layout)?
- Should references/examples/scripts be included?

Do not draft final skill content until these answers are confirmed.

### 2) Define scope
Capture:
- the exact task domain the new Skill covers
- where it should live (use your platform's project-level and user-level skill directories)
- trigger scenarios (keywords and user intents)
- expected outputs and constraints

If details are still missing, ask concise follow-up questions before writing files.

### 3) Design metadata first
Create YAML frontmatter before writing instructions.

Requirements:
- `name`: lowercase letters, numbers, hyphens; max 64 chars
- `description`: non-empty, max 1024 chars, third person, includes both:
  - what the Skill does
  - when to use it (trigger language)

Good description pattern:
"<capabilities>. Use when <user intent/keywords>."

Naming guidance:
- prefer clear gerund names (for example: `writing-documentation`, `testing-code`)
- avoid vague names like `helper`, `utils`, `tools`
- avoid vendor names in `name` unless the skill is intentionally vendor-specific

### 4) Set the right degree of freedom
Match specificity to task fragility:
- high freedom: heuristics and principles (multiple valid approaches)
- medium freedom: templates/pseudocode with configurable parameters
- low freedom: exact commands/scripts for fragile or high-risk operations

Default to one recommended path. Add alternatives only when needed.

### 5) Write concise SKILL.md instructions
Keep `SKILL.md` focused and operational:
- short sections
- explicit steps
- concrete defaults with limited alternatives
- examples only when they improve output consistency

Prefer under 500 lines for fast loading and maintenance.

### 6) Add progressive disclosure
Put essentials in `SKILL.md`, move details to linked files:
- `REFERENCE.md` for deep API/reference detail
- `EXAMPLES.md` for many examples
- `scripts/*` for deterministic repeatable operations

Reference files one level deep from `SKILL.md`.

For longer reference files, include a compact table of contents near the top.

### 7) Build workflows and feedback loops
For complex tasks, include a checklist-style workflow and explicit validation cycle:
- plan
- validate
- execute
- verify

For quality-critical operations, require loop behavior:
"run validator -> fix issues -> rerun validator -> proceed only when clean."

### 8) Evaluation-first iteration
Before large documentation expansion:
1. identify real failures without the skill
2. write at least 3 evaluation scenarios
3. capture baseline behavior
4. add minimal instructions to close the gap
5. retest and iterate

Test with the model families you intend to use.

### 9) Validate before handoff
Check:
- metadata constraints pass
- description clearly states WHAT + WHEN
- terminology is consistent
- paths are POSIX style (`/`)
- no contradictory instructions
- output format is explicit where needed
- references are one level deep
- no unnecessary verbosity
- no stale time-sensitive guidance

## Output Contract For Skill Creation
When asked to create a Skill, produce:
1. Proposed `name`
2. Proposed `description`
3. Directory layout
4. `SKILL.md` contents
5. Any optional helper files (only if useful)
6. A concise verification checklist tailored to that skill

Before this output, include a "Requirements Confirmed" section summarizing the user's answers to clarification questions.

Then implement by creating files in the requested scope.

## Reusable SKILL.md Template
Use this template when generating new Skills:

```markdown
---
name: <skill-name>
description: <what it does>. Use when <when to trigger>.
---

# <Skill Title>

## Purpose
<1-2 lines about intended outcome>

## When To Use
- <trigger 1>
- <trigger 2>

## Inputs
- <required input>
- <optional input>

## Workflow
1. <step 1>
2. <step 2>
3. <step 3>

## Output Format
<exact expected structure, checklist, or schema>

## Validation
- <quality check 1>
- <quality check 2>

## Examples
<minimal high-signal example>
```

## Anti-Patterns To Avoid
- vague names like `helper-skill` or `utils`
- descriptions that omit trigger conditions
- overly long SKILL.md with reference material mixed in
- too many equivalent options without a default recommendation
- embedding fragile, time-sensitive instructions without version notes
- deep reference chains (`SKILL.md -> advanced.md -> details.md`)
- Windows-style paths like `scripts\tool.py`
- scripts that punt on errors instead of handling them
- undocumented constants or unexplained parameter values

## Scripts And Runtime Guidance
If scripts are included:
- prefer deterministic utility scripts for repetitive or fragile operations
- state whether the agent should execute the script or read it as reference
- include explicit error handling in scripts; do not punt failures
- document required packages and verify environment constraints
- for MCP tools, use fully qualified names (`ServerName:tool_name`)

## Quick Quality Rubric
Rate each area from 0-2 (target total >= 10):
- Metadata clarity
- Trigger specificity
- Instruction actionability
- Output structure clarity
- Maintainability/progressive disclosure
- Workflow and feedback-loop quality

## Final Checklist
- [ ] clarification questions were asked and answered before authoring
- [ ] final requirements were explicitly confirmed
- [ ] `name` and `description` satisfy constraints
- [ ] description is third person and trigger-rich
- [ ] `SKILL.md` is concise (prefer under 500 lines)
- [ ] references are one level deep from `SKILL.md`
- [ ] complex tasks include workflow + validation loop
- [ ] scripts are explicit, reliable, and dependency-aware
- [ ] at least three evaluation scenarios exist
