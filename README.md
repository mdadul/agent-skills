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

## Usage

### Claude Code
Place a skill directory under your skills path (e.g. `~/.claude/skills/` for user-wide, or `.claude/skills/` in a project) and the agent will discover it automatically. Invoke explicitly with `/<skill-name>`, or let the agent trigger it when a task matches the description.

### Other tools
The skills are plain Markdown with YAML frontmatter and are portable across any skill-aware agent platform. Point your tool at the `skills/` directory or copy individual skill folders.

## Contributing

When adding a skill, use the [create-skills](skills/create-skills/SKILL.md) skill to keep metadata, descriptions, and structure consistent. For subagents, use [create-subagent](skills/create-subagent/SKILL.md).
