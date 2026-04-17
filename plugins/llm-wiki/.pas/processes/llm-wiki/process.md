---
name: llm-wiki
goal: Compile and maintain a per-project knowledge base from raw sources so understanding accumulates instead of being re-derived on every query
version: 1.0
orchestration: solo
sequential: true
modes: [supervised, autonomous]

input:
  - intent: the operation the user wants (bootstrap, digest, ingest, query, or lint)
  - source: optional source file(s) under .wiki/raw/ for digest or ingest

phases:
  understand-intent:
    agent: orchestrator
    input: user message
    output: chosen operation + slug
    gate: user confirms operation
  execute:
    agent: orchestrator
    input: chosen operation
    output: wiki pages touched and log entry appended
    gate: user approves result

status_file: .pas/workspace/llm-wiki/{slug}/status.yaml
---

# LLM Wiki

A solo-orchestrated process: one orchestrator routes every invocation to one of five operations (`bootstrapping`, `digesting`, `ingesting`, `querying`, `linting`) based on user intent and the state of `.wiki/`. Each operation is a skill under `agents/orchestrator/skills/`. All operations share the conventions defined below.

## Core Principle

**Stop re-deriving, start compiling.** Most LLM+document workflows re-read raw sources every query and rebuild understanding from scratch. A wiki compiles the understanding once and keeps it current. The wiki is a persistent, compounding artifact. The cross-references already exist. Contradictions are already flagged. Every new source strengthens or challenges what's there — it does not start over.

**The LLM owns the wiki layer; the user owns sourcing and direction.** The orchestrator does the grunt work (summarizing, cross-referencing, filing, bookkeeping) that makes humans abandon wikis. The human curates, asks good questions, and thinks about what it all means.

## Phases

**Understand Intent** — `orchestrator` reads the user message plus `.wiki/` state. If `.wiki/` is missing → route to `bootstrapping`. Otherwise detect which operation the user wants and choose a slug (e.g. `ingest-raft-paxos-2026-04-16`). In supervised mode, confirm with the user before continuing.

**Execute** — `orchestrator` reads the matching skill from `agents/orchestrator/skills/{operation}/SKILL.md` and executes it. Before any wiki work on an existing wiki, always read `.wiki/wiki/SCHEMA.md` in full — it encodes domain-specific conventions that override generic defaults. Final output is a list of pages touched and a one-line log entry preview.

## Architecture

Everything lives under `.wiki/` at the project root. Four layers inside it, each with a different owner and lifecycle.

- **`.wiki/raw/`** — user's curated sources. Immutable. The orchestrator reads from it but never modifies it. If the user wants to edit a source, they do it themselves.
- **`.wiki/digested/`** — preprocessed markdown. Mirrors the folder hierarchy of `.wiki/raw/`, with a slug-named subdirectory at each leaf. Contains extracted text, extracted images, and a combined `digest.md`. Orchestrator-owned. The user can inspect it to verify extraction quality.
- **`.wiki/wiki/`** — wiki pages. Orchestrator-owned entirely. The user reads; the orchestrator writes.
- **`.wiki/wiki/SCHEMA.md`** — the rulebook. Tells any future agent how this specific wiki is structured and what workflows to follow. The user and orchestrator co-evolve it over time.

## Index and Log Conventions

**`.wiki/wiki/index.md` is content-oriented.** Organized by category with these fixed headers: `## Sources`, `## Entities`, `## Concepts`, `## Synthesis`. Under each, one line per page: `- [[slug]] — one-line description`. Update on every ingest, every new synthesis page, and every lint-driven promotion.

**`.wiki/wiki/log.md` is chronological and append-only.** Every operation gets one entry. Canonical prefix format: `## [YYYY-MM-DD] <op> | <title>` where `<op>` is `bootstrap`, `digest`, `ingest`, `query`, `lint`, or `schema`. The `## [` prefix is load-bearing — it makes `grep "^## \[" log.md` a usable timeline tool. The body under each heading is free-form: pages touched, contradictions flagged, notes. Do not mix prefix formats; do not put operations anywhere but the log.

## Page Frontmatter

Every wiki page gets this YAML frontmatter. Dataview-compatible.

```yaml
---
type: source | entity | concept | synthesis
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: <integer>
tags: [tag1, tag2]
---
```

`sources` semantics: on a source page, set to `1` (the page represents itself). On entity, concept, or synthesis pages, set to the count of distinct source pages whose claims the page currently cites. Increment when you add a new citation from a new source.

Keep it short. Do not add fields the process does not specify — page frontmatter is a load-bearing convention that downstream tools (Dataview, the lint operation, `llm-wiki-extended`) depend on.

## What Belongs Where

| Kind | Location | Who writes |
|---|---|---|
| User's source files (PDFs, articles, notes, transcripts) | `.wiki/raw/` | User only |
| Preprocessed source extractions (text, images, digest) | `.wiki/digested/<path>/<slug>/` | Orchestrator |
| Source summaries | `.wiki/wiki/sources/` | Orchestrator |
| Entity pages (people, libraries, projects, systems) | `.wiki/wiki/entities/` | Orchestrator |
| Concept pages (ideas, patterns, terms) | `.wiki/wiki/concepts/` | Orchestrator |
| Synthesis pages (comparisons, analyses, answers filed back) | `.wiki/wiki/synthesis/` | Orchestrator |
| Conventions for this specific wiki | `.wiki/wiki/SCHEMA.md` | Orchestrator + user co-evolve |
| Catalog of all pages | `.wiki/wiki/index.md` | Orchestrator |
| Chronological operations log | `.wiki/wiki/log.md` | Orchestrator |
| Pointer from project context | `CLAUDE.md` (one line under `## Wiki`) | Orchestrator during bootstrap |

## Rationalizations to Reject

| Excuse | Reality |
|---|---|
| "I'll just answer the question, no need to file it back" | Good answers disappear into chat history and have to be re-derived. File it — that's the whole point. |
| "The log entry is obvious from the diff, skip it" | The log is how you and the user reconstruct what happened. Skipping it silently decays the wiki's auditability. |
| "This source is too short for a full page" | Short sources still need a slug, a citation target, and a log entry. The page can be short too. |
| "I'll update `index.md` later" | "Later" doesn't happen. The index has to move in lockstep or queries will miss pages. Update it in the same turn. |
| "Put the schema in CLAUDE.md, it's simpler" | CLAUDE.md is for behavioral directives that bloat quickly. The schema belongs in `.wiki/wiki/SCHEMA.md`; CLAUDE.md only points at it. |
| "Just write the new claim, worry about contradictions on the next lint" | Contradictions compound. Detect them at ingest time when you have the new source fresh. Lint is a backstop, not the primary defense. |
| "`README.md` / `INDEX.md` / `LOG.md` uppercase is clearer" | The lowercase canonical names (`SCHEMA.md`, `index.md`, `log.md`) are what downstream tools and future sessions expect. Use them. |

## Common Mistakes

- **Editing files in `.wiki/raw/`.** The raw layer is the user's source of truth. If it's wrong, the user edits it.
- **Writing a new claim without reading existing pages first.** Contradiction detection only works if you read before you write.
- **One big "notes" page instead of entity and concept pages.** Cross-references require named pages. Flat notes don't compound.
- **Forgetting the log entry.** Every operation, every time. No exceptions.
- **Skipping the schema because "it's obvious."** Future sessions (yours and other agents) cannot read your memory. `.wiki/wiki/SCHEMA.md` is how you teach future Claude what this wiki expects.
- **Updating one page and leaving its neighbors stale.** When a source updates an entity, also check concepts and synthesis pages that cite that entity.
- **Running text-only extraction on image-heavy sources.** Floor plans, annotated diagrams, scanned documents — text extraction produces empty or degraded output. Run digest first to extract images and describe them visually.
- **Skipping digest on complex formats.** PDFs, docx, and xlsx files should be digested before ingest. The digest artifact is inspectable — the user can verify extraction quality before wiki synthesis happens.

## Tips

- **Obsidian is the intended reader.** The wiki is just a git repo of markdown, but Obsidian's graph view and wikilink support make it pleasant. Suggest the user open the `.wiki/wiki/` directory as an Obsidian vault.
- **Obsidian Web Clipper** converts articles to markdown — the easiest way to get web sources into `.wiki/raw/`.
- **Dataview** plugin runs queries over page frontmatter. The frontmatter spec above is Dataview-ready.
- **The wiki is a git repo.** Commit after each ingest for a free audit trail.

## Lifecycle

This process follows the shared lifecycle protocol. Read `${CLAUDE_PLUGIN_ROOT}/library/orchestration/lifecycle.md` for:

- Workspace creation and status tracking
- Task creation (required — create a Claude Code task for each phase)
- Shutdown sequence and completion gate
- Ready handshake for multi-agent patterns

## Credits

Based on Andrej Karpathy's [LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). This process encodes Karpathy's pattern as a reusable PAS process; the underlying idea is his.

For larger wikis that need lifecycle management, knowledge graphs, hybrid search, or automation hooks, use the sibling `llm-wiki-extended` plugin. This one stays minimal.
