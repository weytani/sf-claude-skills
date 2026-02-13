# SF Claude Skills — Platform Reference Library

## What This Is

A **platform reference** for Salesforce — documenting how the platform itself works, independent of any team's conventions, patterns, or frameworks.

This is the layer *below* project-level skills. Project skills say "use fflib Selectors" or "log with NebulaLogger." These skills say "here's how SOQL relationship queries work" and "here's every governor limit."

## What This Is Not

- Not a tutorial or learning path
- Not team conventions or code standards
- Not a replacement for Salesforce docs — it's a dense, searchable distillation
- Not opinioned about architecture (no fflib, no specific trigger framework)

## Installation

Copy the `.claude/skills/` directory from this repo into your Salesforce project:

```bash
# From your project root
cp -r /path/to/sf-claude-skills/.claude/skills/* .claude/skills/
```

Or add as a git submodule:

```bash
git submodule add https://github.com/weytani/sf-claude-skills.git .claude/vendor/sf-claude-skills
cp -r .claude/vendor/sf-claude-skills/.claude/skills/* .claude/skills/
```

Claude Code auto-discovers any `SKILL.md` files in `.claude/skills/` — no config, no plugin install, no approval chain. These skills sit alongside your team's own project-level skills.

## Skill Format Conventions

Every `SKILL.md` file uses YAML frontmatter with:
- `name` — technology name
- `description` — one-line purpose
- `when_to_use` — when Claude should consult this skill
- `version` — Salesforce API version this targets

Content priorities:
1. **Tables over prose** — scannable, dense, searchable
2. **Code examples over descriptions** — show, don't tell
3. **Error tables** — "I've seen this error" is the #1 use case
4. **Decision trees** — "which async type?" is the #2 use case

## Design Principles

- **Density**: Every line earns its place. No filler, no "Salesforce is a cloud platform."
- **2am Test**: If you're debugging at 2am, can you find the answer in <10 seconds?
- **Platform Truth**: Document what Salesforce does, not what we wish it did.
- **Legacy Honesty**: Aura and Visualforce are marked LEGACY. They still exist and we document them fully, but new work should use LWC.
- **Flat Structure**: 10 skill directories under `.claude/skills/`, no nested categories. `grep -r "keyword" .claude/skills/` finds anything.

## File Map

All skills live under `.claude/skills/`:

| File | Purpose |
|---|---|
| `.claude/skills/INDEX.md` | Master index with categories and search tips |
| `.claude/skills/sf-patterns/SKILL.md` | Cross-cutting: order of execution, sharing, security |
| `.claude/skills/apex/SKILL.md` | Governor limits, triggers, async, DML, errors |
| `.claude/skills/lwc/SKILL.md` | Decorators, wire, lifecycle, events, navigation |
| `.claude/skills/aura/SKILL.md` | LEGACY — bundle structure, events, migration to LWC |
| `.claude/skills/visualforce/SKILL.md` | LEGACY — pages, controllers, expressions, remote actions |
| `.claude/skills/flows/SKILL.md` | Types, elements, trigger timing, fault paths |
| `.claude/skills/metadata-api/SKILL.md` | package.xml, metadata types, deployment |
| `.claude/skills/slds2/SKILL.md` | Design tokens, utility classes, grid, icons |
| `.claude/skills/sf-cli/SKILL.md` | Commands, scratch orgs, auth, deploy, test |
| `.claude/skills/soql/SKILL.md` | Syntax, relationships, aggregates, date literals |
