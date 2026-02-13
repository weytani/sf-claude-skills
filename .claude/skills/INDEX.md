---
name: SF Claude Skills Index
description: Master index of all Salesforce platform reference skills
when_to_use: When deciding which skill to consult, or when searching across all skills
version: "62.0"
---

# SF Claude Skills — Master Index

## Quick Search

```bash
# Find any keyword across all skills (from your project root)
grep -r "keyword" .claude/skills/

# Find which skill covers a topic
grep -r "when_to_use" .claude/skills/*/SKILL.md
```

## Skills by Category

### Server-Side

| Skill | File | Covers |
|-------|------|--------|
| **Apex** | `apex/SKILL.md` | Governor limits, triggers, async patterns, DML, sharing, CRUD/FLS, errors |
| **SOQL** | `soql/SKILL.md` | Query syntax, relationships, aggregates, date literals, TYPEOF, performance |

### Client-Side

| Skill | File | Covers |
|-------|------|--------|
| **LWC** | `lwc/SKILL.md` | Decorators, wire service, lifecycle, events, navigation, data access |
| **SLDS 2** | `slds2/SKILL.md` | Design tokens, grid, utility classes, component blueprints, icons, accessibility |

### Declarative

| Skill | File | Covers |
|-------|------|--------|
| **Flows** | `flows/SKILL.md` | Flow types, trigger timing, elements, variables, fault handling, Flow vs Apex |

### Tooling

| Skill | File | Covers |
|-------|------|--------|
| **Metadata API** | `metadata-api/SKILL.md` | package.xml, metadata types, source vs MDAPI format, deploy, destructive changes |
| **SF CLI** | `sf-cli/SKILL.md` | Auth, scratch orgs, deploy/retrieve, testing, data ops, CI/CD |

### Legacy

| Skill | File | Covers |
|-------|------|--------|
| **Aura** | `aura/SKILL.md` | ⚠️ LEGACY — Bundle structure, events, Apex integration, migration to LWC |
| **Visualforce** | `visualforce/SKILL.md` | ⚠️ LEGACY — Pages, controllers, expressions, view state, migration to LWC |

## Cross-Cutting Reference

| File | Covers |
|------|--------|
| **`sf-patterns/SKILL.md`** | Order of execution, governor limits philosophy, sharing model, security model, platform events, Custom Metadata vs Custom Settings |

## Skill Layering

These skills document **how the Salesforce platform works** — they are technology reference, not team conventions.

For project-specific patterns (fflib, NebulaLogger, naming conventions, code standards), use project-level skills:

```
your-project/.claude/skills/     ← copy from this repo + your team's own skills
```

## When to Use Which Skill

| I need to... | Start with |
|-------------|-----------|
| Write a SOQL query | `soql/SKILL.md` |
| Debug a trigger or governor limit | `apex/SKILL.md` |
| Build a Lightning component | `lwc/SKILL.md` |
| Style a component | `slds2/SKILL.md` |
| Automate without code | `flows/SKILL.md` |
| Deploy or retrieve metadata | `metadata-api/SKILL.md` + `sf-cli/SKILL.md` |
| Set up a scratch org | `sf-cli/SKILL.md` |
| Maintain an Aura component | `aura/SKILL.md` |
| Maintain a VF page | `visualforce/SKILL.md` |
| Understand save-order bugs | `sf-patterns/SKILL.md` |
| Debug sharing/security | `sf-patterns/SKILL.md` |
