# Subagent Reference

## Purpose
Practical reference for creating portable subagents across AI platforms.

## Core Definition Model
Most platforms can express the same concepts, even when syntax differs:
- identity: `name`
- delegation trigger: `description`
- behavior: system prompt / instructions body
- capability boundaries: `tools` / allowlists / denylists
- runtime control: model, permissions, hooks, background, isolation
- output contract: structured return shape

## Portable Field Mapping
Use this mapping when converting between YAML, JSON, or UI-defined agents.

| Canonical Concept | YAML-style Key | JSON-style Key | UI Equivalent |
|---|---|---|---|
| Agent identifier | `name` | `name` | Agent name |
| Delegation trigger | `description` | `description` | Description / when to use |
| System instructions | Markdown body | `prompt` / `systemPrompt` | System prompt |
| Allowed capabilities | `tools` | `tools` | Enabled tools |
| Denied capabilities | `disallowedTools` | `disallowedTools` | Blocked tools |
| Model routing | `model` | `model` | Model selector |
| Permissions mode | `permissionMode` | `permissionMode` | Approval mode |
| Max execution turns | `maxTurns` | `maxTurns` | Turn/task limits |
| External servers | `mcpServers` | `mcpServers` | Integrations |
| Lifecycle hooks | `hooks` | `hooks` | Automation hooks |
| Persistent memory | `memory` | `memory` | Memory scope |
| Async behavior | `background` | `background` | Run in background |
| Git isolation | `isolation` | `isolation` | Worktree/sandbox mode |
| Visual tag | `color` | `color` | Label color |

## Recommended Output Contract
Use a stable schema for predictable orchestration:
- `status`: `ok | needs_info | blocked`
- `artifacts`: files/paths changed or produced
- `summary`: concise result
- `open_questions`: numbered clarifications

## Permission Design Rules
- Start read-only unless editing is necessary.
- Separate role-specific agents (scout, implementer, verifier).
- Avoid broad shell/tool access unless required by the task.
- Add stop conditions for risky actions and ambiguous requirements.

## Pattern Selection
- **Chain**: dependent tasks with explicit handoffs.
- **Fan-out**: independent research tracks with shared output schema.
- **Safe default**: read-only first, escalate privileges only with evidence.

## Validation Checklist
- Metadata is specific and trigger-aware.
- Instructions are deterministic and role-focused.
- Output contract is explicit and parseable.
- Permissions are least-privilege.
- Examples reflect real usage and match constraints.