# skills

Personal skills marketplace for Claude Code.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| `context-file-hygiene` | Audit and trim CLAUDE.md files to minimal behavioral directives |
| `creating-agent-teams` | Guide for creating and managing Claude Code agent teams with model selection, communication patterns, and prompt templates |

## Plugins

### context-file-hygiene

Audits CLAUDE.md and similar context files, removing anything the agent can discover from the codebase on its own. Use it after `/init`, when context files feel bloated, or when agents misbehave despite having instructions.

Install:

```
/install context-file-hygiene@skills
```

### creating-agent-teams

Guides you through deciding whether a task needs a single agent, parallel subagents, or a coordinated team — and which model tier and agent type to assign each role. Includes an agent roster, communication patterns, and prompt templates.

Install:

```
/install creating-agent-teams@skills
```