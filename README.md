# skills

Personal skills marketplace for Claude Code.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| `context-file-hygiene` | Audit and trim CLAUDE.md files to minimal behavioral directives |
| `creating-agent-teams` | Guide for creating and managing Claude Code agent teams with model selection, communication patterns, and prompt templates |
| `design-system-showcase` | Enforces a living component library page — bootstraps it when missing, gates component work until updated, and audits for missing entries |
| `content-audit` | Multi-persona content audit engine — analyzes writing for clarity, jargon, logical flow, and AI-signature patterns |

## Setup

Add this marketplace to Claude Code (one-time):

```
/plugin marketplace add ZoranSpirkovski/skills
```

## Plugins

### context-file-hygiene

Audits CLAUDE.md and similar context files, removing anything the agent can discover from the codebase on its own. Use it after `/init`, when context files feel bloated, or when agents misbehave despite having instructions.

Install:

```
/plugin install context-file-hygiene@skills
```

### creating-agent-teams

Guides you through deciding whether a task needs a single agent, parallel subagents, or a coordinated team — and which model tier and agent type to assign each role. Includes an agent roster, communication patterns, and prompt templates.

Install:

```
/plugin install creating-agent-teams@skills
```

### design-system-showcase

Enforces a living component library page for visual inspection of every UI component. Bootstraps the page when none exists (detecting framework automatically), hard-gates component work until the showcase is updated, and audits for components missing from the page.

Install:

```
/plugin install design-system-showcase@skills
```

### content-audit

Multi-persona content audit engine that stress-tests any written content through dynamically generated reader personas. Runs 5 audit phases — So What, Jargon, Logical Flow, Checklist, and Editor Pattern checks — then presents a structured report with a change list you approve before applying.

Install:

```
/plugin install content-audit@skills
```