---
name: llm-wiki
description: Use when the user wants to build or maintain a personal knowledge base from sources (papers, articles, notes, transcripts, book chapters, meeting recordings), when they say things like "I want a knowledge base that accumulates", "help me organize my research", "file this source", "ingest this", or when they drop a document into the project and ask to process it. Also use when asked to run a health check, lint, or audit over an existing wiki, or to answer a question against an accumulated collection of notes. This is the minimal pattern — always start here; upgrade to llm-wiki-extended only when specific scaling symptoms appear.
---

# LLM Wiki — Thin Launcher

This is a PAS-managed process. The first invocation in a target repo seeds the process definition into that repo's `.pas/processes/llm-wiki/`; subsequent invocations read and execute from the local copy, which is authoritative and can be customized per-project.

Requires the `pas-framework` plugin (for the orchestration library and feedback hooks).

## Protocol

### 1. PAS first-run setup

Check `<cwd>/.pas/config.yaml`. If missing:

- Create `<cwd>/.pas/config.yaml` with:
  ```yaml
  feedback: enabled
  feedback_disabled_at: ~
  ```
- Create `<cwd>/.pas/workspace/` directory.
- Tell the user: "PAS initialized — `.pas/` directory created with config and workspace."

If `<cwd>/pas-config.yaml` exists at root but `<cwd>/.pas/` does not, migrate: move config, library, workspace, processes, and feedback under `.pas/` (same behavior as the core PAS skill).

### 2. Seed llm-wiki process on first invoke

Check `<cwd>/.pas/processes/llm-wiki/`. If missing:

- Copy `${CLAUDE_PLUGIN_ROOT}/.pas/processes/llm-wiki/` → `<cwd>/.pas/processes/llm-wiki/` recursively. Use the Read + Write tools per file (not `cp`) so the seed works in sandboxed environments. Preserve the directory structure: `process.md`, `modes/`, `changelog.md`, `references/`, `feedback/backlog/.gitkeep`, and `agents/orchestrator/` with its `agent.md`, `changelog.md`, `references/`, `feedback/backlog/.gitkeep`, and `skills/{bootstrapping,digesting,ingesting,querying,linting}/`.
- Announce: "Installed llm-wiki into `.pas/processes/llm-wiki/` — local copy is now authoritative; edit it freely for per-project customization."

If it already exists, compare the plugin's `changelog.md` head against the local one. If the plugin is newer, print one line: `Plugin has newer llm-wiki revision (vX.Y). Local copy is authoritative — run /pas upgrade llm-wiki if you want to pull it in.` Do not overwrite automatically.

### 3. Route to the process

Read `<cwd>/.pas/processes/llm-wiki/process.md` and follow its phases:

1. **understand-intent** — detect which operation the user wants (bootstrap / digest / ingest / query / lint) from their message plus the state of `.wiki/`. In supervised mode, confirm before continuing.
2. **execute** — read the matching skill from `<cwd>/.pas/processes/llm-wiki/agents/orchestrator/skills/{operation}/SKILL.md` and run it.

Follow the solo orchestration pattern from `${CLAUDE_PLUGIN_ROOT:pas-framework}/library/orchestration/solo.md` and lifecycle protocol from `${CLAUDE_PLUGIN_ROOT:pas-framework}/library/orchestration/lifecycle.md`.

Always announce which operation you're running so the user can follow along.

## Operations at a glance

| Operation | When invoked | Skill file |
|---|---|---|
| `bootstrap` | `.wiki/` missing; user wants a new knowledge base | `agents/orchestrator/skills/bootstrapping/SKILL.md` |
| `digest` | Non-trivial source format; "digest this"; auto-called by ingest | `agents/orchestrator/skills/digesting/SKILL.md` |
| `ingest` | User drops a source and asks to file it | `agents/orchestrator/skills/ingesting/SKILL.md` |
| `query` | User asks a question answerable from the wiki | `agents/orchestrator/skills/querying/SKILL.md` |
| `lint` | Health check, recurring cadence, or "wiki feels stale" | `agents/orchestrator/skills/linting/SKILL.md` |

## Local customization

Once seeded, the local copy under `<cwd>/.pas/processes/llm-wiki/` is yours to edit. Common customizations:

- Add domain-specific lint rules in `linting/SKILL.md`.
- Extend `bootstrapping/schema-template.md` with page types specific to your wiki's domain.
- Tighten ingest checkpoint phrasing in `ingesting/SKILL.md` to match how you prefer to steer synthesis.
- Swap to `hub-and-spoke` orchestration with specialist agents if operations grow too large for one orchestrator — the orchestrator already carries `Agent`, `SendMessage`, and `TeamCreate` tools for that upgrade path.

The plugin never overwrites local changes. When a new plugin version ships, you'll see a one-line notice on invocation; pulling updates in is an opt-in action.
