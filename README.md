# Agent Skills

A collection of 33 reusable **Agent Skills** for Claude Code, organized into 8 plugins.

Each skill packages domain expertise — coding principles, design patterns, refactoring techniques, testing strategies — into a portable, model-discoverable unit. When a task matches a skill's `description`, the agent loads it on demand and follows its guidance.

## Repository structure

```
agent-skills/
└── plugins/              # 8 Claude Code plugins
    ├── code-quality/
    ├── design-patterns-behavioral/
    ├── design-patterns-creational/
    ├── design-patterns-structural/
    ├── documentation/
    ├── react-native/
    ├── skill-creation/
    └── testing/
```

Each plugin contains skills organized in `plugins/<name>/skills/`.

## 33 Skills organized in 8 plugins

Skills are bundled into plugins for easy installation:

- **code-quality** — [clean-code](plugins/code-quality/skills/clean-code/SKILL.md), [code-smells](plugins/code-quality/skills/code-smells/SKILL.md), [refactoring](plugins/code-quality/skills/refactoring/SKILL.md), [pragmatic-programmer](plugins/code-quality/skills/pragmatic-programmer/SKILL.md), [dead-code-removal](plugins/code-quality/skills/dead-code-removal/SKILL.md)
- **design-patterns-behavioral** — 10 Gang of Four behavioral patterns
- **design-patterns-creational** — 5 Gang of Four creational patterns
- **design-patterns-structural** — 7 Gang of Four structural patterns
- **react-native** — [expo-modules-api](plugins/react-native/skills/expo-modules-api/SKILL.md), [nativewind-expo](plugins/react-native/skills/nativewind-expo/SKILL.md)
- **testing** — [test-audit](plugins/testing/skills/test-audit/SKILL.md)
- **documentation** — [doc-drift](plugins/documentation/skills/doc-drift/SKILL.md)
- **skill-creation** — [create-skills](plugins/skill-creation/skills/create-skills/SKILL.md), [create-subagent](plugins/skill-creation/skills/create-subagent/SKILL.md)

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

### Claude Code plugins (recommended)

Install the 8 plugins from the Claude Code plugin directory:

1. Open Claude Code
2. Go to **Customize > Plugins**
3. Search for **agent-skills**
4. Install the plugins you need

Each plugin bundles related skills for easy discovery and management.

### Manual installation

To install skills individually from source:

```sh
git clone https://github.com/mdadul/agent-skills.git
cp -r agent-skills/plugins/code-quality/skills/clean-code ~/.claude/skills/
```

Skills are plain Markdown with YAML frontmatter, so you can copy any skill folder to your `~/.claude/skills/` (user-wide) or `.claude/skills/` (project-wide) directory.

## Usage

Once installed, invoke a skill explicitly by name (e.g. `/clean-code` in Claude Code), or simply describe your task — the agent loads the matching skill automatically based on its `description`.

> ⚠️ Skills run with full agent permissions. Review a skill before using it.

## Contributing

When adding a skill, use the [create-skills](skills/create-skills/SKILL.md) skill to keep metadata, descriptions, and structure consistent. For subagents, use [create-subagent](skills/create-subagent/SKILL.md).
