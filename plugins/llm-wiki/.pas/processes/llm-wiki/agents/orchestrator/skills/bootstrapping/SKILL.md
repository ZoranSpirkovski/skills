---
name: bootstrapping
description: Use when `.wiki/` does not yet exist in the current project and the user wants to start a new knowledge base. Creates the directory layout, seeds `SCHEMA.md`, `index.md`, `log.md`, and drops a pointer line into `CLAUDE.md`.
---

# Bootstrapping

Create a fresh `.wiki/` tree at the project root and prime it so future operations have something to work with.

## Steps

1. **Confirm cwd.** Ask the user to confirm the current working directory is where the wiki should live. The wiki is per-project — one wiki per topic, typically one wiki per directory.
2. **Ask about domain and scale.** One or two questions, no more:
   - "What topic is this wiki about?"
   - "What kinds of sources will you be adding — articles, papers, transcripts, book chapters, your own notes?"
   Use the answers to fill in the schema template.
3. **Create the directory structure.**
   ```
   .wiki/
     raw/            # immutable source documents (user-owned inputs)
     digested/       # orchestrator-generated preprocessed markdown per source
     wiki/           # orchestrator-generated wiki pages
       SCHEMA.md     # conventions and workflows for this wiki
       index.md      # content catalog, organized by category
       log.md        # append-only chronological log
   ```
4. **Write `.wiki/wiki/SCHEMA.md`** from the `schema-template.md` shipped alongside this skill. Fill in the domain-specific bits: what "the topic" is, what page types make sense (always keep source / entity / concept / synthesis; add domain-specific types only if the user asks), any custom conventions. Do not skip unfilled fields — replace them with sensible defaults from the user's domain answer.
5. **Seed `.wiki/wiki/index.md`** with the category headers only, no pages yet:
   ```
   ## Sources
   ## Entities
   ## Concepts
   ## Synthesis
   ```
6. **Seed `.wiki/wiki/log.md`** with one entry marking bootstrap:
   ```
   ## [YYYY-MM-DD] bootstrap | wiki initialized for <domain>
   ```
7. **Ensure the `CLAUDE.md` pointer line exists** at project root. If `CLAUDE.md` is absent, create it containing exactly:
   ```
   For wiki operations, read .wiki/wiki/SCHEMA.md before acting.
   ```
   If `CLAUDE.md` exists, append a new section (do not modify existing content):
   ```
   ## Wiki

   For wiki operations, read .wiki/wiki/SCHEMA.md before acting.
   ```
   This ensures future sessions discover the schema automatically.
8. **Tell the user what to do next.** Two sentences: "Drop your first source into `.wiki/raw/` and ask me to ingest it. I'll read it, file it into the wiki, and show you the pages I touched."

## Output

Report back in this shape:

```
Bootstrapped: <domain>

Created:
- .wiki/raw/
- .wiki/digested/
- .wiki/wiki/SCHEMA.md
- .wiki/wiki/index.md
- .wiki/wiki/log.md
- CLAUDE.md pointer (new / appended)

Next: drop a source into .wiki/raw/ and ask me to ingest it.

Log entry:
## [YYYY-MM-DD] bootstrap | wiki initialized for <domain>
```

## Common Mistakes

- Running bootstrap when `.wiki/` already exists. Check first — if it does, route to the operation the user actually wants (digest / ingest / query / lint).
- Leaving unfilled placeholders in `SCHEMA.md`. Every `<filled in during bootstrap>` slot must be replaced — the schema is load-bearing for future sessions.
- Skipping the `CLAUDE.md` pointer. Without it, future sessions won't discover the schema automatically.
