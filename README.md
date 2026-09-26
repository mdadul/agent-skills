# Agent Skills

A collection of reusable **Agent Skills** and **subagents** for AI coding assistants (Claude Code, Claude Agent SDK, and other skill-aware tools).

Each skill packages domain expertise — coding principles, design patterns, refactoring techniques, framework setup — into a portable, model-discoverable unit. When a task matches a skill's `description`, the agent loads it on demand and follows its guidance.

## What's inside

```
agent-skills/
├── agents/                 # Subagent definitions
│   └── refactoring-expert.md
└── skills/                 # Agent Skills
    ├── clean-code/
    ├── code-smells/
    ├── create-skills/
    ├── create-subagent/
    ├── design-pattern/     # The 22 Gang of Four patterns
    ├── doc-drift/
    ├── expo-modules-api/
    ├── nativewind-expo/
    ├── pragmatic-programmer/
    └── refactoring/
```

## Skills

### Code quality & craftsmanship
| Skill | What it does |
|-------|--------------|
| [clean-code](skills/clean-code/SKILL.md) | Apply Robert C. Martin's Clean Code principles — meaningful names, small functions, honest error handling, TDD, SRP, smells & heuristics. |
| [code-smells](skills/code-smells/SKILL.md) | Detect, name, and remediate code smells using the classic taxonomy (Bloaters, Object-Orientation Abusers, Change Preventers, Dispensables, Couplers). |
| [refactoring](skills/refactoring/SKILL.md) | Apply named refactoring techniques (Extract Method, Move Method, Replace Conditional with Polymorphism, …) organized by the classic catalog. |
| [pragmatic-programmer](skills/pragmatic-programmer/SKILL.md) | Apply *The Pragmatic Programmer*'s foundational principles — DRY and Orthogonality. |
| [doc-drift](skills/doc-drift/SKILL.md) | Check function comments against the implementation. Report over-promises and direct mismatches on a selection, a file, or a project. On a large codebase, fan the check out to low-cost subagents. |

### Design patterns
[design-pattern](skills/design-pattern/) covers all 22 Gang of Four patterns, organized by category. Each pattern has its own `SKILL.md`, `REFERENCE.md`, and `EXAMPLES.md`.

- **Creational** — [abstract-factory](skills/design-pattern/creational/abstract-factory/SKILL.md), [builder](skills/design-pattern/creational/builder/SKILL.md), [factory-method](skills/design-pattern/creational/factory-method/SKILL.md), [prototype](skills/design-pattern/creational/prototype/SKILL.md), [singleton](skills/design-pattern/creational/singleton/SKILL.md)
- **Structural** — [adapter](skills/design-pattern/structural/adapter/SKILL.md), [bridge](skills/design-pattern/structural/bridge/SKILL.md), [composite](skills/design-pattern/structural/composite/SKILL.md), [decorator](skills/design-pattern/structural/decorator/SKILL.md), [facade](skills/design-pattern/structural/facade/SKILL.md), [flyweight](skills/design-pattern/structural/flyweight/SKILL.md), [proxy](skills/design-pattern/structural/proxy/SKILL.md)
- **Behavioral** — [chain-of-responsibility](skills/design-pattern/behavioral/chain-of-responsibility/SKILL.md), [command](skills/design-pattern/behavioral/command/SKILL.md), [iterator](skills/design-pattern/behavioral/iterator/SKILL.md), [mediator](skills/design-pattern/behavioral/mediator/SKILL.md), [memento](skills/design-pattern/behavioral/memento/SKILL.md), [observer](skills/design-pattern/behavioral/observer/SKILL.md), [state](skills/design-pattern/behavioral/state/SKILL.md), [strategy](skills/design-pattern/behavioral/strategy/SKILL.md), [template-method](skills/design-pattern/behavioral/template-method/SKILL.md), [visitor](skills/design-pattern/behavioral/visitor/SKILL.md)

### React Native / Expo
| Skill | What it does |
|-------|--------------|
| [nativewind-expo](skills/nativewind-expo/SKILL.md) | Install and configure NativeWind v4 (Tailwind CSS for React Native) in an Expo project. |
| [expo-modules-api](skills/expo-modules-api/SKILL.md) | Build, extend, and review Expo native modules with the Modules API (Swift/Kotlin DSL, views, events, Records). |

### Meta — authoring skills & agents
| Skill | What it does |
|-------|--------------|
| [create-skills](skills/create-skills/SKILL.md) | Create and maintain custom Skills with correct metadata, trigger-aware descriptions, and progressive disclosure. |
| [create-subagent](skills/create-subagent/SKILL.md) | Create and maintain custom subagent definition files for any AI agent platform. |

## Subagents

| Agent | What it does |
|-------|--------------|
| [refactoring-expert](agents/refactoring-expert.md) | Diagnoses code smells and produces prioritized, step-by-step refactoring plans. Always interviews the user first, then plans against named smells and techniques (powered by the `code-smells` and `refactoring` skills). |

## Anatomy of a skill

Each skill lives in its own directory and follows progressive disclosure:

- **`SKILL.md`** — required. YAML frontmatter (`name`, `description`) plus concise instructions. The `description` is what the agent matches against to decide when to load the skill, so it lists concrete triggers.
- **`REFERENCE.md`** — optional. Deeper reference material loaded only when needed.
- **`EXAMPLES.md`** — optional. Worked examples and before/after code.

```yaml
---
name: clean-code
description: Apply Clean Code principles … Use when the user asks for clean code, refactoring, …
---

# Clean Code
## Purpose
…
```

## Installation

The fastest way to install is with [`skills`](https://skills.sh), which works across 17+ agent tools (Claude Code, Cursor, Codex, Gemini CLI, GitHub Copilot, and more):

```sh
# Install all skills from this repo
npx skills@latest add mdadul/agent-skills
```

The installer detects which agent tools you have and writes the skills to the right place for each — as universal files or symlinks. Run it from inside the project (or directory) where you want the skills available.

```sh
# Install a single skill
npx skills@latest add mdadul/agent-skills/clean-code
```

By default the installer stops at the top-level skills. To also pull in the 22 nested [design-pattern](skills/design-pattern/) skills, search the full tree:

```sh
# Include the nested design-pattern/** skills
npx skills@latest add mdadul/agent-skills --full-depth
```

### Installing the subagent

The [`skills`](https://skills.sh) CLI installs **skills only** — the [refactoring-expert](agents/refactoring-expert.md) subagent is a separate artifact you install by hand. For Claude Code, drop it in an agents directory (`~/.claude/agents/` for user-wide, or `.claude/agents/` in a project):

```sh
git clone https://github.com/mdadul/agent-skills.git

# the subagent itself
cp agent-skills/agents/refactoring-expert.md ~/.claude/agents/

# the skills it depends on — it reads code-smells and refactoring at runtime
npx skills@latest add mdadul/agent-skills -s code-smells,refactoring
```

> The `refactoring-expert` agent grounds every diagnosis in the `code-smells` and `refactoring` skills, so install those alongside it (or run it from inside this repo, where it can read them directly).

### Manual install

Skills are plain Markdown with YAML frontmatter, so you can also install them by hand.

**Claude Code** — copy (or symlink) a skill directory under your skills path: `~/.claude/skills/` for user-wide, or `.claude/skills/` in a project. The agent discovers it automatically. Invoke explicitly with `/<skill-name>`, or let it trigger when a task matches the description.

```sh
git clone https://github.com/mdadul/agent-skills.git
cp -r agent-skills/skills/clean-code ~/.claude/skills/
```

**Other tools** — point your tool at the `skills/` directory or copy individual skill folders; the format is portable across any skill-aware agent platform.

## Usage

Once installed, invoke a skill explicitly by name (e.g. `/clean-code` in Claude Code), or simply describe your task — the agent loads the matching skill automatically based on its `description`.

> ⚠️ Skills run with full agent permissions. Review a skill before using it.

## Contributing

When adding a skill, use the [create-skills](skills/create-skills/SKILL.md) skill to keep metadata, descriptions, and structure consistent. For subagents, use [create-subagent](skills/create-subagent/SKILL.md).
