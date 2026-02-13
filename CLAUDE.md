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

Symlink into your Claude skills directory:

```bash
ln -s /path/to/sf-claude-skills ~/.claude/skills/salesforce
```

This sits alongside (not replacing) project-level skills:

```
~/.claude/skills/salesforce/     ← this project (platform reference)
your-project/.claude/skills/     ← project skills (team conventions)
```

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
- **Flat Structure**: 9 skills, no category subdirectories. `grep -r "keyword"` finds anything.

## File Map

| File | Purpose |
|---|---|
| `INDEX.md` | Master index with categories and search tips |
| `_PATTERNS.md` | Cross-cutting: order of execution, sharing, security |
| `apex/SKILL.md` | Governor limits, triggers, async, DML, errors |
| `lwc/SKILL.md` | Decorators, wire, lifecycle, events, navigation |
| `aura/SKILL.md` | LEGACY — bundle structure, events, migration to LWC |
| `visualforce/SKILL.md` | LEGACY — pages, controllers, expressions, remote actions |
| `flows/SKILL.md` | Types, elements, trigger timing, fault paths |
| `metadata-api/SKILL.md` | package.xml, metadata types, deployment |
| `slds2/SKILL.md` | Design tokens, utility classes, grid, icons |
| `sf-cli/SKILL.md` | Commands, scratch orgs, auth, deploy, test |
| `soql/SKILL.md` | Syntax, relationships, aggregates, date literals |
