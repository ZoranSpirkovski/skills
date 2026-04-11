---
name: llm-wiki-extended
description: Use when an existing llm-wiki has outgrown its flat structure — symptoms include silent contradictions piling up, index.md too long for one read (>500 lines), stale pages accumulating, users asking relationship questions flat wikilinks can't answer ("show me everything that touches X"), users forgetting to run lint for weeks, multiple agents sharing the same wiki, or sensitive sources that need governance. Not for fresh projects or wikis under ~100 pages — start with llm-wiki instead.
---

# LLM Wiki Extended

## Required Background

**REQUIRED BACKGROUND:** You MUST use `llm-wiki` first to bootstrap and understand the minimal pattern. This skill extends it; it does not replace it. If `./wiki/` and `./wiki/SCHEMA.md` do not exist yet, stop and run `llm-wiki` first. Everything in this skill assumes the minimal pattern is already in place.

This skill adds layers. It does not change the three-layer architecture (`raw/` → `wiki/` → `wiki/SCHEMA.md`), the canonical filenames, the four page types, the ingest/query/lint operations, or the log prefix format. It adds new frontmatter fields, new operations (`crystallize`, `forget`), and new infrastructure (hybrid search, automation hooks).

## Core Principle

**Knowledge has a lifecycle. Structure earns its keep.**

A flat wiki compounds beautifully up to roughly 100 pages. Past that, three things go wrong:

1. **Trust decays silently.** Old claims and new claims look identical. Contradictions sit next to each other with nothing marking which is current. Users stop trusting the wiki, which is worse than having no wiki at all.
2. **Retrieval breaks.** `index.md` grows past the point where reading the whole thing per query is efficient, and keyword wikilinks can't express "everything that depends on X."
3. **Maintenance debt accumulates.** The lint step that was supposed to run weekly runs never. Stale pages pile up. The user can feel it but can't name it.

This skill adds the machinery that handles all three: confidence and supersession for trust, typed edges and hybrid retrieval for queries, consolidation tiers and automation for maintenance. Add layers as you feel the pain. Not before.

## When to Add Each Layer (Implementation Spectrum)

You do not apply this skill as a bundle. You apply specific layers when specific symptoms appear. Use this table to pick.

| Symptom the user describes | Layer to add | Section below |
|---|---|---|
| "I found contradictions sitting silently next to each other" | Lifecycle — confidence + supersession | Lifecycle Layer |
| "The wiki feels cluttered with old stuff I don't want to delete" | Lifecycle — tiers + forget operation | Lifecycle Layer |
| "`index.md` is too long, queries miss pages" | Break the index, add hybrid retrieval | Retrieval Layer |
| "Show me everything that touches X (relationship query)" | Knowledge graph with typed edges | Graph Layer |
| "I keep forgetting to run lint" | Automation hooks + scheduled agents | Automation Layer |
| "Multiple agents or users sharing the wiki" | Shared/private scoping + mesh sync | Collaboration Layer |
| "Sources contain API keys or PII" | Privacy filter on ingest + audit trail | Privacy Layer |
| "Completed research threads should become wiki pages" | Crystallization operation | Crystallization |

Addressing symptoms in this order works because later layers depend on frontmatter introduced by earlier ones. **Always do Lifecycle before Graph before Retrieval** — the graph depends on edge frontmatter, and hybrid retrieval wants access to `confidence` and `tier` for ranking.

## Lifecycle Layer

Adds trust scoring and structured retirement to every page.

### Extended frontmatter (additive to llm-wiki spec)

```yaml
---
type: source | entity | concept | synthesis
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: <integer>
tags: [tag1, tag2]

# llm-wiki-extended additions:
confidence: high | medium | low
tier: working | episodic | semantic | procedural
last_verified: YYYY-MM-DD
superseded_by: <slug>            # optional, only if retired
supersedes: [slug1, slug2]       # optional, only on replacement pages
```

Defaults when migrating an existing v1 wiki: `confidence: medium`, `tier: working`, `last_verified: <created>`. Script the migration — do not hand-edit pages.

### Confidence semantics

- **high** — page is corroborated by three or more sources, no unresolved contradictions, `last_verified` is within the last 30 days.
- **medium** — single source, or corroborated but `last_verified` is older than 30 days, or user explicitly flagged moderate uncertainty. Default for migrated pages.
- **low** — single source with user-flagged uncertainty, or a page under active contradiction, or `last_verified` older than six months.

Confidence is set on the page as a whole, not per claim. If one page would need per-claim confidence, split it into two pages.

**`last_verified` is the clock.** It gets updated whenever a page is actively re-read and confirmed — during ingest (if the new source corroborates the page), during lint's verification pass, or whenever the user explicitly confirms a page is still correct. Bump `last_verified` to today when you verify; bump `updated` to today when you edit content. They are different: verification without editing updates `last_verified` only. Lint reports pages whose `last_verified` is older than 30 / 90 / 180 days as re-verification candidates.

### Supersession workflow

When a new source contradicts an existing page, you have three choices:

1. **Neither is clearly right** — add a `## Contradictions` section to both, lower both pages' confidence to `low`, report to user.
2. **New supersedes old** — create or keep the new page, add `superseded_by: <new-slug>` to the old page's frontmatter, lower the old page's confidence to `low`, leave the old page's body intact (do NOT delete). Add `supersedes: [old-slug]` to the new page. Queries filter out superseded pages by default; the old page stays readable for audit.
3. **New supplements old** — update the existing page with the new claim, cite the new source, bump `last_verified`. No supersession needed.

Never silently overwrite. Every retirement is tracked.

### Consolidation tiers

Pages live in one of four tiers. Movement between tiers is a named operation.

- **working** — recent, actively cited, may still change. Default for new pages.
- **episodic** — session- or source-specific observations tied to a single event or source. Example: "what I learned from reading the Raft paper on 2026-03-15." Narrow in scope, stable in content, but not promotable to canonical because they describe a specific event rather than a durable fact.
- **semantic** — stable, corroborated, authoritative. The canonical statement about an entity or concept. Pages that have been high confidence for 30+ days and are cited by three or more other pages are candidates for promotion to `semantic`.
- **procedural** — workflows, how-tos, patterns. Extracted from repeated semantic patterns. Example: "the standard way I analyze a distributed systems paper."

**Lifecycle per tier:**
- `working` is the default destination for ingest and the source tier for both `crystallize` (→ semantic) and `forget` (→ archive).
- `episodic` is assigned at ingest time when the user signals "this is about a specific event/session" — it is not a promotion target and does not transition to other tiers. An episodic page is archived via `forget` (same threshold as working) if it becomes stale. Episodic pages are not crystallization candidates because their narrowness is the point.
- `semantic` is the promotion destination for `crystallize`. Semantic pages only demote if their supporting sources get superseded — in which case supersession handles it, not `forget`.
- `procedural` is the promotion destination when three or more semantic pages share a structural pattern the user wants captured as a procedure.

**Promotion triggers** (`crystallize` operation):
- Working → Semantic: confidence `high` for ≥30 days AND cited by ≥3 other pages AND no edits for ≥14 days.
- Semantic → Procedural: three or more semantic pages share the same structural pattern and the user wants the pattern captured as a procedure.

**Demotion trigger** (`forget` operation):
- Working or episodic → archive: confidence `low` AND no edits or citations for ≥180 days AND not cited by any semantic or procedural page. Citations from other working-tier pages do not block archival on their own — if a working-tier citer is itself a forget candidate, archive both; if the citer is still active, lint flags the orphaned reference and the user decides whether to fix the citation or keep the page alive.
- Move archived pages to `wiki/archive/<slug>.md`, leaving a one-line tombstone stub at the original path:
  ```markdown
  ---
  type: tombstone
  archived: YYYY-MM-DD
  see: archive/<slug>
  ---
  ```
  Deletion is never automatic. The archive is reversible.

### New operations

Add these to `wiki/SCHEMA.md` alongside ingest/query/lint:

**Crystallize** — triggered manually or by lint findings. Scan working-tier pages for promotion candidates; propose a list; user approves; promote and update `tier` frontmatter; log with op `crystallize`.

**Forget** — same pattern. Scan working-tier and episodic pages for demotion candidates; propose list; user approves; move to archive with tombstone; log with op `forget`.

**Supersede** — triggered when ingest finds that a new source retires an existing claim cleanly. Create or confirm the replacement page, add `superseded_by: <new-slug>` to the old page, lower the old page's confidence to `low`, leave the old body intact, add `supersedes: [old-slug]` to the replacement, log with op `supersede`. Queries filter superseded pages out by default.

**Verify** — re-read a page against its cited sources, bump `last_verified` to today if the page is still correct, downgrade `confidence` if not. Triggered by lint findings (pages with `last_verified` older than 30 / 90 / 180 days) or by user request. Log with op `verify`.

**Schema** — any edit to `wiki/SCHEMA.md`. Log with op `schema`.

## Graph Layer

Adds typed relationships so you can answer "everything that touches X" queries and render a knowledge graph.

### Edge types

A fixed vocabulary of edge types lives in frontmatter. Do not invent new edge types without updating `wiki/SCHEMA.md`.

```yaml
uses: [slug1, slug2]             # this page uses/depends on these entities
depends_on: [slug]               # stronger form of uses; depends for correctness
implements: [slug]               # e.g. raft implements consensus
contradicts: [slug]              # this page's claims contradict another page
caused: [slug]                   # causal, historical
fixed: [slug]                    # e.g. bugfix relations
related: [slug1, slug2]          # weak link, usually kept minimal
```

Edges are directed. The lint operation builds the inverse (`used_by`, `depended_on_by`, etc.) on demand; do not hand-maintain both sides.

### Graph-aware queries

For relationship queries ("show me everything that touches leader-election"):

1. Start at the target node (e.g. `wiki/concepts/leader-election.md`).
2. Walk inverse edges: find every page with `leader-election` in any of `uses`, `depends_on`, `implements`, or `related`.
3. Group results by edge type.
4. Read the top candidates in full, synthesize with citations, offer to file back.

Obsidian's graph view can render this directly from the frontmatter edges. Dataview queries over the edge arrays also work.

### Migrating existing pages

When adding the graph layer to an existing v1 wiki:

1. Script a pass that scans every page's wikilinks (`[[slug]]`) and, for each, populates a `related: [...]` list in frontmatter. This is a lossy default — `related` is the weakest edge type.
2. Over the next several ingest cycles, upgrade weak `related` edges to specific types as you read each page. Do not try to upgrade all edges at once.
3. Run a lint check that flags pages with zero outgoing edges (probable orphan) and pages with only `related` edges (likely need more specific typing).

## Retrieval Layer

When `index.md` outgrows a single read.

**Depends on Lifecycle.** The `by-tier.md` and `by-confidence.md` sub-indexes read frontmatter fields (`tier`, `confidence`) that the Lifecycle layer introduces. If Lifecycle is not active yet, you can still split by category and by recency, but the full sub-index set requires Lifecycle in place first.

### Split the index

Replace the single `wiki/index.md` with:

- `wiki/index.md` — thin top-level table of contents with category headers, pointers to sub-indexes, under 200 lines. This is what gets loaded into context at query time.
- `wiki/indexes/by-category.md` — entities, concepts, sources, synthesis as separate Dataview tables.
- `wiki/indexes/by-tier.md` — working/semantic/procedural/episodic views.
- `wiki/indexes/by-confidence.md` — surfaces low-confidence pages for review.
- `wiki/indexes/recent.md` — last 30 days of activity, auto-generated from `log.md`.

The sub-indexes are generated by scripts or Dataview queries. They are read on demand, not on every query.

### Hybrid search

Past ~200 pages, index reading is no longer enough. Add three retrieval streams:

- **BM25** — keyword matching with stemming. Catches exact terms.
- **Vector search** — embeddings over page summaries and key claims. Catches semantic similarity.
- **Graph traversal** — start at a node, walk typed edges outward. Catches structural connections.

Fuse results with reciprocal rank fusion. Each stream catches things the others miss.

**Do not implement this from scratch.** Use [qmd](https://github.com/tobi/qmd) — a local-first hybrid search over markdown with BM25, vectors, and LLM re-ranking, shipping both a CLI and an MCP server. When the user hits the scale limit of `index.md`, propose qmd as the default. If they need something simpler, a 50-line Python script over `sqlite-vec` + BM25 is enough for most cases.

## Automation Layer

Solves the "user forgets to run lint" problem and compounds observations across sessions.

### Settings.json hooks

Configure via the `update-config` skill. Candidate events:

- **SessionStart** — check `wiki/log.md` for the most recent `lint` entry. If older than 7 days, print a one-line reminder: *"Wiki lint is N days stale — run lint."*
- **Stop** (end of response) — if the session touched `wiki/` files, append an observation to `wiki/log.md` summarizing what changed.
- **UserPromptSubmit** — if the prompt mentions the wiki's domain, pre-load the relevant sub-indexes into context.

Each hook is a shell command or a script. Keep them under 50 lines and idempotent.

### Scheduled agents

Use the `schedule` skill (or cron) to run a weekly full lint outside sessions. The agent writes findings to `wiki/lint-queue.md` so the user walks in Monday morning with a ready review list. Same for monthly crystallize/forget passes over large wikis.

### On-ingest automation

When the user drops a file into `raw/` without saying anything, a watcher can auto-trigger ingest. For most users this is too aggressive (they prefer to initiate ingests explicitly). Offer it as opt-in, not default.

## Quality and Self-Healing

Lint graduates from "report findings and wait" to "fix what is safely fixable." Safe fixes:

- Broken cross-references (`[[slug]]` pointing at a deleted page) — repair if the page was renamed; otherwise flag.
- Missing inverse edges — rebuild the `used_by` / `depended_on_by` views.
- Orphan pages — leave them, but flag with a priority score (low = archive candidate, high = probably needs a parent).
- Stale tombstones — if an archived page is cited by a semantic page, restore it.
- Schema drift — pages missing required frontmatter fields get defaults filled in.

Unsafe fixes (contradictions, supersession, tier promotion, deletion) always require user approval. The lint operation's output is a two-part report: "fixed automatically" and "needs your review."

For ingest quality, apply a second-pass self-evaluation: after writing pages, re-read the ingest output and score it on a checklist (structure, citations, cross-refs, contradictions flagged, log entry). Pages below threshold trigger a rewrite pass.

## Privacy and Governance

When sources can contain sensitive data (credentials, PII, internal conversations).

### Ingest filter

Before any raw content reaches `wiki/`, scan for:

- API keys, tokens, passwords (regex patterns)
- Email addresses and phone numbers
- Anything in a `.wiki-private` section of the source
- Patterns the user flags in `wiki/SCHEMA.md`'s "Sensitive" section

Redact or refuse to ingest. Never silently paste secrets into wiki pages.

### Audit trail

The `log.md` file already serves as a basic audit trail. For stricter governance, add:

- **Every write is logged** — not just operations. Extend `log.md` with per-page write entries under each operation heading.
- **Reversible bulk operations** — before bulk-archiving, bulk-retiring, or bulk-merging, write the target list to `wiki/operations/<op>-<date>.md` and wait for explicit user approval. Execute only after approval. Keep the operations file as a rollback reference.
- **Scoping** — some pages are private (user-only), some shared. Add a `scope: private | shared` frontmatter field; the graph and retrieval layers respect it.

## Crystallization

Completed explorations are sources too.

When the user finishes a research thread, a debugging session, or an analysis inside a Claude Code session, distill it into a crystallization page:

1. Summarize the thread: what was the question, what did the exploration find, what was the conclusion, what files or entities were involved.
2. Extract any lessons that generalize beyond the original question into standalone entity or concept pages.
3. File the crystallization as a synthesis page with `tier: working` and the usual frontmatter.
4. Log with op `crystallize`.

Your explorations should compound into the wiki the same way external sources do. If you notice yourself re-deriving the same analysis twice, the first analysis should have been crystallized.

## When NOT to use this skill

Pull the rip cord and stay on `llm-wiki` minimal if:

- The wiki has fewer than 100 pages. Confidence scoring and tier tracking are overhead you cannot amortize yet.
- There's a single user and a single agent. Shared/private scoping is unnecessary.
- No contradictions have appeared. Supersession is solving a problem you don't have.
- `index.md` fits in one read. Don't split indexes preemptively.
- The user is still figuring out their domain. Tier promotion thresholds are stable targets; a still-evolving wiki will thrash.
- There are no sensitive sources. Privacy layer adds friction.

Adding extended machinery to a wiki that doesn't need it makes the wiki harder to maintain, not easier. Layers earn their place by solving a felt problem. If nothing is broken, do not upgrade.

## Rationalizations to Reject

| Excuse | Reality |
|---|---|
| "I'll add the graph later, `[[slug]]` wikilinks are fine for now" | Graph queries require typed edges *in frontmatter*. Wikilinks are opaque to Dataview and to retrieval. Retrofitting edges over 200 pages is worse than adding them at ingest time. |
| "Confidence scoring is overkill, I can trust my wiki" | Until you can't, and by then you're rebuilding trust across the whole corpus. Confidence is cheap to write and expensive to retrofit. |
| "Automation hooks are a nice-to-have" | Lint-you-forgot-to-run is the #1 cause of wiki decay in the field. Hooks cost ten minutes once; forgetting costs the whole wiki over months. |
| "I'll handle contradictions as I encounter them" | Contradictions you don't explicitly track are contradictions you've silently adopted. Supersession is how "track them" becomes a structural property, not a discipline. |
| "This is just lifecycle ceremony, the wiki is the point" | The wiki is only useful if you trust it. Lifecycle is what produces trust at scale. |
| "I'll skip migration and apply the new spec only to new pages" | Two-spec wikis break every downstream tool. Migrate all pages at once with a script; the script is not that hard. |
| "The user didn't ask for crystallization — don't bring it up" | Users don't know to ask for what they haven't seen. When you recognize symptoms, name the pattern — that's the whole point of this skill. |

## Common Mistakes

- **Adding all layers at once on day one.** Choose one symptom, add one layer, verify, then move on.
- **Handling migration by hand.** 150 pages × 6 frontmatter fields = 900 manual edits. Always script it.
- **Inventing new edge types ad hoc.** Edge types are a fixed vocabulary. New types require a schema update and a migration.
- **Deleting tombstones.** A tombstone is a redirect. Deleting it breaks every cross-reference to the archived page.
- **Promoting to `semantic` too fast.** Semantic is "this is stable and canonical." A page three days old is not canonical.
- **Skipping the self-evaluation pass.** Ingest quality degrades fastest when the LLM is tired or the source is dense. The second-pass review exists to catch this.
- **Running hooks that block the user.** Hooks should take under a second and never block. Long operations go in scheduled agents.

## Output Format

After any extended operation, report in this shape:

```
Operation: <op name>
Layer: <lifecycle | graph | retrieval | automation | quality | privacy | crystallization>

Changes:
- <page or file> (<created | updated | moved | archived>)
- ...

Frontmatter deltas:
- <slug>: <field> <old> → <new>
- ...

Needs your review:
- <finding> → <proposed fix>

Log entry:
## [YYYY-MM-DD] <op> | <summary>
```

## Credits

Extends Andrej Karpathy's [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) with the v2 additions from Rohit Ghumare's [LLM Wiki v2](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2), which itself draws lessons from building [agentmemory](https://github.com/rohitg00/agentmemory). The underlying ideas belong to Karpathy and Ghumare; this skill encodes them as a reusable plugin that sits cleanly on top of `llm-wiki`.
