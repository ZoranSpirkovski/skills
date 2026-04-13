# Wiki Schema

> Placed at `wiki/SCHEMA.md` by the `llm-wiki` skill during bootstrap.
> This is the rulebook for any LLM agent working on this wiki. Read it in full before making any changes.

## Domain

**Topic:** <filled in during bootstrap — replace with what this wiki is about>

**Source types expected:** <filled in during bootstrap — articles, papers, transcripts, chapters, notes, etc.>

This document and the wiki itself co-evolve. As conventions become clearer, update this file.

## Layers

- `raw/` — immutable source documents. Read-only for the LLM. The user owns this directory; if a source is wrong, the user edits it.
- `wiki/digested/` — LLM-generated preprocessed markdown. One subdirectory per source (`wiki/digested/<slug>/`), containing extracted text (`text.md`), extracted images (`img-001.png`, etc.), and a combined output (`digest.md`). LLM-owned. The user can inspect digests to verify extraction quality.
- `wiki/` — all LLM-generated wiki pages. LLM-owned. The user reads; the LLM writes.
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

## Digest workflow

Digest preprocesses a raw source into clean, structured markdown before ingest. Run digest explicitly ("digest this") or let ingest trigger it automatically for non-trivial formats.

1. **Identify the source** in `raw/` and derive a slug.
2. **Create `wiki/digested/<slug>/`** if it doesn't exist.
3. **Extract text** using the best available tool (see Format handling below). Save to `wiki/digested/<slug>/text.md`. If a tool isn't installed, note what was unavailable and proceed — digest degrades gracefully.
4. **Extract images** (PDF and docx only) using `pdfimages` or equivalent if available. Save as `wiki/digested/<slug>/img-001.png`, etc. Skip if tools aren't available.
5. **Describe images visually** — use the Read tool on each extracted image (or the source file itself for JPEG/PNG) and write a description capturing labels, annotations, spatial relationships, and layout details.
6. **Write `wiki/digested/<slug>/digest.md`** — combined text + image descriptions with extraction metadata (see SKILL.md for format).
7. **Log** with op `digest`. Note format, tools used, and any extraction gaps.
8. **Report** digest path, format, extraction method, and any gaps.

## Ingest workflow

For every new source:

1. **Read the source.** If `wiki/digested/<slug>/digest.md` exists, read that. Otherwise, if the source is a non-trivial format (PDF, docx, xlsx, image file), run digest first. For plain text/markdown, read directly from `raw/`.
2. **Discuss key takeaways with the user** — three to five bullets, short message. This is your checkpoint.
3. **Read existing related pages FIRST** — use `index.md` to find candidates. This is how contradictions get caught at ingest time.
4. **Write the source summary page** at `wiki/sources/<slug>.md`.
5. **Update or create entity and concept pages.** Every entity mentioned gets a page. Every new claim cites the source.
6. **Flag contradictions** by adding a `## Contradictions` section to affected pages — never silently overwrite.
7. **Update `wiki/index.md`** — add the new source and any new entity/concept pages.
8. **Append to `wiki/log.md`** with the canonical prefix:
   ```
   ## [YYYY-MM-DD] ingest | <source title>
   ```
9. **Report pages touched** to the user.

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

Where `<op>` is one of: `bootstrap`, `digest`, `ingest`, `query`, `lint`. This format is load-bearing: it makes `grep "^## \[" log.md | tail -5` a usable timeline tool.

## What NOT to do

- Do not edit files in `raw/`.
- Do not write new claims without reading existing related pages first.
- Do not silently overwrite a claim when new information contradicts it — flag both.
- Do not skip the log entry.
- Do not put wiki conventions in the project `CLAUDE.md` — it only holds a pointer line.
- Do not rename slugs.
- Do not create a page type beyond the four canonical types without also updating this schema.

## Format handling

The digest operation handles format extraction. These are the default tools per format — all are best-effort (digest degrades gracefully if a tool isn't installed).

| Format | Text extraction | Image extraction | Visual pass |
|--------|----------------|------------------|-------------|
| `.md`, `.txt`, `.csv` | Read tool (copy as-is) | N/A | N/A |
| `.html` | Read tool | N/A | N/A |
| `.docx` | Read tool or `python-docx` | Extract embedded images | Describe each image |
| `.xlsx` | `openpyxl` → markdown tables | N/A | N/A |
| `.pdf` (text-heavy) | `pdftotext` (poppler) | `pdfimages` (poppler) | Describe extracted images |
| `.pdf` (image-heavy) | `pdftotext` (poppler) | `pdfimages` (poppler) | Describe each image — primary content source |
| `.jpg`, `.png`, `.svg` | N/A | File is the image | Read tool visual description |

**Domain-specific notes:** <filled in as the wiki evolves — e.g., "venue floor plans in this project are always image-heavy JPEGs; always use visual inspection" or "transcripts are always plain text; skip digest">

## Co-evolve me

This schema is not immutable. As you learn what works for this specific wiki, update this file. Examples of changes worth making:

- New domain-specific page types (e.g. `character` for a book wiki, `experiment` for a research wiki).
- New frontmatter tags the user finds useful.
- Custom lint checks specific to the domain.
- Format handling notes for source types common in this wiki's domain.

Log schema edits in `wiki/log.md` with op `schema`.
