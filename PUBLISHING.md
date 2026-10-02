# Publishing to Claude Code Plugin Marketplace

This guide explains how to submit the agent-skills plugins to the Claude Code plugin marketplace.

## Submission Checklist

Before submitting, ensure:

### Plugin Structure
- [ ] Each plugin has `.claude-plugin/plugin.json` with required fields:
  - `name` (kebab-case, lowercase)
  - `version` (semver: MAJOR.MINOR.PATCH)
  - `description` (one sentence)
  - `author` (name, optional email)
  - `license` (MIT for open-source)
  - `repository` (GitHub URL for public distribution)
  - `keywords` (array of relevant terms)

- [ ] Each plugin has a `README.md` documenting:
  - What the plugin does
  - Which skills it includes
  - When to use it
  - Installation instructions (if needed)

- [ ] All skills follow progressive disclosure:
  - `SKILL.md` — concise instructions + YAML frontmatter
  - `REFERENCE.md` — optional, detailed content
  - `EXAMPLES.md` — optional, worked examples

### Validation
- [ ] Run `claude plugin validate <plugin.json>` for each plugin (or manually verify structure)
- [ ] `.plugin` files are valid zip archives
- [ ] No `.DS_Store`, `.env`, credentials, or secrets in plugins
- [ ] No hardcoded absolute paths (use placeholders if needed)

### Documentation
- [ ] Root `README.md` explains what's in the repo
- [ ] `PLUGINS.md` documents all available plugins
- [ ] Each plugin has a clear description in its `plugin.json`

### Legal
- [ ] License is clear (MIT)
- [ ] Copyright/attribution is correct
- [ ] No GPL or copyleft licenses (conflicts with Anthropic's preferences)

## Submission Steps

### 1. Prepare the Repository
```sh
# Ensure repo is clean
git status

# Tag the release
git tag -a v1.0.0 -m "Initial plugin release: 8 plugins for code quality, design patterns, React Native, testing, documentation, and skill creation"

# Push to GitHub
git push origin main
git push origin v1.0.0
```

### 2. Create Release Assets
```sh
# Create a releases directory
mkdir -p releases/v1.0.0

# Copy all .plugin files
cp *.plugin releases/v1.0.0/

# Create a RELEASE_NOTES.md
cat > releases/v1.0.0/RELEASE_NOTES.md << EOF
# Agent Skills v1.0.0 Release

Initial release of 8 Claude Code plugins:

- **code-quality** — Apply Clean Code principles and refactoring patterns
- **design-patterns-behavioral** — Gang of Four behavioral patterns
- **design-patterns-creational** — Gang of Four creational patterns
- **design-patterns-structural** — Gang of Four structural patterns
- **react-native** — Expo Modules API & NativeWind styling
- **testing** — Test suite analysis & coverage review
- **documentation** — Documentation drift detection
- **skill-creation** — Author custom skills & subagents

See [PLUGINS.md](../PLUGINS.md) for details.
EOF
```

### 3. Submit to Marketplace
Contact Anthropic (or check https://claude.com/plugins) for the submission form:

**Plugin Details:**
- **Repository URL:** https://github.com/mdadul/agent-skills
- **License:** MIT
- **Author:** Emdadul
- **Plugins (8 total):**
  1. code-quality (5 skills)
  2. design-patterns-behavioral (10 skills)
  3. design-patterns-creational (5 skills)
  4. design-patterns-structural (7 skills)
  5. react-native (2 skills)
  6. testing (1 skill)
  7. documentation (1 skill)
  8. skill-creation (2 skills)

**Plugin Description (for marketplace listing):**
> A collection of 8 curated plugins for professional code quality, design patterns, React Native development, testing, documentation, and skill authoring. Includes all 22 Gang of Four design patterns, Clean Code principles, refactoring techniques, and pragmatic programming guidance.

### 4. Post-Release
- [ ] Create GitHub release with `.plugin` files as assets
- [ ] Update repo with version tags
- [ ] Monitor plugin marketplace for listing status
- [ ] Respond to user feedback & issues

## Versioning

Follow semantic versioning:
- **MAJOR** (1.0.0 → 2.0.0): Breaking changes to skill interfaces
- **MINOR** (1.0.0 → 1.1.0): New skills, new features
- **PATCH** (1.0.0 → 1.0.1): Bug fixes, documentation updates

## Maintenance

After release:
1. Accept issues & feature requests on GitHub
2. Create new versions with bug fixes & enhancements
3. Update plugins & re-test before releasing
4. Maintain backward compatibility (or document breaking changes)

## Resources

- Claude Code plugin spec: [Link TBD]
- Plugin marketplace: [Link TBD]
- Example plugins: [Link TBD]

---

**Contact:** For questions about submitting to the marketplace, reach out to Anthropic support or check the plugin guidelines.
