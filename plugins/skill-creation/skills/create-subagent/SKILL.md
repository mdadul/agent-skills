---
name: create-subagent
description: Create and maintain custom subagent definition files for any AI agent platform, with strong descriptions, tight tool scope, and explicit handoff contracts. Use when the user asks to create, improve, or standardize a subagent.
---

# Create Subagent Skill

## Purpose
Create reliable custom subagents that are easy to invoke, safe to run, and portable across AI platforms.

## When To Use
Use this skill when the user asks to:
- create a new subagent
- improve an existing subagent definition
- split one broad agent into specialized agents
- tighten tool permissions for an agent
- add structured outputs and handoff rules

## Inputs
- User intent and target workflow (review, debug, implementation, orchestration)
- Target platform and format (YAML file, JSON config, or UI-defined agent)
- Scope (project-level, user-level, or session-only)
- Permission and safety constraints
- Expected output contract and handoff requirements

## Workflow

### 1) Capture intent and scope
Identify:
- subagent purpose
- location (project vs user level)
- expected inputs and outputs
- whether it should be read-only or editing-capable

If details are unclear, ask concise follow-up questions.

### 2) Design metadata first
Always define:
- `name`: lowercase and hyphenated
- `description`: explicit trigger phrase with "Use proactively when..."

Optionally define when supported by the target platform:
- `tools`
- `model`
- `permissionMode`
- `maxTurns`
- `mcpServers`
- `hooks`
- `memory`
- `background`
- `isolation`
- `color`

If the target platform does not use YAML frontmatter, map these fields into the platform's equivalent JSON or UI form.

### 3) Write a focused system prompt
The body should include:
- role and objective
- ordered execution steps
- guardrails and stop conditions
- explicit output contract

Keep single responsibility. Do not mix unrelated duties in one subagent.

### 4) Enforce output contract
Prefer this minimal schema:
- `status`: `ok | needs_info | blocked`
- `artifacts`: paths created/changed
- `summary`: concise result
- `open_questions`: numbered follow-ups

This keeps parent-agent synthesis predictable and low-noise.

### 5) Permission hygiene
- Start with least privilege
- Add write/edit/bash only when clearly needed
- Separate explorer/reviewer roles from implementer roles
- Avoid broad MCP/tool access unless required

### 6) Add examples and references
- Add one minimal read-only example
- Add one minimal edit-capable example
- Include source references only when they improve implementation quality
- Keep references shallow and directly actionable

### 7) Validate before finishing
Check:
- valid syntax for the target platform (YAML/JSON/UI config)
- unique, specific `name`
- description matches actual behavior
- prompt contains clear done criteria
- no contradictory tool/policy instructions
- output contract is explicit and parseable
- examples match the documented constraints

## Progressive Disclosure
Keep this file concise and operational. Use companion files for depth:
- `REFERENCE.md` for portable field mapping, permission guidance, and validation criteria
- `EXAMPLES.md` for reusable YAML/JSON agent definitions and workflow prompts

Before drafting a subagent:
1. Read `REFERENCE.md` for platform-specific mapping decisions.
2. Pick the closest pattern from `EXAMPLES.md`.
3. Adapt minimally to the user's requested scope and constraints.

## Output Format
When asked to create or improve a subagent, return:
1. Proposed `name`
2. Proposed `description`
3. Target scope and file/config location
4. Final subagent definition content
5. Optional supporting files (only if useful)
6. Validation checklist and known risks

## Best Practices
- Keep one responsibility per subagent; split planner, implementer, and reviewer roles.
- Use explicit delegation triggers in `description` (for example: "Use proactively when...").
- Start with least privilege tool access, then add capabilities only when justified.
- Require a strict output contract so parent agents can reliably parse responses.
- Prefer deterministic steps and stop conditions over broad, open-ended instructions.
- For parallel execution, keep scopes disjoint (different files/modules) to avoid conflicts.
- Add lightweight validation loops: produce draft -> self-check against checklist -> return.
- Treat subagent outputs as proposed work; verify before merge/deploy.

## Example Patterns

### Chain pattern (sequential handoff)
Use when tasks depend on each other.
1. `repo-scout` finds relevant code paths.
2. `implementer` applies changes based on scoped inputs.
3. `verifier` checks correctness, safety, and tests.

### Fan-out pattern (parallel research)
Use when tasks are independent.
1. Spawn multiple read-only scouts for separate modules.
2. Require each scout to return the same response schema.
3. Synthesize results and run one implementer on approved paths.

### Safe defaults pattern
If uncertain, generate:
- read-only tools first
- concise prompt with explicit "ask before risky actions"
- output schema with `status`, `artifacts`, `summary`, `open_questions`

## Anti-Patterns To Avoid
- vague names such as `helper-agent` or `general-agent`
- descriptions that do not include trigger conditions
- mixing exploration, implementation, and review in one subagent
- granting broad tools/MCP access without task need
- output with no schema or inconsistent keys across examples
- long reference dumps inside the core instructions

## Quick Quality Rubric
Rate each area from 0-2 (target total >= 10):
- Metadata clarity
- Trigger specificity
- Tool/permission safety
- Instruction actionability
- Output contract clarity
- Example quality and portability

## Final Checklist
- [ ] `name` and `description` are specific and trigger-aware
- [ ] responsibilities are non-overlapping
- [ ] tools are least-privilege for the intended task
- [ ] output contract is explicit and consistent
- [ ] syntax is valid for the target platform
- [ ] examples are minimal, practical, and aligned to guidance
- [ ] references are useful and not excessive