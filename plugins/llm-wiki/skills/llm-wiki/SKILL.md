---
name: llm-wiki
description: Use when the user wants to build or maintain a personal knowledge base from sources (papers, articles, notes, transcripts, book chapters, meeting recordings), when they say things like "I want a knowledge base that accumulates", "help me organize my research", "file this source", "ingest this", or when they drop a document into the project and ask to process it. Also use when asked to run a health check, lint, or audit over an existing wiki, or to answer a question against an accumulated collection of notes. This is the minimal pattern — always start here; upgrade to llm-wiki-extended only when specific scaling symptoms appear.
---

# LLM Wiki

## Core Principle

**Stop re-deriving, start compiling.** Most LLM+document workflows re-read raw sources every query and rebuild understanding from scratch. A wiki compiles the understanding once and keeps it current. The wiki is a persistent, compounding artifact. The cross-references already exist. Contradictions are already flagged. Every new source strengthens or challenges what's there — it does not start over.

**The LLM owns the wiki layer; the user owns sourcing and direction.** You do the grunt work (summarizing, cross-referencing, filing, bookkeeping) that makes humans abandon wikis. The human curates, asks good questions, and thinks about what it all means.

## When Invoked

1. Check whether `./raw/` and `./wiki/SCHEMA.md` exist in the current working directory.
   - **Neither exists** → Bootstrap Flow (below)
   - **Both exist** → Operate Flow (identify whether the user wants digest, ingest, query, or lint, then run it)
2. Always read `wiki/SCHEMA.md` in full before doing any wiki work on an existing wiki. It encodes conventions specific to this wiki's domain that override generic defaults.

## Bootstrap Flow

Run these steps when `wiki/` does not yet exist.

1. **Confirm cwd.** Ask the user to confirm that the current working directory is where the wiki should live. The wiki is per-project — one wiki per topic, typically one wiki per directory.
2. **Ask about domain and scale.** One or two questions, no more: "What topic is this wiki about?" and "What kinds of sources will you be adding — articles, papers, transcripts, book chapters, your own notes?" Use the answers to fill in the schema template.
3. **Create the directory structure.**
   ```
   ./raw/            # immutable source documents (user-owned inputs)
   ./wiki/
     digested/       # LLM-generated preprocessed markdown per source
     SCHEMA.md       # conventions and workflows for this wiki
     index.md        # content catalog, organized by category
     log.md          # append-only chronological log
   ```
4. **Write `wiki/SCHEMA.md`** from `schema-template.md` (shipped with this skill). Fill in domain-specific bits: what "the topic" is, what page types make sense (always keep source/entity/concept/synthesis; add domain-specific types if the user needs them), any custom conventions the user asks for. Do not skip unfilled fields — replace them with sensible defaults from the user's domain answer.
5. **Seed `wiki/index.md`** with the category headers only, no pages yet: `## Sources`, `## Entities`, `## Concepts`, `## Synthesis`.
6. **Seed `wiki/log.md`** with one entry marking bootstrap: `## [YYYY-MM-DD] bootstrap | wiki initialized for <domain>`.
7. **Ensure `CLAUDE.md` pointer line exists** at project root. If `CLAUDE.md` is absent, create it containing exactly:
   ```
   For wiki operations, read wiki/SCHEMA.md before acting.
   ```
   If `CLAUDE.md` exists, append a new section (do not modify existing content):
   ```
   ## Wiki

   For wiki operations, read wiki/SCHEMA.md before acting.
   ```
   This ensures future sessions discover the schema automatically.
8. **Tell the user what to do next.** Two sentences: "Drop your first source into `./raw/` and ask me to ingest it. I'll read it, file it into the wiki, and show you the pages I touched."

## Architecture

Three layers. Each has a different owner and lifecycle.

- **`raw/`** — user's curated sources. Immutable. You read from it but never modify it. If the user wants to edit a source, they do it themselves.
- **`wiki/`** — LLM-generated markdown. You own this entirely. Includes wiki pages, preprocessed digests (`wiki/digested/<slug>/`), the index, and the log. The user reads it; you write it.
- **`wiki/SCHEMA.md`** — the rulebook. Tells any future agent how this specific wiki is structured and what workflows to follow. The user and you co-evolve it over time.

## Operations

Four named operations. Always say which one you're running so the user can follow along.

### Digest

Triggered when: the user says "digest this", "preprocess these files", drops files into `raw/` and asks to prepare them, or says "digest all undigested files in raw/". Also triggered automatically by ingest when a source has no existing digest.

Digest preprocesses a raw source into clean, structured markdown so that ingest works from a normalized, inspectable artifact instead of wrestling with file formats.

1. **Identify the source** in `raw/` and derive a slug for it.
2. **Create `wiki/digested/<slug>/`** if it doesn't exist.
3. **Extract text.** Use the best available tool for the format:
   - `.md`, `.txt`, `.html` — Read tool, copy content as-is.
   - `.pdf` — `pdftotext` (poppler-utils) if available, otherwise Read tool.
   - `.docx` — Read tool or `python-docx` if available.
   - `.xlsx` — `openpyxl` to render markdown tables if available, otherwise note as unextractable.
   - `.csv` — Read tool, copy as-is.
   Save extracted text to `wiki/digested/<slug>/text.md`.
   If a tool isn't installed, note what was unavailable and proceed with what works. Digest degrades gracefully — never fail because an optional tool is missing.
4. **Extract images** (PDF and docx only). Use `pdfimages` (poppler-utils) or equivalent if available. Save extracted images as `wiki/digested/<slug>/img-001.png`, `img-002.png`, etc. If extraction tools aren't available, skip this step and note it.
5. **Describe images visually.** For each extracted image (or for the source file itself if it's a JPEG/PNG), use the Read tool to view it and write a description capturing labels, annotations, spatial relationships, and layout details.
6. **Write `wiki/digested/<slug>/digest.md`** combining everything:
   ```markdown
   # <Source Title>

   **Raw file:** `<path to original>`
   **Format:** <format> | **Digested:** <date>
   **Extraction notes:** <what tools were used, what was lossy, what was unavailable>

   ## Text Content
   <extracted text, cleaned up>

   ## Image 1: <filename>
   <visual description>

   ## Image 2: <filename>
   ...
   ```
   For text-only sources, the `## Image` sections are omitted.
7. **Append to `wiki/log.md`** with the canonical prefix:
   ```
   ## [YYYY-MM-DD] digest | <source title>
   ```
   Note the format, tools used, and any extraction gaps.
8. **Report back.** Show the digest path, format detected, extraction method used, and any gaps the user should review.

### Ingest

Triggered when: the user drops a file into `raw/` and asks to file it, pastes a source in chat, or says "ingest this".

1. **Read the source.** If `wiki/digested/<slug>/digest.md` exists, read that — it's the normalized, complete extraction. Otherwise, check whether the source would benefit from digesting first (non-trivial formats like PDF, docx, xlsx, or image files). If so, run digest automatically before proceeding. For plain text/markdown sources pasted in chat or already in a simple format, read directly from `raw/`.
2. **Discuss key takeaways with the user in one short message.** Three to five bullet points. This is a checkpoint — it lets the user steer the synthesis before you commit pages. (Exception: if bootstrap and the first ingest happen in the same turn, fold the checkpoint into the final report rather than pausing mid-turn.)
3. **Read relevant existing pages FIRST.** Use `wiki/index.md` to find existing entity and concept pages related to the source. Read them in full. This is how you detect contradictions before you write them in — it is not optional. (Vacuous on the very first ingest when the index is empty; mandatory every time after that.)
4. **Write a source summary page** at `wiki/sources/<slug>.md` with frontmatter (see Page Frontmatter below), a summary section, key claims as a numbered list, entities mentioned, and open questions.
5. **Update or create entity and concept pages.** Every entity mentioned in the source gets a page (create if missing, update if existing). Every claim on an entity page cites its source: `- Uses Raft consensus [sources/raft-vs-paxos-practical]`. When new information contradicts an existing claim, don't overwrite — add a `## Contradictions` section to the affected page flagging both claims and their sources, then tell the user in your summary.
6. **Update `wiki/index.md`.** Add the new source under `## Sources` with a one-line description. Add any new entity/concept pages under the matching header with one-line descriptions.
7. **Append to `wiki/log.md`** with the canonical prefix:
   ```
   ## [YYYY-MM-DD] ingest | <source title>
   ```
   Under the heading, list pages touched and any contradictions flagged. The prefix format matters: `grep "^## \[" log.md | tail -5` must give a clean last-five-operations view.
8. **Report back.** List pages touched, show a one-line diff of the index, and paste the log entry preview. A single source typically touches 5–15 pages; do not artificially limit yourself.

### Query

Triggered when: the user asks a question that should be answered from the wiki.

1. **Read `wiki/index.md`** first to find candidate pages. At small scale (under ~100 sources), the index is enough; you do not need embedding search.
2. **Read the candidate pages in full.** Follow cross-references as needed.
3. **Synthesize an answer with citations.** Every claim in your answer cites a wiki page slug so the user can verify.
4. **Offer to file the answer back.** Say: "This answer touches X, Y, and Z pages. Should I file it as a new synthesis page at `wiki/synthesis/<slug>.md`?" Good answers should compound into the wiki — they should not disappear into chat history. If the user agrees, write the page, update the index, and append a log entry with the `query` op.

### Lint

Triggered when: the user asks for a health check, says the wiki feels stale, or at a recurring cadence (weekly is a reasonable default).

Run these checks and report findings in a structured list:

- **Contradictions** — pages with a `## Contradictions` section that has not been resolved.
- **Orphan pages** — pages with no inbound links from other wiki pages. Either link them in or delete them.
- **Stale claims** — claims cited from one old source that newer sources don't confirm or actively weaken.
- **Missing cross-refs** — a page mentions an entity by name but doesn't link to the entity's page.
- **Concepts without pages** — a term appears in three or more pages but has no page of its own. Promote it.
- **Data gaps worth a web search** — open questions the wiki hasn't answered that a web search could close.

For each finding, propose a specific fix. Do not fix silently — list the fix and let the user approve. Append a log entry:
```
## [YYYY-MM-DD] lint | <summary>
```

## Index and Log Conventions

**`wiki/index.md` is content-oriented.** Organized by category with these fixed headers: `## Sources`, `## Entities`, `## Concepts`, `## Synthesis`. Under each, one line per page: `- [[slug]] — one-line description`. Update on every ingest, every new synthesis page, and every lint-driven promotion.

**`wiki/log.md` is chronological and append-only.** Every operation gets one entry. Canonical prefix format: `## [YYYY-MM-DD] <op> | <title>` where `<op>` is `bootstrap`, `digest`, `ingest`, `query`, `lint`, or `schema`. The `## [` prefix is load-bearing — it makes `grep "^## \[" log.md` a usable timeline tool. The body under each heading is free-form: pages touched, contradictions flagged, notes. Do not mix prefix formats; do not put operations anywhere but the log.

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

Keep it short. Do not add fields the skill does not specify — page frontmatter is a load-bearing convention that downstream tools (Dataview, the lint operation, `llm-wiki-extended`) depend on.

## What Belongs Where

| Kind | Location | Who writes |
|---|---|---|
| User's source files (PDFs, articles, notes, transcripts) | `raw/` | User only |
| Preprocessed source extractions (text, images, digest) | `wiki/digested/<slug>/` | LLM |
| Source summaries | `wiki/sources/` | LLM |
| Entity pages (people, libraries, projects, systems) | `wiki/entities/` | LLM |
| Concept pages (ideas, patterns, terms) | `wiki/concepts/` | LLM |
| Synthesis pages (comparisons, analyses, answers filed back) | `wiki/synthesis/` | LLM |
| Conventions for this specific wiki | `wiki/SCHEMA.md` | LLM + user co-evolve |
| Catalog of all pages | `wiki/index.md` | LLM |
| Chronological operations log | `wiki/log.md` | LLM |
| Pointer from project context | `CLAUDE.md` (one line under `## Wiki`) | LLM during bootstrap |

## Rationalizations to Reject

| Excuse | Reality |
|---|---|
| "I'll just answer the question, no need to file it back" | Good answers disappear into chat history and have to be re-derived. File it — that's the whole point. |
| "The log entry is obvious from the diff, skip it" | The log is how you and the user reconstruct what happened. Skipping it silently decays the wiki's auditability. |
| "This source is too short for a full page" | Short sources still need a slug, a citation target, and a log entry. The page can be short too. |
| "I'll update `index.md` later" | "Later" doesn't happen. The index has to move in lockstep or queries will miss pages. Update it in the same turn. |
| "Put the schema in CLAUDE.md, it's simpler" | CLAUDE.md is for behavioral directives that bloat quickly. The schema belongs in `wiki/SCHEMA.md`; CLAUDE.md only points at it. |
| "Just write the new claim, worry about contradictions on the next lint" | Contradictions compound. Detect them at ingest time when you have the new source fresh. Lint is a backstop, not the primary defense. |
| "`README.md` / `INDEX.md` / `LOG.md` uppercase is clearer" | The lowercase canonical names (`SCHEMA.md`, `index.md`, `log.md`) are what downstream tools and future sessions expect. Use them. |

## Common Mistakes

- **Editing files in `raw/`.** The raw layer is the user's source of truth. If it's wrong, the user edits it.
- **Writing a new claim without reading existing pages first.** Contradiction detection only works if you read before you write.
- **One big "notes" page instead of entity and concept pages.** Cross-references require named pages. Flat notes don't compound.
- **Forgetting the log entry.** Every operation, every time. No exceptions.
- **Skipping the schema because "it's obvious."** Future sessions (yours and other agents) cannot read your memory. SCHEMA.md is how you teach future Claude what this wiki expects.
- **Updating one page and leaving its neighbors stale.** When a source updates an entity, also check concepts and synthesis pages that cite that entity.
- **Running text-only extraction on image-heavy sources.** Floor plans, annotated diagrams, scanned documents — text extraction produces empty or degraded output. Run digest first to extract images and describe them visually.
- **Skipping digest on complex formats.** PDFs, docx, and xlsx files should be digested before ingest. The digest artifact is inspectable — the user can verify extraction quality before wiki synthesis happens.

## Output Format

After every digest, report in this shape:

```
Digested: <source title>

Output: wiki/digested/<slug>/digest.md
Format: <format detected>
Extraction: <tools used, e.g. "pdftotext + pdfimages + visual Read">
Images: <N> extracted and described (or "none")
Gaps: <anything lossy or unreadable> (or "none")

Log entry:
## [YYYY-MM-DD] digest | <title>
```

After every ingest, report in this shape:

```
Ingested: <source title>

Pages touched:
- wiki/sources/<slug>.md (created)
- wiki/entities/<slug>.md (created)
- wiki/entities/<other>.md (updated — added claim, new cross-ref)
- wiki/index.md (updated — +2 entities, +1 source)

Contradictions flagged: <N> (or "none")

Log entry:
## [YYYY-MM-DD] ingest | <title>
- Touched: ...
```

After every query, cite every claim. After every lint, list findings with proposed fixes.

## Tips

- **Obsidian is the intended reader.** The wiki is just a git repo of markdown, but Obsidian's graph view and wikilink support make it pleasant. Suggest the user open the `wiki/` directory as an Obsidian vault.
- **Obsidian Web Clipper** converts articles to markdown — the easiest way to get web sources into `raw/`.
- **Dataview** plugin runs queries over page frontmatter. Our frontmatter spec is Dataview-ready.
- **The wiki is a git repo.** Commit after each ingest for a free audit trail.

## Credits

Based on Andrej Karpathy's [LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). This skill encodes Karpathy's pattern as a reusable plugin; the underlying idea is his.

For larger wikis that need lifecycle management, knowledge graphs, hybrid search, or automation hooks, use the sibling `llm-wiki-extended` skill. This one stays minimal.
