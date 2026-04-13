# Wiki Schema

> Placed at `wiki/SCHEMA.md` by the `llm-wiki` skill during bootstrap.
> This is the rulebook for any LLM agent working on this wiki. Read it in full before making any changes.

## Domain

**Topic:** <filled in during bootstrap — replace with what this wiki is about>

**Source types expected:** <filled in during bootstrap — articles, papers, transcripts, chapters, notes, etc.>

This document and the wiki itself co-evolve. As conventions become clearer, update this file.

## Layers

- `raw/` — immutable source documents. Read-only for the LLM. The user owns this directory; if a source is wrong, the user edits it.
- `wiki/` — all LLM-generated pages. LLM-owned. The user reads; the LLM writes.
- `wiki/SCHEMA.md` — this file. The rulebook.

## Page types

Four canonical types. Each page has frontmatter declaring its `type`.

### Source pages (`wiki/sources/<slug>.md`)

One per ingested source. Contains:
- Frontmatter (see below)
- `## Summary` — two to five sentences
- `## Key claims` — numbered list, each claim stands alone
- `## Entities mentioned` — list, each linked with `[[entity-slug]]`
- `## Open questions` — things the source raised but didn't answer

### Entity pages (`wiki/entities/<slug>.md`)

One per person, library, project, system, organization. Contains:
- Frontmatter
- A one-paragraph description
- `## Claims` — bullet list, each claim cites its source: `- Uses Raft consensus [sources/raft-vs-paxos-practical]`
- `## Relationships` — bullet list of links to other entities
- `## Contradictions` — only if contradictions exist. Never overwrite a claim on contradiction; flag both.

### Concept pages (`wiki/concepts/<slug>.md`)

One per idea, pattern, term, or abstraction. Same structure as entity pages.

### Synthesis pages (`wiki/synthesis/<slug>.md`)

One per comparison, analysis, or filed-back query answer. Contains:
- Frontmatter
- A narrative section that synthesizes multiple sources
- Every claim cites the source page it comes from

## Page frontmatter

Every wiki page starts with this YAML block:

```yaml
---
type: source | entity | concept | synthesis
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: <integer>
tags: [tag1, tag2]
---
```

`sources` semantics: source pages use `1` (self). Entity, concept, and synthesis pages use the count of distinct source pages whose claims they currently cite.

Keep it short. Do not add fields beyond this spec unless you're also extending this schema document.

## Slug rules

- Lowercase, hyphens, no spaces or underscores.
- Source slugs derive from the source title: trim punctuation, drop leading articles (`a`, `an`, `the`), keep the rest. "Raft vs Paxos: A Practical Comparison" → `raft-vs-paxos-practical-comparison`.
- Entity and concept slugs use the canonical name: `raft`, `paxos`, `leslie-lamport`, `leader-election`.
- Never rename a slug once it exists. If a name changes, create a new page and add a redirect stub in the old one.

## Ingest workflow

For every new source:

1. **Read the source in full** — no skimming.
2. **Assess source format.** If the source contains images, diagrams, or visual layouts (e.g., image-heavy PDFs, scanned documents, floor plans, annotated maps):
   - First extract text (Read tool, `pdftotext`, or similar).
   - Then view the source visually (Read tool on PDF/image files) to capture labels, annotations, spatial relationships, and layout details that text extraction misses.
   - Note in the source page summary what was extracted via text vs. visual inspection, and flag any content that remains unreadable.
   Text-only sources skip this step.
3. **Discuss key takeaways with the user** — three to five bullets, short message. This is your checkpoint.
4. **Read existing related pages FIRST** — use `index.md` to find candidates. This is how contradictions get caught at ingest time.
5. **Write the source summary page** at `wiki/sources/<slug>.md`.
6. **Update or create entity and concept pages.** Every entity mentioned gets a page. Every new claim cites the source.
7. **Flag contradictions** by adding a `## Contradictions` section to affected pages — never silently overwrite.
8. **Update `wiki/index.md`** — add the new source and any new entity/concept pages.
9. **Append to `wiki/log.md`** with the canonical prefix:
   ```
   ## [YYYY-MM-DD] ingest | <source title>
   ```
10. **Report pages touched** to the user.

A single source typically touches 5–15 pages. Do not artificially limit yourself.

## Query workflow

1. Read `wiki/index.md` to find candidate pages.
2. Read candidate pages in full, following cross-references.
3. Synthesize an answer with one citation per claim.
4. Offer to file the answer back as a synthesis page. If the user agrees, write it, update the index, log it with op `query`.

## Lint workflow

Run weekly, or whenever the wiki feels stale. Check for:

- **Contradictions** — pages with unresolved `## Contradictions` sections.
- **Orphan pages** — no inbound links. Either link them or delete them.
- **Stale claims** — claims from one old source not confirmed or actively weakened by newer ones.
- **Missing cross-refs** — a page mentions an entity by name but doesn't link to its page.
- **Concepts without pages** — a term appears in three or more pages with no page of its own. Promote it.
- **Data gaps** — open questions a web search could close.

Report findings as a list with proposed fixes. Do not fix silently — let the user approve. Log with op `lint`.

## Index structure

`wiki/index.md` uses these fixed top-level headers:

```
## Sources
## Entities
## Concepts
## Synthesis
```

Under each, one line per page: `- [[slug]] — one-line description`. Sorted alphabetically within each category.

## Log format

`wiki/log.md` is append-only. Every operation gets one entry. Canonical prefix:

```
## [YYYY-MM-DD] <op> | <title>
```

Where `<op>` is one of: `bootstrap`, `ingest`, `query`, `lint`. This format is load-bearing: it makes `grep "^## \[" log.md | tail -5` a usable timeline tool.

## What NOT to do

- Do not edit files in `raw/`.
- Do not write new claims without reading existing related pages first.
- Do not silently overwrite a claim when new information contradicts it — flag both.
- Do not skip the log entry.
- Do not put wiki conventions in the project `CLAUDE.md` — it only holds a pointer line.
- Do not rename slugs.
- Do not create a page type beyond the four canonical types without also updating this schema.

## Format handling

Some sources are text-extractable (articles, transcripts, plain markdown); others are image-heavy (scanned PDFs, floor plans, annotated diagrams, JPEG proposals). The ingest workflow's format-assessment step handles this automatically, but domain-specific wikis can document preferred tools and known quirks here.

**Default tools by format:**
- `.md`, `.txt`, `.html` — Read tool (text extraction)
- `.pdf` — Read tool for visual inspection; `pdftotext` (poppler-utils) for text extraction. Use both passes on image-heavy PDFs.
- `.jpg`, `.png`, `.svg` — Read tool (visual inspection only)

**Domain-specific notes:** <filled in as the wiki evolves — e.g., "venue floor plans in this project are always image-heavy JPEGs; always use visual inspection" or "transcripts are always plain text; skip format assessment">

## Co-evolve me

This schema is not immutable. As you learn what works for this specific wiki, update this file. Examples of changes worth making:

- New domain-specific page types (e.g. `character` for a book wiki, `experiment` for a research wiki).
- New frontmatter tags the user finds useful.
- Custom lint checks specific to the domain.
- Format handling notes for source types common in this wiki's domain.

Log schema edits in `wiki/log.md` with op `schema`.
