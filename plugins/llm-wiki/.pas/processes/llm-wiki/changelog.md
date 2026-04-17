# LLM Wiki Changelog

## 1.0.0 — 2026-04-16

- Converted from a monolithic `SKILL.md` to a PAS process with solo orchestration.
- Five operations (`bootstrapping`, `digesting`, `ingesting`, `querying`, `linting`) are now separate skills under `agents/orchestrator/skills/`.
- Cross-cutting conventions (architecture, index/log format, page frontmatter, what-belongs-where, rationalizations, common mistakes) consolidated into `process.md`.
- Plugin thin launcher auto-seeds this process tree into `<cwd>/.pas/processes/llm-wiki/` on first invoke.
