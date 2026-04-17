---
name: ingesting
description: Use when the user drops a file into `.wiki/raw/` and asks to file it, pastes a source in chat, or says "ingest this". This is the core synthesis step — it reads a source, extracts claims, files them across entity and concept pages, and updates the index and log.
---

# Ingesting

File a source into the wiki as a source summary page plus updates to every entity and concept page it touches.

## Steps

1. **Read the source.** If a `digest.md` exists in the source's mirrored digest directory, read that — it's the normalized, complete extraction. Otherwise, check whether the source would benefit from digesting first (non-trivial formats like PDF, docx, xlsx, or image files). If so, run the digesting skill automatically before proceeding. For plain text / markdown sources pasted in chat or already in a simple format, read directly from `.wiki/raw/`.
2. **Discuss key takeaways with the user in one short message.** Three to five bullet points. This is a checkpoint — it lets the user steer the synthesis before you commit pages. (Exception: if bootstrap and the first ingest happen in the same turn, fold the checkpoint into the final report rather than pausing mid-turn.)
3. **Read relevant existing pages FIRST.** Use `.wiki/wiki/index.md` to find existing entity and concept pages related to the source. Read them in full. This is how you detect contradictions before you write them in — it is not optional. (Vacuous on the very first ingest when the index is empty; mandatory every time after that.) For large indexes, parallelize via fire-and-forget Agent subagents, one per candidate page.
4. **Write a source summary page** at `.wiki/wiki/sources/<slug>.md` with:
   - frontmatter (see `process.md` → Page Frontmatter)
   - `## Summary` — two to five sentences
   - `## Key claims` — numbered list, each claim stands alone
   - `## Entities mentioned` — list, each linked with `[[entity-slug]]`
   - `## Open questions` — things the source raised but didn't answer
5. **Update or create entity and concept pages.** Every entity mentioned in the source gets a page (create if missing, update if existing). Every claim on an entity page cites its source: `- Uses Raft consensus [sources/raft-vs-paxos-practical]`. When new information contradicts an existing claim, don't overwrite — add a `## Contradictions` section to the affected page flagging both claims and their sources, then tell the user in your summary.
6. **Update `.wiki/wiki/index.md`.** Add the new source under `## Sources` with a one-line description. Add any new entity / concept pages under the matching header with one-line descriptions.
7. **Append to `.wiki/wiki/log.md`** with the canonical prefix:
   ```
   ## [YYYY-MM-DD] ingest | <source title>
   ```
   Under the heading, list pages touched and any contradictions flagged. The prefix format matters: `grep "^## \[" log.md | tail -5` must give a clean last-five-operations view.
8. **Report back.** List pages touched, show a one-line diff of the index, and paste the log entry preview. A single source typically touches 5–15 pages; do not artificially limit yourself.

## Output

```
Ingested: <source title>

Pages touched:
- .wiki/wiki/sources/<slug>.md (created)
- .wiki/wiki/entities/<slug>.md (created)
- .wiki/wiki/entities/<other>.md (updated — added claim, new cross-ref)
- .wiki/wiki/index.md (updated — +2 entities, +1 source)

Contradictions flagged: <N> (or "none")

Log entry:
## [YYYY-MM-DD] ingest | <title>
- Touched: ...
```

## Common Mistakes

- **Writing a new claim without reading existing pages first.** Contradiction detection only works if you read before you write. Skipping step 3 silently corrupts the wiki.
- **One big "notes" page instead of entity and concept pages.** Cross-references require named pages. Flat notes don't compound.
- **Silently overwriting a contradicted claim.** Always flag both claims with a `## Contradictions` section and surface it in your report.
- **Skipping the index update.** The index has to move in lockstep or future queries miss pages.
- **Forgetting the log entry.** Every ingest, every time.
- **Artificially limiting pages touched.** 5–15 is typical; a thorough source may touch 20+. Don't stop early to keep the output short.
