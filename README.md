# SF Claude Skills — Salesforce Platform Reference for Claude Code

**A Salesforce MCP server alternative that actually ships.**

## The Problem

Salesforce MCP servers exist, but most businesses can't get them into the hands of their developers. Between security reviews, IT approval chains, infrastructure provisioning, and org-level API access concerns, the path from "this would be useful" to "my team can use it" is measured in quarters, not days.

Meanwhile, your developers are context-switching to Salesforce documentation dozens of times a day — looking up governor limits, trigger context variables, SOQL date literals, deployment flags, and wire adapter syntax. Every tab switch is a break in flow. Every "what's the async heap limit again?" is 30 seconds lost.

## The Solution

Skip the server. Put the reference material directly in Claude's skill context.

This project is 9 dense, searchable skill files covering the Salesforce platform — not opinions about how your team should work, but documentation of how Salesforce itself works. No server to provision. No API credentials to manage. No security review. Copy the skills into your project and every Claude Code session has a Salesforce-fluent copilot.

```bash
# Clone this repo
git clone https://github.com/weytani/sf-claude-skills.git

# Copy the skills into your Salesforce project
cp -r sf-claude-skills/.claude/skills/* your-project/.claude/skills/
```

That's it. No infrastructure. No tokens. No approval chain. No plugin install that IT can block. Just files in your project repo that Claude Code auto-discovers.

## What's Covered

| Skill | File | What's In It |
|-------|------|-------------|
| **Apex** | `apex/SKILL.md` | Governor limits table, trigger context matrix, async decision tree, DML ops, sharing keywords, CRUD/FLS enforcement, common errors |
| **SOQL** | `soql/SKILL.md` | Full syntax, WHERE operators, relationship queries, aggregates, 26 date literals, TYPEOF, semi/anti-joins, performance tuning, dynamic SOQL |
| **LWC** | `lwc/SKILL.md` | Decorators, wire service patterns, lifecycle hooks, event communication (3 patterns with code), navigation, data access priority ladder |
| **Flows** | `flows/SKILL.md` | Flow types, before/after timing decision tree, 15 elements, fault handling, Flow vs Apex decision guide |
| **Metadata API** | `metadata-api/SKILL.md` | package.xml anatomy, 30 metadata types, source vs MDAPI format, deploy/retrieve commands, destructive changes |
| **SF CLI** | `sf-cli/SKILL.md` | Auth (4 methods), scratch orgs, deploy/retrieve, test execution, data operations, CI/CD patterns |
| **SLDS 2** | `slds2/SKILL.md` | Design tokens, grid system, utility classes, 15 component blueprints, icons, accessibility |
| **Aura** | `aura/SKILL.md` | ⚠️ LEGACY — Bundle structure, events, `$A.enqueueAction`, 12-row migration-to-LWC table |
| **Visualforce** | `visualforce/SKILL.md` | ⚠️ LEGACY — Pages, controllers, expressions, view state, RemoteAction |

Plus cross-cutting reference (`sf-patterns/SKILL.md`) covering order of execution, sharing model, security model, platform events, and Custom Metadata vs Custom Settings.

## Why Not an MCP Server?

| | MCP Server | This Project |
|---|---|---|
| **Setup** | Provision server, configure auth, manage API tokens | `cp -r` into your project |
| **Security review** | Needs org API access, credential management, network policies | Read-only markdown files. No API access. No plugins. |
| **Rollout** | Per-user or per-team infrastructure | Commit to your repo — everyone gets it |
| **Maintenance** | Server uptime, API version updates, token rotation | `git pull` |
| **Offline** | No | Yes |
| **Works on day one** | Rarely | Yes |
| **Cost** | Server hosting + API calls | Zero |

An MCP server gives you live org data — object schemas, record queries, real-time metadata. That's powerful when you can get it. This project gives you platform knowledge — the kind of reference material that doesn't change between orgs and doesn't need an API connection to be useful.

They're complementary. But only one of them ships today.

## How It's Designed

**Density over prose.** Every section is tables, code examples, and decision trees. No "Salesforce is a cloud-based CRM platform" filler. Optimized for the 2am debugging session where you need the answer in 10 seconds.

**Platform truth, not team opinions.** These skills document how Salesforce works — governor limits, trigger context, wire adapter behavior. They don't prescribe architectural patterns like fflib or specific logging frameworks. That's what project-level skills are for.

**Flat and searchable.** 10 skill directories, no nested categories. `grep -r "MIXED_DML" .claude/skills/` finds the answer instantly.

## How It Works

Claude Code auto-discovers any `SKILL.md` file inside `.claude/skills/` in your project. No registration, no config file, no plugin install. The YAML frontmatter in each skill tells Claude when to load it:

```yaml
name: Salesforce Apex Platform Reference
when_to_use: When writing, reviewing, or debugging Salesforce Apex code
```

**Why this matters for enterprise teams:** Plugins can be blocked by IT via managed settings. MCP servers need infrastructure approval. But `.claude/skills/` files are just markdown committed to your repo — they travel with your code and can't be gatekept separately from the codebase itself.

## Installation

**Copy into your project:**
```bash
git clone https://github.com/weytani/sf-claude-skills.git
mkdir -p your-project/.claude/skills
cp -r sf-claude-skills/.claude/skills/* your-project/.claude/skills/
```

**Or add as a git submodule** (for centralized updates):
```bash
cd your-project
git submodule add https://github.com/weytani/sf-claude-skills.git .claude/vendor/sf-claude-skills
cp -r .claude/vendor/sf-claude-skills/.claude/skills/* .claude/skills/
```

Your team's own project-level skills sit alongside these in the same `.claude/skills/` directory. They answer different questions — "how does the platform work?" vs "how does our team work?" — so they don't conflict.

## Contributing

PRs welcome. The bar is: would this help a Salesforce developer at 2am? If yes, it belongs here. Keep it dense, keep it accurate, keep it platform-factual rather than opinion-driven.

## License

MIT
