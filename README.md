# skills

Personal skills marketplace for Claude Code.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| `context-file-hygiene` | Audit and trim CLAUDE.md files to minimal behavioral directives |
| `creating-agent-teams` | Guide for creating and managing Claude Code agent teams with model selection, communication patterns, and prompt templates |
| `design-system-showcase` | Enforces a living component library page — bootstraps it when missing, gates component work until updated, and audits for missing entries |
| `content-audit` | Multi-persona content audit engine — analyzes writing for clarity, jargon, logical flow, and AI-signature patterns |
| `llm-wiki` | PAS-managed persistent markdown wiki — seeds a customizable process into `.pas/` on first invoke (requires `pas-framework`) |
| `llm-wiki-extended` | Extends `llm-wiki` with memory lifecycle, knowledge graph, hybrid search, automation hooks, and quality controls for wikis past ~100 pages |

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

### llm-wiki

PAS-managed persistent markdown wiki in the current project, based on Andrej Karpathy's [LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). On first invoke, seeds a process into `.pas/processes/llm-wiki/` containing the orchestrator and five per-operation skills (`bootstrapping`, `digesting`, `ingesting`, `querying`, `linting`). After seeding, the local copy is authoritative — edit it freely for per-project customization. Creates `.wiki/raw/`, `.wiki/digested/`, `.wiki/wiki/SCHEMA.md`, `.wiki/wiki/index.md`, `.wiki/wiki/log.md`, and a `CLAUDE.md` pointer; handles digest/ingest/query/lint with canonical naming, contradiction detection at ingest time, and a grep-friendly log format.

Requires the [`pas-framework`](https://github.com/ZoranSpirkovski/PAS) plugin for the orchestration library and feedback hooks.

Install:

```
/plugin install llm-wiki@skills
```

### llm-wiki-extended

Extends `llm-wiki` with the v2 pattern from Rohit Ghumare's [LLM Wiki v2 gist](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2). Adds confidence scoring, page supersession, consolidation tiers (working/episodic/semantic/procedural), a typed knowledge graph, hybrid BM25+vector+graph search, automation hooks, and new operations (`crystallize`, `forget`, `supersede`, `verify`). Use only when an existing `llm-wiki` has outgrown its flat structure — not for fresh projects.

Install:

```
/plugin install llm-wiki-extended@skills
```