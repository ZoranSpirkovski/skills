# Wiki Schema (Extended)

> Placed at `wiki/SCHEMA.md` by the `llm-wiki-extended` skill, either as a fresh install or as an upgrade from the `llm-wiki` minimal template.
> This is the rulebook for any LLM agent working on this wiki. Read it in full before making any changes.

## Domain

**Topic:** <filled in during bootstrap — replace with what this wiki is about>

**Source types expected:** <filled in during bootstrap>

**Extended layers in use:** <check the ones active for this wiki; uncheck the others>
- [ ] Lifecycle (confidence, supersession, tiers)
- [ ] Graph (typed edges)
- [ ] Retrieval (split index, hybrid search)
- [ ] Automation (hooks, scheduled lint)
- [ ] Quality (self-healing lint, second-pass review)
- [ ] Privacy (ingest filter, audit trail, scoping)
- [ ] Crystallization

Only layers checked above are enforced. Do not apply an unchecked layer without updating this list and running a migration.

## Layers

- `raw/` — immutable source documents.
- `wiki/` — all LLM-generated pages.
- `wiki/archive/` — tombstoned pages, reversible. Never delete from here.
- `wiki/indexes/` — sub-indexes (present only if Retrieval layer is active).
- `wiki/operations/` — pending bulk operation files awaiting user approval (present only if Privacy layer is active).
- `wiki/SCHEMA.md` — this file.

## Page types

Four canonical types plus `tombstone`:

- `source` — one per ingested source, lives under `wiki/sources/`.
- `entity` — person, library, project, system, organization. Lives under `wiki/entities/`.
- `concept` — idea, pattern, term, abstraction. Lives under `wiki/concepts/`.
- `synthesis` — comparison, analysis, filed-back answer, crystallization. Lives under `wiki/synthesis/`.
- `tombstone` — redirect stub left behind when a page is archived. Lives at the original path.

## Page frontmatter (extended)

Every wiki page (except tombstones) starts with this YAML block:

```yaml
---
# base fields (from llm-wiki)
type: source | entity | concept | synthesis
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: <integer>
tags: [tag1, tag2]

# lifecycle layer
confidence: high | medium | low
tier: working | episodic | semantic | procedural
last_verified: YYYY-MM-DD
superseded_by: <slug>            # optional
supersedes: [slug, ...]          # optional

# graph layer (all optional, empty list = no edges of that type)
uses: [slug, ...]
depends_on: [slug, ...]
implements: [slug, ...]
contradicts: [slug, ...]
caused: [slug, ...]
fixed: [slug, ...]
related: [slug, ...]

# privacy layer (optional)
scope: private | shared
---
```

Only include fields for layers this wiki has enabled (see the layers checklist above). Tombstone pages use this minimal form:

```yaml
---
type: tombstone
archived: YYYY-MM-DD
see: archive/<slug>
---
```

## Slug rules

Same as `llm-wiki`:

- Lowercase, hyphens, no spaces.
- Source slugs derive from the title with punctuation trimmed and leading articles dropped.
- Entity and concept slugs use the canonical name.
- Never rename a slug. If a name changes, archive and create a new page.

## Operations

Beyond the base ingest / query / lint, this wiki supports:

- **`crystallize`** — promote a working-tier page to semantic (or a set of semantic pages to procedural). Triggered manually or by lint findings.
- **`forget`** — demote a working- or episodic-tier page to archive. Triggered by lint findings.
- **`supersede`** — retire a page in favor of a replacement; mark the old `superseded_by`, add `supersedes` to the new one, lower the old page's confidence to `low`.
- **`verify`** — re-read a page against its sources, bump `last_verified` to today if still correct, downgrade `confidence` if not.
- **`schema`** — edit this file. Always log.

All operations go in `wiki/log.md` with the canonical prefix:

```
## [YYYY-MM-DD] <op> | <title>
```

## Lifecycle rules

### Confidence

- `high` — ≥3 corroborating sources, no unresolved contradictions, verified in the last 30 days.
- `medium` — single source, or corroborated but verification is older than 30 days. Default for new and migrated pages.
- `low` — single source with user-flagged uncertainty, or under active contradiction, or unverified for 6+ months.

### Tiers

- `working` — active, may still change. Default for new pages.
- `episodic` — session- or source-specific observations, stable but narrow.
- `semantic` — stable, corroborated, canonical. Promotion candidates: confidence `high` ≥30 days, cited by ≥3 pages, no edits for ≥14 days.
- `procedural` — extracted workflows and patterns, derived from repeated semantics.

### Forget triggers

Working- or episodic-tier page becomes a `forget` candidate when all of:
- `confidence: low`
- No edits or citations for ≥180 days
- Not cited by any semantic or procedural page

Citations from other working- or episodic-tier pages do not block archival on their own. If a citer is still active, lint flags the orphaned reference after archival and the user decides.

Forget = move to `wiki/archive/<slug>.md`, leave a tombstone at the original path. Never delete.

## Graph rules

Edge vocabulary (fixed): `uses`, `depends_on`, `implements`, `contradicts`, `caused`, `fixed`, `related`.

- Edges are directed. The lint operation builds inverse views (`used_by`, etc.) on demand.
- Do not invent new edge types without updating this schema and migrating.
- During ingest, prefer specific edge types (`uses`, `depends_on`) over weak `related`.

## Retrieval rules (if Retrieval layer is active)

- `wiki/index.md` stays under 200 lines: top-level TOC only.
- Sub-indexes live in `wiki/indexes/`. Generated by script or Dataview, not hand-edited.
- Hybrid search (BM25 + vector + graph) via `qmd` or equivalent. Re-indexed on every ingest.

## Privacy rules (if Privacy layer is active)

- Ingest scans sources for API keys, tokens, passwords, email, phone numbers, and patterns in the "Sensitive" section below. Matches are redacted or refuse-to-ingest.
- Bulk operations (archive, supersession passes, migration) write proposed targets to `wiki/operations/<op>-<date>.md` first; execution waits on user approval.
- `scope: private` pages are excluded from shared indexes and retrieval. `scope: shared` (the default) is visible to all agents.

### Sensitive patterns

<Add wiki-specific patterns here. Examples:>
- <regex or description>
- <regex or description>

## Automation rules (if Automation layer is active)

Configured outside this file (settings.json hooks or scheduled agents). This section is a record of what's wired:

- **SessionStart hook:** <path to script / description, or "none">
- **Stop hook:** <description, or "none">
- **Scheduled lint:** <cadence, e.g. "weekly, Monday 08:00">
- **Scheduled crystallize/forget pass:** <cadence, or "none">

## What NOT to do

All of the llm-wiki rules, plus:

- Do not apply an extended layer that isn't checked in the layers list.
- Do not hand-maintain inverse edges — lint builds them on demand.
- Do not delete tombstones. They are redirects.
- Do not promote pages to `semantic` without meeting the threshold.
- Do not silently migrate pages to the extended frontmatter — script it, log it, keep the migration diff reviewable.
- Do not mix v1 and v2 frontmatter in the same wiki. Migrate all or none.

## Co-evolve me

As you learn what works for this specific wiki, update this file. Log schema edits with op `schema`.
